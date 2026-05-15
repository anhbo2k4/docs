# ADR-003: SQLite as the durable local store

**Status:** Accepted  
**Date:** 2026-05-13  
**Deciders:** Engineering Lead, Mobile Lead  
**Consulted:** Backend Lead, QA Lead  
**Informed:** All

## Context

The legacy Field Work PWA used `sessionStorage` and IndexedDB sketchily. We have hard requirements: workers go offline for hours or days; captures must survive app kill, OS reboot, battery death; the sync queue must be durable and replay-safe. We need typed, queryable, transactional storage. `sessionStorage`, `shared_preferences`, in-memory state, and even Hive lack transactional guarantees, schema migrations, or query power for sync semantics.

## Decision

Use **SQLite** as the single local durable store, accessed through **Drift 2.18+** (preferred) or **sqflite 2.3+** if the team explicitly chooses raw SQL. SQLite is the **first durable write** for every capture; UI never displays a "saved" state without a confirmed SQLite insert.

## Rationale

- SQLite is battle-tested on iOS and Android.
- ACID transactions make the sync_queue model trustworthy.
- Drift gives compile-time SQL checking and type-safe DAOs.
- Schema migrations are explicit and versioned.
- Backup, encryption, and inspection tools are mature.

## Alternatives considered

| Option | Pros | Cons | Why not chosen |
|---|---|---|---|
| Hive | Fast key-value | No SQL, weak relations, no real migrations | Sync queue model needs SQL |
| ObjectBox | Fast, good DX | Native deps complicate iOS builds; smaller ecosystem | Risk for pilot timeline |
| Isar | Modern API | Younger project, breaking changes | Stability concern |
| sessionStorage / shared_preferences | Trivial | Not durable, not queryable | Unsafe for evidence |

## Consequences

### Positive
- Durable across app kill and OS restart.
- Sync queue, captures, and shifts are queryable with SQL.
- Migrations are versioned and testable.
- Encrypted-at-rest via OS file encryption + Keystore-protected key for PII columns.

### Negative
- More boilerplate than key-value stores.
- Migration discipline required; bad migrations corrupt user data.
- DAO + repository layering increases initial code.

### Neutral
- Team standardises on either Drift or sqflite. Drift recommended; choice locked at S1.

## Compliance / verification

- SQLite schema and migrations live in `data/sqlite-schema.md` and `mobile/lib/@core/db/migrations/`.
- No capture is shown as saved without a corresponding SQLite row.
- Migration rollback is tested in CI for every schema PR.

## Related

- FR-009 to FR-015 (all capture and sync FRs)
- ADR-007 (idempotency model relies on SQLite-stored `client_id`)
- Stories: `epics/EPIC-04-offline-persistence/`
