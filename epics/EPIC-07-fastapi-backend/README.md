---
epic: EPIC-07
title: FastAPI Integration Backend
sprint: S8
priority: P0
status: Ready
owner: Backend Lead
---

# EPIC-07 — FastAPI Integration Backend

## 1. Goal

Stand up the integration layer that mobile talks to. FastAPI accepts envelopes, enforces auth, persists to Postgres, dedupes by `client_id`, and enqueues Celery jobs for the Odoo write path. Mobile must never call Odoo directly (ADR-005).

## 2. In scope

- FastAPI service with structured config, request IDs, and structured logs.
- Endpoints: `/v1/auth/exchange`, `/v1/shifts`, `/v1/shifts/{id}`, `/v1/sync/envelope`, `/v1/sync/status/{client_id}`, `/v1/media/upload-url`, `/v1/media/finalize`.
- JWT issuance and validation; refresh + revocation list.
- Idempotency middleware keyed on `(employee_id, client_id)`.
- Postgres schema for `envelopes`, `media_blobs`, `auth_sessions`, `dead_letter`, `audit_log`.
- Celery worker stub: enqueue + retry + dead-letter routing (real Odoo write happens in EPIC-08).
- Redis as broker + cache for OTP rate limiting.
- Object-storage signed URLs (per DEC-007 provider).
- OpenAPI spec generated and committed under `api-contracts/`.
- Observability: Sentry, OpenTelemetry traces, Prometheus metrics.

## 3. Out of scope

- Odoo writes (EPIC-08).
- Mobile UI (EPIC-02..06).
- Pilot deployment (EPIC-09 / S10).

## 4. Functional requirements covered

FR-002, FR-003, FR-004, FR-005, FR-014, FR-015 (server side).

## 5. Stories

| ID | Title | Pri | Estimate | Status | Owner |
|---|---|---|---|---|---|
| US-API-001 | FastAPI scaffold + config + observability | P0 | M | Ready | Backend Lead |
| US-API-002 | `/v1/auth/exchange` + JWT + refresh + revoke | P0 | M | Ready | Backend |
| US-API-003 | `/v1/shifts` list + detail (Odoo read passthrough or cache) | P0 | M | Ready | Backend |
| US-API-004 | `/v1/sync/envelope` ingest + idempotency + Celery enqueue | P0 | L | Ready | Backend Lead |
| US-API-005 | `/v1/media/upload-url` + finalize + EXIF check | P0 | M | Ready | Backend |

## 6. Dependencies

- DEC-001, DEC-002, DEC-007, DEC-009 closed.
- EPIC-02 mobile contract aligned for `/v1/auth/exchange`.

## 7. Risks

- RISK-070 Idempotency middleware misses race window → duplicate Odoo writes.
- RISK-071 Signed URL provider differs in CORS behaviour from spec.
- RISK-072 Cache poisoning via shift list TTL too long.
- RISK-073 Celery DLQ silently grows without alerting.

## 8. Definition of Done (epic-level)

- All TC-SYNC-001..010 and TC-AUTH backend cases pass against staging.
- p95 server time on `/v1/sync/envelope` ≤ 200 ms (TC-PERF-008).
- Postgres migrations versioned; rollback rehearsed.
- OpenAPI matches `api-contracts/` markdown (lint check in CI).

## 9. Open questions

DEC-007 (storage) and DEC-009 (orchestration) must be closed.
