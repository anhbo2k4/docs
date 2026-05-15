# ADR-011: Odoo 19 Enterprise as test target; RPC contract for production portability

**Status:** Accepted (amended 2026-05-15, blueprint v2.1.0)
**Date:** 2026-05-15
**Last amended:** 2026-05-15
**Deciders:** Sponsor, Engineering Lead, Odoo Specialist
**Consulted:** Backend Lead, DevOps, PO
**Informed:** QA Lead, Mobile Lead, Security Champion

## Context

DEC-001 originally asked which Odoo version and edition the integration must target. Initial answer: Odoo 19 Enterprise.

A reconfirmation on 2026-05-15 widened the production contract: the solution **must run on Odoo Online** and **must work across versions and editions** (Community, Enterprise, Online). Odoo 19 Enterprise remains the *test* target; it is no longer the *only* deployment target.

This drives:

- Available out-of-the-box modules (DEC-002).
- API surface expectations.
- The custom module question (DEC-003 — answered NO in [ADR-015](ADR-015-no-custom-module-rpc-adapter.md)).
- Hosting decisions: Odoo Online, Odoo.sh, or self-host all in scope.

Constraints:

- Sponsor has acquired Odoo 19 Enterprise license for testing.
- Production deployments may target any reasonably current Odoo (17+) with the required modules installed.
- Odoo Online prohibits custom Python modules; only Studio-level customisation is permitted.

## Decision

- **Test target:** Odoo 19 Enterprise (latest stable point release at S1 kickoff). All CI integration tests, fake-Odoo fixtures, and pilot infrastructure are pinned to this version.
- **Production contract:** the FastAPI integration layer (ADR-004) communicates with Odoo strictly via JSON-RPC. The RPC adapter (ADR-015) detects available modules and adapts behaviour at runtime. Supported deployment targets: Odoo Online, Odoo.sh, self-hosted Enterprise, self-hosted Community 17+.
- **Required Odoo modules** (hard requirements, validated at startup): `base`, `hr`, `project`, `contacts`. Plus the `x_ngynapp_*` field set per `data/odoo-x-fields-spec.md`, created manually via Studio or Developer mode.
- **Soft capabilities** (graceful degrade): `industry_fsm`, `planning`, `hr_attendance`, `hr_timesheet`, `survey`, `quality`, `maintenance`, `documents`. Adapter probes on startup and selects appropriate write paths.

## Rationale

- **Test target Odoo 19 EE** gives us a known, fully-featured environment for CI: Field Service available, Planning available, modern JSON-RPC. Issues found there are easier to triage than on a customer-specific tenant.
- **Production via RPC adapter** keeps the deployment door open for Odoo Online customers who cannot install a custom module. It also removes the version pin: when Odoo 20 ships, the same adapter probes for capabilities and continues to work.
- **Studio-level `x_` fields** are tolerable across editions and easy for an Odoo admin to provision manually.
- **No edition branching in code.** The adapter branches on observed module presence, not on edition strings, because Online does not expose a clean edition signal.

## Alternatives considered

| Option | Pros | Cons | Why not chosen |
|---|---|---|---|
| Odoo 17 Community only | Free; well-documented | No Field Service; limits production reach | Forces customers to upgrade before adopting our app |
| Odoo 17 Enterprise only | Mature, broad community knowledge | Older; locks customers to single version | Same lock-in problem |
| Odoo 18 Enterprise only | Stable, recent | Same lock-in problem | Not flexible enough |
| Odoo 19 Enterprise only (previous decision) | Latest LTS, longest support | Excludes Online customers; excludes other editions | Violates new sponsor requirement |
| Odoo 19 EE as test target + RPC adapter for prod (chosen) | Works on Online + EE + Community; no version pin | Adapter complexity; manual Studio setup per tenant | Best fit for constraint set |

## Consequences

### Positive

- Mobile shift entity maps to `project.task` (always available) or `fsm.task` (Enterprise FSM, when available) via the adapter.
- Field Service worksheet flow remains usable when present; degrade path uses `project.task` + Survey when not.
- Customers on Odoo Online can adopt the platform without leaving SaaS.
- No lock-in to a single Odoo major version; adapter survives upgrades.

### Negative

- **Adapter complexity.** Capability probing + per-capability write paths add code and tests.
- **Manual Studio setup per tenant** for `x_ngynapp_*` fields. Documented but human-driven.
- Test target version 19 EE is newer than typical consultant pool; expect 1–2 vendor-side bug reports during build.
- Production validation effort increases: smoke tests must run against more than just Odoo 19 EE.

### Neutral

- DevOps standardises Odoo 19 EE Docker image for CI and dev (`engineering/dev-environment.md`).
- Odoo SLA for production tenants is delegated to the customer's Odoo provider (Odoo Online SLA, Odoo.sh SLA, or self-host) — see DEC-008.

## Compliance / verification

- `engineering/odoo-module-mapping.md` documents the **RPC adapter capability matrix** (replaces the previous Path A / Path B paths).
- `data/odoo-x-fields-spec.md` lists the required `x_ngynapp_*` field set for any tenant.
- CI runs an Odoo 19 EE Docker image for primary integration tests (`engineering/ci-pipeline.md`).
- A weekly smoke matrix runs against Odoo 17 Community to assert capability fallback paths.
- Pre-pilot acceptance: a fresh Odoo Online tenant dry-run validates the manual setup steps complete in under 30 minutes.
- Odoo Specialist signs off field-level mapping in `data/odoo-mapping.md` against an Odoo 19 EE staging instance before S8 starts.

## Related

- FR / NFR: FR-004, FR-005, FR-009 (shifts, GPS, sync ack), NFR-PORT-01 (Odoo edition portability), NFR-OPS-03 (upgrade safety)
- Stories: US-DISC-001, US-DISC-003, US-ODOO-001..006
- Other ADRs: ADR-004 (FastAPI), ADR-005 (no direct mobile-to-Odoo), ADR-015 (no custom module — RPC adapter), ADR-016 (GPS in FastAPI Postgres)
- Decisions closed: DEC-001 (amended), DEC-002 (now Decided — capability detection)
