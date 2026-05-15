# Decision Log (Open + Closed)

**Status:** Active
**Owner:** Project Manager
**Last updated:** 2026-05-15 (blueprint v2.1.0)

This is the running ledger of significant decisions: the 9 Phase 0 decisions plus any decision raised during build that does not warrant a full ADR.

## Status legend

- `Open` — needs answer, blocking listed scope
- `In Discussion` — owner working with stakeholders
- `Decided` — answered; if architectural, see linked ADR
- `Provisional` — answered with default assumption pending sponsor confirmation; safe to start work
- `Deferred` — answer postponed beyond MVP

## Phase 0 decisions — all closed (2026-05-15, v2.1.0)

All nine Phase 0 decisions are now `Decided`. Sponsor reconfirmation on 2026-05-15 flipped four `Provisional` items and amended two `Decided` items to reflect the Odoo Online + multi-version contract.

| ID | Topic | Status | Source | Linked ADR |
|---|---|---|---|---|
| **DEC-001** | Odoo version + edition | **Decided (amended)** | Odoo 19 EE for testing; RPC contract for production portability across versions and editions | [ADR-011](../adr/ADR-011-odoo-19-enterprise.md) |
| **DEC-002** | Installed Odoo modules | **Decided** | Capability detection at runtime; only modules in use need to be installed; hard requirements: `hr`, `project`, `contacts`, `x_ngynapp_*` field set | [ADR-015](../adr/ADR-015-no-custom-module-rpc-adapter.md) |
| **DEC-003** | Custom Odoo module `field_mobile_sync` | **Decided (flipped)** | **No custom module.** Solution must run on Odoo Online; capabilities relocated to FastAPI + `x_ngynapp_*` fields | [ADR-015](../adr/ADR-015-no-custom-module-rpc-adapter.md) |
| **DEC-004** | GPS storage target | **Decided (flipped)** | FastAPI Postgres for full history; Odoo holds summary `x_ngynapp_gps_*` only | [ADR-016](../adr/ADR-016-gps-in-fastapi-postgres.md) |
| **DEC-005** | SMS provider | **Decided** | Twilio Verify only for MVP; account + key issued; fallback re-evaluated post-pilot | [ADR-010](../adr/ADR-010-twilio-sms-only.md) |
| **DEC-006** | Phone-to-`hr.employee` mapping | **Decided** | `hr.employee.work_phone` first, fallback `private_phone`, then `mobile_phone`; normalised to E.164 (+84…); selected source recorded in `x_ngynapp_phone_source` | [ADR-015](../adr/ADR-015-no-custom-module-rpc-adapter.md), `data/odoo-mapping.md` |
| **DEC-007** | Object storage provider | **Decided (amended)** | DigitalOcean Spaces in `sgp1`; S3-compatible API for portability | [ADR-012](../adr/ADR-012-aws-s3-object-storage.md) |
| **DEC-008** | Odoo SLA / scaling / DR | **Decided** | Production tenants delegate SLA to their Odoo provider (Odoo Online SLA, Odoo.sh SLA, or self-host operator). FastAPI side carries its own SLA in `architecture/non-functional-requirements.md`. DR for our side: Postgres PITR + Spaces versioning | — (recorded in `runbooks/odoo-dr-upgrade.md`) |
| **DEC-009** | Container orchestration target | **Decided** | DigitalOcean Droplets running Docker Compose for testing and pilot; managed Postgres + managed Redis on DO; Spaces for storage. Production scales by Droplet size first, DOKS only when concurrent worker count exceeds threshold | — (recorded in `architecture/deployment.md`) |

## Stretch / non-Phase-0 decisions

