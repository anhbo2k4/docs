---
id: US-AUTH-005
epic: EPIC-02
sprint: S2
fr: [FR-002]
priority: P0
estimate: M
status: Ready
owner: Backend
---

# US-AUTH-005 — Map phone to hr.employee

## User story

**As** the system
**I want** to resolve a verified phone to exactly one `hr.employee`
**so that** every subsequent API call is scoped to the right person.

## Acceptance criteria

1. Implementation respects DEC-006: phone field on `hr.employee` is read in priority order configured by env (`work_phone`, `mobile_phone`, custom field).
2. Lookup is normalised to E.164 on both sides.
3. Zero matches → 403 `EMPLOYEE_NOT_MAPPED`. Multiple matches → 409 `MAPPING_AMBIGUOUS` with audit log entry. Single match → returns `employee_id`.
4. Audit row written to `auth_sessions` with `employee_id`, `device_id`, `created_at`.
5. Mapping cache TTL ≤ 30 s to balance Odoo load and stale data.
6. Operational error path: if Odoo is unavailable, return 503 `ERP_UNAVAILABLE` with request id.

## Tasks

### Backend
- [ ] `MappingService` with provider per DEC-006.
- [ ] Redis cache with TTL.
- [ ] Audit insert.
- [ ] Metrics: `auth.mapping.hits`, `auth.mapping.ambiguous`, `auth.mapping.misses`.

### Testing
- [ ] Unit tests for each match branch.
- [ ] Contract test against Odoo mock.
- [ ] Soak test for cache stampede.

## Dependencies

- DEC-006.
- Odoo read access to `hr.employee`.

## Risks

- RISK-021 Mapping ambiguity if shared phone numbers exist.

## FR mapping

FR-002.

## Test cases

TC-AUTH-006.

## Definition of Done

- TC-AUTH-006 green in staging against Odoo mirror.
- Ambiguous case alerts to ops within 1 minute.
- 99th percentile mapping latency ≤ 200 ms.
