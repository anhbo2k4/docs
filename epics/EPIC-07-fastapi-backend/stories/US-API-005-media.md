---
id: US-API-005
epic: EPIC-07
sprint: S8
fr: [FR-009, FR-010, FR-011]
priority: P0
estimate: M
status: Ready
owner: Backend
---

# US-API-005 — `/v1/media/upload-url` + finalize + EXIF check

## Acceptance criteria

1. `POST /v1/media/upload-url` returns a short-lived signed URL for direct PUT to object storage.
2. `POST /v1/media/finalize` records the blob with metadata: `client_id`, `mime`, `size_bytes`, `sha256`.
3. Finalize verifies `sha256` against object storage HEAD response; mismatch → 400 `MEDIA_CHECKSUM_FAILED`.
4. Server-side EXIF strip if any non-empty EXIF detected (defense-in-depth; mobile already strips).
5. Max file sizes per type enforced (image 10 MB, audio 25 MB).
6. Pre-signed URL TTL ≤ 5 min.

## Tasks

- [ ] Endpoint handlers.
- [ ] Signed URL provider abstraction (DEC-007).
- [ ] EXIF strip post-finalize (best-effort).
- [ ] Tests for size cap, checksum, expiration.

## Dependencies

DEC-007.

## FR mapping

FR-009, FR-010, FR-011.

## Test cases

TC-MEDIA-001..006, TC-SEC-007.

## DoD

- Signed URL flow verified end-to-end.
- Checksum mismatch path tested.
