---
id: US-QA-001
epic: EPIC-09
sprint: S10
fr: [FR-001..FR-015]
priority: P0
estimate: L
status: Ready
owner: QA
---

# US-QA-001 — Device matrix regression run

## Acceptance criteria

1. Full regression suite executed on every device row in `qa/device-matrix.md`.
2. Pass/fail/blocker recorded per row + per test area; blockers escalate to release call.
3. Defects logged with `repro`, `severity`, `screen recording or log`.
4. SEV-1 → release-blocker; SEV-2 → conditional; SEV-3 → allowed with workaround documented; SEV-4 → tracked post-pilot.

## Tasks

- [ ] Trigger automation suites for each device.
- [ ] Coordinate manual passes for OEM-specific paths.
- [ ] Compile results report.

## FR mapping

All MVP FRs.

## Test cases

All TC-* in `qa/test-cases.md`.

## DoD

- Report attached to release ticket.
- Pilot gate row "device matrix" green.
