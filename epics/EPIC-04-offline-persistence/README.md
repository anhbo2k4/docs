---
epic: EPIC-04
title: Offline Local Persistence
sprint: S4
priority: P0
status: Ready
owner: Mobile Lead
---

# EPIC-04 — Offline Local Persistence

## 1. Goal

Make local SQLite the **first durable write** for every capture. Define every table, DAO, and the sync-state machine so S5/S6 captures can rely on a stable storage contract.

> Authoritative schema lives in [`data/sqlite-schema.md`](../../data/sqlite-schema.md). Code-level entities mirror it 1:1. The list below is the working set covered by EPIC-04 stories.

## 2. In scope

- Drift tables (per `data/sqlite-schema.md`):
  `employee`, `shift`, `ppe_check`, `form_response`, `gps_event`, `work_result`, `media`, `voice_note`, `sync_queue`, `sync_attempt`, `kv_store`.
- Sync state machine: `PENDING → SYNCING → CONFIRMED | FAILED → DEAD_LETTER`.
- DAOs with transactional writes.
- Migration framework with version + rollback strategy.
- Encrypted DB file on Android (SQLCipher) and iOS (Keychain-backed key).
- `sync_attempt` table for forensic replay (the project's "outbox audit" implementation).
- Disk-quota guard (alert when SQLite > 200 MB).
- DB wipe on logout (TC-SEC-008).
- Read repository contracts ready for S5 capture flows.

## 3. Out of scope

- Sync engine itself (S6/S7).
- Server-side ingest (S8).

## 4. Functional requirements covered

FR-012 (durability), foundation for FR-006..FR-011.

## 5. Stories

| ID | Title | Pri | Estimate | Status | Owner |
|---|---|---|---|---|---|
| US-OFF-001 | Author Drift schema v1 | P0 | M | Ready | Mobile Lead |
| US-OFF-002 | DAOs + repositories with txn writes | P0 | L | Ready | Mobile Lead |
| US-OFF-003 | Sync state machine + transitions | P0 | M | Ready | Mobile |
| US-OFF-004 | DB encryption (SQLCipher / Keychain key) | P0 | M | Ready | Mobile + Security |
| US-OFF-005 | Migration harness + downgrade guard | P0 | M | Ready | Mobile |
| US-OFF-006 | Logout wipe + disk-quota guard | P0 | S | Ready | Mobile |

## 6. Dependencies

- EPIC-01 (Drift skeleton).
- `data/sqlite-schema.md` finalized.

## 7. Risks

- RISK-040 Schema v1 underspecifies media references → S6 rework.
- RISK-041 SQLCipher dependency size pushes APK over budget (NFR-110).
- RISK-042 Migration v1→v2 path missing → first OTA breaks.

## 8. Definition of Done (epic-level)

- All TC-OFF-001..005 green.
- Schema reviewed and signed off by Backend Lead (Odoo mapping risk).
- Encryption verified on both platforms (TC-SEC-004).
- Migration harness exercised with at least one no-op migration.

## 9. Open questions

DEC-011 (Drift vs sqflite) must be locked at S1 review; this epic assumes Drift.
