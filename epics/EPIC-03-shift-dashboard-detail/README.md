---
epic: EPIC-03
title: Shift Dashboard & Detail
sprint: S3
priority: P0
status: Ready
owner: Mobile Lead
---

# EPIC-03 — Shift Dashboard & Shift Detail

## 1. Goal

Authenticated workers see the shifts assigned to them and can drill into one shift to launch the capture flows. Sync-state badges are visible from this point on.

## 2. In scope

- `My Shifts` screen: list, sort by `start_at`, filter by status.
- Shift detail screen: title, site, time, instructions, action CTAs.
- Cursor pagination for shift list.
- Local cache of shifts in SQLite read-through with TTL.
- Empty / error / loading states with consistent UI.
- Pull-to-refresh and stale-while-revalidate behaviour.
- Sync state badges scaffolded (Pending / Syncing / Confirmed / Failed) ready for capture flows in S5+.

## 3. Out of scope

- Editing shifts. Workers do not edit shifts; they capture against shifts.
- Supervisor views (Odoo only for MVP).
- Map view (post-MVP).

## 4. Functional requirements covered

FR-004, FR-005.

## 5. Stories

| ID | Title | Pri | Estimate | Status | Owner |
|---|---|---|---|---|---|
| US-SHIFT-001 | Fetch and render My Shifts list | P0 | M | Ready | Mobile |
| US-SHIFT-002 | Cursor pagination + pull-to-refresh | P0 | M | Ready | Mobile |
| US-SHIFT-003 | Local cache with TTL (read-through) | P0 | M | Ready | Mobile Lead |
| US-SHIFT-004 | Shift detail with capture CTAs | P0 | M | Ready | Mobile |
| US-SHIFT-005 | Sync state badge component | P0 | S | Ready | Mobile |

## 6. Dependencies

- EPIC-02 (auth) complete; `employee_id` available in claims.
- `api-contracts/shifts.md` agreed with backend.
- DEC-002 closed (knows whether shifts come from `project.task` or `planning.slot`).

## 7. Risks

- RISK-030 Shift fields differ across Odoo modules; mapping fragile.
- RISK-031 Pagination key contract drift between mobile and backend.

## 8. Definition of Done (epic-level)

- TC-SHIFT-001..004 green.
- 100-row shift list renders ≤ 500 ms (TC-PERF-003).
- Offline open shows cached list with stale banner.
- Detail screen launches each capture screen as a stub (real flows arrive in S5/S6).

## 9. Open questions

DEC-002 must be closed.
