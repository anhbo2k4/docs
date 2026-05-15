# Stakeholder Map

**Status:** Active  
**Owner:** Project Manager

## Internal

| Stakeholder | Role | Interest | Influence | Engagement strategy |
|---|---|---|---|---|
| Sponsor | Funds project, owns ROI | High | High | Monthly steering, weekly digest |
| Product Owner | Owns scope and backlog | High | High | Daily, sprint events |
| Engineering Lead | Owns architecture and delivery | High | High | Daily, architecture reviews |
| Mobile Lead | Owns Flutter app | High | Medium | Daily, sprint events |
| Backend Lead | Owns FastAPI + Odoo integration | High | Medium | Daily, sprint events |
| QA Lead | Owns quality and pilot gate | High | Medium | Daily, sprint events, pilot gate |
| Security Champion | Owns threat model | Medium | Medium | Architecture review, security checkpoints (S1, S5, S9) |
| DevOps / SRE | Owns infra and CI/CD | Medium | Medium | Sprint events, runbook reviews |
| Designer | Owns UX | Medium | Low | Sprint planning, design reviews |
| Odoo Specialist | Owns ERP mapping | High | Medium | Phase 0 + S8/S9 + integration tests |

## External

| Stakeholder | Role | Interest | Influence | Engagement strategy |
|---|---|---|---|---|
| Pilot site supervisor | Daily user, primary feedback | Very High | High | Pilot kickoff, weekly check-in during pilot |
| Pilot field workers | End users | Very High | Low | Onboarding, in-app feedback channel |
| HR / People Ops | Employee data owner | Medium | Medium | Phase 0 employee mapping (DEC-006) |
| Compliance / Legal | Privacy, data retention | Medium | High | Threat model review, data retention policy sign-off |
| Twilio | OTP provider | Low | Low | Vendor support channel only |
| Supabase | Edge functions provider | Low | Low | Vendor support channel only |

## Communication preferences

| Stakeholder | Preferred channel | Frequency |
|---|---|---|
| Sponsor | Email digest + monthly meeting | Weekly + monthly |
| Product Owner | Slack + sprint events | Daily |
| Pilot supervisor | Slack + weekly call | Weekly during pilot |
| Field workers | In-app feedback + onboarding sessions | Per release |
| Compliance | Email + scheduled reviews | Per milestone |

## Engagement timeline

```
Phase 0 (S0)        Phase 1 (S1-S6)         Phase 2 (S7-S9)         Phase 3 (S10)
─────────────       ──────────────────      ──────────────────      ─────────────
Sponsor: kick       Sponsor: monthly        Sponsor: monthly        Sponsor: pilot decision
PO: deep            PO: daily               PO: daily               PO: daily + pilot
Stake: discovery    Stake: monthly          Stake: bi-weekly        Stake: pilot kickoff
Compliance: scope   Compliance: review S5   Compliance: review S9   Compliance: pilot sign-off
Odoo: must close    Odoo: light             Odoo: heavy             Odoo: validation
```
