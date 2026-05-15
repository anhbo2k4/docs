---
id: US-CAP-004
epic: EPIC-05
sprint: S5
fr: [FR-007]
priority: P0
estimate: M
status: Ready
owner: Mobile
---

# US-CAP-004 — Pre-shift form UI

## Acceptance criteria

1. Pre-shift form renders fields per the static MVP schema (text, single-select, multi-select, numeric, signature placeholder for now).
2. Validation rules per field; submit disabled while invalid.
3. Drafts auto-save every 5 s and on navigation.
4. Schema versioned (`schema_version: 1`) and embedded in the envelope.
5. Reset confirms with the user before discarding draft.

## Tasks

- [ ] `PreShiftFormScreen` with declarative form widgets.
- [ ] Schema definition under `screen/field_work/forms/schemas/preshift_v1.dart`.
- [ ] Auto-save service.

## FR mapping

FR-007.

## Test cases

TC-FORM-001, TC-FORM-002.

## DoD

- All field types render and validate.
- Auto-save survives app kill.
