# Entity-Relationship Diagrams

**Status:** Active
**Owner:** Backend Lead + Mobile Lead

Visual references for the data models. Source of truth for column-level detail is `data/sqlite-schema.md`, `data/postgres-schema.md`, and `data/odoo-mapping.md`.

## Mobile (SQLite)

```mermaid
erDiagram
    EMPLOYEE ||--o{ SHIFT : "assigned_to"
    SHIFT ||--o{ PPE_CHECK : "has"
    SHIFT ||--o{ FORM_RESPONSE : "has"
    SHIFT ||--o{ GPS_EVENT : "has"
    SHIFT ||--o{ WORK_RESULT : "has"
    SHIFT ||--o{ VOICE_NOTE : "has"
    WORK_RESULT ||--o{ MEDIA : "references"
    WORK_RESULT ||--o{ VOICE_NOTE : "references"
    SYNC_QUEUE ||--|| PPE_CHECK : "tracks"
    SYNC_QUEUE ||--|| FORM_RESPONSE : "tracks"
    SYNC_QUEUE ||--|| GPS_EVENT : "tracks"
    SYNC_QUEUE ||--|| WORK_RESULT : "tracks"
    SYNC_QUEUE ||--|| MEDIA : "tracks"
    SYNC_QUEUE ||--|| VOICE_NOTE : "tracks"
    SYNC_QUEUE ||--o{ SYNC_ATTEMPT : "history"

    EMPLOYEE {
      int id PK
      string name
      string phone
    }
    SHIFT {
      int id PK
      int employee_id FK
      datetime start_at
      datetime end_at
      string status
      string sync_state
    }
    PPE_CHECK {
      uuid client_id PK
      int shift_id FK
      json items
      string notes
      datetime captured_at
      string sync_state
    }
    FORM_RESPONSE {
      uuid client_id PK
      int shift_id FK
      string form_type
      int schema_version
      json answers
      string sync_state
    }
    GPS_EVENT {
      uuid client_id PK
      int shift_id FK
      string event_type
      double latitude
      double longitude
      double accuracy_m
      datetime captured_at
      string sync_state
    }
    WORK_RESULT {
      uuid client_id PK
      int shift_id FK
      string notes
      string sync_state
    }
    MEDIA {
      uuid client_id PK
      int shift_id FK
      string kind
      string path
      string sha256
      int bytes
      string sync_state
    }
    VOICE_NOTE {
      uuid client_id PK
      int shift_id FK
      uuid media_client_id FK
      string transcript
      int duration_ms
      string sync_state
    }
    SYNC_QUEUE {
      uuid client_id PK
      string envelope_type
      int attempt_count
      string state
      datetime next_attempt_at
    }
    SYNC_ATTEMPT {
      int id PK
      uuid client_id FK
      datetime attempted_at
      string outcome
      string error_code
    }
```

## Backend (Postgres)

```mermaid
erDiagram
    EMPLOYEE_SESSION ||--o{ REFRESH_TOKEN : "has"
    EMPLOYEE_SESSION ||--o{ DEVICE : "binds"
    EMPLOYEE_SESSION ||--o{ SYNC_ENVELOPE : "owns"
    SYNC_ENVELOPE ||--o{ SYNC_ENVELOPE_ATTEMPT : "history"
    SYNC_ENVELOPE ||--o| MEDIA_OBJECT : "references"

    EMPLOYEE_SESSION {
      uuid id PK
      int employee_id
      string status
      datetime created_at
      datetime last_seen_at
    }
    REFRESH_TOKEN {
      uuid id PK
      uuid session_id FK
      string family_id
      datetime issued_at
      datetime rotated_at
      datetime revoked_at
      string revoked_reason
    }
    DEVICE {
      uuid id PK
      uuid session_id FK
      string platform
      string device_name
      string app_version
    }
    SYNC_ENVELOPE {
      uuid client_id PK
      int employee_id
      string type
      int shift_id
      json payload
      int schema_version
      string status
      string error_code
      string odoo_ref
      datetime received_at
      datetime processed_at
    }
    SYNC_ENVELOPE_ATTEMPT {
      bigserial id PK
      uuid client_id FK
      datetime attempted_at
      string outcome
      string error_code
      int duration_ms
    }
    MEDIA_OBJECT {
      uuid client_id PK
      int employee_id
      string kind
      int bytes
      string sha256
      string s3_key
      string status
      string odoo_attachment_ref
    }
```

## Sync state machine (visual reference)

```mermaid
stateDiagram-v2
    [*] --> PENDING
    PENDING --> SYNCING: begin attempt
    SYNCING --> CONFIRMED: 2xx + Odoo wrote
    SYNCING --> FAILED: 5xx / network
    SYNCING --> DEAD_LETTER: 4xx validation
    FAILED --> PENDING: backoff elapsed
    DEAD_LETTER --> [*]: support intervention or wipe-on-logout
    CONFIRMED --> [*]
```

## How to keep these in sync

- Any column added in `sqlite-schema.md` or `postgres-schema.md` must update the matching ERD entity here.
- Lint reviews check these diagrams compile via `mermaid-cli` in CI.
- ERDs are illustrative; the schema files are normative when conflicts occur.
