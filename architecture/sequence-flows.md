# Sequence Flows

**Status:** Active  
**Owner:** Engineering Lead

Critical sequences described as Mermaid diagrams. Each links to the relevant FR, story, and test case.

## 1. Login + OTP verification

**Maps to:** FR-001, FR-002 · EPIC-02 · US-AUTH-001..005 · TC-AUTH-001..006

```mermaid
sequenceDiagram
    autonumber
    actor U as User
    participant A as Mobile App
    participant E as Supabase Edge
    participant T as Twilio
    participant FA as FastAPI
    participant OD as Odoo

    U->>A: enters phone +84xxx
    A->>E: POST /request_otp {phone}
    E->>E: rate-limit check
    E->>T: send SMS code
    T-->>U: SMS code 6 digits
    U->>A: enters code
    A->>E: POST /verify_otp {phone, code}
    E-->>A: 200 {supabase_jwt}
    A->>FA: POST /v1/auth/exchange<br/>Authorization: Bearer supabase_jwt
    FA->>FA: verify Supabase JWT (JWKS)
    FA->>OD: hr.employee.search_read by phone
    OD-->>FA: employee_id (or 404)
    alt employee found
        FA->>FA: issue internal JWT (15m) + refresh (30d)
        FA-->>A: 200 {access, refresh, employee}
        A->>A: store in flutter_secure_storage
    else not found
        FA-->>A: 403 EMPLOYEE_NOT_MAPPED
        A-->>U: shows "Contact admin" screen
    end
```

## 2. Shift list (read-with-cache)

**Maps to:** FR-004 · EPIC-03 · US-SHIFT-001 · TC-SHIFT-001

```mermaid
sequenceDiagram
    autonumber
    participant A as Mobile App
    participant FA as FastAPI
    participant OD as Odoo
    participant PG as PostgreSQL (cache)

    A->>FA: GET /v1/shifts?from=...&to=...
    FA->>PG: read cache (TTL 60s)
    alt cache hit
        PG-->>FA: rows
    else cache miss
        FA->>OD: search_read project.task / planning.slot
        OD-->>FA: shift rows
        FA->>PG: upsert cache rows
    end
    FA-->>A: 200 [shifts]
    A->>A: upsert into local SQLite (shifts table)
    A-->>A: render
```

## 3. PPE check-in (offline-tolerant)

**Maps to:** FR-006 · EPIC-05 · US-CAP-001 · TC-PPE-001

```mermaid
sequenceDiagram
    autonumber
    actor U as User
    participant A as Mobile App
    participant DB as SQLite
    participant Q as sync_queue
    participant FA as FastAPI

    U->>A: completes PPE checklist
    A->>DB: INSERT ppe_check (status=PENDING)
    A->>Q: ENQUEUE envelope {client_id, type=PPE_CHECK, payload}
    A-->>U: shows "Saved offline (PENDING)"
    note over Q,FA: when online + WorkManager fires
    Q->>FA: POST /v1/sync/envelope (idempotency-key=client_id)
    FA-->>Q: 202 ACCEPTED
    Q->>DB: UPDATE ppe_check SET sync_state=SYNCING
    note over FA: Celery processes → Odoo
    FA-->>Q: (poll) /v1/sync/status/{client_id} → CONFIRMED
    Q->>DB: UPDATE ppe_check SET sync_state=CONFIRMED
    A-->>U: badge updates to CONFIRMED
```

## 4. Work result with photos (capture + upload)

**Maps to:** FR-009, FR-010 · EPIC-05/06 · US-CAP-010..012 · TC-MEDIA-001..006

