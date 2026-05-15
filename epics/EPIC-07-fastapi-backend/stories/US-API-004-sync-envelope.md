---
id: US-API-004
epic: EPIC-07
sprint: S8
fr: [FR-012, FR-013, FR-014, FR-015]
priority: P0
estimate: L
status: Ready
owner: Backend Lead
---

# US-API-004 — `/v1/sync/envelope` ingest + idempotency + Celery enqueue

## User story

**As** the backend
**I want** to accept envelopes from mobile, deduplicate by `client_id`, and enqueue Odoo writes
**so that** mobile retries are safe and Odoo is never written twice for the same logical action.

## Acceptance criteria

1. Endpoint accepts JSON envelopes per `api-contracts/sync.md`.
2. Idempotency middleware:
   - Unique key on `(employee_id, client_id)`.
   - Same `client_id` + same payload → 200 with current status.
   - Same `client_id` + different payload → 409 `CLIENT_ID_CONFLICT`.
3. Validates per-type payload schema; 400 `INVALID_ENVELOPE` on failure.
4. Persists envelope to Postgres in a single transaction with status `PENDING`.
5. Enqueues a Celery task per envelope type; failure to enqueue rolls back the persist (transactional outbox).
6. Returns 202 with `client_id` and `status`.
7. p95 server time ≤ 200 ms (TC-PERF-008).
8. Backpressure: 429 on tenant burst (Redis sliding window).
9. Auth: requires valid access JWT; 401 on missing or expired.

## Tasks

- [ ] Schemas per envelope type with discriminator.
- [ ] Idempotency middleware (Redis-assisted lookup, Postgres unique-key authoritative).
- [ ] Transactional outbox for Celery enqueue.
- [ ] Per-tenant rate limit.
- [ ] Tests: dup, conflict, invalid, expired token, rate limit.

## Dependencies

US-API-001, US-API-002.

## FR mapping

FR-012, FR-013, FR-014, FR-015.

## Test cases

TC-SYNC-001, TC-SYNC-002, TC-SYNC-003, TC-SYNC-004, TC-SYNC-024, TC-PERF-008.

## DoD

- Retry storm test: 50 concurrent same-`client_id` POSTs → exactly one persisted row.
- Storm test: 50 concurrent different `client_id` → all persisted, all enqueued.
- p95 budget verified in staging under 100 RPS.
