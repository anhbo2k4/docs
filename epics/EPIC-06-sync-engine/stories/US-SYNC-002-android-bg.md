---
id: US-SYNC-002
epic: EPIC-06
sprint: S7
fr: [FR-013]
priority: P0
estimate: M
status: Ready
owner: Mobile
---

# US-SYNC-002 — Background scheduling Android (WorkManager)

## Acceptance criteria

1. WorkManager periodic worker (interval 15 min) and event-driven worker on connectivity change.
2. Constraints: requires network, battery not low, storage not low.
3. Coexists with foreground drain without duplicate uploads.
4. Survives app kill and process death.
5. Verified on Xiaomi MIUI and Samsung One UI (RISK-062).

## Tasks

- [ ] WorkManager setup + constraints.
- [ ] Coordinated locking with foreground manager.
- [ ] OEM compatibility test plan.

## FR mapping

FR-013.

## Test cases

TC-SYNC-002, TC-SYNC-003.

## DoD

- 24-hour soak shows scheduled drain on both OEMs.
