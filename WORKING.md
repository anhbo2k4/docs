# Working — Current Focus

**Updated:** 2026-05-15
**Phase:** Pre-build → Sprint 1 ready (Phase 0 closed; v2.1.0 retargeted for Odoo Online + DigitalOcean)

## Current focus

Sprint 1 (S1 — Project Foundation) kickoff prep. Phase 0 fully closed in v2.1.0. All nine Phase 0 decisions are now `Decided`; the four previously `Provisional` items flipped after sponsor reconfirmation on 2026-05-15. Major architectural pivots:

- **No custom Odoo module.** Solution must run on Odoo Online (which prohibits custom modules). Capabilities relocated to FastAPI + manually-created `x_ngynapp_*` fields. See ADR-015.
- **GPS history in FastAPI Postgres**, not in Odoo. Odoo retains only summary fields. See ADR-016.
- **Odoo target relaxed.** Odoo 19 EE for testing; production contract is RPC-only across Community / Enterprise / Online.
- **Compute + storage on DigitalOcean.** Droplet + managed Postgres + managed Redis + Spaces (`sgp1`). Single vendor, ~$59/mo testing/pilot.

EPIC-08 story US-ODOO-001 (custom module skeleton) is rewritten in-place to "RPC adapter scaffold + capability probe" before S1 work begins.

## Last decisions

- 2026-05-15 — Blueprint **v2.1.0** released. ADR-015 (no custom module) and ADR-016 (GPS in FastAPI Postgres) accepted; ADR-013 + ADR-014 superseded. ADR-011 amended (test target vs production contract). ADR-012 rewritten (DigitalOcean Spaces). Engineering RPC adapter spec replaces Path A / Path B model. Deployment doc moved to DO topology. Decision log Phase 0 fully closed: 9 Decided. New stretch decisions DEC-014 (GPS retention 180 days), DEC-015 (`x_ngynapp_*` prefix).
- 2026-05-15 — Blueprint v2.0.0 released. Phase 0 closed with 5 ADRs (ADR-010..014); 4 Provisional defaults pending PM confirmation. v2.1.0 supersedes by closing all Provisional items and pivoting to Online-compatible architecture.
- 2026-05-13 — Blueprint v1.7.0..v1.0.0 — see `CHANGELOG.md`.

## Open blockers

**None for S1 entry.** All Phase 0 decisions are `Decided`. The pivot in v2.1.0 cancels two pieces of S1-adjacent work (Odoo custom module repo + Odoo CI pipeline) — no replacement work blocks S1; the FastAPI RPC adapter scaffold is naturally in S1 scope.

## Customer-side prerequisites (per Odoo tenant)

These are not S1 blockers because v2.1.0 makes them runtime-discovered, but every new tenant onboarding must complete them before sync goes live:

- Create `x_ngynapp_*` fields per `data/odoo-x-fields-spec.md` via Studio (Online/Enterprise) or Developer mode (Community). Estimated 20–30 minutes.
- Provision Odoo service account + API key.
- Confirm `work_phone` populated for active employees in E.164 (DEC-006 fallback to `private_phone` / `mobile_phone` is automatic).
- Run FastAPI startup probe; resolve any `errors` / `warnings` in the `ProbeReport`.

## Next 3 things

1. Update EPIC-08 story US-ODOO-001 from "custom module skeleton" → "RPC adapter scaffold + capability probe"; align acceptance criteria with `engineering/odoo-module-mapping.md` §4 (`OdooAdapter` interface) + `data/odoo-x-fields-spec.md` §6 (`ProbeReport`).
2. Stand up Sprint 1 ceremonies: planning, daily, review, retro per `governance/sprint-ceremonies.md`.
3. Kick off `US-PLAT-001` (Flutter scaffold) and `US-PLAT-002` (Riverpod skeleton) — first two stories in S1.

## Notes for agents

- Phase 0 is closed. Implementation code is welcome for any S1 story whose status is `Ready`.
- New architectural questions still warrant a `Proposed` ADR via `adr/_template.md`.
- `x_ngynapp_*` fields are the contract surface with Odoo. Adding or renaming any field is a breaking schema change — see `engineering/api-versioning.md`.
- Storage migration off DO Spaces (to AWS S3 / R2) is a config swap, not a refactor — keep all storage code S3-API-compatible.
- Every change to docs requires a `CHANGELOG.md` bullet under the next version.
