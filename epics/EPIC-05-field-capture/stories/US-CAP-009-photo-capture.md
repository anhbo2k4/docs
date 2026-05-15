---
id: US-CAP-009
epic: EPIC-05
sprint: S5
fr: [FR-009]
priority: P0
estimate: L
status: Ready
owner: Mobile
---

# US-CAP-009 — Photo capture + compression pipeline

## Acceptance criteria

1. In-app camera launches via plugin; falls back to system camera if plugin unavailable.
2. Each captured photo is downscaled and re-encoded to JPEG with target ≤ 500 KB and max edge 1920 px.
3. Compression p95 ≤ 800 ms on the mid-tier device (TC-PERF-005).
4. Photos persist as `media_blob` rows (path, sha256, mime, size_bytes) and link to the parent capture.
5. Failure during compression yields a typed error; user can retry or keep the original (sized warning shown).
6. Deletion before submit is supported.

## Tasks

- [ ] Camera plugin wrapper under `plugins/camera/`.
- [ ] `MediaCompressionService`.
- [ ] Persistence + linking helpers.

## Dependencies

US-OFF-002, US-CAP-006.

## FR mapping

FR-009, NFR-103.

## Test cases

TC-MEDIA-001, TC-MEDIA-002, TC-PERF-005.

## DoD

- Compression budget hit on mid-device.
- Edge cases (rotated EXIF, HEIC source) handled.
