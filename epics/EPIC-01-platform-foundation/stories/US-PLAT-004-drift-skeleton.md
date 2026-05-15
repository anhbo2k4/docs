---
id: US-PLAT-004
epic: EPIC-01
sprint: S1
fr: []
priority: P0
estimate: M
status: Ready
owner: Mobile Lead
---

# US-PLAT-004 — Add Drift skeleton + migration harness

## Acceptance criteria

1. Drift database under `@core/db/app_database.dart` initialised lazily, opens once per process.
2. Migration harness supports `from → to` with an explicit registry; downgrade is forbidden and surfaced as an error.
3. Build runner generates schema with no warnings.
4. SQLite file is opened with encryption at rest (SQLCipher Android, key in Keychain on iOS).
5. Empty schema v1 ships with no business tables (those land in EPIC-04).
6. A `pragma integrity_check` runs at first open in `dev` and `staging`.

## Tasks

- [ ] Add `drift`, `drift_dev`, `sqlcipher_flutter_libs`.
- [ ] Implement encryption key strategy (`SecureSessionStore` pre-existing).
- [ ] Author migration registry stub.
- [ ] CI build_runner step.

## Dependencies

US-PLAT-003.

## DoD

- Schema generates cleanly.
- Encryption verified by reading the on-disk file with `sqlite3` and confirming non-readable bytes.
