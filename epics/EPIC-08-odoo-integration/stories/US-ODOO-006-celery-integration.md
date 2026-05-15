---
id: US-ODOO-006
epic: EPIC-08
sprint: S9
fr: [FR-013, FR-014, FR-015]
priority: P0
estimate: L
status: Ready
owner: Backend Lead
---

# US-ODOO-006 — Celery integration: retry, DLQ, ACK

## Acceptance criteria

1. Celery tasks defined per envelope type, all decorated with retry policy:
   - 5xx + transient errors: retry up to 8 with capped exponential backoff and jitter.
   - 4xx Odoo: no retry; route to DLQ with `error_code`.
2. On success, task updates `envelopes.status=CONFIRMED`, `confirmed_at`, `odoo_ref`.
3. On final failure, status `DEAD_LETTER`, `error_code`, `error_message` (sanitised).
4. Tasks are idempotent: re-running an already-confirmed envelope is a no-op.
5. Concurrency capped per worker pool to protect Odoo.
6. Replay tool processes a list of `client_id`s from DLQ and updates rows.
7. Metrics + alerts: DLQ growth rate; queue lag; task failure rate.

## Tasks

- [ ] Task implementations.
- [ ] Retry policy decorator.
- [ ] Replay CLI under `tools/sync_replay.py`.
- [ ] Alerts + dashboards.

## Dependencies

US-ODOO-001..005, US-API-004.

## FR mapping

FR-013, FR-014, FR-015.

## Test cases

TC-SYNC-006..010, TC-SYNC-024, TC-ODOO-024.

## DoD

- Replay tool exercised on a synthetic DLQ.
- Alerts wired in staging.
- 24-hour soak shows zero duplicates.
