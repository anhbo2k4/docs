---
id: US-CAP-011
epic: EPIC-05
sprint: S5
fr: [FR-009]
priority: P0
estimate: M
status: Ready
owner: Mobile
---

# US-CAP-011 — Photo gallery review/delete

## Acceptance criteria

1. Gallery view per shift shows thumbnails grouped by capture (PPE, work result).
2. Tap opens full-screen with pinch-zoom.
3. Delete pre-submit removes the row and the file; post-submit asks for confirmation and writes a deletion-pending envelope (handled in EPIC-06).
4. Empty state and offline behaviour verified.

## Tasks

- [ ] `MediaGalleryScreen`.
- [ ] Thumbnail cache (in-memory + disk LRU).
- [ ] Delete flows.

## FR mapping

FR-009.

## Test cases

TC-MEDIA-004, TC-MEDIA-006.

## DoD

- Gallery interactions verified.
- Delete pre/post submit paths covered.
