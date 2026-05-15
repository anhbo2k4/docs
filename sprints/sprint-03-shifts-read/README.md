---
sprint: S3
title: Shifts Read Workflow
duration: 2 weeks
priority: P0
status: Ready
owner: Mobile Lead
---

# Sprint 3 — Shifts Read Workflow

## 1. Sprint goal

Authenticated workers see assigned shifts and drill into a detail view that launches capture flow stubs. Sync-state badges are scaffolded for use in S5+.

## 2. Theme

First user-visible feature. Solidifies the read path and the cache strategy that capture flows will rely on.

## 3. Scope (in)

- My Shifts list with sort, filter, empty/error/loading states.
- Cursor pagination + pull-to-refresh.
- Read-through cache with TTL.
- Shift detail screen + capture CTA stubs.
- `SyncStateBadge` component.

## 4. Out of scope

- Capture flows (PPE, GPS, photos) — S5/S6.
- Server-side ingest — S8.

## 5. Pre-conditions

- S2 done.
- DEC-002 closed (shift entity known).
- `api-contracts/shifts.md` ratified.
- Backend ships a mock or Odoo-backed `/v1/shifts` (if backend not ready, mobile uses a local fixture and swaps in S8).

## 6. Stories committed

| ID | Title | Owner | Estimate |
|---|---|---|---|
| US-SHIFT-001 | List render | Mobile | M |
| US-SHIFT-002 | Cursor pagination + pull-to-refresh | Mobile | M |
| US-SHIFT-003 | Local cache with TTL | Mobile Lead | M |
| US-SHIFT-004 | Shift detail with CTAs | Mobile | M |
| US-SHIFT-005 | `SyncStateBadge` component | Mobile | S |

## 7. Risks for this sprint

- RISK-030 Shift fields differ across Odoo modules.
- RISK-031 Pagination contract drift between mobile and backend.

## 8. Sprint-level Definition of Done

- TC-SHIFT-001..004 + TC-PERF-003 green.
- Offline open shows cached list with stale banner.
- Detail screen launches each capture stub.
- A11y labels on all states.

## 9. Demo script

- Login → My Shifts → tap a shift → see detail.
- Toggle airplane mode → list still renders from cache.
- Show stale banner.

## 10. Retro inputs

- Was the shift contract stable across the sprint?
- Did the cache TTL produce surprising behaviour?
- How well did the design system hold up?
