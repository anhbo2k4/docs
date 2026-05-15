# Sync API

**Status:** Active  
**Owner:** Backend Lead

Maps to FR-012, FR-013, FR-014, FR-015. ADR-007.

The single ingress for offline-captured work. **All envelopes are idempotent by `client_id`.**

## 1. POST /v1/sync/envelope

Submit one envelope. Server persists and enqueues to Celery for Odoo write.

### Request

```http
POST /v1/sync/envelope
Authorization: Bearer <access>
Idempotency-Key: 7e5ab0bc-3e5f-4a31-b8f1-c2d8b9b5b7a4
Content-Type: application/json
```

```json
{
  "client_id": "7e5ab0bc-3e5f-4a31-b8f1-c2d8b9b5b7a4",
  "type": "PPE_CHECK",
  "shift_id": 101,
  "captured_at": "2026-05-14T07:55:12Z",
  "device_id": "...",
  "schema_version": 1,
  "payload": {
    "items": [
      { "key": "helmet",  "label": "Helmet", "checked": true  },
      { "key": "gloves",  "label": "Gloves", "checked": true  },
      { "key": "harness", "label": "Harness","checked": false }
    ],
    "notes": "Harness damaged, replacement requested."
  }
}
```

### Envelope types

| `type` | `payload` schema |
|---|---|
| `PPE_CHECK` | `items[]`, `notes` |
| `FORM_RESPONSE` | `form_type`, `schema_version`, `answers{}` |
| `GPS_EVENT` | `event_type`, `latitude`, `longitude`, `accuracy_m`, `altitude_m?` |
| `WORK_RESULT` | `notes`, `completion_pct?`, `media_client_ids[]`, `voice_client_ids[]` |
| `MEDIA_FINALIZE` | (handled by media API; envelope only references) |
| `VOICE_NOTE` | `transcript`, `duration_ms`, `media_client_id` (the audio) |

### Response (202 Accepted, first time)

```json
{
  "data": {
    "client_id": "7e5ab0bc-...",
    "status": "PENDING"
  }
}
```

### Response (200, duplicate or already known)

```json
{
  "data": {
    "client_id": "7e5ab0bc-...",
    "status": "CONFIRMED",
    "odoo_ref": "field_mobile_sync.ppe_check,98"
  }
}
```

### Errors

| HTTP | Code | When |
|---|---|---|
| 400 | `MISSING_CLIENT_ID` | header / body missing or mismatched |
| 400 | `INVALID_ENVELOPE` | schema validation failed |
| 401 | `TOKEN_EXPIRED` | refresh and retry |
| 403 | `NOT_ASSIGNED` | shift_id not assigned to this employee |
| 409 | `CLIENT_ID_CONFLICT` | same client_id seen with different payload |
| 422 | `INVALID_SHIFT_STATE` | e.g. submitting PPE for a COMPLETED shift |
| 429 | `RATE_LIMIT` | |
| 503 | `ODOO_UNAVAILABLE` | will accept and enqueue regardless; 503 only if write to Postgres fails |

`409 CLIENT_ID_CONFLICT` is rare; it happens when mobile reuses a `client_id` after editing a captured row. Mobile must treat it as a programming error and surface to Sentry.

---

## 2. GET /v1/sync/status/{client_id}

Poll status for a previously submitted envelope.

### Response (200)

```json
{
  "data": {
    "client_id": "7e5ab0bc-...",
    "status": "CONFIRMED",
    "odoo_ref": "field_mobile_sync.ppe_check,98",
    "received_at": "2026-05-14T07:55:13Z",
    "processed_at": "2026-05-14T07:55:14Z"
  }
}
```

`status` ∈ `PENDING | PROCESSING | CONFIRMED | DEAD_LETTER`.

For `DEAD_LETTER`:

```json
{
  "data": {
    "client_id": "7e5ab0bc-...",
    "status": "DEAD_LETTER",
    "error_code": "VALIDATION_FAILED",
    "error_message": "Field 'notes' exceeds 4000 chars",
    "received_at": "...",
    "processed_at": "..."
  }
}
```

### Errors

| HTTP | Code |
|---|---|
| 404 | `ENVELOPE_NOT_FOUND` |
| 401 | `TOKEN_EXPIRED` |

---

## 3. POST /v1/sync/batch (optional, post-MVP)

Submit up to 25 envelopes in one call. Each entry follows the single-envelope schema. Server processes in order; partial success allowed.

```json
{ "envelopes": [ { ... }, { ... } ] }
```

```json
{ "data": { "results": [ { "client_id": "...", "status": "PENDING" }, ... ] } }
```

Disabled in MVP unless explicit need surfaces. Deferred to EPIC-10.

---

## Idempotency contract (recap)

- Mobile sends `client_id` in body **and** header `Idempotency-Key`. They must match.
- Server unique key: `(employee_id, client_id)`.
- Same `client_id` with same payload → return current status.
- Same `client_id` with different payload → 409.
- Server stores envelope before enqueueing; Celery is the only writer to Odoo.

## Test cases

- TC-SYNC-001 PENDING after first POST
- TC-SYNC-002 idempotent on retry (same client_id)
- TC-SYNC-003 conflict on different payload (409)
- TC-SYNC-004 DEAD_LETTER on validation
- TC-SYNC-005 status poll happy path
- TC-SYNC-006..010 envelope-type-specific
- TC-SYNC-024 retry storm dedup
