# Shifts API

**Status:** Active  
**Owner:** Backend Lead

Maps to FR-004, FR-005.

## 1. GET /v1/shifts

List shifts assigned to the authenticated employee.

### Request

```http
GET /v1/shifts?from=2026-05-13T00:00:00Z&to=2026-05-20T00:00:00Z&status=in_progress,scheduled
Authorization: Bearer <access>
```

| Query | Type | Required | Default |
|---|---|---|---|
| `from` | ISO-8601 | no | now - 1 day |
| `to` | ISO-8601 | no | now + 14 days |
| `status` | comma-list | no | all |
| `cursor` | string | no | — |
| `limit` | int (1..100) | no | 50 |

### Response (200)

```json
{
  "data": {
    "items": [
      {
        "id": 101,
        "title": "AC Maintenance — Building 5",
        "site_name": "FPT Tower",
        "site_address": "...",
        "project_id": 17,
        "project_name": "Q2 HVAC",
        "start_at": "2026-05-14T08:00:00Z",
        "end_at":   "2026-05-14T12:00:00Z",
        "status": "SCHEDULED"
      }
    ],
    "next_cursor": null
  }
}
```

### Caching

Server-side: Postgres cache, TTL 60 s. Mobile may use a longer client cache (5 min) and revalidate.

### Errors

| HTTP | Code |
|---|---|
| 401 | `TOKEN_EXPIRED` |
| 503 | `ODOO_UNAVAILABLE` |

---

## 2. GET /v1/shifts/{id}

Detail for a single shift, including any prior captures already CONFIRMED in Odoo.

### Response (200)

```json
{
  "data": {
    "id": 101,
    "title": "AC Maintenance — Building 5",
    "site_name": "FPT Tower",
    "site_address": "...",
    "site_geo": { "lat": 21.028511, "lon": 105.804817 },
    "instructions": "Use entrance B; coordinate with security.",
    "project_id": 17,
    "project_name": "Q2 HVAC",
    "start_at": "2026-05-14T08:00:00Z",
    "end_at":   "2026-05-14T12:00:00Z",
    "actual_start_at": null,
    "actual_end_at":   null,
    "status": "SCHEDULED",
    "submissions": {
      "ppe_check":  { "status": "NONE" },
      "pre_shift":  { "status": "NONE" },
      "work_result":{ "status": "NONE" },
      "post_shift": { "status": "NONE" }
    },
    "team": [
      { "id": 42, "name": "Nguyen Van A", "role": "lead" },
      { "id": 51, "name": "Tran Thi B",   "role": "member" }
    ]
  }
}
```

`submissions.*.status` is one of `NONE`, `CONFIRMED`. Mobile compares with its own local sync_state to render the badge.

### Errors

| HTTP | Code |
|---|---|
| 404 | `SHIFT_NOT_FOUND` |
| 403 | `NOT_ASSIGNED` |

## Tests

- TC-SHIFT-001 list happy path
- TC-SHIFT-002 list pagination
- TC-SHIFT-003 detail happy path
- TC-SHIFT-004 not assigned
