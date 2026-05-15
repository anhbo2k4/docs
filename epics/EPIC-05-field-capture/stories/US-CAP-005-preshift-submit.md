---
id: US-CAP-005
epic: EPIC-05
sprint: S5
fr: [FR-007]
priority: P0
estimate: S
status: Ready
owner: Mobile
---

# US-CAP-005 — Pre-shift submit → SQLite

## Acceptance criteria

1. Submit writes `form_response` row + `sync_queue` row of type `FORM_RESPONSE` in one txn.
2. `payload.form_type = 'preshift'` and `payload.schema_version = 1`.
3. Status badge wired (re-uses US-CAP-003 pattern).
4. Submit is idempotent (same client_id on retry).

## Tasks

- [ ] `SubmitPreShiftFormUseCase`.
- [ ] Validation gate before write.

## FR mapping

FR-007.

## Test cases

TC-FORM-001, TC-FORM-002.

## DoD

- Same DoD as US-CAP-002.
