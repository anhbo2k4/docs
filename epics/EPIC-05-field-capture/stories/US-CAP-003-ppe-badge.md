---
id: US-CAP-003
epic: EPIC-05
sprint: S5
fr: [FR-006, FR-015]
priority: P0
estimate: S
status: Ready
owner: Mobile
---

# US-CAP-003 — PPE state badge wired

## Acceptance criteria

1. PPE row in shift detail shows `SyncStateBadge` reflecting the latest state.
2. Badge updates in real time when the sync engine transitions the row.
3. Tap-on-FAILED routes to Sync Center entry (stub before EPIC-06 lands).

## Tasks

- [ ] Bind `SyncStateBadge` to a Riverpod stream from DAO.
- [ ] Tap handler with route stub.

## Dependencies

US-SHIFT-005, US-OFF-003.

## FR mapping

FR-006, FR-015.

## Test cases

TC-PPE-002, TC-SYNC-008.

## DoD

- Real-time updates verified by integration test.
