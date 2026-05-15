---
id: US-ODOO-002
epic: EPIC-08
sprint: S9
fr: [FR-006, FR-007, FR-014]
priority: P0
estimate: M
status: Ready
owner: Odoo Specialist
---

# US-ODOO-002 — PPE + Form write methods (idempotent)

## Acceptance criteria

1. Methods `field.mobile.envelope.process_ppe_check(envelope)` and `process_form_response(envelope)` write to the chosen Odoo target (per DEC-002/003).
2. Lookup by `client_id` external_id; if a record exists, return it (idempotent).
3. Validation matches FastAPI: missing fields → 422 mapped to `VALIDATION_FAILED`.
4. Returns `odoo_ref` (model + id).

## Tasks

- [ ] PPE write logic.
- [ ] Form write logic.
- [ ] Unit tests with Odoo test framework.

## FR mapping

FR-006, FR-007, FR-014.

## Test cases

TC-ODOO-001, TC-ODOO-024.

## DoD

- 100 retries on same client_id → 1 record.
- Validation paths produce typed errors.
