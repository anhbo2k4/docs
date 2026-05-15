# ADR-009: Use Drift (over raw sqflite) for the local SQLite layer

**Status:** Accepted
**Date:** 2026-05-13
**Deciders:** Mobile Lead, Engineering Lead
**Consulted:** Backend Lead, QA Lead
**Informed:** All
**Closes:** DEC-011

## Context

EPIC-04 needs a versioned, type-safe SQLite access layer that supports:

- Compile-time SQL validation (sync envelope correctness).
- Generated DAOs and entities so repositories stay thin.
- Multi-table transactional writes (capture row + `sync_queue` + `sync_attempt`).
- Migration registry with snapshot tests.
- SQLCipher encryption (DB at rest, key in Keystore/Keychain).

`tech-stack.md` already lists Drift as the primary candidate and `sqflite` as the fallback. WORKING.md tentatively defaulted to Drift on 2026-05-13 pending ADR.

## Decision

Adopt **Drift 2.18+** as the canonical SQLite ORM for the mobile app. `sqflite` is permitted only inside Drift's underlying executor or for one-off scripts (e.g. test fixtures); no business code may import `sqflite` directly.

## Rationale

- **Type safety.** Drift's compile-time SQL eliminates an entire class of runtime errors that plague raw `sqflite` codebases.
- **Generated DAOs.** Reduces boilerplate that EPIC-04 would otherwise hand-roll across 11 tables.
- **Migration tooling.** Schema diffing + snapshot tests align with US-OFF-005 (migration harness + downgrade guard).
- **Transactions.** Idiomatic transaction API matches the capture-write pattern in US-OFF-002.
- **Active maintenance.** Drift ships releases on Flutter stable cadence; sqflite is stable but lower velocity.
- **Encryption support.** `sqlcipher_flutter_libs` plugs cleanly into Drift's executor.
- **Team familiarity.** Lower onboarding cost than introducing Floor or Isar.

## Alternatives considered

| Option | Pros | Cons | Why not |
|---|---|---|---|
| Raw `sqflite` | Zero generation, smallest dep | Hand-rolled DAOs, no compile-time SQL, more boilerplate | Higher long-term cost in our 11-table sync surface |
| Floor | Codegen, room-like API | Smaller community, slower release cadence | Drift better fit, more momentum |
| Isar | Fast, no SQL | Non-SQL store; harder to reason about with our state machine + envelope contract | Mismatch with relational sync model |
| ObjectBox | Fast, mature | Closed-source kernel, less inspection in audits | Audit / ops friction |

## Consequences

### Positive

- One canonical pattern across all data stories (US-OFF-001..006).
- Build-runner schema generation wired into `make bootstrap`.
- Snapshot tests for every migration become trivial.

### Negative

- Adds `build_runner` to CI runtime (~30–60 s warm).
- Generated files (`*.g.dart`) must be checked into source for predictable builds.
- Engineers must learn Drift's SQL DSL; mitigated by linking the official guide in `engineering/coding-standards.md`.

### Neutral

- Encryption strategy unchanged; ADR remains compatible with SQLCipher decision in US-OFF-004.

## Compliance / verification

- `pubspec.yaml` lists `drift`, `drift_dev`, `sqlcipher_flutter_libs` at the pinned versions in `architecture/tech-stack.md`.
- CI lint rule forbids `import 'package:sqflite/sqflite.dart'` outside `mobile/test/` and Drift's own executor wiring.
- `make bootstrap` runs `dart run build_runner build --delete-conflicting-outputs` once.
- `flutter analyze` clean on generated files.

## Related

- `data/sqlite-schema.md` (canonical schema)
- `data/sync-state-machine.md`
- ADR-007 (idempotency by `client_id`)
- Stories: US-PLAT-004, US-OFF-001..006
- Closed: DEC-011
