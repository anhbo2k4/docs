---
id: US-SHIFT-001
epic: EPIC-03
sprint: S3
fr: [FR-004]
priority: P0
estimate: M
status: Ready
owner: Mobile
---

# US-SHIFT-001 — Fetch and render My Shifts list

## User story

**As** a field worker
**I want** to see all shifts assigned to me, sorted by start time
**so that** I know what work is queued and what is current.

## Acceptance criteria

1. `GET /v1/shifts?employee_id=me&from=...&to=...` returns shifts; mobile renders them sorted ascending by `start_at`.
2. Each row shows: title, site, start–end, status pill (SCHEDULED / IN_PROGRESS / COMPLETED), sync state badge.
3. Empty state: friendly message with refresh CTA.
4. Loading state: shimmer placeholder for ≥ 3 rows.
5. Error state: inline message with retry; respects offline-first (US-SHIFT-003 cache).
6. Tapping a row routes to detail (US-SHIFT-004).
7. Render time ≤ 500 ms p95 for 100 cached rows on the mid-tier device.

## Tasks

### Mobile
- [ ] `ShiftsRepository.list(employee_id, range)`.
- [ ] `ShiftsListScreen` with Riverpod async notifier.
- [ ] Row widget with status + sync badges.
- [ ] States: loading / empty / error / data.

### Testing
- [ ] Widget tests for each state.
- [ ] Perf test against 100-row dataset (TC-PERF-003).

## Dependencies

- US-SHIFT-003 (cache), US-API-003 backend ready.

## FR mapping

FR-004.

## Test cases

TC-SHIFT-001, TC-PERF-003.

## DoD

- All four states implemented and tested.
- Sort order verified.
- A11y labels present.
