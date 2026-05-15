# Odoo DR / Upgrade Runbook

**Status:** Active (skeleton; finalised when DEC-001, DEC-002, DEC-008 close)
**Owner:** Odoo Specialist + DevOps
**Audience:** on-call DevOps, Backend Lead, Internal IT.

This runbook covers what we do when Odoo is degraded, down, undergoing major-version upgrade, or recovering from a region outage. Mobile and FastAPI behaviour during these events is explicitly documented so on-call doesn't have to improvise.

---

## 1. Inputs we depend on

| Input | Source | Status |
|---|---|---|
| Odoo version + edition | `traceability/decision-log.md` DEC-001 | Open |
| Installed modules | DEC-002 | Open |
| SLA / scaling / DR profile | DEC-008 | Open |
| Custom module `field_mobile_sync` deploy story | EPIC-08, US-ODOO-001 | Open until S8 |

Until DEC-001/002/008 close, this document captures **principles** and **placeholders**. Specific HA topology + RPO/RPO numbers are filled after sponsor sign-off.

---

## 2. Topology assumptions (placeholders)

| Concern | Placeholder |
|---|---|
| Hosting | Self-hosted on cloud VMs OR Odoo.sh — TBD |
| Database | PostgreSQL 15.x managed (separate from FastAPI Postgres) |
| Filestore | Object storage (DEC-007) — never local disk |
| Backups | Daily logical (`pg_dump`) + hourly WAL — retention 30 days |
| Replicas | 1 standby in same region, 1 in DR region (DEC-008) |
| RTO target | 4 hours (placeholder, refine in DEC-008) |
| RPO target | 15 minutes (placeholder, refine in DEC-008) |

---

## 3. Behaviour when Odoo is degraded

If Odoo write path slows or returns errors, FastAPI must shield the mobile fleet.

| Symptom | FastAPI behaviour | Mobile UX | Operator action |
|---|---|---|---|
| Odoo p95 > 5 s | Celery worker throttles to 50 % concurrency | Sync slows; banner "Sync is delayed" after 60 s | Page on-call; check Odoo dashboards |
| Odoo HTTP 5xx > 5 % | Backoff + retry per `backend/app/odoo/retry.py`; envelope stays `SYNCING` | Sync paused notification | Engage Odoo specialist |
| Odoo HTTP 5xx > 30 % for 5 min | Circuit-breaker opens; envelopes go to `RETRY_LATER`; FastAPI returns 503 with `Retry-After: 300` | "Service degraded — your data is safe locally" banner | Declare SEV-2 incident |
| Odoo unreachable > 15 min | Backend stops accepting envelopes for storage-bound endpoints; auth still works | Login + view shifts (cached) still works; capture still works locally | Declare SEV-1 incident |

Mobile guarantees: **local SQLite stays the source of truth**. No data is lost. The user's UX is degraded, not broken.

---

## 4. Standby failover (intra-region)

When the primary Odoo instance fails:

1. Acknowledge incident; notify per `runbooks/incident-response.md`.
2. DBA promotes standby Postgres to primary.
3. Update Odoo's database connection (DNS swap or config push).
4. Restart Odoo workers.
5. Validate:
   - `xmlrpc/2/common version` returns OK.
   - `field_mobile_sync.shift_lookup_employee` JSON-RPC works.
   - One synthetic envelope round-trips end-to-end.
6. Re-enable the FastAPI circuit breaker; envelopes drain from `RETRY_LATER`.
7. Post update to #field-incidents.

Target: 15 min from declaration to validated recovery.

---

## 5. Region failover (DR)

For a region-wide outage:

1. Declare SEV-1; activate DR plan; notify sponsor + Internal IT.
2. Promote DR-region standby Postgres to primary.
3. Stand up Odoo workers in DR region pointing at promoted Postgres.
4. Failover object storage if applicable (S3 cross-region replication or DR bucket).
5. Update DNS / load balancer to route to DR region.
6. Validate as in §4 plus a 30-minute soak with synthetic envelopes.
7. Decide on returning to primary region after primary is healthy: schedule a planned cutover during a low-traffic window. **Do not auto-fail-back.**

Target RTO: 4 hours (placeholder, DEC-008).

---

## 6. Backup and restore drill

Frequency: monthly (calendar in `engineering/oncall.md`).

Steps:

1. Pick a non-production Odoo replica.
2. `pg_restore` the most recent daily backup.
3. Verify row counts on `hr.employee`, `project.task`, `account.analytic.line`.
4. Apply 1 hour of WAL.
5. Run synthetic envelope round-trip.
6. Document in `runbooks/_postmortem-template.md`-derived "drill notes" doc.

Failed drills are SEV-2.

---

## 7. Major version upgrade (e.g. Odoo X → Odoo Y)

This is a planned change managed via `governance/change-management.md`.

Phases:

1. **Discovery (1 sprint).** Custom module `field_mobile_sync` smoke-tested against a pre-prod Odoo Y. Identify breaking model changes (XML-RPC field renames, new permissions, new constraints).
2. **Adapter sprint (1 sprint).** FastAPI side: ship a feature-flag-gated dual-path adapter — same envelope can target Odoo X or Y.
3. **Pre-prod validation (3–5 days).** Replay last 7 days of envelopes against Odoo Y; reconcile.
4. **Cutover window (4 h, off-peak).**
   1. Drain Celery queue.
   2. Snapshot Odoo X DB.
   3. Run upgrade script.
   4. Switch FastAPI feature flag to `odoo_target=Y`.
   5. Run synthetic envelopes.
   6. Re-open queue.
5. **Hot-back stop (24 h).** If success rate drops below 99 %, flip flag back; queue stays drained.

Communication template: `governance/comms-templates.md` §3 (release notes adapt).

---

## 8. Schema or model migrations

If a custom model in `field_mobile_sync` changes:

1. Add migration script under `infra/odoo/field_mobile_sync/migrations/<version>/` (per Odoo conventions).
2. Test in staging by replaying production-shape envelopes (anonymised).
3. Roll out only after CI integration job passes (`pr-backend / contract`).
4. Announce per `governance/comms-templates.md` §3.

Reference: `governance/change-management.md`.

---

## 9. Failure to reach Odoo from FastAPI (network)

| Layer | What we check |
|---|---|
| DNS | `dig field-erp.example.com` resolves consistently |
| TLS | `openssl s_client` shows a valid chain |
| Auth | API user can read `res.users` |
| Queue | Celery workers respond to ping |

Each runs as a synthetic in `nightly-soak.yml`. A failure pages on-call.

---

## 10. Cross-references

- Incident response: `runbooks/incident-response.md`
- Sync recovery: `runbooks/sync-recovery.md`
- Rollback (mobile / backend): `runbooks/rollback.md`
- Communication plan: `governance/communication-plan.md`
- Comms templates: `governance/comms-templates.md`
- Decision log: `traceability/decision-log.md` DEC-001/002/008
- ADR-005: no direct mobile-to-Odoo (the basis for this isolation)
