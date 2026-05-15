---
id: US-SHIFT-005
epic: EPIC-03
sprint: S3
fr: [FR-005, FR-015]
priority: P0
estimate: S
status: Ready
owner: Mobile
---

# US-SHIFT-005 — Sync state badge component

## Acceptance criteria

1. `SyncStateBadge` widget renders one of: PENDING, SYNCING, CONFIRMED, FAILED, DEAD_LETTER.
2. Color and icon are tokens from `resource/theme/`.
3. Accessible via screen reader with explicit text label, not just color.
4. Used by shift list rows and detail header.
5. Tap on FAILED or DEAD_LETTER routes to the Sync Center entry for the related envelope (Sync Center built in EPIC-06; route stubbed here).

## Tasks

- [ ] Author widget with tests.
- [ ] Add a11y semantics.
- [ ] Document tokens used.

## FR mapping

FR-005, FR-015.

## Test cases

TC-SHIFT-003, TC-SYNC-005.

## DoD

- Widget golden tests for all five states.
- A11y label per state present.
