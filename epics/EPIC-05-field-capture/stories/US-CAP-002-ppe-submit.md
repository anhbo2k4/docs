---
id: US-CAP-002
epic: EPIC-05
sprint: S5
fr: [FR-006]
priority: P0
estimate: M
status: Ready
owner: Mobile
---

# US-CAP-002 — PPE submit → SQLite + sync_queue

## Acceptance criteria

1. Submit creates a `ppe_check` row with `client_id` (UUID v4 generated at tap time) and `sync_state = PENDING`.
2. In the same transaction, insert a `sync_queue` row of type `PPE_CHECK` with the same `client_id`.
3. `sync_attempt` row records the initial enqueue with `outcome = 'PENDING'`.
4. UI updates the badge to PENDING and routes back to shift detail.
5. Submit works fully offline (airplane mode test).
6. Idempotent: if the user double-taps, only one row inserts (debounce + DB constraint).

## Tasks

- [ ] `SubmitPpeUseCase` with txn write.
- [ ] UUID v4 generator with platform-safe randomness.
- [ ] Debounce on submit button.

## Dependencies

US-CAP-001, US-OFF-002, US-OFF-003.

## FR mapping

FR-006, FR-012.

## Test cases

TC-PPE-001, TC-PPE-002, TC-OFF-001.

## DoD

- Offline submit verified by integration test.
- Double-tap test produces exactly one row.
