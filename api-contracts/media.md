# Media API

**Status:** Active  
**Owner:** Backend Lead

Maps to FR-009 (photos), FR-011 (voice audio).

The mobile app uploads bytes directly to object storage via presigned URLs. Backend issues presigns, then finalizes uploads after the bytes land.

## Flow

```
1. Mobile compresses photo / records audio
2. Mobile asks FastAPI for presigned URL
3. Mobile PUTs bytes to S3
4. Mobile calls FastAPI to finalize (provides sha256, bytes, etc.)
5. Mobile includes media client_id in WORK_RESULT envelope
```

---

## 1. POST /v1/media/presign

Request a presigned PUT URL for one media object.

### Request

```json
{
  "client_id": "9b2c91e2-3d4f-4f5b-93f1-aabbccddeeff",
  "kind": "PHOTO",                 // 'PHOTO' | 'AUDIO'
  "mime_type": "image/jpeg",
  "expected_bytes": 412300,
  "sha256_hex": "...",
  "shift_id": 101
}
```

### Response (200)

```json
{
  "data": {
    "object_key": "media/2026/05/14/42/9b2c91e2.jpg",
    "upload_url": "https://s3.example.com/...",
    "method": "PUT",
    "headers": {
      "Content-Type": "image/jpeg",
      "x-amz-meta-sha256": "..."
    },
    "expires_in": 600
  }
}
```

### Constraints

- Expires in 10 minutes.
- One presign per `client_id`. Re-requesting with the same `client_id` returns the **same** URL (idempotent).
- Photo `expected_bytes` capped at 1 MB (we target ≤ 500 KB after compression).
- Audio `expected_bytes` capped at 5 MB (m4a, 60 s typical).

### Errors

| HTTP | Code |
|---|---|
| 400 | `INVALID_MIME` |
| 400 | `BYTES_TOO_LARGE` |
| 401 | `TOKEN_EXPIRED` |

---

## 2. PUT to S3

Mobile uploads bytes directly. On 2xx → step 3.

### Failure handling

- 5xx, network errors → exponential backoff up to 6 attempts.
- 4xx (signature mismatch, expired URL) → request a new presign.

---

## 3. POST /v1/media/finalize

Notifies backend that bytes have landed and were validated client-side.

### Request

```json
{
  "client_id": "9b2c91e2-3d4f-4f5b-93f1-aabbccddeeff",
  "object_key": "media/2026/05/14/42/9b2c91e2.jpg",
  "sha256_hex": "...",
  "bytes": 412300
}
```

### Response (200)

```json
{ "data": { "client_id": "9b2c91e2-...", "status": "UPLOADED" } }
```

### Server validation

- Confirms object exists in S3 with HEAD.
- Compares actual bytes vs declared.
- Optionally streams and verifies SHA-256 (sampled in MVP, full check in prod).
- Updates `media_uploads` to `UPLOADED`.

### Errors

| HTTP | Code |
|---|---|
| 404 | `OBJECT_NOT_FOUND` |
| 409 | `SHA256_MISMATCH` |
| 409 | `BYTES_MISMATCH` |
| 401 | `TOKEN_EXPIRED` |

---

## 4. Inclusion in envelope

The `WORK_RESULT` envelope references media by `client_id`:

```json
{
  "type": "WORK_RESULT",
  "client_id": "...",
  "shift_id": 101,
  "captured_at": "...",
  "payload": {
    "notes": "Completed AC unit 3 service.",
    "completion_pct": 100,
    "media_client_ids": [ "9b2c91e2-...", "a1b2c3d4-..." ],
    "voice_client_ids": [ "..." ]
  }
}
```

Server validates that every referenced `client_id` is in `UPLOADED` state for the same employee. If any is missing, server **rejects the envelope** with `MEDIA_NOT_FINALIZED` and lists offending IDs. Mobile then re-uploads and resubmits.

---

## Retention

Media bytes retained 24 months by default. Lifecycle policy in S3 transitions to cold storage at 12 months. Mobile retains local copy for 7 days after `CONFIRMED` (configurable, OQ-006).

## Privacy

- Photos: EXIF location stripped server-side unless explicitly required for the work result.
- Audio: never auto-uploaded for transcription; only uploaded as part of a deliberate work result submit.
- Object keys do not embed PII (use anonymous prefixes).

## Test cases

- TC-MEDIA-001 presign happy path
- TC-MEDIA-002 bytes mismatch
- TC-MEDIA-003 sha256 mismatch
- TC-MEDIA-004 expired presign
- TC-MEDIA-005 finalize without object
- TC-MEDIA-006 work_result rejected when media not finalized