| ID | Topic | Owner | Status | Blocks | Target | Notes |
|---|---|---|---|---|---|---|
| **DEC-010** | Whisper model bundling: ship inside app vs first-launch download | Mobile Lead | Open | S6 | End of S5 | Bundling +75 MB; download fails offline first launch. ADR-008 deferred this |
| **DEC-011** | Drift vs sqflite for SQLite | Mobile Lead | **Decided** | — | 2026-05-13 | Drift adopted; see ADR-009 |
| **DEC-012** | Force update enforcement threshold | PO + EL | Open | S10 | End of S9 | Tied to release plan |
| **DEC-013** | Crash reporter opt-in default (pilot vs prod) | Security Champion + PO | Open | S10 | End of S9 | Pilot opt-in; prod TBD |
| **DEC-014** | GPS retention window in FastAPI Postgres | Backend Lead + Security | **Decided** | — | 2026-05-15 | 180 days. Daily Celery purge job. Summary fields on Odoo never purged. |
| **DEC-015** | `x_ngynapp_*` field prefix | Backend Lead + Odoo Specialist | **Decided** | — | 2026-05-15 | Reserved program prefix. See `data/odoo-x-fields-spec.md`. |

## Decision template

When a decision is made:

```markdown
### DEC-NNN: <Title> — Decided YYYY-MM-DD

**Decision:** <one-line decision>
**Rationale:** <why>
**Implications:** <what changes>
**Linked ADR:** ADR-XXX (if architectural)
**Approved by:** <names per RACI>
```

## Closed decisions (full record)

### DEC-001: Odoo version + edition — Decided 2026-05-15 (amended 2026-05-15, v2.1.0)

**Decision:** Odoo 19 Enterprise is the test target. Production contract is RPC-only against any reasonably current Odoo (17+) on Community, Enterprise, or Online. The integration never branches on edition strings; it branches on observed module presence.
**Rationale:** Sponsor reconfirmed the production solution must run on Odoo Online and any Odoo version. Pinning to one edition would exclude SaaS customers and force premature upgrades.
**Implications:** RPC adapter (ADR-015) replaces fixed-contract integration. CI primary against Odoo 19 EE; weekly smoke matrix includes Odoo 17 Community to validate fallback. Path A / Path B model removed from `engineering/odoo-module-mapping.md`.
**Linked ADR:** ADR-011 (amended), ADR-015.
**Approved by:** Sponsor, Engineering Lead, Odoo Specialist.

### DEC-002: Installed Odoo modules — Decided 2026-05-15 (v2.1.0)

**Decision:** Hard requirements are `hr`, `project`, `contacts`, plus the `x_ngynapp_*` field set per `data/odoo-x-fields-spec.md`. All other modules (`industry_fsm`, `planning`, `hr_attendance`, `hr_timesheet`, `survey`, `quality`, `maintenance`, `documents`) are soft capabilities — install if you want them, the adapter detects and degrades gracefully when absent.
**Rationale:** Sponsor reconfirmed flexibility: any module can be installed if needed; the system must not require a specific set. Capability detection is the only durable contract across versions and editions.
**Implications:** FastAPI startup probe enumerates modules and emits a `ProbeReport`. Service refuses writes if hard requirements are missing. Admin alert fires with remediation steps.
**Linked ADR:** ADR-015.
**Approved by:** Sponsor, Engineering Lead, Backend Lead, Odoo Specialist.

### DEC-003: Custom Odoo module — Decided 2026-05-15 (flipped 2026-05-15, v2.1.0)

**Decision:** **No custom Odoo module.** All capabilities formerly delivered by `field_mobile_sync` are relocated:
- Idempotency: Redis SETNX in FastAPI + `x_ngynapp_client_id` Char on Odoo target records.
- Sync acknowledgement: read-after-write RPC; `SyncAck` carries Odoo `id` + `write_date`.
- GPS history: FastAPI Postgres `gps_pings` table (ADR-016); Odoo gets summary fields only.
- Sync state: FastAPI Postgres authoritative; mirror on `x_ngynapp_sync_state`.
- Audit: FastAPI structured logs + Postgres `audit_event` table.

