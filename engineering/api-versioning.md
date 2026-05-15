# API Versioning Policy

**Status:** Active
**Owner:** Backend Lead
**Last updated:** 2026-05-13

This policy defines how the FastAPI integration backend evolves without breaking pinned mobile clients in the field. It complements `api-contracts/openapi.yaml` (the contract) and `engineering/release-process.md` (the cadence).

---

## 1. Versioning model

- URL-based major versioning: `/api/v1/...`, `/api/v2/...` only when a breaking change is unavoidable.
- Single live major in production at any time, plus a deprecation window of **180 days** running in parallel during cutover.
- Minor and patch changes are **additive-only** within a major and never break clients.

The mobile app pins to a specific major (`/api/v1`). Server is responsible for backwards compatibility within `v1` for the deprecation window.

---

## 2. What is breaking vs additive

### Breaking (requires a new major)

- Removing a field from a response.
- Renaming a field in a request or response.
- Changing a field's type or units.
- Changing a field from optional to required (in requests).
- Removing or renaming an endpoint, method, or status code.
- Tightening validation in a way that previously accepted requests now fail.
- Changing the meaning of an existing error code.

### Additive (safe, ship in any minor)

- Adding a new endpoint.
- Adding a new optional field to a request.
- Adding a new field to a response (clients ignore unknown fields per §3).
- Adding a new error code.
- Loosening validation (accepting more inputs than before).
- Adding a new authentication scope while keeping existing ones.

### Grey area — treat as breaking unless reviewed

- Reordering enum values.
- Changing default values for optional request fields.
- Changing pagination defaults.
- Tightening rate limits.

---

## 3. Client contract — required behaviours

The mobile app and any future client MUST:

- **Tolerate unknown fields** in JSON responses. No "strict-mode" decoders that fail on extra keys.
- **Tolerate new error codes** by mapping unknown codes to a generic "unknown error, retry later" path (`api-contracts/error-codes.md`).
- **Tolerate new enum values** by mapping unknown values to a documented fallback (e.g., shift status `unknown` shows as "Other").
- **Send a `User-Agent`** of the form `FieldWork/<app-version> (<platform>; <build>)` so the server can correlate client builds.
- **Honour `Sunset` headers** (RFC 8594) from the server.

Tests under `qa/test-cases.md` group `OBS` validate these tolerances.

---

## 4. Server-side responsibilities

- Emit a `Deprecation: true` and `Sunset: <date>` header on any endpoint marked deprecated.
- Add a `Warning: 299` header explaining the next action when behaviour is about to change.
- Log every request bucketed by client `User-Agent`; surface "old client share" in dashboards.
- Maintain `api-contracts/openapi.yaml` as the single contract source. Generated FastAPI spec must match in CI (see `engineering/ci-pipeline.md` `pr-backend / contract`).

---

## 5. Deprecation playbook

When a new major must ship:

1. Open an ADR proposing the breaking change. Examples: schema overhaul, auth flow change, signed-envelope addition.
2. Add `/api/v2/...` endpoints alongside `/api/v1/...`.
3. Mobile pins are updated, but the server keeps `/v1` live for 180 days.
4. Server adds `Deprecation` + `Sunset` headers on `/v1`.
5. Mobile app increments min-version (`DEC-012`) so devices still on `/v1` are nudged to update.
6. Dashboards track `/v1` traffic; cutover plan documented in `runbooks/`.
7. After 180 days **and** `/v1` traffic < 1 % of requests for 30 consecutive days, retire `/v1`. Sponsor sign-off required.

If forced retirement is needed for a security reason: see `runbooks/incident-response.md`.

---

## 6. Mobile-side compatibility budget

The app supports **N** and **N-1** server majors at runtime:

- App version 1.x → talks to `/api/v1` only.
- App version 2.x → can talk to both `/api/v1` and `/api/v2`, with feature-flag gating, until 180-day window closes.

The compatibility matrix is maintained in `engineering/feature-flags.md`.

---

## 7. Field naming and schema evolution

- Field names: `snake_case`, lowercase, no abbreviations except those in `traceability/glossary.md`.
- IDs: UUID v4 strings, never integers, never reused.
- Timestamps: ISO-8601 UTC, suffix `_at` (`created_at`, `synced_at`).
- Booleans: prefixed `is_` / `has_` / `can_`.
- Enums: documented in `api-contracts/openapi.yaml` with explicit `description` per value.
- Money / numerics: integer cents in JSON; never floats for currency.

Renaming a field is a breaking change; never rename in place. Add the new field, deprecate the old one over the next major.

---

## 8. Pagination, filtering, sorting

- Pagination: cursor-based (`cursor` + `limit`); offset-based not allowed for new endpoints.
- Cursors: opaque base64-encoded strings the server can rotate without client changes.
- Filters: `?filter[field]=value` with `description` per filter in OpenAPI.
- Sorting: `?sort=-created_at,name` (RFC 6570 style); document available sort keys per endpoint.

Adding new filters or sort keys is additive. Removing one is breaking.

---

## 9. Error envelope

Single envelope across all endpoints (`api-contracts/error-codes.md`):

```json
{
  "error": {
    "code": "AUTH_OTP_INVALID",
    "message": "OTP code is invalid or expired.",
    "details": {"attempts_left": 2},
    "trace_id": "...",
    "documentation_url": "https://docs.../errors#AUTH_OTP_INVALID"
  }
}
```

- `code` is stable per major version.
- `message` is human-friendly; localisation handled in app via code lookup.
- `details` is shape-flexible per code; clients access fields defensively.

Adding a new code: additive. Removing one: breaking.

---

## 10. Idempotency, concurrency, conditional requests

- Mutation endpoints accept `Idempotency-Key` header (UUID v4); server replays prior response within 24 h. (ADR-007.)
- `POST /sync/envelope` uses `client_id` from the body as the idempotency key (per ADR-007); the header is optional reinforcement.
- Reads support `If-None-Match` / `ETag` where relevant for shifts list.
- Writes support `If-Match` for optimistic concurrency on shift updates.

---

## 11. Versioning of the OpenAPI document itself

`api-contracts/openapi.yaml` carries a top-level `info.version` string:

- `MAJOR.MINOR.PATCH`.
- MAJOR matches the URL major.
- MINOR bumps on additive change.
- PATCH bumps on docs-only change (description, example).

CI (`engineering/ci-pipeline.md` `pr-backend / contract`) blocks PRs that:

- Change MAJOR without an ADR.
- Bump MINOR without an entry in `CHANGELOG.md`.
- Diverge from FastAPI-emitted spec without explanation.

---

## 12. Cross-references

- ADR-005 — no direct mobile-to-Odoo (the reason this layer matters).
- ADR-007 — idempotency by `client_id`.
- `api-contracts/openapi.yaml` — the contract itself.
- `engineering/feature-flags.md` — gate version-specific behaviour.
- `engineering/ci-pipeline.md` — contract diff tooling.
- `runbooks/rollback.md` — emergency rollback to `/v1`.
- `traceability/decision-log.md` DEC-012 — force-update enforcement.
