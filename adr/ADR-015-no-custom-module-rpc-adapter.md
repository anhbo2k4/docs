# ADR-015: No custom Odoo module — RPC adapter pattern + `x_` fields

**Status:** Accepted
**Date:** 2026-05-15
**Deciders:** Sponsor, Engineering Lead, Backend Lead, Odoo Specialist
**Consulted:** DevOps, Security Champion, Mobile Lead
**Informed:** PO, QA Lead
**Supersedes:** [ADR-014](ADR-014-field-mobile-sync-module.md)

## Context

Sponsor reconfirmed (2026-05-15) two binding constraints that override the earlier custom-module decision:

1. **Must run on Odoo Online.** Odoo Online (SaaS) does not allow installing custom Python modules. Only Studio-level customisation is available: new `x_*` fields on existing models, new `x_*` models (limited), Studio views, and Studio automations.
2. **Must work across Odoo versions and editions** (Community, Enterprise, Online). Odoo 19 Enterprise is the test target, but the production contract is "any reasonably current Odoo with the right modules installed".

ADR-014 had specified a custom module `field_mobile_sync` to deliver five capabilities:

- Idempotency anchor (`x_client_id` field).
- Sync acknowledgement (`field.sync.ack` model).
- GPS history (`field.gps.ping` model).
- Sync state visibility (`x_sync_state` field).
- Audit hooks on sensitive writes.

Under the new constraints, four of those five cannot live in Odoo. We need a pattern that delivers the same outcomes without code that ships into the Odoo instance.

## Decision

**No custom Odoo module. The FastAPI integration layer (ADR-004) becomes the single point of integration; it interacts with Odoo only via JSON-RPC.**

The five capabilities relocate as follows:

| Capability (was ADR-014) | New home | Mechanism |
|---|---|---|
| Idempotency anchor | FastAPI + Odoo `x_ngynapp_client_id` | Redis `SETNX` keyed on `client_id`; mirrored as `x_ngynapp_client_id` Char on the Odoo target record for auditability. |
| Sync acknowledgement | FastAPI Postgres + read-after-write RPC | FastAPI writes via `web_save` / `create`, immediately reads back the record, returns the Odoo `id` + `write_date` to mobile as the ack payload. No Odoo-side model. |
| GPS history | **FastAPI Postgres** (see [ADR-016](ADR-016-gps-in-fastapi-postgres.md)) | First-class table `gps_pings` in our own DB. Odoo gets only summary `x_ngynapp_gps_*` fields on `project.task`. |
| Sync state visibility | FastAPI Postgres `sync_envelope` table + Odoo `x_ngynapp_sync_state` | Authoritative state in FastAPI; mirror to Odoo Selection field for ops dashboards. |
| Audit hooks | FastAPI structured logging + Postgres `audit_event` table | All sync-time mutations logged at the integration layer with correlation IDs. |

### RPC adapter pattern

To support multiple Odoo versions and editions, the FastAPI Odoo client uses a **capability-detection adapter** rather than a fixed model contract:

```
OdooAdapter
 ├─ probe()            # called at startup + on cache refresh
 │   reads ir.module.module to detect: project, industry_fsm, planning,
 │   hr_attendance, hr_timesheet, survey, quality, maintenance, documents
 │   reads ir.model.fields to detect required x_ngynapp_* fields exist
 │   classifies edition (Community / Enterprise / Online) via
 │   `ir.module.module` codename heuristics + version probe
 │
 ├─ shift_model        # → 'fsm.task' if industry_fsm else 'project.task'
 ├─ phone_field        # → 'work_phone' (fallback 'mobile_phone' / 'private_phone')
 ├─ supports(capability)  # boolean: e.g. 'planning', 'survey'
 └─ write paths        # implemented per detected capability set
```

Behaviour:

- **Hard requirements** (must be present, otherwise startup fails fast): `hr`, `project`, `contacts`, the configured `x_ngynapp_*` field set on the chosen shift model.
- **Soft capabilities** (graceful degrade): `industry_fsm`, `planning`, `hr_attendance`, `hr_timesheet`, `survey`, `quality`, `maintenance`, `documents`. Missing capabilities fall back to lighter write paths and surface a warning in the admin dashboard.
- **Edition detection** is best-effort. The adapter does not branch on edition directly — it branches on observed module presence, which is the only signal Odoo Online exposes consistently.

### `x_` fields contract

