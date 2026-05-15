# Sprints — Index

**Status:** Active
**Owner:** Scrum Master + Engineering Lead

11 two-week sprints (`S0` through `S10`) covering MVP delivery. Each sprint folder contains a `README.md` written in **Sprint Planning Prompt** format: goal, scope, dependencies, exit criteria, and the stories committed.

| # | Sprint | Theme | Stories committed | Folder |
|---|---|---|---|---|
| S0 | Discovery & Build Authorization | Close 9 Phase 0 decisions, sign build SOW | US-DISC-001..009 | [sprint-00-discovery/](./sprint-00-discovery/) |
| S1 | Project Foundation | Flutter scaffold, CI, secure storage, SQLite skeleton | US-PLAT-001..006 | [sprint-01-foundation/](./sprint-01-foundation/) |
| S2 | Auth & Employee Context | OTP login, internal JWT exchange, employee mapping | US-AUTH-001..007 | [sprint-02-auth-context/](./sprint-02-auth-context/) |
| S3 | Shifts Read Workflow | My Shifts dashboard, Shift Detail, sync state badges | US-SHIFT-001..005 | [sprint-03-shifts-read/](./sprint-03-shifts-read/) |
| S4 | SQLite Offline Foundation | DAOs, sync_queue, sync state machine | US-OFF-001..006 | [sprint-04-sqlite-foundation/](./sprint-04-sqlite-foundation/) |
| S5 | Core Field Capture | PPE, pre-shift form, GPS check-in/out | US-CAP-001..008 | [sprint-05-core-capture/](./sprint-05-core-capture/) |
| S6 | Evidence Capture | Photos with compression, voice notes + Whisper | US-CAP-009..013, US-VOICE-001..004 | [sprint-06-evidence-capture/](./sprint-06-evidence-capture/) |
| S7 | Mobile Sync Engine | WorkManager / BGTaskScheduler, retry, dead-letter, Sync Center | US-SYNC-001..006 | [sprint-07-mobile-sync/](./sprint-07-mobile-sync/) |
| S8 | Backend Sync Intake | FastAPI ingest, Celery enqueue, idempotency, status API | US-API-001..005 | [sprint-08-backend-intake/](./sprint-08-backend-intake/) |
| S9 | Odoo Write Path | Odoo JSON-RPC, custom module `field_mobile_sync`, retries | US-ODOO-001..006 | [sprint-09-odoo-write-path/](./sprint-09-odoo-write-path/) |
| S10 | Hardening & Pilot | Perf, security audit, device matrix, pilot launch | US-QA-001..006 | [sprint-10-hardening-pilot/](./sprint-10-hardening-pilot/) |

## Sprint Planning Prompt format

Each sprint README answers, in order:

1. **Sprint goal** — single sentence.
2. **Theme** — one paragraph.
3. **Scope (in)** — bullet list of what ships.
4. **Out of scope** — what is intentionally deferred.
5. **Pre-conditions** — what must be true to start (closed DECs, prior stories, environments).
6. **Stories committed** — table referencing `epics/.../stories/`.
7. **Risks for this sprint** — references `backlog/risk-register.md`.
8. **Sprint-level Definition of Done** — exit criteria.
9. **Demo script** — what is shown at sprint review.
10. **Retro inputs** — recurring questions for the team.

## Capacity assumptions

- Team: 4–6 (1 Mobile Lead, 1–2 Mobile, 1 Backend, 1 QA, 1 PM/PO; +Designer part-time).
- Sprint length: 2 weeks (10 working days).
- Capacity buffer: 20% reserved for spikes, defects, code review.
- Velocity baseline: estimated at S2 close, recalibrated each sprint.

## Sprint-to-Epic mapping at a glance

```
S0  → EPIC-00
S1  → EPIC-01
S2  → EPIC-02
S3  → EPIC-03
S4  → EPIC-04
S5  → EPIC-05 (core)
S6  → EPIC-05 (evidence) + EPIC-06 (voice)
S7  → EPIC-06 (mobile sync engine)
S8  → EPIC-07
S9  → EPIC-08
S10 → EPIC-09
```

EPIC-10 is post-MVP and not bound to any sprint here.
