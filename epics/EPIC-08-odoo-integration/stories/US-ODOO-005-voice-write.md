---
id: US-ODOO-005
epic: EPIC-08
sprint: S9
fr: [FR-011, FR-014]
priority: P0
estimate: M
status: Ready
owner: Odoo Specialist
---

# US-ODOO-005 — Voice note + transcript write

## Acceptance criteria

1. `process_voice_note(envelope)` stores transcript, `duration_ms`, link to associated audio media record.
2. Both `transcript_original` (mobile-edited) and `transcript_at_submit` are stored if differing.
3. Idempotent by `client_id`.

## Tasks

- [ ] Voice note model + write.
- [ ] Transcript fields.
- [ ] Tests.

## FR mapping

FR-011, FR-014.

## Test cases

TC-ODOO-005.

## DoD

- Replay on duplicate yields no new record.
