# ADR-013: GPS pings stored in custom Odoo model `field.gps.ping`

**Status:** Superseded by [ADR-016](ADR-016-gps-in-fastapi-postgres.md) (2026-05-15)
**Date:** 2026-05-15
**Superseded:** 2026-05-15
**Deciders:** Backend Lead, Odoo Specialist, Engineering Lead
**Consulted:** Mobile Lead, PO, Security Champion
**Informed:** DevOps, QA Lead

> **Superseded notice (2026-05-15, blueprint v2.1.0).** Sponsor reconfirmed that the solution must run on Odoo Online, which prohibits custom modules and custom ORM models. GPS history now lives in the FastAPI Postgres database; Odoo retains only `x_ngynapp_*` summary fields on `project.task`. See ADR-016.

## Context

DEC-004 asked where GPS pings (check-in / check-out coordinates and periodic location samples during a shift) should live in Odoo.

Two patterns exist:

1. **Inline on `project.task`** — store latitude/longitude as float fields on the shift record. Pros: zero new models. Cons: only the latest ping is retained; no historical trail; no support for periodic sampling.
2. **Custom model `field.gps.ping`** — one row per ping, FK to the shift. Pros: full history, easy reporting, supports overlay maps. Cons: requires custom module (DEC-003) and migration thinking.

The pilot scope (FR-005, FR-008) requires:

- Check-in and check-out GPS captured per shift.
- Optional 5-minute interval sampling during active shift (feature flagged).
- Map overlay in supervisor Odoo view to verify presence.

## Decision

Create custom Odoo model **`field.gps.ping`** inside the `field_mobile_sync` custom module (ADR-014). Each row records one ping; FK `shift_id` links to `project.task`.

Inline fields on `project.task` (`gps_check_in_lat`, `gps_check_in_lon`, `gps_check_out_lat`, `gps_check_out_lon`) are kept as **denormalised summary fields** updated when the corresponding ping is committed. This gives reports a fast read path without joining.

## Rationale

- Inline-only loses history. The map overlay and dispute resolution flows need it.
- Denormalised summary fields preserve simple report queries.
- Custom model is cheap once we have the custom module; the marginal cost is one ORM model + one tree/form view.
- Schema is forward-compatible with future continuous tracking without re-modelling.

## Alternatives considered

| Option | Pros | Cons | Why not chosen |
|---|---|---|---|
| Inline fields only | Zero new models | No history; no periodic sampling support | Loses pilot requirements |
| Custom model only (no denormalised summary) | Single source of truth | Reports must always join | Slight perf cost on common views |
| Custom model + denormalised summary (chosen) | Full history + fast common reads | Two write paths to keep consistent | Best balance |
| Audit log table only (no first-class model) | Generic | Hard to query; no Odoo views | Defeats purpose |

## Consequences

### Positive

- Map overlay in supervisor view becomes straightforward.
- Periodic sampling can be enabled per pilot site without schema changes.
- Dispute resolution has a queryable trail.

### Negative

- Two write paths: insert into `field.gps.ping` and update summary fields on `project.task`. Consistency enforced via Odoo `create()` override on `field.gps.ping`.
- Denormalisation drift if the override is bypassed; mitigated by a constraint test in CI (`tests/test_gps_summary_consistency.py`).

### Neutral

- Privacy: GPS classified as L3 data per `security/data-classification.md`. Retention rule: pings older than 180 days purged via cron (`cron_purge_gps_pings`).

## Compliance / verification

- Migration script `field_mobile_sync/migrations/19.0.1.0.0/post-create-gps-model.py` creates the model.
- ORM constraint: `(shift_id, captured_at)` unique pair to prevent duplicate writes from sync replays.
- TC-GPS-001..003 cover check-in, check-out, and periodic ping flows.
- TC-GPS-004 verifies summary fields on `project.task` stay in sync with the latest ping.

## Related

- FR / NFR: FR-005, FR-008 (GPS check-in / check-out, periodic ping), NFR-PRIV-02 (consent)
- Stories: US-DISC-001, US-FIELD-003, US-FIELD-004, US-ODOO-005
- Other ADRs: ADR-011 (Odoo 19 EE), ADR-014 (`field_mobile_sync` module)
- Decisions closed: DEC-004
