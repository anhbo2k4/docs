---
id: US-API-003
epic: EPIC-07
sprint: S8
fr: [FR-004, FR-005]
priority: P0
estimate: M
status: Ready
owner: Backend
---

# US-API-003 — `/v1/shifts` list + detail (Odoo read passthrough or cache)

## Acceptance criteria

1. `GET /v1/shifts?from=&to=&cursor=` returns shifts assigned to the authenticated employee.
2. Source per DEC-002: read-through cache from Postgres, refreshed by a background sync from Odoo.
3. Cursor pagination (opaque cursor); page size default 25, max 100.
4. `GET /v1/shifts/{id}` returns full detail; 403 `NOT_ASSIGNED` if not in the employee's set.
5. Cache TTL 60 s default; configurable via env.
6. Read latency p95 ≤ 250 ms.

## Tasks

- [ ] List + detail handlers.
- [ ] Background sync worker pulling from Odoo.
- [ ] Cursor implementation.
- [ ] Tests: list, detail, not-assigned, pagination.

## Dependencies

DEC-002, US-API-001, US-API-002.

## FR mapping

FR-004, FR-005.

## Test cases

TC-SHIFT-001, TC-SHIFT-002, TC-SHIFT-003, TC-SHIFT-004.

## DoD

- Cache invalidation on Odoo update verified.
- Pagination key stable across requests.
