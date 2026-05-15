---
id: US-DISC-003
epic: EPIC-00
sprint: S0
fr: []
priority: P0
estimate: M
status: Ready
owner: Backend Lead + Odoo Specialist
---

# US-DISC-003 — Decide custom Odoo module scope (DEC-003, DEC-004)

## User story

**As** the engineering lead
**I want** the scope and timeline for the custom Odoo module `field_mobile_sync`
**so that** EPIC-08 can be planned and staffed.

## Acceptance criteria

1. Module manifest drafted: models, methods, security groups, dependencies.
2. GPS storage target selected (custom model recommended) — DEC-004 closed.
3. Timeline agreed: development, testing, deployment to staging by S8 mid-sprint.
4. Owner named (Odoo Specialist).
5. DEC-003 closed in decision log.

## Tasks

- [ ] Draft module manifest including `field.mobile.envelope`, `field.mobile.gps_event`, etc.
- [ ] Walk through manifest with Odoo admin.
- [ ] Confirm install/uninstall does not affect production data.
- [ ] Update `data/odoo-mapping.md` once DEC-004 lands.

## Dependencies

- US-DISC-001.

## FR mapping

Gates FR-006..FR-014.

## Test cases

None directly; informs TC-ODOO-001..006.

## Definition of Done

- DEC-003 and DEC-004 closed.
- Manifest committed under `data/`.
