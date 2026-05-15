# ADR-016: GPS history in FastAPI Postgres; Odoo holds summary `x_ngynapp_gps_*` only

**Status:** Accepted
**Date:** 2026-05-15
**Deciders:** Backend Lead, Engineering Lead, Odoo Specialist
**Consulted:** Sponsor, Mobile Lead, Security Champion, DevOps
**Informed:** PO, QA Lead
**Supersedes:** [ADR-013](ADR-013-custom-gps-model.md)

## Context

ADR-013 placed GPS history in a custom Odoo model `field.gps.ping`. ADR-015 forbids custom Odoo modules (Odoo Online constraint). GPS history therefore needs a different home that:

- Supports the same use cases: check-in / check-out coordinates per shift, optional 5-minute interval sampling, supervisor map overlay, dispute resolution trail.
- Lets us enforce retention (180 days) and privacy controls (`security/data-classification.md` L3).
- Stays portable across Odoo versions and editions.

## Decision

GPS history lives in the **FastAPI Postgres database** as a first-class table `gps_pings`. Odoo carries only **denormalised summary fields** on `project.task` (or `fsm.task` if Field Service is installed):

```
x_ngynapp_gps_check_in_lat      Float
x_ngynapp_gps_check_in_lon      Float
x_ngynapp_gps_check_in_at       Datetime
x_ngynapp_gps_check_out_lat     Float
x_ngynapp_gps_check_out_lon     Float
x_ngynapp_gps_check_out_at      Datetime
x_ngynapp_gps_last_lat          Float   # latest known position
x_ngynapp_gps_last_lon          Float
x_ngynapp_gps_last_at           Datetime
x_ngynapp_gps_ping_count        Integer
```

Map overlays and audit views are served from FastAPI endpoints (`GET /v1/shifts/{id}/gps-trail`) consumed by an embedded iframe / link in the Odoo shift form (an Odoo URL field opening the FastAPI dashboard).

### Postgres schema

```sql
CREATE TABLE gps_pings (
    id              BIGSERIAL PRIMARY KEY,
    client_id       UUID NOT NULL,                 -- idempotency (ADR-007)
    employee_id     INTEGER NOT NULL,              -- mirror of hr.employee.id
    odoo_shift_id   INTEGER,                       -- mirror of project.task.id (nullable until shift sync)
    captured_at     TIMESTAMPTZ NOT NULL,
    received_at     TIMESTAMPTZ NOT NULL DEFAULT now(),
    latitude        DOUBLE PRECISION NOT NULL,
    longitude       DOUBLE PRECISION NOT NULL,
    accuracy_m      REAL,
    altitude_m      REAL,
    speed_mps       REAL,
    heading_deg     REAL,
    source          TEXT NOT NULL CHECK (source IN ('check_in','check_out','interval','manual')),
    correlation_id  UUID,                           -- sync envelope id
    UNIQUE (client_id)
);

CREATE INDEX idx_gps_shift_time ON gps_pings (odoo_shift_id, captured_at DESC);
CREATE INDEX idx_gps_employee_time ON gps_pings (employee_id, captured_at DESC);
CREATE INDEX idx_gps_received_at ON gps_pings (received_at) WHERE received_at > now() - INTERVAL '7 days';
```

Retention: nightly job deletes rows where `captured_at < now() - INTERVAL '180 days'`. Summary fields on Odoo are not purged.

### Write path

1. Mobile sync envelope arrives at FastAPI with one or more GPS pings.
2. FastAPI inserts each ping (idempotent on `client_id`).
3. For pings of source `check_in` / `check_out` / `interval` (latest), Celery worker updates the corresponding summary field set on Odoo via JSON-RPC (`web_save` on `project.task` / `fsm.task`).
4. The summary update is non-blocking — sync envelope is acked once Postgres write succeeds. Odoo summary mirror has its own retry queue.

### Read path

- **Mobile** does not read GPS history (capture-only).
- **Supervisor inside Odoo** sees the summary fields directly and a "Open GPS trail" URL that loads the FastAPI map view (signed JWT, 15-minute expiry).
- **Audit / dispute** queries hit FastAPI `/admin/gps` endpoints with admin role.

## Rationale

- **Odoo Online cannot host a custom model.** Postgres next to FastAPI is the natural authoritative home.
- **Performance.** GPS volume scales with shift count × ping frequency. At 100 workers × 8 pings/shift × 22 shifts/month ≈ 17,600 rows/month; 5-minute sampling could push that to ~600k/month. Better in dedicated Postgres than in shared Odoo DB.
- **Reporting.** PostGIS or simple lat/lon queries are easy in Postgres without burdening Odoo's ORM.
- **Privacy.** Tighter access control on a separate DB; Odoo users only see derived summary, reducing PII surface.
- **Portability.** Works the same on Odoo Online, Enterprise, Community.

## Alternatives considered

| Option | Pros | Cons | Why not chosen |
|---|---|---|---|
| Custom Odoo model `field.gps.ping` (ADR-013) | First-class in Odoo | Blocked by Odoo Online | Hard constraint violation |
| Studio `x_` model on Odoo | No custom Python | Studio models are weak (no compute, weak migrations); still Online-restricted in practice; performance unknown for high-volume inserts | Brittle, slow, not version-stable |
| Append-only file logs (S3) | Cheap | Hard to query; no joins; bad UX for ops | Loses dispute-resolution UX |
| External time-series DB (InfluxDB) | Purpose-built | New stack to operate; overkill for pilot scale | Not justified |
| FastAPI Postgres (chosen) | Same DB as sync state, easy joins, cheap, portable | Two stores of business data (Odoo + ours) | Acceptable; reconciled via sync state machine |

## Consequences

### Positive

- Works on Odoo Online out of the box.
- Sub-millisecond inserts at pilot scale; horizontal scale path via Postgres read replicas.
- GPS map UI iterates fast — owned by our FastAPI codebase, not blocked by Odoo release cadence.
- Privacy posture improves: Odoo PII surface shrinks to the summary.

### Negative

- Operators querying ad-hoc inside Odoo cannot see full trail; must use linked FastAPI dashboard.
- Two systems to backup; Postgres backup policy must include `gps_pings`.
- A fresh Odoo tenant onboarding still requires `x_ngynapp_gps_*` fields to exist (per `data/odoo-x-fields-spec.md`).

### Neutral

- Retention enforcement moves from Odoo cron (`cron_purge_gps_pings`) to a FastAPI-scheduled job (Celery beat).
- TC-GPS-001..004 are rewritten to assert Postgres rows + Odoo summary mirror equivalence.

## Compliance / verification

- Migration `alembic/versions/xxxx_create_gps_pings.py` creates the table and indexes.
- Retention job `tasks.gps_retention.purge_old_pings` runs daily 02:00 UTC.
- TC-GPS-001 verifies check-in ping inserts row + updates summary.
- TC-GPS-002 verifies check-out ping inserts row + updates summary.
- TC-GPS-003 verifies interval pings only update summary on the latest.
- TC-GPS-004 verifies summary equals latest ping for each shift after a replay storm.
- Privacy: GPS L3 per `security/data-classification.md`; access via `/admin/gps` requires `field_admin` role.

## Related

- FR / NFR: FR-005, FR-008, NFR-PRIV-02
- Stories: US-CAP-006, US-CAP-007, US-CAP-008, US-ODOO-003, US-API-004
- Other ADRs: ADR-004 (FastAPI), ADR-007 (idempotency), ADR-011 (Odoo as system of record for shifts), ADR-015 (no custom module)
- Decisions closed: DEC-004 (flipped — Postgres, not custom Odoo model)
