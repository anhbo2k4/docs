---
id: US-DISC-002
epic: EPIC-00
sprint: S0
fr: []
priority: P0
estimate: S
status: Ready
owner: Backend Lead + Odoo Specialist
---

# US-DISC-002 — Confirm employee phone-mapping strategy (DEC-006)

## User story

**As** the backend lead
**I want** a chosen phone-to-`hr.employee` mapping field
**so that** US-AUTH-005 can implement a deterministic lookup.

## Acceptance criteria

1. Field-priority order picked: e.g. `mobile_phone` > `work_phone` > `x_field_mobile`.
2. Sample dataset (≥ 50 employees) verified: zero / one / multiple matches counted.
3. Ambiguous-match policy chosen (reject vs prefer most-recent vs admin-resolve).
4. DEC-006 status moves to `Decided`.

## Tasks

- [ ] Pull anonymised export from `hr.employee`.
- [ ] Run mapping simulation and capture metrics.
- [ ] Lock priority order in decision log.
- [ ] Update `security/auth-flow.md` if behaviour changes.

## Dependencies

- US-DISC-001 (Odoo version known).

## Risks

- RISK-021 ambiguous mapping.

## FR mapping

Gates FR-002.

## Test cases

None directly; informs TC-AUTH-006.

## Definition of Done

- DEC-006 closed.
- Sample run report committed under workshop notes.
