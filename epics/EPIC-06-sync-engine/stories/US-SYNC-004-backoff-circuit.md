---
id: US-SYNC-004
epic: EPIC-06
sprint: S7
fr: [FR-013]
priority: P0
estimate: M
status: Ready
owner: Mobile
---

# US-SYNC-004 — Backoff + jitter + circuit breaker

## Acceptance criteria

1. Exponential backoff with jitter (e.g. base 2 s, cap 5 min) across attempts.
2. Circuit breaker opens after N consecutive 5xx and closes after a half-open success window.
3. After max retries (configurable, default 8) the row goes to `DEAD_LETTER` with reason.
4. 4xx responses (except 429) move directly to `DEAD_LETTER` with `error_code`.
5. Retry storm test: 50 concurrent retries produce exactly one Odoo write (TC-SYNC-024).

## Tasks

- [ ] Backoff util.
- [ ] Circuit breaker state.
- [ ] Test harness for retry storm.

## FR mapping

FR-013.

## Test cases

TC-SYNC-002, TC-SYNC-024.

## DoD

- Storm test green.
- DLQ classification verified for representative 4xx and 5xx.
