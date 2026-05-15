# Observability

**Status:** Active
**Owner:** DevOps + Backend Lead + Mobile Lead

## Pillars

- **Logs** for individual events (structured, JSON).
- **Metrics** for aggregates (counters, histograms, gauges).
- **Traces** for causality across services.
- **Crash reports** for unhandled errors (Sentry).

## Stack

- Backend logs: stdout JSON → Loki (or vendor equivalent).
- Backend metrics: Prometheus exposition + Grafana dashboards.
- Backend traces: OpenTelemetry → Tempo (or vendor).
- Mobile crashes + breadcrumbs: Sentry (`SENTRY_DSN_MOBILE`).
- Mobile logs (debug builds only) → app_logger console; release builds emit only to Sentry breadcrumbs (PII scrubbed).

## Naming conventions

- Metrics: `<service>_<noun>_<unit>`. Examples:
  - `sync_envelope_total{status}` (counter)
  - `sync_envelope_latency_seconds{phase}` (histogram)
  - `odoo_write_total{status,model}` (counter)
- Trace span names: `<service>.<operation>`. Examples:
  - `fastapi.sync.envelope.post`
  - `celery.odoo.write_ppe_check`
- Sentry transaction names mirror screen names mobile-side.

## Cardinality discipline

- High-cardinality labels (`employee_id`, `client_id`) are never on metrics; reserved for traces and structured logs.
- Allowed metric labels: `status`, `type`, `phase`, `error_code`, `platform`, `app_version_minor`.
- Adding a new label requires CODEOWNER review.

## Required signals (MVP)

### Mobile

| Signal | Type | Where |
|---|---|---|
| App start time | Sentry transaction | `application/bootstrap` |
| Capture submitted | breadcrumb + metric event to backend | `screen/field_work` |
| Sync queue size | gauge logged on Sync Center open | `screen/sync_center` |
| Sync attempt result | breadcrumb | `@core/sync` |
| Crash | event | global handler |

### Backend

| Signal | Type |
|---|---|
| Request rate, latency, error rate per route | RED metrics |
| Sync envelope status transitions | counter `sync_envelope_total{status}` |
| Celery task duration + outcome | histogram |
| Odoo write outcome | counter + histogram |
| Auth attempts (success/fail/lock) | counter |
| Token refresh outcome | counter |
| Dead-letter creation rate | counter; alert threshold > 5/min |

### SLO targets (initial)

| SLO | Target |
|---|---|
| `/v1/sync/envelope` availability | 99.5 % monthly |
| `/v1/sync/envelope` p95 latency | ≤ 200 ms |
| Mobile crash-free sessions | ≥ 99.5 % |
| Sync success rate (CONFIRMED / submitted) | ≥ 99.0 % within 1 h |

Error budget violations trigger sprint freeze on non-reliability work.

## Dashboards

- `Pilot Health` — single-pane for sponsor: shifts active, sync success, crashes, OTP success.
- `Backend Service Health` — RED + Celery + Odoo timings.
- `Mobile Release Health` — crash-free sessions, ANR rate, slow frames.
- `Security Watch` — auth failures, refresh reuse, rate-limit hits, dead-letters.

Dashboards live in `infra/observability/dashboards/` as JSON; reviewed at sprint review.

## Alerting

- Page on: prod 5xx > 1 % for 5 min; sync DEAD_LETTER > 50/h; OTP success < 90 % for 15 min; crash-free < 99 % rolling 1 h.
- Ticket-only on: cache hit < threshold; latency p95 within 1.2× SLO; lower-severity anomalies.
- Every alert has a runbook link. Pages without runbooks are P0 to fix.

## Logging rules

- One log per externally-observable event; do not stutter.
- Structured fields: `event`, `service`, `route`, `client_id`, `employee_id_hash`, `status`, `error_code`, `latency_ms`.
- PII scrubber runs as the last step before emission.
- Sampling: 100 % of errors and warnings; INFO at 10 % in prod (configurable per route).

## Trace context propagation

- Mobile sends `traceparent` on every backend call.
- Backend propagates to Celery via task headers.
- Celery propagates into Odoo client via custom header logged by Odoo middleware.
