---
id: US-CAP-001
epic: EPIC-05
sprint: S5
fr: [FR-006]
priority: P0
estimate: M
status: Ready
owner: Mobile
---

# US-CAP-001 — PPE checklist UI

## User story

**As** a field worker
**I want** to complete a PPE checklist before starting my shift
**so that** safety compliance is recorded and visible to supervisors.

## Acceptance criteria

1. PPE screen lists checklist items per static schema (helmet, gloves, harness, boots, hi-vis, glasses) with toggles.
2. Items required for shift completion are marked; submit is disabled until all required items are confirmed or a justification note is entered.
3. Notes field supports up to 1000 characters with character counter.
4. Save button persists the capture (US-CAP-002 handles persistence).
5. Loading from existing draft: if the user navigates away and back, the partial state restores.
6. Accessible: each toggle has a label and is reachable by screen reader.

## Tasks

- [ ] `PpeCheckScreen` widget under `screen/field_work/ppe/`.
- [ ] Domain model `PpeChecklist` with required-item rules.
- [ ] Local draft persistence keyed by `(shift_id, employee_id)`.

## Dependencies

EPIC-04 (storage) for draft + final write.

## FR mapping

FR-006.

## Test cases

TC-PPE-001 (offline write), TC-PPE-003 (a11y).

## Definition of Done

- AC met across iOS and Android.
- Widget tests cover required-rule and draft restore.
