---
id: US-OFF-002
epic: EPIC-04
sprint: S4
fr: [FR-012]
priority: P0
estimate: L
status: Ready
owner: Mobile Lead
---

# US-OFF-002 — DAOs + repositories with transactional writes

## Acceptance criteria

1. Each table has a DAO under `@core/db/dao/`. Every multi-table write goes through a Drift transaction.
2. Capture writes pattern is: insert capture row + insert `sync_queue` row + insert `sync_attempt` row (audit), all in one transaction.
3. Repositories are thin: they coordinate DAOs and do not contain business logic.
4. All DAO methods are nullable-aware and return typed entities, not generic `Map`s.
5. Errors surface as typed exceptions (`DbConflictException`, `DbCorruptedException`) with the original SQLite code.

## Tasks

- [ ] DAOs and entity classes for all 9 tables.
- [ ] `EnvelopeRepository.enqueue(...)` reusable from every capture repo.
- [ ] Unit tests for each DAO with in-memory DB.

## Dependencies

US-OFF-001.

## FR mapping

FR-012.

## Test cases

TC-OFF-001..003.

## DoD

- 100% txn-write coverage on capture repos.
- Failure tests for each typed exception.