```mermaid
sequenceDiagram
    autonumber
    actor U as User
    participant A as Mobile App
    participant FS as Local FS
    participant DB as SQLite
    participant FA as FastAPI
    participant S3 as Object Storage

    U->>A: takes photo
    A->>A: compress to ≤ 500 KB
    A->>FS: write photo + checksum
    A->>DB: INSERT media (status=PENDING, local_path, sha256)
    U->>A: submits work result with N photos
    A->>DB: INSERT work_result (sync_state=PENDING)
    A->>DB: INSERT sync_queue rows: WORK_RESULT + N MEDIA_UPLOAD
    note over A,FA: when online
    A->>FA: POST /v1/media/presign {client_id, sha256, content_type}
    FA-->>A: {upload_url, object_key, expires_in}
    A->>S3: PUT photo bytes
    S3-->>A: 200
    A->>FA: POST /v1/media/finalize {client_id, object_key, sha256}
    FA-->>A: 200
    A->>FA: POST /v1/sync/envelope (WORK_RESULT with media_ids)
    FA-->>A: 202
```

## 5. Voice note + Whisper transcription

**Maps to:** FR-011 · EPIC-06 · US-VOICE-001..004 · TC-VOICE-001..005

```mermaid
sequenceDiagram
    autonumber
    actor U as User
    participant A as Mobile App
    participant W as Whisper (on-device)
    participant FS as Local FS
    participant DB as SQLite

    U->>A: presses Record
    A->>U: shows recording indicator + waveform
    U->>A: presses Stop
    A->>FS: write audio.m4a + checksum
    A->>DB: INSERT voice_note (status=TRANSCRIBING)
    A->>W: invoke transcription
    W-->>A: transcript text
    A->>DB: UPDATE voice_note SET transcript, status=PENDING
    A->>DB: INSERT sync_queue (VOICE_NOTE)
    A-->>U: shows transcript + audio playback
```

## 6. Background sync (resume from kill)

**Maps to:** FR-012, FR-013, FR-014 · EPIC-06 · US-SYNC-001..006 · TC-SYNC-001..010

```mermaid
sequenceDiagram
    autonumber
    participant OS as OS Scheduler
    participant W as Sync Worker
    participant DB as SQLite
    participant FA as FastAPI

    OS->>W: WorkManager / BGTask fires
    W->>DB: SELECT * FROM sync_queue WHERE state IN (PENDING,FAILED) ORDER BY created_at
    loop per row
        W->>DB: UPDATE state=SYNCING
        W->>FA: POST /v1/sync/envelope (Idempotency-Key=client_id)
        alt 2xx
            FA-->>W: 202
            W->>DB: UPDATE state=CONFIRMED
        else 4xx (validation)
            FA-->>W: 4xx error
            W->>DB: UPDATE state=DEAD_LETTER, last_error
        else 5xx or timeout
            FA-->>W: 5xx
            W->>DB: UPDATE state=FAILED, attempts+=1
            note over W: backoff: 30s, 2m, 10m, 30m, 2h, 6h (cap)
        end
    end
```

## 7. Token refresh on 401

**Maps to:** FR-003 · EPIC-02 · US-AUTH-006 · TC-AUTH-007

```mermaid
sequenceDiagram
    autonumber
    participant A as Mobile App
    participant FA as FastAPI

    A->>FA: GET /v1/shifts (access_token expired)
    FA-->>A: 401 TOKEN_EXPIRED
    A->>FA: POST /v1/auth/refresh (refresh_token)
    alt refresh valid
        FA-->>A: 200 {access, refresh}
        A->>FA: retry original request
        FA-->>A: 200
    else refresh invalid / revoked
        FA-->>A: 401 SESSION_EXPIRED
        A-->>A: clear secure storage, route to login
    end
```

## 8. Pilot rollback

**Maps to:** runbooks/rollback.md

```mermaid
sequenceDiagram
    autonumber
    actor PM as PM
    participant OPS as DevOps
    participant FA as FastAPI
    participant CW as Celery
    participant ST as Store

    PM->>OPS: rollback decision (per gate)
    OPS->>FA: deploy previous version
    OPS->>CW: deploy previous version
    OPS->>FA: feature flag SYNC_DRAIN_ONLY=true
    note over FA,CW: workers drain queue but reject new envelopes
    OPS->>ST: halt mobile rollout (stay at last stable)
    OPS->>OPS: verify metrics return to baseline
    OPS->>PM: confirm rollback complete
```
