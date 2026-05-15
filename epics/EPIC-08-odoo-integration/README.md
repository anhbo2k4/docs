---
epic: EPIC-08
title: Odoo Integration
sprint: S9
priority: P0
status: Ready
owner: Backend Lead + Odoo Specialist
---

# EPIC-08 — Odoo Integration

## 1. Goal

Persist every confirmed envelope into Odoo through Celery workers via JSON-RPC. The custom Odoo module `field_mobile_sync` carries the `external_id = client_id` lookup and the per-type write methods. Mobile never sees Odoo (ADR-005).

## 2. In scope

- Custom Odoo module `field_mobile_sync` with:
  - `field.mobile.envelope` model storing `client_id` external_id.
  - Per-type write methods: PPE check, form response, GPS event, work result, voice note, media reference.
  - Idempotent re-write that returns the existing record on duplicate.
- Celery tasks per envelope type with retries, jitter, and DLQ on permanent failure.
- Odoo connection pool + circuit breaker on FastAPI side.
- ACK update path: Celery writes `confirmed_at` and `odoo_ref` back to Postgres.
- Status visibility: failures surfaced via `/v1/sync/status/{client_id}` (DEAD_LETTER includes error_code).
- Re-process tooling for ops: replay one or many DLQ entries.

## 3. Out of scope

- Schema changes inside core Odoo (only the custom module).
- Reading shifts back into mobile — that path lives in EPIC-07 `/v1/shifts`.

## 4. Functional requirements covered

FR-014 (idempotent backend ingest), and the Odoo-side completion of FR-006..FR-011.

## 5. Stories

| ID | Title | Pri | Estimate | Status | Owner |
|---|---|---|---|---|---|
| US-ODOO-001 | Custom module `field_mobile_sync` skeleton | P0 | M | Ready | Odoo Specialist |
| US-ODOO-002 | PPE + Form write methods (idempotent) | P0 | M | Ready | Odoo Specialist |
| US-ODOO-003 | GPS event write method (model per DEC-004) | P0 | M | Ready | Odoo Specialist |
| US-ODOO-004 | Work result + media reference write | P0 | L | Ready | Odoo Specialist |
| US-ODOO-005 | Voice note + transcript write | P0 | M | Ready | Odoo Specialist |
| US-ODOO-006 | Celery integration: retry, DLQ, ACK | P0 | L | Ready | Backend Lead |

## 6. Dependencies

- DEC-001, DEC-002, DEC-003, DEC-004 closed.
- EPIC-07 staging deployment reachable from Odoo network.
- Read-only access to `hr.employee`, `project.task` / `planning.slot`.

## 7. Risks

- RISK-080 Odoo upgrade window during build invalidates module APIs.
- RISK-081 Duplicate writes on retry storm if external_id index is missing.
- RISK-082 Long Odoo write p95 violates pilot SLA (NFR-101).
- RISK-083 Network policy blocks FastAPI → Odoo on internal segment.

## 8. Definition of Done (epic-level)

- TC-ODOO-001..006, TC-ODOO-009, TC-ODOO-024 green.
- Replay tool tested end-to-end on a simulated DLQ batch.
- Odoo module deployed to staging via reproducible install script.
- p95 Odoo write ≤ 2 s under nominal load.

## 9. Open questions

DEC-001..DEC-004 all required closed before sprint S9 starts.
