# ADR-014: Build custom Odoo module `field_mobile_sync`

**Status:** Superseded by [ADR-015](ADR-015-no-custom-module-rpc-adapter.md) (2026-05-15)
**Date:** 2026-05-15
**Superseded:** 2026-05-15
**Deciders:** Backend Lead, Odoo Specialist, Engineering Lead
**Consulted:** Sponsor, Mobile Lead, Security Champion
**Informed:** DevOps, QA Lead, PO

> **Superseded notice (2026-05-15, blueprint v2.1.0).** Sponsor reconfirmed that the solution must run on Odoo Online, which prohibits installing custom modules. All capabilities formerly delivered by `field_mobile_sync` are now provided by the FastAPI integration layer plus manually-created `x_ngynapp_*` fields on existing Odoo models (Studio / Developer mode). See ADR-015.

## Context

DEC-003 asked whether the integration requires a custom Odoo module, and if so, what its scope should be.

Out of the box, Odoo 19 Enterprise (ADR-011) does not provide:

- A field on `project.task` to carry the mobile client's `client_id` (UUID v4 from ADR-007) for idempotent upserts.
- A sync acknowledgement mechanism the mobile can poll to confirm a write committed.
- A first-class `field.gps.ping` model (ADR-013).
- A queryable employee-to-mobile-phone mapping that tolerates E.164 normalisation (DEC-006).
- Server actions to enqueue a `web.sync` audit row when sensitive writes happen (PII / GPS).

Without a custom module, the FastAPI layer would have to monkey-patch core models through `ir.model.fields` programmatic creation, which is unsupported in Enterprise and breaks on minor upgrades.

## Decision

Build a custom Odoo module **`field_mobile_sync`** owned by the program. Initial scope (v0.1.0) for MVP:

| Element | Purpose |
|---|---|
| Manifest | Depends on `project`, `industry_fsm`, `planning`, `hr`, `survey`, `mail` |
| Model: `field.gps.ping` | GPS history (ADR-013) |
| Model: `field.sync.ack` | Sync acknowledgement record per `client_id` |
| Field: `project.task.x_client_id` (Char) | Idempotency anchor (ADR-007) |
| Field: `project.task.x_sync_state` (Selection) | `pending / synced / failed` for ops visibility |
| Fields: `project.task.gps_check_in_lat/lon`, `gps_check_out_lat/lon` | Denormalised summary (ADR-013) |
| Server action: `enqueue_sync_audit` | On write to sensitive models, enqueue audit row |
| Security group: `field_mobile_sync.group_mobile_writer` | Restricted role used by FastAPI service account |
| Cron: `cron_purge_gps_pings` | 180-day GPS retention |
| Views | Tree + form for `field.gps.ping`, `field.sync.ack`; backend-only |

Module is hosted in repo `field-mobile-sync` (separate from the mobile + backend monorepo) and pinned via `requirements-odoo.txt` in the staging Odoo image.

## Rationale

- All five missing capabilities are tightly coupled to the mobile sync contract; bundling them into one module keeps versioning aligned with the mobile/backend release train.
- One module, one upgrade path, one CI pipeline.
- Security group lets us follow least privilege on the FastAPI service account.
- Future scope (post-MVP) can extend the same module without forcing a second.

## Alternatives considered

| Option | Pros | Cons | Why not chosen |
|---|---|---|---|
| No custom module — patch via `ir.model.fields` programmatically | No new module | Unsupported in EE; breaks on upgrade; no version control | Disqualified |
| Multiple small modules (one per concern) | Granular | More upgrade churn; harder ACL story | Unnecessary fragmentation |
| One module (chosen) | One pipeline, aligned versioning, clear ownership | Slightly larger blast radius if the module breaks | Best fit for pilot |
| Use OCA modules (e.g. `web_api`) | Free, community-maintained | Don't fit our exact contract; would require forks | Risk of drift |

## Consequences

### Positive

- Mobile sync envelope (ADR-007) lands cleanly because `x_client_id` exists.
- GPS history (ADR-013) has a real ORM home with views, security, and migrations.
- Audit visibility for sensitive writes is built-in.

### Negative

- Adds a third repository for the program (mobile, backend, odoo-module).
- Odoo Specialist becomes a hard dependency for module changes.
- Enterprise upgrade path now must validate the module against new Odoo versions.

### Neutral

- Module is internal / proprietary; no plan to publish to OCA.

## Compliance / verification

- `engineering/odoo-module-mapping.md` documents module skeleton, manifest, and migration files.
- CI pipeline `odoo-module-ci` runs `flake8`, `pylint-odoo`, and `pytest-odoo` on every PR.
- Module version follows Odoo conventions: `19.0.MAJOR.MINOR.PATCH`. Pinned in `requirements-odoo.txt`.
- Pre-merge: Odoo Specialist + Backend Lead approval required (`engineering/code-owners.md`).

## Related

- FR / NFR: FR-009, FR-012, FR-013 (sync ack, idempotency, audit), NFR-OPS-03 (upgrade safety)
- Stories: US-DISC-001, US-DISC-003, US-ODOO-001..010, US-SYNC-001..006
- Other ADRs: ADR-007 (idempotency), ADR-011 (Odoo 19 EE), ADR-013 (custom GPS model)
- Decisions closed: DEC-003
