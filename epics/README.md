# Epics — Index

**Status:** Active
**Owner:** Engineering Lead + Product Owner

The 11 epics that compose the MVP (`EPIC-00` through `EPIC-09`) plus post-MVP scope (`EPIC-10`). Each epic folder contains a `README.md` (epic charter) and a `stories/` subfolder with user stories.

| Epic | Title | Sprint(s) | Priority | Folder |
|---|---|---|---|---|
| EPIC-00 | Discovery & Build Authorization | S0 | P0 | [EPIC-00-discovery/](./EPIC-00-discovery/) |
| EPIC-01 | Platform Foundation | S1 | P0 | [EPIC-01-platform-foundation/](./EPIC-01-platform-foundation/) |
| EPIC-02 | Authentication & Employee Mapping | S2 | P0 | [EPIC-02-auth-employee-mapping/](./EPIC-02-auth-employee-mapping/) |
| EPIC-03 | Shift Dashboard & Detail | S3 | P0 | [EPIC-03-shift-dashboard-detail/](./EPIC-03-shift-dashboard-detail/) |
| EPIC-04 | Offline Local Persistence | S4 | P0 | [EPIC-04-offline-persistence/](./EPIC-04-offline-persistence/) |
| EPIC-05 | Field Capture Workflow | S5 | P0 | [EPIC-05-field-capture/](./EPIC-05-field-capture/) |
| EPIC-06 | Sync Engine & Background Work | S6, S7 | P0 | [EPIC-06-sync-engine/](./EPIC-06-sync-engine/) |
| EPIC-07 | FastAPI Integration Backend | S8 | P0 | [EPIC-07-fastapi-backend/](./EPIC-07-fastapi-backend/) |
| EPIC-08 | Odoo Integration | S9 | P0 | [EPIC-08-odoo-integration/](./EPIC-08-odoo-integration/) |
| EPIC-09 | QA, Security & Pilot Readiness | S10 | P0 | [EPIC-09-qa-pilot/](./EPIC-09-qa-pilot/) |
| EPIC-10 | Post-MVP Enhancements | post-MVP | P2 | [EPIC-10-post-mvp/](./EPIC-10-post-mvp/) |

## Story ID prefixes

| Prefix | Used in epics |
|---|---|
| `US-DISC-NNN` | EPIC-00 |
| `US-PLAT-NNN` | EPIC-01 |
| `US-AUTH-NNN` | EPIC-02 |
| `US-SHIFT-NNN` | EPIC-03 |
| `US-OFF-NNN` | EPIC-04 |
| `US-CAP-NNN`, `US-VOICE-NNN` | EPIC-05, EPIC-06 (capture portion) |
| `US-SYNC-NNN` | EPIC-06 |
| `US-API-NNN` | EPIC-07 |
| `US-ODOO-NNN` | EPIC-08 |
| `US-QA-NNN` | EPIC-09 |
| `US-POST-NNN` | EPIC-10 |

## How an epic is structured

Each `EPIC-XX/README.md` contains:

1. **Goal** — one-line outcome.
2. **In scope / Out of scope** — explicit boundaries.
3. **Functional requirements covered** — FR-NNN list.
4. **Stories** — table with status, priority, estimate, owner.
5. **Dependencies** — other epics, decisions (DEC-NNN), ADRs.
6. **Risks** — RISK-NNN list (links to `backlog/risk-register.md`).
7. **Definition of Done (epic-level)** — exit criteria.
8. **Open questions** — DEC-NNN that block stories in this epic.

Each story file under `stories/` follows the template documented in `CONTRIBUTING.md` §5.
