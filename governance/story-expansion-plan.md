# Story expansion plan

**Status:** Active
**Owner:** Project Manager + Engineering Lead
**Last updated:** 2026-05-13

## Why this exists

Story files vary in depth. Self-critique gap A: stories under ~1 KB lack enough AC, tasks, test mapping, and DoD for an autonomous agent to execute without conversation. The fix is sprint-by-sprint expansion during refinement.

## Expansion checklist (per story)

A story is "fully expanded" when it has:

- [ ] Front matter complete and current.
- [ ] User story sentence (`As / I want / so that`).
- [ ] **Background** paragraph linking to the relevant ADR(s) and contract(s).
- [ ] **Acceptance criteria** numbered, each measurable and pass/fail. Minimum 5 for any P0.
- [ ] **Tasks** broken into `Mobile / Backend / Tests` (omit sections that don't apply).
- [ ] **Dependencies** with story IDs and DEC-NNN.
- [ ] **Risks / Open questions** linking RISK and DEC IDs.
- [ ] **FR mapping** to one or more `FR-NNN`.
- [ ] **Test cases** to one or more `TC-AREA-NNN`.
- [ ] **Definition of Done** specific to this story (delta over the global DoD).

## Reference exemplars

These are full-template stories you can copy from when expanding:

- `epics/EPIC-02-auth-employee-mapping/stories/US-AUTH-001-phone-entry.md` — UI-leaning, security-sensitive.
- `epics/EPIC-06-sync-engine/stories/US-SYNC-001-queue-manager.md` — infrastructure, multi-area, large.

## Per-sprint expansion target

Each sprint's stories must be expanded **before sprint planning** of that sprint. PM owns the calendar.

| Sprint | Stories to expand by Day -3 of sprint planning | Owner |
|---|---|---|
| S1 | All `US-PLAT-*` | Mobile Lead + Backend Lead |
| S2 | All `US-AUTH-*` | Mobile Lead + Security Champion |
| S3 | All `US-SHIFT-*` | Mobile Lead |
| S4 | All `US-OFF-*` | Mobile Lead |
| S5 | `US-CAP-001..008` (PPE, form, GPS) | Mobile Lead |
| S6 | `US-CAP-009..013`, all `US-VOICE-*` | Mobile Lead |
| S7 | All `US-SYNC-*` | Mobile Lead |
| S8 | All `US-API-*` | Backend Lead |
| S9 | All `US-ODOO-*` | Backend Lead + Odoo Specialist |
| S10 | All `US-QA-*` | QA Lead |

## Process

1. PM picks the next sprint's backlog 3 days before planning.
2. Story owner copies the exemplar template.
3. Story owner fills missing sections; missing info becomes a `DEC-NNN` or a story dependency.
4. PM reviews against the checklist; rejects if AC < 5 for P0 or any "TBD" remains.
5. Refined stories enter the planning meeting Ready.

## Tracking

PM keeps a one-line status per story in this file's appendix; a green check means the story has passed the checklist for the upcoming sprint.

## Appendix — current expansion status

| Story | Sprint | Bytes | Expanded? | Notes |
|---|---|---|---|---|
| US-AUTH-001 | S2 | 2.0 KB | ✅ | Exemplar. |
| US-AUTH-002..007 | S2 | 1.5–2.0 KB | partial | Expand before S2 planning. |
| US-SYNC-001 | S7 | 4+ KB | ✅ | Exemplar. |
| US-SYNC-002..006 | S7 | 0.7–0.9 KB | ⏳ | Expand before S7 planning. |
| US-API-004 | S8 | 1.9 KB | ⏳ | Already medium; add tests + DoR confirmation. |
| US-PLAT-001..006 | S1 | 1.0–1.9 KB | ⏳ | Expand before S1 planning. |
| US-OFF-001..006 | S4 | 0.8–1.5 KB | ⏳ | Expand before S4 planning. |
| US-CAP-001..013 | S5–S6 | 0.6–1.4 KB | ⏳ | Expand before S5/S6 planning. |
| US-VOICE-001..004 | S6 | 0.6–0.9 KB | ⏳ | Expand before S6 planning. |
| US-ODOO-001..006 | S9 | 0.6–1.4 KB | ⏳ | Expand before S9 planning. |
| US-QA-001..006 | S10 | 0.6–0.9 KB | ⏳ | Expand before S10 planning. |
| US-DISC-001..009 | S0 | 0.5–1.7 KB | ⏳ | Expand within S0 (discovery; lighter template). |
