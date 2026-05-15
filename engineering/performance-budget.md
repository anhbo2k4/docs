# Performance budget

**Status:** Active
**Owner:** Mobile Lead + Backend Lead
**Last updated:** 2026-05-13

Single page that consolidates every perf target in the blueprint. Stories must cite the budget IDs here instead of re-stating numbers.

## Mobile budgets

| ID | What | Budget | Measured by | NFR |
|---|---|---|---|---|
| PB-M-001 | Cold start (login visible) | ≤ 2.5 s p95 | TC-PERF-001 | NFR-001 |
| PB-M-002 | Warm start | ≤ 1.0 s p95 | TC-PERF-002 | NFR-002 |
| PB-M-003 | My Shifts list (100 items) render | ≤ 500 ms p95 | TC-PERF-003 | NFR-003 |
| PB-M-004 | Photo capture → preview | ≤ 300 ms p95 | TC-PERF-004 | NFR-004 |
| PB-M-005 | Photo compression (5 MP → ≤ 500 KB) | ≤ 800 ms p95 mid-tier | TC-PERF-005 | NFR-005 |
| PB-M-006 | Whisper transcription of 60 s audio | ≤ 25 s mid-tier Android | TC-VOICE-005 | NFR-006 |
| PB-M-007 | APK size (Android) | ≤ 80 MB | TC-PERF-010 | NFR-010 |
| PB-M-008 | IPA size (iOS) | ≤ 120 MB | TC-PERF-011 | NFR-011 |
| PB-M-009 | RSS during 1k-envelope soak | ≤ 250 MB peak | TC-PERF-012 | — |
| PB-M-010 | Slow-frame rate steady-state | ≤ 1 % | Sentry mobile dashboard | NFR-081 |

## Backend budgets

| ID | What | Budget | Measured by | NFR |
|---|---|---|---|---|
| PB-B-001 | `/v1/sync/envelope` p95 server time | ≤ 200 ms | TC-PERF-008 | NFR-008 |
| PB-B-002 | `/v1/sync/envelope` p99 server time | ≤ 500 ms | k6 (load) | derived |
| PB-B-003 | `/v1/shifts` p95 server time | ≤ 250 ms | k6 | NFR-002 (mobile-side) |
| PB-B-004 | Odoo write per envelope p95 | ≤ 2 s | TC-ODOO-009 | NFR-009 |
| PB-B-005 | Sustained throughput (pilot) | ≥ 5 000 envelopes / h | LP-001 | NFR-032 |
| PB-B-006 | Sustained throughput (year 1) | ≥ 50 000 envelopes / h | LP-002 | NFR-033 |
| PB-B-007 | Burst handling | 1 000 envelopes in 60 s w/o 5xx | LP-003 | NFR-031 |

## Network / payload budgets

| ID | What | Budget |
|---|---|---|
| PB-N-001 | Sync envelope POST p95 over 4G | ≤ 800 ms (NFR-007) |
| PB-N-002 | Single envelope payload (excl. media refs) | ≤ 8 KB |
| PB-N-003 | Photo upload size after compression | ≤ 500 KB |
| PB-N-004 | Voice recording bitrate | 32 kbps mono AAC (M4A) |

## Battery / disk budgets (mobile)

| ID | What | Budget |
|---|---|---|
| PB-D-001 | Local data warning threshold | 200 MB (US-OFF-006) |
| PB-D-002 | Local data block threshold | 500 MB (US-OFF-006) |
| PB-D-003 | Battery-aware non-essential pause | < 15 % on cellular pauses photos/audio (US-SYNC-001 AC11) |

## Verification cadence

- **Per PR:** unit benchmarks for any change touching capture/sync/photos/voice. CI fails on > 10 % regression vs main on the affected micro-bench.
- **Per sprint:** k6 smoke run against staging; results posted in sprint review.
- **S10 gate:** full TC-PERF-001..013, TC-SYNC-024 (idempotency storm), LP-001..008.
- **Pilot:** dashboards `Pilot Health` and `Backend Service Health` are watched; a PB violation for > 1 h on prod creates a `nfr-debt` story (NFR-073).

## When to bend a budget

A budget exception requires:

1. ADR or RFC explaining the new budget and trade-off.
2. Sponsor + EL sign-off if it widens cost or risk.
3. A `nfr-debt` backlog story tracking the path back to budget if it's a temporary exception.

## Mid-tier baseline reminder

- Android: Pixel 6a.
- iOS: iPhone 12.
- Low-end (informational only at MVP): Galaxy A13, iPhone SE 2.