**Rationale:** Sponsor reconfirmed solution must run on Odoo Online. Online prohibits custom Python modules. The capabilities are still required; they move to where we control the runtime.
**Implications:** ADR-014 superseded by ADR-015. The `field-mobile-sync` repo + Odoo CI pipeline is cancelled. EPIC-08 stories US-ODOO-001 (module skeleton) is rewritten to "RPC adapter scaffold + probe". Studio setup is the customer-side prerequisite.
**Linked ADR:** ADR-015 (supersedes ADR-014).
**Approved by:** Sponsor, Backend Lead, Odoo Specialist, Engineering Lead.

### DEC-004: GPS storage target — Decided 2026-05-15 (flipped 2026-05-15, v2.1.0)

**Decision:** GPS history lives in FastAPI Postgres (`gps_pings` table). Odoo retains only denormalised summary fields `x_ngynapp_gps_*` on `project.task` (or `fsm.task`).
**Rationale:** Custom Odoo model is no longer permitted (DEC-003). High-volume inserts and ad-hoc map queries fit better in Postgres than in Odoo's shared DB.
**Implications:** ADR-013 superseded by ADR-016. Map overlay surfaces via FastAPI dashboard linked from Odoo. Retention enforced in Postgres (180-day Celery purge — DEC-014). TC-GPS-001..004 rewritten.
**Linked ADR:** ADR-016 (supersedes ADR-013).
**Approved by:** Sponsor, Backend Lead, Engineering Lead, Odoo Specialist.

### DEC-005: SMS provider — Decided 2026-05-15

**Decision:** Twilio Verify only for MVP. Twilio account + API key issued and stored in environment vault. Fallback evaluated post-pilot via `DEC-005-FOLLOWUP`.
**Rationale:** Single provider; mature API; pilot scale does not justify dual-routing engineering cost. Sponsor confirmed credentials available now.
**Implications:** Auth flow (ADR-002) unchanged; fallback abstraction can be added later without breaking the OTP API contract.
**Linked ADR:** ADR-010.
**Approved by:** Sponsor, Backend Lead, Engineering Lead.

### DEC-006: Phone-to-employee mapping — Decided 2026-05-15

**Decision:** Resolve in order: `hr.employee.work_phone` first, then `hr.employee.private_phone`, then `hr.employee.mobile_phone`. Normalise the chosen value to E.164 (`+84…`) and store in `x_ngynapp_normalised_phone`; record which field supplied it in `x_ngynapp_phone_source`.
**Rationale:** Sponsor confirmed `work_phone` is the primary expectation, with `private_phone` as the realistic fallback for field workers without a corporate line.
**Implications:** A pre-pilot data-cleanup script populates `x_ngynapp_normalised_phone` for all active employees. OTP lookup queries this single field; no per-tenant branching. If a phone changes in Odoo, a Celery sync detects and updates the normalised mirror.
**Linked ADR:** ADR-015.
**Approved by:** Sponsor, Backend Lead, Odoo Specialist.

### DEC-007: Object storage — Decided 2026-05-15 (amended 2026-05-15, v2.1.0)

**Decision:** DigitalOcean Spaces in `sgp1`. SSE at rest (DO-managed AES-256), TLS 1.2+ in transit, public-access blocked, lifecycle archive prefix at 365 days. S3-compatible endpoint allows future migration to AWS S3 or Cloudflare R2 with config-only changes.
**Rationale:** Compute host is DO (DEC-009); single-vendor billing; flat-rate pricing fits pilot scale. S3-compatibility removes lock-in risk.
**Implications:** Bucket naming `field-mobile-{env}-evidence-sgp1`. Access keys scoped per environment, rotated 90 days. SDK config: `boto3` with `endpoint_url=https://sgp1.digitaloceanspaces.com`.
**Linked ADR:** ADR-012 (amended).
**Approved by:** Sponsor, DevOps, Backend Lead.

### DEC-008: Odoo SLA / scaling / DR — Decided 2026-05-15 (v2.1.0)

