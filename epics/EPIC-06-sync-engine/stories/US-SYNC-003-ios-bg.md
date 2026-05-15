---
id: US-SYNC-003
epic: EPIC-06
sprint: S7
fr: [FR-013]
priority: P0
estimate: M
status: Ready
owner: Mobile
---

# US-SYNC-003 — Background scheduling iOS (BGTaskScheduler)

## Acceptance criteria

1. BGAppRefreshTask + BGProcessingTask registered with appropriate identifiers.
2. iOS quotas respected; UI never assumes background runtime > 30 s.
3. Coexistence with foreground manager (lock-aware).
4. Verified on iOS 14, 15, 16+ devices.

## Tasks

- [ ] Register tasks in `Info.plist`.
- [ ] Native bridge for scheduling.
- [ ] Quota fallback (defer to next foreground).

## FR mapping

FR-013.

## Test cases

TC-SYNC-002, TC-SYNC-003.

## DoD

- Soak shows opportunistic drain.
- Foreground catches up within 30 s after app open.
