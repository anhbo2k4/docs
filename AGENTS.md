# PROJECT AGENTS — Field Work Delivery

**Status:** Active
**Owner:** Engineering Lead
**Last updated:** 2026-05-13

This file tells AI agents and new engineers how this project is organised and what to read before doing anything.

## Project at a glance

- **Product:** Unified Flutter mobile app for field workers + contractors. Offline-first. Syncs to Odoo through a FastAPI integration layer.
- **MVP scope:** EPIC-00 through EPIC-09 (S0 → S10, ~22 weeks, 4–6 people).
- **Single source of truth:** this directory (`field_work_delivery/`).
- **Mobile language:** Dart (Flutter). **Backend:** Python (FastAPI + Celery). **ERP:** Odoo (Python). **Edge:** TypeScript (Deno).
- **Doc language:** English. Vietnamese allowed in `traceability/decision-log.md` comments only.

## First action when joining a task

1. Read `WORKING.md` for current focus and last decisions.
2. Read `README.md` for navigation.
3. Read `traceability/glossary.md` so terms align.
4. Locate your story under `epics/EPIC-XX/stories/US-...md`.
5. Follow the **routing rules** below.
6. Check `traceability/decision-log.md` for blockers tagged on the story.
7. Update story status front-matter when starting and when finishing.

## Tier classifier (per agents-system convention)

| Task | Tier |
|---|---|
| Typo, link fix, single-file copy edit | T0 |
| Single story implementation, single file or small change | T1 |
| Multi-file feature within one epic | T2 |
| Cross-epic change, schema migration, API contract change, security area | T3 |

For T2/T3 always run pre-mortem (`agents-system/protocols/pre-mortem.md` if present) and write an ADR if the change is architectural.

## Routing rules — what to read before coding

| Task area | Read first | Read also |
|---|---|---|
| Auth (mobile) | `epics/EPIC-02-auth-employee-mapping/` + `security/auth-flow.md` | `api-contracts/auth.md` |
| Auth (backend) | `epics/EPIC-07-fastapi-backend/stories/US-API-002-*` | `security/auth-flow.md`, `api-contracts/auth.md`, `data/postgres-schema.md` |
| Shifts list / detail | `epics/EPIC-03-shift-dashboard-detail/` | `api-contracts/shifts.md`, `data/sqlite-schema.md` (`shift` table) |
| SQLite + persistence | `epics/EPIC-04-offline-persistence/` | `data/sqlite-schema.md`, `data/erd.md`, `data/sync-state-machine.md` |
| Field capture (PPE / form / GPS / photos) | `epics/EPIC-05-field-capture/` | `data/sqlite-schema.md`, relevant `qa/test-cases.md` rows |
| Voice + Whisper | `epics/EPIC-06-sync-engine/stories/US-VOICE-*` | `adr/ADR-008-on-device-whisper.md` |
| Sync engine (mobile) | `epics/EPIC-06-sync-engine/` | `adr/ADR-007-idempotency-by-client-id.md`, `data/sync-state-machine.md` |
| Sync ingest (backend) | `epics/EPIC-07-fastapi-backend/stories/US-API-004-*` | `api-contracts/sync.md`, `data/postgres-schema.md` |
| Odoo write | `epics/EPIC-08-odoo-integration/` | `data/odoo-mapping.md`, ADR-005 |
| QA / pilot | `epics/EPIC-09-qa-pilot/` | `qa/pilot-gate.md`, `qa/test-cases.md` |
| Any code change | `engineering/coding-standards.md`, `engineering/git-workflow.md`, `engineering/code-review-checklist.md` | `governance/definition-of-ready.md`, `governance/definition-of-done.md` |
| Observability change | `engineering/observability.md` | metric naming + SLOs |
| Incident / on-call | `engineering/oncall.md`, `runbooks/incident-response.md` | `runbooks/_postmortem-template.md` |
| Security finding | `SECURITY.md`, `security/threat-model.md` | `security/pii-policy.md` |

## Hard rules

1. **Mobile never calls Odoo directly.** Always FastAPI in between (ADR-005). If you find yourself wanting to bypass, stop and read ADR-005.
2. **Idempotency by `client_id`.** Every sync envelope MUST carry a UUID v4 generated at capture time. Reusing one with a different payload is a programming error (409). (ADR-007.)
3. **Local SQLite is the first durable write.** Never block UX on a network call for capture. (ADR-003.)
4. **No PII in logs or Sentry.** Phone, lat, lng, OTP, tokens are never logged. Verified by tests.
5. **Static schemas in MVP.** Dynamic forms are EPIC-10. Don't sneak them in.
6. **Test the sync, auth, and Odoo write paths first.** These three areas are where bugs cost the most.
7. **Don't modify Accepted ADRs. Supersede them with a new one.**

## How story IDs map to code

| Story prefix | Mobile code area | Backend code area |
|---|---|---|
| `US-PLAT-` | `mobile/lib/@core`, `application/`, build config | infrastructure |
| `US-AUTH-` | `mobile/lib/screen/auth`, `@core/auth` | `backend/app/auth`, Edge functions |
| `US-SHIFT-` | `mobile/lib/screen/shifts` | `backend/app/shifts` |
| `US-OFF-` | `mobile/lib/@core/db` | — |
| `US-CAP-` | `mobile/lib/screen/field_work` | `backend/app/sync` (intake) |
| `US-VOICE-` | `mobile/lib/screen/field_work/voice`, `plugins/audio` | `backend/app/media` |
| `US-SYNC-` | `mobile/lib/@core/sync` | `backend/app/sync`, Celery |
| `US-API-` | — | `backend/app/*` |
| `US-ODOO-` | — | `backend/app/odoo`, `infra/odoo/field_mobile_sync` |
| `US-QA-` | tests + dashboards | tests + dashboards |

## When in doubt

Don't guess. Add an entry under `traceability/open-questions.md` and ping the role above the affected area in `governance/raci.md`.
