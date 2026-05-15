# Coding standards

**Status:** Active
**Owner:** Engineering Lead

These standards apply to all code in the repository. Linters and formatters enforce them in CI; reviewers enforce the spirit.

## Universal

- Code is read more than written. Optimise for the next reader.
- A function does one thing at one level of abstraction.
- Names describe what, not how. Avoid `Manager`, `Helper`, `Util` unless truly generic.
- Errors are typed. No raw exceptions thrown across module boundaries.
- All public functions have doc comments stating purpose, params, returns, and failure modes.
- TODOs reference a story ID. Untracked TODOs are blocked by lint.

## Dart / Flutter

- Lints: `package:lints/recommended.yaml` + `flutter_lints` + project rules in `analysis_options.yaml`.
- Formatter: `dart format` with default 80-col width. CI fails on diff.
- Imports: relative within a feature, package-style across features.
- Layering (mobile):
  - `screen/<feature>/state/` — Riverpod notifiers, immutable state.
  - `screen/<feature>/widget/` — UI; never holds business logic.
  - `screen/<feature>/data/` — repositories called by state.
  - `@core/` — infrastructure, shared by features.
- State: Riverpod (per ADR-006). No `setState` in non-trivial screens. No global singletons outside `@core`.
- Async: never `Future.delayed` to "fix" race conditions. Resolve causally.
- Null safety: full sound null safety. No `as Type` casts without runtime check.
- Logging: `app_logger.d/i/w/e`. Never `print`. PII scrubbed by default formatter.
- Tests:
  - Pure logic → unit tests.
  - Widget contracts → widget tests.
  - User journeys → integration_test.

## Python (FastAPI + Celery)

- Lints: `ruff` (configured in `pyproject.toml`) + `mypy --strict` for `app/`.
- Formatter: `ruff format`. CI fails on diff.
- Type hints required on all public functions. `Any` is a code smell.
- Layering (backend):
  - `app/<feature>/router.py` — FastAPI routes; no business logic.
  - `app/<feature>/service.py` — business logic; pure where possible.
  - `app/<feature>/repository.py` — DB access via SQLAlchemy.
  - `app/<feature>/schemas.py` — Pydantic models.
  - `app/<feature>/tasks.py` — Celery tasks.
- Errors: domain errors are Pydantic-shaped; mapped to HTTP via a single exception handler. No `HTTPException` raised inside services.
- DB: parameterized queries only. No string-built SQL. Migrations via Alembic.
- Idempotency: `client_id` is the unique key per `(employee_id, client_id)`. Do not invent secondary keys.
- Logging: `structlog` with `before_send` PII scrubber. Never `print`.
- Tests: pytest + pytest-asyncio. Integration tests use real Postgres + Redis (in containers); Odoo writes mock unless tagged `@odoo`.

## TypeScript (Supabase Edge)

- Lints: `eslint:recommended` + `@typescript-eslint/recommended`.
- Strict mode on. No `any`.
- One function per file, default export typed.
- Secrets via Deno env only; never logged.

## Odoo (Python)

- Follow Odoo OCA guidelines for module `field_mobile_sync`.
- Models prefixed `field.mobile.sync.*`.
- All writes via `external_id = client_id` for idempotency.
- No `sudo()` outside narrowly scoped service-account utilities.

## SQL

- Migrations are forward-only in production. Roll-forward to fix; never edit applied migrations.
- Indexes named `idx_<table>_<col>[_<col>]`. Foreign keys named `fk_<table>_<ref>`.
- Every table has `created_at`, `updated_at`. Soft-delete with `deleted_at` only when business demands; otherwise hard delete.

## Errors and exceptions

- User-visible error messages are short, actionable, and localized through the app's i18n layer.
- Internal errors carry a stable `error_code` (SCREAMING_SNAKE) so clients can branch without parsing English.
- Error catalog is `api-contracts/error-codes.md`. Adding a code needs a CHANGELOG entry.

## Performance

- No N+1 queries in hot paths. Use eager loading or explicit batch.
- Mobile: avoid rebuilding lists; use `select` Riverpod selectors and `const` widgets.
- Backend: cache idempotently-resolved data with explicit TTLs; never silently.

## Security in code

- Authn / authz checks at the router layer, never deeper. Make services pure.
- Validate all external input via Pydantic / Dart DTOs at the boundary.
- Never log JWTs, OTPs, phone numbers in full, lat/lng, or media bytes. Logger redacts by default; do not bypass.
- For mobile: data at rest is in SQLite (encrypted per ADR-003 stance) and Keychain/Keystore. No `SharedPreferences` for sensitive data.

## Comments

- Why, not what. The code shows what.
- Reference story IDs and ADRs near non-obvious decisions: `// ADR-007: idempotency by client_id`.

## Files

- One public class per Dart file unless they're tightly coupled.
- One concept per Python module; if a file exceeds ~400 lines without good reason, split.
- README.md in every non-trivial folder. Empty `__init__.py` discouraged; document the package's purpose if it exists.
