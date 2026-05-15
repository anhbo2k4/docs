---
id: US-CAP-006
epic: EPIC-05
sprint: S5
fr: [FR-008]
priority: P0
estimate: M
status: Ready
owner: Mobile
---

# US-CAP-006 — GPS plugin wrapper + consent

## Acceptance criteria

1. Wrapper API: `requestPermission()`, `getCurrentLocation(timeout, accuracy)`, `watchLocation(stream)`.
2. First-use consent dialog explains why GPS is needed; choice recorded in `sync_attempt`.
3. Permission denied path returns a typed error and disables GPS-dependent CTAs.
4. Cross-platform error mapping: timeouts, insufficient accuracy, OS rejection.
5. Mocked in unit tests via dependency injection.

## Tasks

- [ ] `LocationPlugin` under `plugins/gps/`.
- [ ] Consent dialog widget.
- [ ] DI binding for tests.

## FR mapping

FR-008.

## Test cases

TC-GPS-001, TC-GPS-002.

## DoD

- Wrapper documented with sample usage.
- Permission flow verified.
