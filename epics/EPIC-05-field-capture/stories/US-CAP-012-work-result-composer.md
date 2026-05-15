---
id: US-CAP-012
epic: EPIC-05
sprint: S5
fr: [FR-010]
priority: P0
estimate: M
status: Ready
owner: Mobile
---

# US-CAP-012 — Work result composer

## Acceptance criteria

1. Composer collects: notes (≤ 4000 chars), completion percentage (0–100, optional), linked photos (selected from gallery), linked voice notes (S6 hook).
2. Submit writes `work_result` row + `WORK_RESULT` envelope referencing media `client_id`s.
3. References to media are validated: every linked `client_id` must exist locally.
4. Submit blocked while a referenced media has `sync_state = DEAD_LETTER`.
5. Submit is offline-safe.

## Tasks

- [ ] `WorkResultScreen`.
- [ ] `SubmitWorkResultUseCase`.
- [ ] Validation gates.

## FR mapping

FR-010.

## Test cases

TC-MEDIA-005, TC-MEDIA-006.

## DoD

- Reference validation enforced.
- Offline path verified.
