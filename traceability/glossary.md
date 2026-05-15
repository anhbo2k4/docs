# Glossary

**Status:** Active  
**Owner:** Engineering Lead

Authoritative definitions for terms used across this blueprint. When a term in code, comments, or docs differs from this list, this list wins.

## Domain terms

| Term | Definition |
|---|---|
| **Field worker** | A person assigned to a shift who performs PPE checks, forms, captures, and submissions in the app. |
| **Contractor** | A field worker employed by a contracting partner rather than the primary org. Same UX in this app. |
| **Supervisor** | Field manager who oversees one or more field workers. Reads submissions in Odoo, not in the mobile app (MVP). |
| **Shift** | A time-bound assignment of work for a worker. In Odoo, a `project.task` or `planning.slot` (TBD per DEC-002). |
| **PPE check** | Personal Protective Equipment checklist completed before a shift starts. |
| **Pre-shift form** | Structured form the worker fills before starting work (safety, conditions, declarations). |
| **Post-shift form** | Structured form the worker fills after completing work (issues, sign-off). |
| **Work result** | The body of evidence (photos, notes, GPS, voice notes) the worker captures during a shift. |
| **Voice note** | An audio recording with an on-device Whisper transcript attached. |
| **Evidence** | Any captured artefact: photo, audio, GPS event, form response, signature. |
| **Capture** | Verb / noun for any field worker action that produces evidence. |

## Sync terms

| Term | Definition |
|---|---|
| **Sync queue** | The local SQLite table that holds pending envelopes. |
| **Envelope** | A single sync unit submitted to FastAPI. Contains `client_id`, type, payload, references. |
| **client_id** | A UUID v4 generated at the moment of capture on device. The idempotency key. (See ADR-007.) |
| **Sync state** | The state of a row's synchronisation: `PENDING`, `SYNCING`, `CONFIRMED`, `FAILED`, `DEAD_LETTER`. |
| **PENDING** | Captured locally, not yet sent. |
| **SYNCING** | Sent to FastAPI, awaiting confirmation. |
| **CONFIRMED** | FastAPI accepted and Odoo wrote successfully. |
| **FAILED** | Last attempt failed; will retry per backoff. |
| **DEAD_LETTER** | Exceeded retry budget or rejected with 4xx; needs human intervention. |
| **Idempotency-Key** | HTTP header carrying `client_id`. Server uses it to deduplicate. |
| **Dead-letter queue (DLQ)** | Backend table holding envelopes that failed permanently and need ops review. |
| **Drain** | Worker mode that processes existing queue without accepting new envelopes (used during rollback). |

## Auth terms

| Term | Definition |
|---|---|
| **OTP** | One-Time Password (6-digit SMS code via Twilio). |
| **Supabase JWT** | Token issued by Supabase Edge after OTP verification. Short-lived (5 min). |
| **Internal JWT** | Token issued by FastAPI after exchanging Supabase JWT. Carries `employee_id`. Access 15 min, refresh 30 days. |
| **Employee mapping** | Process that links a phone number to an `hr.employee` in Odoo. |
| **Device ID** | UUID generated at first launch, stored in secure storage. Used for audit and threat detection. |

## Architecture terms

| Term | Definition |
|---|---|
| **Mobile** | The Flutter app. iOS or Android. |
| **Backend** | FastAPI service plus Celery workers plus Postgres plus Redis. |
| **Edge** | Supabase Edge Function (Deno + TypeScript). |
| **ERP** | Odoo. The system of record. |
| **System of record (SoR)** | The store whose data is authoritative. Odoo for HR + projects + tasks. |
| **Cache** | A copy of data whose authoritative source is elsewhere. Postgres caches Odoo reads. |
| **DAO** | Data Access Object. Mobile DAOs wrap SQLite tables. |
| **Repository** | Coordinator across DAOs, sync queue, and remote calls. |
| **Use case** | A pure-Dart business operation under `domain/use_cases/`. |

## Process terms

| Term | Definition |
|---|---|
| **Epic** | A grouping of stories that delivers a coherent capability. 11 in this project. |
| **Story** | A user-facing increment with AC and tasks. |
| **Sprint** | A 2-week iteration. |
| **DoR** | Definition of Ready — story fit to enter a sprint. |
| **DoD** | Definition of Done — increment is production-ready. |
| **Pilot** | Limited rollout to a small group of real users for validation. |
| **Phase 0** | Discovery sprint (S0). Closes 9 open decisions. |
| **MVP** | Minimum Viable Product. EPIC-00 through EPIC-09. |

## Quality terms

| Term | Definition |
|---|---|
| **AC** | Acceptance Criterion. Numbered, testable. |
| **TC** | Test Case. ID format `TC-<AREA>-NNN`. |
| **FR** | Functional Requirement. ID format `FR-NNN`. |
| **NFR** | Non-Functional Requirement. ID format `NFR-NNN`. |
| **DEC** | Decision (open or closed). ID format `DEC-NNN`. |
| **Risk** | Tracked uncertainty. ID format `RISK-NNN`. |
| **Pilot gate** | Set of criteria a release must pass before pilot launch. See `qa/pilot-gate.md`. |

## Severity / priority

| Code | Meaning |
|---|---|
| **P0** | Blocker for MVP. |
| **P1** | MVP target. |
| **P2** | Post-MVP. |
| **SEV-1** | Pilot or production unusable. |
| **SEV-2** | Major feature broken. |
| **SEV-3** | Degraded with workaround. |
| **SEV-4** | Cosmetic. |
