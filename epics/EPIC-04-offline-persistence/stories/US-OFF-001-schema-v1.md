---
id: US-OFF-001
epic: EPIC-04
sprint: S4
fr: [FR-012]
priority: P0
estimate: M
status: Ready
owner: Mobile Lead
---

# US-OFF-001 — Author Drift schema v1

## User story

**As** the mobile lead
**I want** a versioned, reviewed schema v1 covering every capture and sync table
**so that** S5/S6 capture flows have a stable storage contract.

## Acceptance criteria

1. Tables in `data/sqlite-schema.md` exist in code with matching columns and indexes:
   `employee`, `shift`, `ppe_check`, `form_response`, `gps_event`, `work_result`, `media`, `voice_note`, `sync_queue`, `sync_attempt`, `kv_store`.
2. Every business row that participates in sync carries `client_id` UUID v4, `created_at`, `updated_at`, `sync_state`.
3. Indexes on `(employee_id, shift_id)`, `(sync_state)`, `(client_id UNIQUE)` per table where applicable (full list in `data/sqlite-schema.md`).
4. Schema reviewed and signed off by Backend Lead (Odoo mapping check).
5. Code-generated DAOs build cleanly under `build_runner`.

## Tasks

- [ ] Add `drift` schemas under `@core/db/tables/`.
- [ ] Run `build_runner` and commit generated files.
- [ ] Cross-check `data/sqlite-schema.md` and align both directions.
- [ ] Backend Lead review.

## Dependencies

- EPIC-01 Drift skeleton (US-PLAT-004).
- `data/sqlite-schema.md`.

## FR mapping

FR-012.

## Test cases

TC-OFF-001.

## Definition of Done

- Schema generated, all DAOs compile.
- ERD diagram updated under `data/`.
- Backend Lead approval recorded in PR.
