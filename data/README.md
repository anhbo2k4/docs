# Data Layer

**Status:** Active  
**Owner:** Mobile Lead + Backend Lead

| File | Purpose |
|---|---|
| [sqlite-schema.md](./sqlite-schema.md) | Mobile SQLite schema, indexes, migrations |
| [postgres-schema.md](./postgres-schema.md) | Backend Postgres schema |
| [sync-state-machine.md](./sync-state-machine.md) | Formal state automaton + idempotency contract |
| [odoo-mapping.md](./odoo-mapping.md) | Mobile entities ↔ Odoo records (depends on Phase 0) |

## Data flow summary

```
[Capture screen]
   ↓ INSERT
[SQLite source row + sync_queue row] (PENDING)
   ↓ background trigger
[Sync worker reads queue → POST envelope]
   ↓ HTTP
[FastAPI sync_envelopes (PENDING) → Celery enqueue]
   ↓ Celery
[Odoo write via field_mobile_sync] ← idempotent by external_id = client_id
   ↓ ack
[FastAPI updates sync_envelopes (CONFIRMED)]
   ↓ poll or ack
[Mobile updates sync_state to CONFIRMED]
```

## Source of truth per data class

| Data | Source of truth | Synced to | Notes |
|---|---|---|---|
| Employee profile | Odoo `hr.employee` | Mobile cache + FastAPI cache | Read-only on mobile |
| Shift schedule | Odoo `project.task` (or `planning.slot`) | Mobile cache + Postgres cache | Read-only on mobile |
| Shift status (start/end) | Mobile (capture moment) | Odoo via FastAPI | Eventually consistent |
| PPE check | Mobile | Odoo | Eventually consistent |
| Forms | Mobile | Odoo | Eventually consistent |
| GPS | Mobile | Odoo | Eventually consistent |
| Photos / voice | Mobile filesystem + S3 | Odoo metadata | Bytes in S3, refs in Odoo |

## Data residency

TBD per pilot agreement (DEC-008). Document the chosen region in `architecture/deployment.md`.
