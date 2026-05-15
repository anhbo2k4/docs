---
sprint: S9
title: Odoo Write Path
duration: 2 weeks
priority: P0
status: Ready
owner: Backend Lead + Odoo Specialist
---

# Sprint 9 — Odoo Write Path

## 1. Sprint goal

Persist every confirmed envelope into Odoo via Celery. The custom module `field_mobile_sync` carries `external_id = client_id`. Replay tooling exists for ops.

## 2. Theme

Closing the loop. After this sprint, work captured on a mobile device flows all the way into Odoo as the system of record.

## 3. Scope (in)

- Custom Odoo module `field_mobile_sync` (skeleton, models, security).
- Per-type write methods: PPE, form, GPS, work result, voice note, media reference.
- Celery integration: retries, DLQ, ACK back to Postgres.
- Replay tool for DLQ entries.
- Metrics + alerts.

## 4. Out of scope

- Performance hardening at scale (S10).
- Pilot launch (S10).

## 5. Pre-conditions

- DEC-001, DEC-002, DEC-003, DEC-004 closed.
- S8 deployed to staging.
- Read-only access to `hr.employee`, shift entity.
- Odoo install pipeline reachable from CI.

## 6. Stories committed

| ID | Title | Owner | Estimate |
|---|---|---|---|
| US-ODOO-001 | Module skeleton | Odoo Specialist | M |
| US-ODOO-002 | PPE + Form writes | Odoo Specialist | M |
| US-ODOO-003 | GPS write | Odoo Specialist | M |
| US-ODOO-004 | Work result + media ref write | Odoo Specialist | L |
| US-ODOO-005 | Voice note + transcript write | Odoo Specialist | M |
| US-ODOO-006 | Celery integration (retry, DLQ, ACK) | Backend Lead | L |

## 7. Risks for this sprint

- RISK-080 Odoo upgrade window invalidates module APIs.
- RISK-081 Duplicate writes if external_id index missing.
- RISK-082 Long Odoo write p95 violates pilot SLA.
- RISK-083 Network policy blocks FastAPI → Odoo segment.

## 8. Sprint-level Definition of Done

- TC-ODOO-001..006, TC-ODOO-009, TC-ODOO-024, TC-SYNC-006..010 green.
- p95 Odoo write ≤ 2 s under nominal load.
- Replay tool processes a synthetic DLQ batch.
- Module deploys to staging via reproducible install script.

## 9. Demo script

- Submit a full shift end-to-end on a real device pointing at staging.
- Open Odoo and walk through every record (PPE, form, GPS, work result with photos and voice).
- Force a 422 in payload, watch DLQ, run replay tool, see Odoo update.

## 10. Retro inputs

- Were Odoo internals stable enough?
- Was the replay tool ergonomic?
- Did we surface DLQ growth fast enough?
