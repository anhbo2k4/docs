---
id: US-CAP-013
epic: EPIC-05
sprint: S5
fr: [FR-010, FR-011]
priority: P0
estimate: S
status: Ready
owner: Mobile
---

# US-CAP-013 — Work result link voice notes (S6 hook)

## Acceptance criteria

1. Composer surfaces a "Add voice note" CTA that routes to the recorder (US-VOICE-001 in S6).
2. After voice envelope creation, the voice `client_id` is linked to the work result.
3. UI shows the voice's transcript preview (first 80 chars) inline.
4. Removal supported pre-submit.

## Tasks

- [ ] CTA + binding.
- [ ] Live update on transcript readiness.

## FR mapping

FR-010, FR-011.

## Test cases

TC-VOICE-005.

## DoD

- Linkage round-trip verified.
- Stub navigates correctly even before US-VOICE-001 lands.
