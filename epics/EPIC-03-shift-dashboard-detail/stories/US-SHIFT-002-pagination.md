---
id: US-SHIFT-002
epic: EPIC-03
sprint: S3
fr: [FR-004]
priority: P0
estimate: M
status: Ready
owner: Mobile
---

# US-SHIFT-002 — Cursor pagination + pull-to-refresh

## Acceptance criteria

1. List supports cursor-based pagination using `next_cursor` returned by `/v1/shifts`.
2. Reaching the end loads the next page; loading row appears at the bottom while fetching.
3. Pull-to-refresh re-issues the first-page request and reconciles the cache.
4. Stale cache flagged with a small banner if last sync > 5 min.
5. Pagination consumer never duplicates rows on cursor reuse.

## Tasks

- [ ] Add cursor state to repository.
- [ ] Implement `RefreshIndicator` integration.
- [ ] Stale-banner widget driven by `last_synced_at`.
- [ ] Test: 60-row scenario across two pages.

## FR mapping

FR-004.

## Test cases

TC-SHIFT-002.

## DoD

- Two-page scenario verified.
- Pull-to-refresh works offline (returns cached, marks stale).
