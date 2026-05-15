---
id: US-CAP-010
epic: EPIC-05
sprint: S5
fr: [FR-009]
priority: P0
estimate: M
status: Ready
owner: Mobile + Security
---

# US-CAP-010 — EXIF strip + checksum

## Acceptance criteria

1. EXIF metadata removed before persistence (orientation tag rebuilt explicitly, GPS tags stripped unconditionally).
2. SHA-256 checksum computed on the persisted bytes and stored on `media_blob`.
3. Backend verifies checksum on `/v1/media/finalize`; mismatch returns 400 `MEDIA_CHECKSUM_FAILED`.
4. Test corpus of 10 sample photos verifies EXIF removal.

## Tasks

- [ ] `ExifSanitiser` utility.
- [ ] Hashing helper.
- [ ] CI test fixture corpus.

## FR mapping

FR-009, NFR-040.

## Test cases

TC-SEC-007, TC-MEDIA-003.

## DoD

- All fixtures pass; no GPS or device-id tags survive.
