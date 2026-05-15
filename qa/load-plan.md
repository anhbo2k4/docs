# Load test plan

**Status:** Active
**Owner:** Backend Lead + QA Lead

## Goals

- Verify backend SLOs under expected pilot load and 5× burst.
- Detect regressions on every release before pilot deploy.
- Validate idempotency under concurrent retry storms.
- Surface bottlenecks (Postgres, Redis, Celery, Odoo) with per-stage timing.

## Tooling

- **k6** for HTTP load against FastAPI.
- **locust** for scenarios needing rich Python state (token rotation, multi-step flows).
- Run targets:
  - Local: against `make up` stack for smoke.
  - Staging: nightly via CI; full pilot scenario.
  - Pre-release: dedicated run before any pilot deploy.

## Scenarios

### LP-001 — Sync envelope steady state

- Profile: 10 RPS sustained for 30 min, 1 envelope/request, 60 % PPE / 30 % FORM / 10 % GPS mix.
- Assertions:
  - p50 ≤ 80 ms, p95 ≤ 200 ms, p99 ≤ 500 ms (NFR-011, TC-PERF-008).
  - Error rate < 0.1 %.
  - Celery queue depth < 50 at all times.
- Maps to: TC-PERF-008.

### LP-002 — Sync envelope burst

- Profile: ramp to 50 RPS over 1 min, hold 5 min, ramp down.
- Assertions:
  - p95 ≤ 400 ms during burst.
  - No 5xx; only 429 if rate-limit reached.
  - System recovers to LP-001 baseline within 2 min.
- Maps to: NFR-012, TC-PERF-009.

### LP-003 — Idempotency storm

- Profile: 50 concurrent virtual users send the same envelope (`client_id=X`) within 10 s.
- Assertions:
  - Exactly 1 Postgres row, 1 Celery dispatch, 1 Odoo write.
  - All 50 callers receive consistent status responses.
  - No deadlocks.
- Maps to: TC-SYNC-024, NFR-024.

### LP-004 — Auth exchange under load

- Profile: 5 RPS of `/v1/auth/exchange` for 10 min.
- Assertions:
  - p95 ≤ 250 ms.
  - Token issuance rate matches request rate.
  - JWT signing key in cache (no per-request KMS hit).
- Maps to: NFR-011, TC-AUTH-003 latency.

### LP-005 — Media presign + finalize

- Profile: 2 RPS presign, 2 RPS finalize for 10 min, 500 KB payloads.
- Assertions:
  - Presign p95 ≤ 100 ms.
  - Finalize p95 ≤ 250 ms.
  - S3 upload errors → orderly 5xx propagation, never silent.
- Maps to: TC-MEDIA-002, TC-MEDIA-003.

### LP-006 — Backpressure on Odoo unavailable

- Profile: simulate Odoo container stopped for 15 min while LP-001 runs.
- Assertions:
  - Envelopes accepted (PROCESSING).
  - Celery exponential backoff respected.
  - On Odoo restore: drain backlog within 10 min.
- Maps to: TC-ODOO-007, FR-013.

### LP-007 — Token refresh storm

- Profile: 100 sessions with synchronized expiries hit `/v1/auth/refresh` within 1 s.
- Assertions:
  - No DB row contention.
  - Each session gets a unique new refresh.
  - Old refresh marked rotated, then re-use on any of the old refreshes triggers TC-SEC-010.
- Maps to: TC-SEC-009/010.

### LP-008 — End-to-end sync from mobile harness

- Profile: 20 simulated mobile clients, each captures 50 envelopes over 30 min, with offline windows of 5 min sprinkled in.
- Assertions:
  - 100 % envelopes reach CONFIRMED.
  - p95 mobile-perceived sync latency ≤ 30 s after reconnect.
  - No DEAD_LETTER unless intentionally faulted.
- Maps to: TC-OFF-002, TC-SYNC-013, FR-013/015.

## Cadence

| Run | When | Trigger |
|---|---|---|
| Smoke | Every PR | CI on staging-lite |
| LP-001 + LP-003 | Nightly | Cron in CI |
| Full suite | Pre-release | Manual dispatch + sign-off |
| LP-008 | Pre-pilot | Once before S10 gate |

## Reporting

- k6 → JSON + summary in CI artifact.
- Trends tracked in Grafana ("Load Test History" dashboard).
- Regressions > 20 % on any p95 fail the build and require sign-off to ship.

## Test data

- Synthetic employees, shifts, and `client_id`s seeded in staging.
- No production data ever copied into load environments.
- Cleanup script wipes synthetic rows after each run.

## Capacity assumptions (initial)

| Resource | MVP target | Notes |
|---|---|---|
| FastAPI pods | 2 → 8 (HPA) | Triggered at 70 % CPU. |
| Celery workers | 4 | Concurrency 4 each. |
| Postgres | single primary, 4 vCPU / 16 GB | Read replica post-MVP. |
| Redis | 2 GB | Used by rate-limit + Celery broker. |
| Odoo | per DEC-008 | Pilot can degrade. |

These are starting points. Re-validate against LP-002 results before pilot.
