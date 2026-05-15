---
id: US-QA-002
epic: EPIC-09
sprint: S10
fr: []
priority: P0
estimate: M
status: Ready
owner: QA + Mobile Lead
---

# US-QA-002 — Perf budget verification

## Acceptance criteria

1. Each NFR-100..NFR-115 perf budget verified on the agreed device set.
2. Cold start ≤ 2.5 s p95 (TC-PERF-001).
3. `/v1/sync/envelope` server time ≤ 200 ms p95 (TC-PERF-008).
4. Photo compression p95 ≤ 800 ms on mid-tier (TC-PERF-005).
5. APK ≤ 80 MB, IPA ≤ 120 MB (TC-PERF-010, TC-PERF-011).
6. Results captured in `qa/pilot-gate.md`.

## Tasks

- [ ] Run perf scripts.
- [ ] Inspect bundle sizes.
- [ ] Profile cold start.

## DoD

- All perf rows green or rationalised with mitigation.
