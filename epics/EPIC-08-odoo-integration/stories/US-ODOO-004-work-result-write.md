---
id: US-ODOO-004
epic: EPIC-08
sprint: S9
fr: [FR-009, FR-010, FR-014]
priority: P0
estimate: L
status: Ready
owner: Odoo Specialist
---

# US-ODOO-004 — Work result + media reference write

## Acceptance criteria

1. `process_work_result(envelope)` creates a work-result record linked to the shift; references media records via their `client_id`s.
2. Media references are stored with object-storage URI plus `sha256`; binary blobs are not embedded in Odoo.
3. Voice note linkage carried through (`voice_note` references in payload).
4. Idempotent and resilient to partial failure (creates orphan-safe rows that DLQ replay can finish).

## Tasks

- [ ] Work result model + write.
- [ ] Media reference table + write.
- [ ] Partial-failure recovery story.

## FR mapping

FR-009, FR-010, FR-014.

## Test cases

TC-ODOO-004, TC-ODOO-024.

## DoD

- Replay completes orphaned envelopes after a simulated mid-write failure.
