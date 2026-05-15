# ADR-004: FastAPI as the integration layer between mobile and Odoo

**Status:** Accepted  
**Date:** 2026-05-13  
**Deciders:** Engineering Lead, Backend Lead  
**Consulted:** Mobile Lead, Security Champion, Odoo Specialist  
**Informed:** All

## Context

Mobile cannot call Odoo directly (see ADR-005). We need a backend that brokers requests, handles slow Odoo writes asynchronously, validates sync envelopes, deduplicates by `client_id`, retries on failure, issues short-lived internal JWTs after Supabase OTP, and provides metrics, tracing, and a clean test surface. We also need a queue + worker model with retry and dead-letter semantics.

## Decision

Use **FastAPI 0.110+** (Python 3.11, Pydantic v2) for the synchronous HTTP layer plus **Celery 5.4+** with **Redis 7** as the broker for asynchronous tasks (Odoo writes, media finalisation, retries). State persists in **PostgreSQL 15** (sessions, sync_envelopes, audit log, shift cache).

## Rationale

- Pydantic v2 strict mode gives strong runtime validation aligned with the typed contract we want with mobile.
- FastAPI's async stack handles concurrent IO well.
- Celery is the de-facto standard for retried task queues in Python with mature operational tooling.
- Postgres for FastAPI's own state keeps Odoo as the system of record without duplication.
- Team has Python expertise.

## Alternatives considered

| Option | Pros | Cons | Why not chosen |
|---|---|---|---|
| Node + Express | Same lang as Edge functions | Weaker validation story than Pydantic | Higher bug risk in sync layer |
| Go (Gin / Fiber) | Fast | Smaller team, fewer Odoo libs | Team velocity penalty |
| Django REST | Batteries included | Heavier, worse async support | Overhead for our scope |
| Direct Lambda + step functions | Minimal infra | Operational complexity for sync flows | Harder to evolve |

## Consequences

### Positive
- Strong validation in both directions.
- Clear separation of fast HTTP path and slow Odoo path.
- Mature observability stack.

### Negative
- Two runtimes (web + worker) to deploy and monitor.
- Celery's operational learning curve.
- Pydantic v2 has migration cost from v1 for engineers with v1 muscle memory.

### Neutral
- Must commit to one validation library project-wide; no fallback to manual JSON parsing in services.

## Compliance / verification

- Backend code lives in `backend/` with the layout in `architecture/c4-component-backend.md`.
- Every sync envelope passes through `services/sync_service.py` and is enqueued via Celery; no direct Odoo calls in HTTP handlers.
- Health and readiness endpoints required: `/healthz`, `/readyz`.

## Related

- FR-007, FR-008 (shift read), FR-012 to FR-015 (sync)
- ADR-005, ADR-007
- Stories: `epics/EPIC-07-fastapi-backend/`, `epics/EPIC-08-odoo-integration/`
