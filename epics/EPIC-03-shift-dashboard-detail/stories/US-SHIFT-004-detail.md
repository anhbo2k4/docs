---
id: US-SHIFT-004
epic: EPIC-03
sprint: S3
fr: [FR-005]
priority: P0
estimate: M
status: Ready
owner: Mobile
---

# US-SHIFT-004 — Shift detail with capture CTAs

## Acceptance criteria

1. Detail screen shows: title, site, start–end, status, instructions, attachments (if any), and four CTAs:
   - PPE check
   - Pre-shift form
   - Work result (composer)
   - Post-shift form
2. CTAs are stubs in S3; full flows arrive in S5/S6 — they render but route to placeholder screens labelled "Coming soon".
3. Status pill matches list, plus a sync-state badge for any captures linked to this shift (visible after EPIC-04 lands).
4. Deep-link from notifications opens the right shift (post-MVP push, but the route is wired now).
5. Not-assigned deep link returns a friendly error.

## Tasks

- [ ] `ShiftDetailScreen` widget.
- [ ] CTA registry per capture type.
- [ ] Deep-link route registration.

## FR mapping

FR-005.

## Test cases

TC-SHIFT-003, TC-SHIFT-004.

## DoD

- Detail screen renders all required fields.
- CTAs route to stubs.