**Decision:** Production tenants delegate Odoo SLA to their Odoo provider (Odoo Online SLA, Odoo.sh SLA, or the customer's self-host operator). Our integration layer carries its own SLA defined in `architecture/non-functional-requirements.md`. Disaster recovery for our side: Postgres point-in-time recovery (managed DO Postgres) + DO Spaces versioning + warm-standby Droplet.
**Rationale:** Sponsor reconfirmed reliance on Odoo provider SLA; our team does not run Odoo. DO managed services provide the durability primitives we need without bespoke ops.
**Implications:** RTO 2h / RPO 15min targets apply to FastAPI + Postgres + Spaces. Odoo downtime is treated as upstream incident — mobile sync queue holds writes; on Odoo recovery, drains via standard backoff. `runbooks/odoo-dr-upgrade.md` updated to reference the provider SLA links.
**Linked ADR:** —.
**Approved by:** Sponsor, DevOps, Engineering Lead.

### DEC-009: Container orchestration target — Decided 2026-05-15 (v2.1.0)

**Decision:** DigitalOcean Droplets running Docker Compose for testing and pilot. Managed Postgres and managed Redis on DO. DO Spaces for object storage. Production scales by Droplet vertical size first (4GB → 8GB → 16GB); migration to DOKS (DigitalOcean Kubernetes) only when concurrent worker count exceeds threshold defined in `backlog/capacity-plan.md`.
**Rationale:** Same vendor as storage (DEC-007); predictable monthly cost; lowest ops complexity for testing and pilot scale; clean upgrade path.
**Implications:** `architecture/deployment.md` updated with DO topology. Single-region deployment in `sgp1`. Provisioning via Terraform module `tf-do-stack`. Estimated baseline cost ~$54/month for pilot (Droplet 4GB + managed Postgres + managed Redis).
**Linked ADR:** —.
**Approved by:** Sponsor, DevOps, Engineering Lead.

### DEC-011: Drift vs sqflite — Decided 2026-05-13

**Decision:** Adopt Drift 2.18+ for the mobile SQLite layer; sqflite forbidden in business code.
**Rationale:** Compile-time SQL, generated DAOs, mature migration tooling, clean SQLCipher integration.
**Implications:** `build_runner` lands in CI; generated `*.g.dart` checked in; lint forbids direct `sqflite` imports outside test fixtures.
**Linked ADR:** ADR-009.
**Approved by:** Mobile Lead, Engineering Lead.

### DEC-014: GPS retention window — Decided 2026-05-15 (v2.1.0)

**Decision:** GPS pings in FastAPI Postgres are retained 180 days, purged daily by Celery beat job `tasks.gps_retention.purge_old_pings` at 02:00 UTC. Summary fields on Odoo are never purged.
**Rationale:** 180 days covers typical labour-dispute and audit windows for the pilot population without unbounded growth.
**Implications:** Retention configurable via `GPS_RETENTION_DAYS` env; longer windows require disk capacity review.
**Linked ADR:** ADR-016.
**Approved by:** Backend Lead, Security Champion.

### DEC-015: `x_ngynapp_*` field prefix — Decided 2026-05-15 (v2.1.0)

**Decision:** All Odoo customisations introduced by this program live under the `x_ngynapp_` prefix. The prefix is reserved; no other namespace is used.
**Rationale:** Single recognisable namespace simplifies probing, auditing, and future cleanup. Shorter than full product name; long enough to avoid accidental collision.
**Implications:** Documented in `data/odoo-x-fields-spec.md`. Probe filters on this prefix. CI checks new specs do not leak other prefixes.
**Linked ADR:** ADR-015.
**Approved by:** Backend Lead, Odoo Specialist.

## How this connects

- Each story's "Open Questions" field references `DEC-NNN`. With Phase 0 fully closed in v2.1.0, all P0 stories are eligible to start.
- Sprint plans previously listed DEC-NNN dependencies; those gates are now satisfied.
- Future flips (very unlikely) will be recorded as amendments here and referenced from `CHANGELOG.md`.
