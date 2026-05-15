---
sprint: S0
title: Discovery & Build Authorization
duration: 2 weeks
priority: P0
status: Ready
owner: PM + Sponsor
---

# Sprint 0 — Discovery & Build Authorization

## 1. Sprint goal

Close the 9 Phase 0 decisions and obtain a signed SOW so S1 can start on schedule with no unanswered structural questions.

## 2. Theme

Discovery only. The team does not write production code in S0. The deliverable is **clarity**: who owns what, what Odoo we are integrating with, which providers we use, and what we are committing to build.

## 3. Scope (in)

- Workshops with sponsor, IT, HR, Odoo admin.
- Closing DEC-001 through DEC-009.
- Drafting and signing the SOW.
- Onboarding plan for S1.
- Local dev environment requirements documented.

## 4. Out of scope

- Any Flutter or FastAPI scaffolding (lands in S1).
- Pilot user recruitment (S10).
- Any Odoo customisation work.

## 5. Pre-conditions

- Sponsor available for sign-off.
- Odoo admin available for live walkthrough.
- HR contact available for phone-mapping data sample.

## 6. Stories committed

| ID | Title | Owner | Estimate |
|---|---|---|---|
| [US-DISC-001](../../epics/EPIC-00-discovery/stories/US-DISC-001-odoo-workshop.md) | Run Odoo discovery workshop (DEC-001, DEC-002) | PM | M |
| [US-DISC-002](../../epics/EPIC-00-discovery/stories/US-DISC-002-employee-mapping.md) | Confirm employee phone-mapping strategy (DEC-006) | Backend Lead | S |
| [US-DISC-003](../../epics/EPIC-00-discovery/stories/US-DISC-003-odoo-module-scope.md) | Decide custom Odoo module scope (DEC-003, DEC-004) | Backend Lead | M |
| [US-DISC-004](../../epics/EPIC-00-discovery/stories/US-DISC-004-sms-provider.md) | Pick SMS provider (DEC-005) | Backend Lead | S |
| [US-DISC-005](../../epics/EPIC-00-discovery/stories/US-DISC-005-object-storage.md) | Pick object storage and region (DEC-007) | DevOps | S |
| [US-DISC-006](../../epics/EPIC-00-discovery/stories/US-DISC-006-orchestration-target.md) | Pick container orchestration target (DEC-009) | DevOps | S |
| [US-DISC-007](../../epics/EPIC-00-discovery/stories/US-DISC-007-odoo-sla.md) | Confirm Odoo SLA, scaling, DR (DEC-008) | Sponsor + IT | M |
| [US-DISC-008](../../epics/EPIC-00-discovery/stories/US-DISC-008-sow.md) | Produce SOW + cost bracket | PM | M |
| [US-DISC-009](../../epics/EPIC-00-discovery/stories/US-DISC-009-signoff.md) | Sponsor sign-off + kickoff schedule | Sponsor | S |

## 7. Risks for this sprint

- RISK-001 Odoo decisions slip → S1 cannot start.
- RISK-002 SMS provider not approved in pilot region.
- RISK-003 Custom Odoo module timeline underestimated.

## 8. Sprint-level Definition of Done

- All DEC-001..DEC-009 closed in `traceability/decision-log.md`.
- SOW signed by sponsor.
- Kickoff for S1 scheduled.
- Communication plan live (`governance/communication-plan.md`).
- All discovery output committed under `traceability/` or sub-page.

## 9. Demo script

- Walk through closed decision log.
- Walk through SOW summary.
- Confirm S1 kickoff date.

## 10. Retro inputs

- Did stakeholders show up on time?
- Did any decision require more context than we anticipated?
- What did we leave unresolved that will leak into S1?
