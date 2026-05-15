---
id: US-API-001
epic: EPIC-07
sprint: S8
fr: []
priority: P0
estimate: M
status: Ready
owner: Backend Lead
---

# US-API-001 — FastAPI scaffold + config + observability

## Acceptance criteria

1. FastAPI service runs locally via Docker Compose with Redis + Postgres.
2. Structured logs (JSON) with `request_id`, `employee_id` (when authenticated), `route`, `status`, `latency_ms`.
3. OpenTelemetry traces exported to staging collector; spans on every route handler and DB call.
4. Sentry integrated for unhandled exceptions; PII scrubber configured.
5. Prometheus metrics: request count, latency histogram, queue depth.
6. `/healthz` and `/readyz` endpoints; `readyz` checks Postgres + Redis + signing key availability.

## Tasks

- [ ] Project scaffold with `app/`, `routers/`, `schemas/`, `core/` layers.
- [ ] Settings via `pydantic-settings` reading from env / SSM.
- [ ] Structured logging filter.
- [ ] OTel + Sentry + Prom wiring.
- [ ] Compose file for local dev.

## Dependencies

DEC-009 (orchestration) for image base.

## FR mapping

Foundation.

## Test cases

Smoke + observability assertions in CI.

## DoD

- Local compose up clean.
- Staging deploy reachable from CI.
