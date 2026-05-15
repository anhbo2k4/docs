# Runbook: Rollback

**Status:** Active  
**Owner:** DevOps / SRE  
**Last tested:** TBD (target: end of S10)

## Symptom

A release has been deployed (backend or mobile) and triggers automatic or human rollback criteria. See `qa/pilot-gate.md` "Rollback triggers" section.

## Severity

SEV-1 or SEV-2. Rollback is itself a high-impact operation.

## Decision authority

Sponsor, PM, EL, or on-call engineer (escalating). For SEV-1 the on-call engineer can initiate without waiting for sign-off; record decision in incident timeline.

## Backend rollback

### Pre-flight

- [ ] Identify last known-good image tag (`git log` or release notes).
- [ ] Confirm Postgres migration history. If the bad release applied a migration, check whether it is reversible.
- [ ] Notify `#field-app-incidents` with planned action.

### Steps

```
1. Set SYNC_DRAIN_ONLY=true via feature flag.
   → Workers continue draining queue.
   → API rejects new envelopes with 503 RETRY_LATER.
2. kubectl set image deploy/fastapi fastapi=<previous-tag>
   kubectl set image deploy/celery celery=<previous-tag>
   kubectl rollout status deploy/fastapi
   kubectl rollout status deploy/celery
3. Verify /healthz and /readyz on both deployments.
4. Run smoke script: scripts/smoke.sh staging
5. If migrations were forward-only:
   - Down-migrate manually using prepared SQL in incident doc.
   - If unsafe, leave schema; verify previous-tag tolerates new schema (we test this in CI).
6. Set SYNC_DRAIN_ONLY=false.
7. Verify queue depth, sync success rate return to baseline.
```

### Verify

- `/healthz` 200 on both deployments.
- Queue depth trending down.
- Sync success rate ≥ 99 % over last 5 min.
- No 5xx spikes in API logs.

### After-action

- Postmortem within 5 business days.
- File RCA at `runbooks/incidents/INC-YYYYMMDD-NNN.md`.

---

## Mobile rollback

Mobile apps cannot be "rolled back" — only halted and replaced. Once a binary is in the user's hands, only forward fixes work, plus force-update.

### Steps

```
1. Halt rollout in stores.
   - Play Console: Pause production rollout.
   - App Store Connect: Pause Phased Release.
2. Promote a previous known-good build:
   - Play: Promote previous track release to production.
   - App Store: Submit a new build identical to last good (rejected for "no changes" sometimes — see fallback).
3. Set min_app_version on backend to a version below the bad one.
4. Notify users via in-app banner (if backend can serve one).
5. If the bad version is critical (data loss): set min_app_version to the new good version, force update.
```

### Fallback when stores reject "no-change" resubmit

- Ship a hot-fix build with a tiny visible change (e.g. version string).
- TestFlight / Play Internal first; promote to closed track within 4 h; production within 24 h depending on severity.

### Verify

- Crash-free sessions trend up.
- Sync success rate trend up.
- New installs go to good version.

---

## Combined backend + mobile rollback

If both layers need rollback (rare):

1. Backend first (faster, fewer downstream effects).
2. Drain queue.
3. Mobile rollback per above.
4. Re-evaluate after each step.

## Verify rollback completed

- [ ] Last known-good tag in production for both deployments.
- [ ] Smoke tests passing.
- [ ] Metrics returned to baseline for ≥ 30 min.
- [ ] Stakeholders informed (Slack + email digest).
- [ ] Incident timeline updated.

## Anti-patterns

- Do not skip the SYNC_DRAIN_ONLY step. Mid-rollback envelope intake leads to confusion in the queue.
- Do not down-migrate Postgres in a panic. Confirm reversibility first.
- Do not delete S3 objects to "clean state" — they may be referenced by Odoo records.
