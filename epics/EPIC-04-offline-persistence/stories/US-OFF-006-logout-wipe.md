---
id: US-OFF-006
epic: EPIC-04
sprint: S4
fr: [FR-012]
priority: P0
estimate: S
status: Ready
owner: Mobile
---

# US-OFF-006 — Logout wipe + disk-quota guard

## Acceptance criteria

1. Logout deletes the SQLite file, secure storage keys, and any local media in app sandbox.
2. Disk-quota guard fires a `WARN` event when local data > 200 MB and a `BLOCK` event at > 500 MB (capture flows surface a friendly "free up space" prompt).
3. Quota state is observable via Riverpod for the Sync Center.
4. Wipe flow tolerates partial failure (file lock) and retries once.

## Tasks

- [ ] Wipe routine in `LogoutUseCase`.
- [ ] `DiskQuotaService` periodic check (15 min).
- [ ] Riverpod provider exposing quota state.

## FR mapping

FR-012.

## Test cases

TC-OFF-005, TC-SEC-008.

## DoD

- Post-logout DB file size = 0.
- Quota events emitted at thresholds.
