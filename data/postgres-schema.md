# Backend Database Schema (PostgreSQL)

**Status:** Active  
**Owner:** Backend Lead

FastAPI-side persistence. **This is not a copy of Odoo.** Only stores: sessions, sync envelopes, audit, and a small read cache. Migrations via Alembic.

## Tables

### `sessions`

```sql
CREATE TABLE sessions (
    id              BIGSERIAL PRIMARY KEY,
    employee_id     INTEGER NOT NULL,
    device_id       TEXT NOT NULL,
    refresh_jti     TEXT NOT NULL UNIQUE,
    revoked         BOOLEAN NOT NULL DEFAULT FALSE,
    issued_at       TIMESTAMPTZ NOT NULL DEFAULT now(),
    expires_at      TIMESTAMPTZ NOT NULL,
    last_seen_at    TIMESTAMPTZ
);
CREATE INDEX idx_sessions_employee ON sessions(employee_id);
CREATE INDEX idx_sessions_device ON sessions(device_id);
```

### `sync_envelopes`

```sql
CREATE TYPE envelope_state AS ENUM ('PENDING','PROCESSING','CONFIRMED','DEAD_LETTER');
CREATE TYPE envelope_type  AS ENUM ('PPE_CHECK','FORM_RESPONSE','GPS_EVENT','WORK_RESULT','MEDIA_FINALIZE','VOICE_NOTE');

CREATE TABLE sync_envelopes (
    id              BIGSERIAL PRIMARY KEY,
    employee_id     INTEGER NOT NULL,
    client_id       UUID NOT NULL,
    type            envelope_type NOT NULL,
    payload         JSONB NOT NULL,
    state           envelope_state NOT NULL DEFAULT 'PENDING',
    odoo_ref        TEXT,
    error_code      TEXT,
    error_message   TEXT,
    received_at     TIMESTAMPTZ NOT NULL DEFAULT now(),
    processed_at    TIMESTAMPTZ,
    attempts        INTEGER NOT NULL DEFAULT 0,
    UNIQUE (employee_id, client_id)
);
CREATE INDEX idx_envelopes_state ON sync_envelopes(state);
CREATE INDEX idx_envelopes_received ON sync_envelopes(received_at);
```

The `(employee_id, client_id)` unique constraint is the deduplication mechanism.

### `media_uploads`

```sql
CREATE TABLE media_uploads (
    id              BIGSERIAL PRIMARY KEY,
    employee_id     INTEGER NOT NULL,
    client_id       UUID NOT NULL,
    object_key      TEXT NOT NULL UNIQUE,
    sha256          TEXT NOT NULL,
    bytes           BIGINT,
    mime_type       TEXT,
    state           TEXT NOT NULL DEFAULT 'PRESIGNED',  -- PRESIGNED | UPLOADED | FAILED | EXPIRED
    presigned_at    TIMESTAMPTZ NOT NULL DEFAULT now(),
    finalized_at    TIMESTAMPTZ,
    UNIQUE (employee_id, client_id)
);
CREATE INDEX idx_media_state ON media_uploads(state);
```

### `shift_cache`

```sql
CREATE TABLE shift_cache (
    employee_id     INTEGER NOT NULL,
    odoo_id         INTEGER NOT NULL,
    payload         JSONB NOT NULL,
    fetched_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    PRIMARY KEY (employee_id, odoo_id)
);
CREATE INDEX idx_shift_cache_fetched ON shift_cache(fetched_at);
```

TTL by `fetched_at` (60 s default). Cleared by a periodic Celery beat task.

### `audit_log`

```sql
CREATE TABLE audit_log (
    id              BIGSERIAL PRIMARY KEY,
    occurred_at     TIMESTAMPTZ NOT NULL DEFAULT now(),
    actor_type      TEXT NOT NULL,                  -- 'employee' | 'system' | 'admin'
    actor_id        TEXT,
    action          TEXT NOT NULL,
    target_type     TEXT,
    target_id       TEXT,
    request_id      TEXT,
    ip              INET,
    metadata        JSONB
);
CREATE INDEX idx_audit_occurred ON audit_log(occurred_at);
CREATE INDEX idx_audit_actor ON audit_log(actor_type, actor_id);
```

Retained for 7 years (NFR-055).

### `dead_letters`

```sql
CREATE TABLE dead_letters (
    id              BIGSERIAL PRIMARY KEY,
    envelope_id     BIGINT REFERENCES sync_envelopes(id),
    parked_at       TIMESTAMPTZ NOT NULL DEFAULT now(),
    reason          TEXT NOT NULL,
    operator_notes  TEXT,
    resolved_at     TIMESTAMPTZ,
    resolved_by     TEXT
);
CREATE INDEX idx_dl_parked ON dead_letters(parked_at);
```

## Migrations

Each migration is an Alembic revision. Naming: `YYYYMMDD_HHMM_short_desc.py`. CI rejects PRs that modify `models.py` without a matching revision file.

Down migrations are best-effort but mandatory; they exist to enable local rollback during development.

## Indexes plan

- `sync_envelopes(received_at)` for ops dashboards.
- `sync_envelopes(state) WHERE state IN ('PENDING','PROCESSING')` partial for queue depth queries.
- `media_uploads(state) WHERE state = 'PRESIGNED'` partial for stale-presign cleanup.

## Retention

| Table | Retention | Cleanup mechanism |
|---|---|---|
| `sessions` | 90 days past expiry | Daily Celery beat |
| `sync_envelopes` | 24 months | Cold-storage export, then delete |
| `media_uploads` | 24 months | Match `sync_envelopes` |
| `shift_cache` | 60 s TTL | Celery beat sweep every 5 min |
| `audit_log` | 7 years | Cold storage after 1 year |
| `dead_letters` | 12 months | Manual archive |
