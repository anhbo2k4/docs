---
sprint: S4
title: SQLite Offline Foundation
duration: 2 weeks
priority: P0
status: Ready
owner: Mobile Lead
---

# Sprint 4 — SQLite Offline Foundation

## 1. Sprint goal

Make local SQLite the first durable write for every capture. Define every table, DAO, and the sync state machine before S5 starts.

## 2. Theme

Storage-first sprint. Capture-flow speed in S5/S6 depends on how clean these foundations are.

## 3. Scope (in)

- Drift schema v1 with every business table.
- DAOs + repositories with transactional writes.
- Sync state machine + transitions.
- DB encryption (SQLCipher / Keychain).
- Migration harness + downgrade guard.
- Logout wipe + disk-quota guard.

## 4. Out of scope

- Sync engine implementation (S6/S7).
- Server ingest (S8).

## 5. Pre-conditions

- S3 done (cache requirements known).
- DEC-011 closed (Drift confirmed).
- Backend Lead available to review schema for Odoo mapping risk.

## 6. Stories committed

| ID | Title | Owner | Estimate |
|---|---|---|---|
| US-OFF-001 | Drift schema v1 | Mobile Lead | M |
| US-OFF-002 | DAOs + repositories with txn writes | Mobile Lead | L |
| US-OFF-003 | Sync state machine + transitions | Mobile | M |
| US-OFF-004 | DB encryption | Mobile + Security | M |
| US-OFF-005 | Migration harness + downgrade guard | Mobile | M |
| US-OFF-006 | Logout wipe + disk-quota guard | Mobile | S |

## 7. Risks for this sprint

- RISK-040 Schema underspec → S6 rework.
- RISK-041 SQLCipher dep size pushes APK budget.
- RISK-042 Migration v1→v2 path missing.

## 8. Sprint-level Definition of Done

- Schema reviewed and signed off by Backend Lead.
- TC-OFF-001..005 + TC-SEC-004, TC-SEC-008 green.
- Disk inspection of DB file shows non-readable bytes.
- Migration harness exercised with one no-op migration.

## 9. Demo script

- Show ER diagram in `data/`.
- Toggle airplane mode, write a fake PPE row, kill app, reopen → row persists with PENDING.
- Logout → file size 0.

## 10. Retro inputs

- Was the schema review thorough?
- Did SQLCipher add unexpected friction (build size, encryption key bootstrap)?
- Are the DAOs ergonomic for capture work in S5?
