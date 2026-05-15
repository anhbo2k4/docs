# Communication Plan

**Status:** Active  
**Owner:** Project Manager

## Cadence

| Event | Frequency | Duration | Attendees | Output |
|---|---|---|---|---|
| Daily standup | Daily, Mon–Fri | 15 min | EL, ML, BL, QA, PM | Yesterday / today / blockers |
| Sprint planning | Start of sprint (every 2 weeks) | 2 h | PO, PM, EL, ML, BL, QA, DES | Sprint backlog, capacity |
| Backlog refinement | Mid-sprint | 1 h | PO, EL, ML, BL, QA | Stories ready for next sprint |
| Sprint review / demo | End of sprint | 1 h | PO, PM, EL, ML, BL, QA, STAKE, SP | Demo, feedback, accepted stories |
| Sprint retrospective | End of sprint | 45 min | PM, EL, ML, BL, QA, DES | Action items |
| Architecture review | On-demand | 1 h | EL, ML, BL, SEC, OPS | ADR draft |
| Risk review | Bi-weekly | 30 min | PM, EL, QA, SEC | Updated risk register |
| Steering committee | Monthly | 1 h | SP, PO, PM, EL, STAKE | Status, scope, budget |
| Pilot readiness review | Before pilot | 2 h | All | Pilot gate decision |
| Incident postmortem | Within 5 business days of incident | 1 h | EL, ML, BL, QA, OPS, SEC | Root cause + actions |

## Channels

| Channel | Purpose | Retention |
|---|---|---|
| **Slack `#field-app-dev`** | Daily dev chatter, blockers | 1 year |
| **Slack `#field-app-incidents`** | Incidents, alerts, paging | Permanent |
| **Slack `#field-app-stakeholders`** | Stakeholder updates, demos | Permanent |
| **Email (weekly digest)** | Status to sponsors | Permanent |
| **Confluence / wiki (this repo)** | Source of truth for docs | Permanent |
| **Jira / Linear** | Story tracking | Permanent |
| **PagerDuty** | On-call rotation, P0/P1 alerts | Permanent |
| **Zoom** | Meetings (recorded for sprint review and steering) | 90 days |

## Status reporting

- **Daily:** standup notes posted to `#field-app-dev`.
- **Weekly:** PM sends digest email every Friday with: progress, risks, decisions needed, next week plan.
- **Per sprint:** sprint review deck + acceptance log committed to `sprints/sprint-NN/review.md`.
- **Per release:** release notes per `runbooks/release-notes-template.md`.

## Escalation

See [escalation.md](./escalation.md).

## Stakeholder map

See [stakeholders.md](./stakeholders.md).
