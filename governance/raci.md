# RACI Matrix

**Status:** Active  
**Owner:** Project Manager  
**Last updated:** 2026-05-13

R = Responsible (does the work). A = Accountable (signs off, only one). C = Consulted (gives input). I = Informed (kept updated).

## Roles

| Code | Role | Notes |
|---|---|---|
| **SP** | Sponsor | Funds the project, approves scope changes |
| **PO** | Product Owner | Owns backlog, AC sign-off |
| **PM** | Project Manager | Schedule, risk, governance |
| **EL** | Engineering Lead | Architecture, technical decisions |
| **ML** | Mobile Lead | Flutter implementation |
| **BL** | Backend Lead | FastAPI + Celery + Odoo |
| **QA** | QA Lead | Test strategy, pilot gate |
| **SEC** | Security Champion | Threat model, secrets, audit |
| **OPS** | DevOps / SRE | CI/CD, infra, runbooks |
| **DES** | Designer | UX, accessibility |
| **ODOO** | Odoo Specialist (vendor or internal) | Custom module + mapping |
| **STAKE** | Pilot site stakeholder | Field manager / supervisor |

## Activity matrix

| Activity | SP | PO | PM | EL | ML | BL | QA | SEC | OPS | DES | ODOO | STAKE |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Approve scope | A | R | C | C | I | I | I | I | I | I | C | C |
| Approve budget | A | C | R | C | I | I | I | I | I | I | C | I |
| Approve timeline | A | C | R | C | I | I | I | I | I | I | I | I |
| Phase 0 close (DEC-001..009) | A | R | C | R | C | C | C | C | I | I | R | C |
| ADR creation | I | C | I | A | R | R | C | C | C | I | C | I |
| ADR sign-off | I | C | I | A | C | C | C | C | I | I | I | I |
| Sprint planning | I | A | R | C | R | R | C | I | I | C | I | I |
| Sprint review | C | A | R | C | R | R | R | I | I | C | I | C |
| Backlog grooming | I | A | R | C | C | C | C | I | I | I | I | I |
| Story acceptance | I | A | I | C | R | R | R | I | I | I | I | I |
| Architecture diagrams | I | I | I | A | C | C | C | C | C | I | C | I |
| API contract changes | I | C | I | A | C | R | C | C | I | I | C | I |
| SQLite schema changes | I | I | I | A | R | C | C | C | I | I | I | I |
| Odoo mapping | I | C | I | A | C | R | C | I | I | I | R | C |
| Threat model | I | I | I | A | C | C | C | R | C | I | C | I |
| Secrets handling | I | I | I | A | C | C | C | R | R | I | I | I |
| CI/CD pipelines | I | I | I | A | C | C | C | C | R | I | I | I |
| Test strategy | I | C | I | C | C | C | A | C | I | I | I | I |
| Test case authoring | I | I | I | I | C | C | A | I | I | I | I | I |
| Pilot gate sign-off | A | R | C | C | C | C | R | R | C | I | C | C |
| Production release | A | C | R | C | C | C | C | C | R | I | I | I |
| Rollback decision | A | C | R | C | C | C | C | C | R | I | I | C |
| Incident response | I | I | C | C | C | C | C | C | A | I | I | I |
| Risk register | I | C | A | R | C | C | C | C | C | I | I | I |
| Stakeholder comms | A | C | R | I | I | I | I | I | I | I | I | C |
| Pilot user training | I | A | R | I | C | I | C | I | I | C | I | R |
| Change requests | A | R | R | C | C | C | C | C | I | I | C | C |

## Conflict resolution

If two roles disagree on a technical matter, EL decides. If business vs technical conflict, PO and EL must reach consensus; if not, escalate to SP per `escalation.md`.
