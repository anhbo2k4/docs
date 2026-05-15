---
id: US-OFF-003
epic: EPIC-04
sprint: S4
fr: [FR-012, FR-015]
priority: P0
estimate: M
status: Ready
owner: Mobile
---

# US-OFF-003 — Sync state machine + transitions

## Acceptance criteria

1. Single state-machine module under `@core/sync/state_machine.dart`.
2. States: `PENDING`, `SYNCING`, `CONFIRMED`, `FAILED`, `DEAD_LETTER`.
3. Allowed transitions encoded as a table; illegal transitions throw `InvalidTransitionException` and emit a Sentry event.
4. Every transition writes a `sync_attempt` row with `started_at`, `finished_at`, `outcome`, `http_status`, `error_code`, `error_message`.
5. Public API: `markSyncing`, `markConfirmed`, `markFailed(reason)`, `markDead(reason)`, `retry()`.
6. Unit tests cover all legal and illegal transitions.

## Tasks

- [ ] Implement state machine.
- [ ] Wire into `EnvelopeRepository`.
- [ ] Author transition table + tests.
- [ ] Update `data/sync-state-machine.md` to match.

## FR mapping

FR-012, FR-015.

## Test cases

TC-OFF-003, TC-SYNC-008.

## DoD

- All transitions tested.
- Doc and code in sync.
