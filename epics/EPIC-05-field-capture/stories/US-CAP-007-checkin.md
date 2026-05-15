---
id: US-CAP-007
epic: EPIC-05
sprint: S5
fr: [FR-008]
priority: P0
estimate: M
status: Ready
owner: Mobile
---

# US-CAP-007 — Check-in capture + accuracy guard

## Acceptance criteria

1. Check-in CTA on shift detail captures GPS with timeout 15 s and required accuracy ≤ 50 m.
2. If accuracy not met, the user can retry or skip with reason; skip writes a `gps_event` with `accuracy_m = null` and a `reason_code`.
3. Successful capture writes `gps_event` (`event_type = CHECK_IN`) + `sync_queue` envelope of type `GPS_EVENT`.
4. Capture timestamp is monotonic (uses device monotonic clock + wall clock; both fields persisted).

## Tasks

- [ ] `CheckInUseCase`.
- [ ] Retry / skip dialog.
- [ ] Time service with monotonic + wall.

## Dependencies

US-CAP-006, US-OFF-002.

## FR mapping

FR-008.

## Test cases

TC-GPS-001, TC-GPS-003.

## DoD

- Accuracy gate verified.
- Skip path produces a structured envelope.
