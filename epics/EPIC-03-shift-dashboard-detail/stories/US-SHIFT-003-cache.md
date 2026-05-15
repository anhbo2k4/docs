---
id: US-SHIFT-003
epic: EPIC-03
sprint: S3
fr: [FR-004]
priority: P0
estimate: M
status: Ready
owner: Mobile Lead
---

# US-SHIFT-003 — Local cache with TTL (read-through)

## Acceptance criteria

1. Read-through cache: read from SQLite first, then refresh in background; UI never blocks on the network.
2. TTL configurable per environment (default 5 min).
3. Conflict resolution: server wins; local-only edits do not exist for shifts (mobile is read-only on shifts).
4. Cache survives app kill.
5. Logout wipes cache (US-AUTH-007 hook).

## Tasks

- [ ] Add `shifts_cache` table to Drift (or stage entry to EPIC-04 if S4 lands first).
- [ ] Repository read-through implementation.
- [ ] Background refresh strategy (Riverpod `refresh()` or `keepAlive`).

## Dependencies

- EPIC-04 (`shifts_cache` table). If S4 lands after S3, ship a thin in-memory cache and migrate at S4 close.

## FR mapping

FR-004.

## Test cases

TC-SHIFT-001 (cache hit path).

## DoD

- Cold start with cached data renders in ≤ 300 ms.
- TTL respected.
