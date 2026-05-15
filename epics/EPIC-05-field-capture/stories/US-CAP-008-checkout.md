---
id: US-CAP-008
epic: EPIC-05
sprint: S5
fr: [FR-008]
priority: P0
estimate: M
status: Ready
owner: Mobile
---

# US-CAP-008 — Check-out capture + reconcile timing

## Acceptance criteria

1. Check-out CTA captures GPS with the same accuracy/timeout policy as check-in.
2. Validates that a check-in exists for the same shift; if missing, prompts to perform check-in first or capture both with a justification note.
3. Records `event_type = CHECK_OUT` and the elapsed duration computed from check-in.
4. Visible badge for both events from shift detail.

## Tasks

- [ ] `CheckOutUseCase`.
- [ ] Reconciliation logic (`HasCheckInForShiftQuery`).

## Dependencies

US-CAP-007.

## FR mapping

FR-008.

## Test cases

TC-GPS-002, TC-GPS-003.

## DoD

- Missing-check-in path verified.
- Duration calculation correct across timezone changes.
