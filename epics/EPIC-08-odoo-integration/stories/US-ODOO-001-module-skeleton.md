---
id: US-ODOO-001
epic: EPIC-08
sprint: S9
fr: [FR-014]
priority: P0
estimate: M
status: Ready
owner: Odoo Specialist
---

# US-ODOO-001 — Custom module `field_mobile_sync` skeleton

## User story

**As** the Odoo specialist
**I want** a clean Odoo module that owns mobile-sync models and write methods
**so that** core Odoo modules remain unmodified and upgrades stay safe.

## Acceptance criteria

1. Module manifest declares dependencies (`base`, `hr`, modules per DEC-002), version, license.
2. Models created:
   - `field.mobile.envelope`: stores `client_id` (indexed unique), `type`, `employee_id`, `shift_id`, `received_at`, `confirmed_at`, `status`, `error_code`, `payload_json`.
   - Per-type child references (e.g. `field.mobile.gps_event`) where DEC-004 chose custom models.
3. Security groups: `Field Mobile Sync / User`, `Field Mobile Sync / Manager`.
4. Reproducible install script under `infra/odoo/install_module.sh`.
5. Uninstall is non-destructive (data retained; tables left in place).
6. Module deploys to staging via CI step.

## Tasks

- [ ] Module skeleton + manifest.
- [ ] Models + security XML.
- [ ] Install script.
- [ ] CI deploy step.

## Dependencies

DEC-001, DEC-002, DEC-003, DEC-004.

## FR mapping

FR-014.

## Test cases

TC-ODOO-001 (per envelope type, end-to-end).

## DoD

- Module installs cleanly on staging mirror.
- Idempotent re-install verified.