A dedicated spec (`data/odoo-x-fields-spec.md`) lists every `x_ngynapp_*` field that must be created in the customer's Odoo (manual Studio setup or Developer-mode XML import). The prefix `x_ngynapp_` is reserved for this program; no other namespace is used.

The startup probe verifies this set exists; if any required field is missing, the service refuses to serve writes and emits an admin alert with the exact Studio steps to remediate.

## Rationale

- **Odoo Online support is non-negotiable** per sponsor. Custom modules are categorically blocked there.
- **Version-agnostic via capability probing**, not version pinning. We never branch on `odoo_version == 19`; we branch on `supports('industry_fsm')`. This keeps the integration durable across upgrades and edition changes.
- **All durable state we own lives in our Postgres**, not in Odoo. Odoo becomes the system of record for business entities (employees, tasks, attachments) and a thin mirror of mobile sync state. Our FastAPI Postgres is the system of record for sync envelopes, idempotency, GPS history, and audit.
- **Manual Studio setup is acceptable** because the field set is small (≤ 15 fields) and stable across the MVP. The setup steps are documented and version-controlled in `data/odoo-x-fields-spec.md`.
- **Reversibility:** if a future customer is on self-hosted Odoo and wants the convenience of a real module, ADR-014 can be re-instated as an optional optimisation. Our contract does not depend on it.

## Alternatives considered

| Option | Pros | Cons | Why not chosen |
|---|---|---|---|
| Custom module `field_mobile_sync` (ADR-014) | First-class ORM models, server actions, audit hooks | Cannot run on Odoo Online; locks us to self-host | Violates hard constraint |
| Studio-only `x_` models | Some isolation in Odoo | Studio models have limited features (no Python compute, weak migration story); still Online-restricted in practice | Not enough power for sync state, too brittle |
| Sidecar service that proxies Odoo (no FastAPI) | Lighter | We already need FastAPI for mobile contract; two integration layers is worse | Duplication |
| RPC adapter + FastAPI as integration owner (chosen) | Works on Online, version-agnostic, single integration layer | Manual Studio setup needed; capability detection complexity | Best fit for the constraint set |

## Consequences

### Positive

- **Odoo Online compatible.** The mobile platform can be sold to Online tenants without re-architecture.
- **Version / edition tolerance.** Adapter handles Community / Enterprise / Online, Odoo 17+ targets, with one code path.
- **Faster onboarding.** New customer = create `x_ngynapp_*` fields in Studio, point FastAPI at the tenant. No Odoo deployment.
- **Cleaner separation of concerns.** Sync-engine state never pollutes the Odoo schema.

### Negative

- **Manual Studio setup** is required per tenant; documented but human-driven.
- **Adapter complexity** is real. Capability probing + per-capability write paths add code and tests.
- **Two systems of record** (FastAPI Postgres + Odoo). Reconciliation requires care; covered by sync-state machine + read-after-write acks.
- **GPS history is not directly visible inside Odoo.** Operators see only the latest summary; full trail is queried from FastAPI dashboards.

### Neutral

- Audit chain shifts from Odoo `mail.message` to FastAPI structured logs + Postgres `audit_event`.
- Existing ADR-007 (idempotency by `client_id`) is unchanged at the API contract level; only its server-side implementation moves out of Odoo.

## Compliance / verification

- `data/odoo-x-fields-spec.md` defines the required `x_ngynapp_*` field set with types, sizes, and Studio steps.
- `engineering/odoo-module-mapping.md` is rewritten as the RPC adapter spec (no Path A / Path B).
- FastAPI startup probe failure → service unhealthy + Slack alert with remediation steps.
- CI runs against Odoo 19 Enterprise Docker (`engineering/ci-pipeline.md`); a smoke matrix runs against Odoo 17 Community to assert capability fallback paths.
- Pre-pilot acceptance: a "fresh Odoo Online tenant" dry-run is performed to validate the manual setup steps in under 30 minutes.

## Related

- FR / NFR: FR-009, FR-012, FR-013 (sync ack, idempotency, audit), NFR-OPS-03 (upgrade safety), NFR-PORT-01 (Odoo edition portability)
- Stories: US-DISC-001, US-DISC-003, US-ODOO-001..006, US-SYNC-001..006, US-API-004
- Other ADRs: ADR-004 (FastAPI), ADR-005 (no direct mobile-to-Odoo), ADR-007 (idempotency), ADR-011 (Odoo 19 EE as test target), ADR-016 (GPS in FastAPI Postgres)
- Decisions closed: DEC-001 (amended), DEC-002 (now Decided — capability detection), DEC-003 (flipped — no custom module)
