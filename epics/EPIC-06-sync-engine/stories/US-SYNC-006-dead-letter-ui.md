---
id: US-SYNC-006
epic: EPIC-06
sprint: S7
fr: [FR-015]
priority: P0
estimate: M
status: Ready
owner: Mobile
---

# US-SYNC-006 — Dead-letter UI + diagnostics export

## Acceptance criteria

1. Dead-letter list shows envelopes in `DEAD_LETTER` with `error_code`, `error_message`, `last_attempt_at`.
2. "Retry" attempts a single re-submit; on success the row becomes `PENDING → CONFIRMED`.
3. "Copy diagnostics" exports a redacted JSON blob (no PII) to clipboard for support.
4. "Discard" requires confirmation and writes an audit row.

## Tasks

- [ ] Dead-letter screen.
- [ ] Diagnostic export with redaction.
- [ ] Tests for retry transitions.

## FR mapping

FR-015.

## Test cases

TC-SYNC-008, TC-SYNC-009.

## DoD

- Redaction verified.
- All actions tested end-to-end.
