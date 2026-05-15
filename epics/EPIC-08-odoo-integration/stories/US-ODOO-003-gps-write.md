---
id: US-ODOO-003
epic: EPIC-08
sprint: S9
fr: [FR-008, FR-014]
priority: P0
estimate: M
status: Ready
owner: Odoo Specialist
---

# US-ODOO-003 — GPS event write method (model per DEC-004)

## Acceptance criteria

1. `process_gps_event(envelope)` writes to the GPS storage target chosen in DEC-004.
2. Stores `event_type`, `latitude`, `longitude`, `accuracy_m`, `altitude_m`, `captured_at`, `employee_id`, `shift_id`.
3. Idempotent by `client_id`.
4. Optional reverse-geocode field deferred to post-MVP.

## Tasks

- [ ] Model writes per DEC-004.
- [ ] Tests.

## FR mapping

FR-008, FR-014.

## Test cases

TC-ODOO-003.

## DoD

- Same idempotency proof as US-ODOO-002.
