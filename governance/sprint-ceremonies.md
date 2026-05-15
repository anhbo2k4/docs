# Sprint ceremonies

**Status:** Active
**Owner:** Project Manager

Two-week sprint cadence. Times below are illustrative; PM confirms per locale.

## Calendar

| Ceremony | Cadence | Duration | Attendees | Output |
|---|---|---|---|---|
| Sprint Planning | Day 1, Mon | 2 h | PO, EL, Mobile Lead, Backend Lead, QA, PM | Sprint backlog committed |
| Daily Standup | Daily, Mon–Fri | 15 min | All engineers + QA + PM | Status + blockers |
| Backlog Refinement | Day 5, Fri | 1 h | PO, EL, leads, QA | Next sprint candidates Ready |
| Mid-sprint Review | Day 6, Mon | 30 min | EL + PM + leads | Risk + scope adjustment |
| Sprint Review | Day 10, Fri am | 1 h | + Sponsor | Demo + acceptance |
| Sprint Retrospective | Day 10, Fri pm | 1 h | Engineers + QA + PM | Action items |

## Sprint Planning agenda

1. Confirm Sprint Goal (1 sentence).
2. Review carried-over stories.
3. Pull stories meeting DoR (`traceability/definitions.md`).
4. Apply WIP limits.
5. Identify dependencies and DECs that must close inside the sprint.
6. Confirm capacity (PTO, on-call, holidays).
7. Restate non-goals.
8. Sign-off from PO + EL.

## Daily Standup format

- What I shipped yesterday.
- What I'll ship today.
- Blockers (only blockers; not status).
- After standup: PM hosts a 5-min "blockers" room to unstick.

## Sprint Review

- Demo by feature owner (mobile + backend pair when relevant).
- PO accepts or rejects each story against AC.
- Stories not Done roll over only if the next sprint has capacity; otherwise they return to backlog.

## Retrospective

- Format: Start / Stop / Continue or Mad / Sad / Glad. Rotate.
- One owner per action item; due date in the next sprint.
- Action items reviewed at the start of the next retro.

## Sprint zero-defects rule

A story counts toward velocity only if it passes DoD. Bugs filed against it within the same sprint reduce that count.

## Stakeholder review

Sponsor reviews `Pilot Health` dashboard weekly and joins Sprint Review every other sprint or on demand.
