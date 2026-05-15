---
sprint: S8
title: Backend Sync Intake
duration: 2 weeks
priority: P0
status: Ready
owner: Backend Lead
---

# Sprint 8 — Backend Sync Intake

## 1. Sprint goal

FastAPI accepts envelopes from mobile, deduplicates by `client_id`, persists, and enqueues Celery jobs. Mobile and backend exchange real envelopes end-to-end (without Odoo yet).

## 2. Theme

Server-side reliability and contract enforcement. Idempotency by `client_id` is the single most important property; everything else is built on top.

## 3. Scope (in)

- FastAPI scaffold, structured logs, OTel, Sentry, Prom.
- `/v1/auth/exchange`, refresh, logout.
- `/v1/shifts` list + detail (Odoo read passthrough).
- `/v1/sync/envelope` ingest + idempotency + Celery enqueue (Celery destination is a stub task in S8 — Odoo writes arrive in S9).
- `/v1/media/upload-url` + finalize + EXIF check.
- Postgres schema + migrations.
- Per-tenant rate limit.

## 4. Out of scope

- Odoo writes (S9).
- Pilot deployment (S10).

## 5. Pre-conditions

- DEC-001, DEC-002, DEC-007, DEC-009 closed.
- S7 staging mock retired in favour of real FastAPI.
- Postgres + Redis provisioned.

## 6. Stories committed

| ID | Title | Owner | Estimate |
|---|---|---|---|
| US-API-001 | FastAPI scaffold + observability | Backend Lead | M |
| US-API-002 | `/v1/auth/exchange` + refresh + revoke | Backend | M |
| US-API-003 | `/v1/shifts` list + detail | Backend | M |
| US-API-004 | `/v1/sync/envelope` ingest + idempotency | Backend Lead | L |
| US-API-005 | `/v1/media/upload-url` + finalize | Backend | M |

## 7. Risks for this sprint

- RISK-070 Idempotency middleware misses race window.
- RISK-071 Signed URL provider differs in CORS behaviour.
- RISK-072 Cache poisoning via long shift TTL.
- RISK-073 Celery DLQ silently grows.

## 8. Sprint-level Definition of Done

- p95 server time on `/v1/sync/envelope` ≤ 200 ms (TC-PERF-008).
- TC-SYNC-001..010 green against staging.
- OpenAPI matches `api-contracts/` markdown (lint check).
- Postgres migrations versioned; one rehearsed rollback in staging.
- Mobile points at staging without code changes (env switch).

## 9. Demo script

- Mobile against real FastAPI: capture, sync, refresh.
- Show idempotency: 50 concurrent same-`client_id` POSTs → 1 row.
- Force a payload mismatch → 409.

## 10. Retro inputs

- Did the idempotency contract hold under storm?
- Were signed URLs reliable?
- Were the migrations safe?
