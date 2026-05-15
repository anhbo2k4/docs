---
id: US-PLAT-006
epic: EPIC-01
sprint: S1
fr: []
priority: P0
estimate: S
status: Ready
owner: Mobile
---

# US-PLAT-006 — Sentry init + redaction filter scaffolding

## Acceptance criteria

1. `sentry_flutter` initialised under build flavours; disabled in `dev` by default.
2. `BeforeSendCallback` strips PII keys (`phone`, `otp`, `token`, `email`, `lat`, `lng`) from breadcrumbs and events.
3. User context sets `device_id` only — never `phone` or `email`.
4. Crash → Sentry verified by a test crash button in `dev`.
5. Sentry sample rate configured via `--dart-define`.

## Tasks

- [ ] Add `sentry_flutter` and configure flavour-aware init.
- [ ] Implement `PiiRedactor` with deny-list keys.
- [ ] Document opt-in policy (DEC-013 will refine for pilot vs prod).

## Dependencies

US-PLAT-002.

## FR mapping

Cross-cutting.

## Test cases

TC-SEC-003 (Sentry scrubber removes PII), TC-SEC-006.

## DoD

- Manual test crash visible in Sentry without PII.
- Redaction filter unit-tested.
