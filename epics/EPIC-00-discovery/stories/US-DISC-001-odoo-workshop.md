---
id: US-DISC-001
epic: EPIC-00
sprint: S0
fr: []
priority: P0
estimate: M
status: Ready
owner: PM + Backend Lead
---

# US-DISC-001 — Run Odoo discovery workshop (DEC-001, DEC-002)

## User story

**As** the project sponsor and engineering lead
**I want** a definitive list of Odoo version, edition, and installed modules
**so that** the team can pick the right shift entity and write path before S1 starts.

## Acceptance criteria

1. Live walkthrough with Odoo admin captures: version (e.g. 17.0), edition (Community vs Enterprise), and modules installed (Project, Field Service, Planning, Survey, HR, Maintenance).
2. Output document records exact module list, with screenshots of `Apps` page and version banner.
3. DEC-001 and DEC-002 status in `traceability/decision-log.md` move to `Decided`.
4. Implication notes: which Odoo entity will represent a Shift (`project.task` vs `planning.slot` vs custom).
5. Risk owners assigned for any "module not installed but required" gap.

## Tasks

- [ ] Schedule 90-minute workshop with Odoo admin + IT.
- [ ] Capture live screenshots and module manifest.
- [ ] Update `traceability/decision-log.md` with DEC-001, DEC-002.
- [ ] If gap found, raise issue and assign owner.

## Dependencies

- Sponsor authorising the meeting.
- Odoo admin available with prod-equivalent instance.

## Risks / Open questions

- DEC-001, DEC-002 themselves.
- RISK-001 Odoo decisions slip.

## FR mapping

None directly; gates EPIC-03, EPIC-08.

## Test cases

None (process story).

## Definition of Done

- DEC-001 and DEC-002 closed in decision log.
- Workshop notes attached as a sub-page or PR.
- Sponsor email confirming version + edition is in the project record.
