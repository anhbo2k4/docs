# NFR Register

**Status:** Active  
**Owner:** Engineering Lead + QA Lead

Tracks the status of every non-functional requirement defined in `architecture/non-functional-requirements.md`. Updated each sprint review.

## Status legend

- `Not started` — no implementation yet
- `In progress` — partial coverage
- `Met` — measured and passing
- `At risk` — measured but trending below target
- `Failed` — measured below target, mitigation needed
- `Waived` — accepted with documented reason

## Performance

| ID | Target | Status | Sprint introduced | Last measured | Notes |
|---|---|---|---|---|---|
| NFR-001 | Cold start ≤ 2.5 s p95 | Not started | S1 | — | Measure on Pixel 6a baseline |
| NFR-002 | Warm start ≤ 1.0 s p95 | Not started | S1 | — | |
| NFR-003 | Shift list 100 items ≤ 500 ms | Not started | S3 | — | |
| NFR-004 | Photo capture preview ≤ 300 ms | Not started | S5 | — | |
| NFR-005 | Photo compression ≤ 800 ms p95 mid | Not started | S6 | — | |
| NFR-006 | Whisper 60 s ≤ 25 s mid | Not started | S6 | — | |
| NFR-007 | Sync POST ≤ 800 ms p95 over 4G | Not started | S7 | — | |
| NFR-008 | `/sync/envelope` ≤ 200 ms p95 | Not started | S8 | — | |
| NFR-009 | Odoo write ≤ 2 s p95 | Not started | S9 | — | |
| NFR-010 | Android APK ≤ 80 MB | Not started | S6 | — | Risk RISK-007 |
| NFR-011 | iOS IPA ≤ 120 MB | Not started | S6 | — | |

## Reliability

| ID | Target | Status | Sprint | Notes |
|---|---|---|---|---|
| NFR-020 | 99.5 % monthly uptime | Not started | S10 | Synthetic check |
| NFR-021 | 0 lost envelopes / 10k | Not started | S7 | TC-OFF-021 |
| NFR-022 | ≥ 99 % sync success after 24 h | Not started | S7 | |
| NFR-023 | ≥ 99.5 % crash-free | Not started | S6 | Sentry |
| NFR-024 | 100 % idempotent retry safety | Not started | S7 | TC-SYNC-024 |
| NFR-025 | ≥ 72 h offline | Not started | S7 | TC-OFF-002 |
| NFR-026 | DB integrity after force-kill | Not started | S4 | TC-OFF-003 |

## Security

| ID | Target | Status | Sprint | Notes |
|---|---|---|---|---|
| NFR-040 | TLS 1.2+ enforced | In progress | S1 | Config in deployment |
| NFR-041 | Tokens at rest in Keychain/Keystore | Not started | S1 | flutter_secure_storage |
| NFR-042 | OTP rate limit | Not started | S2 | TC-AUTH-041 |
| NFR-043 | JWT lifetimes | Not started | S2 | |
| NFR-044 | Local DB encryption | Not started | S1 | |
| NFR-045 | PII minimization in logs | Not started | S1 | TC-SEC-005 |
| NFR-046 | Secret scanning in CI | Not started | S1 | gitleaks |
| NFR-047 | Dep vuln SLA | Not started | S1 | Renovate config |
| NFR-048 | Sentry scrubber | Not started | S6 | TC-SEC-006 |

## Privacy

| ID | Target | Status | Sprint | Notes |
|---|---|---|---|---|
| NFR-050 | GPS per-action consent | Not started | S5 | TC-GPS-002 |
| NFR-051 | Visible recording indicator | Not started | S6 | |
| NFR-052 | Photo per-action consent + EXIF strip | Not started | S6 | TC-SEC-007 |
| NFR-053 | Right to erasure ≤ 30 days | Not started | S10 | data-deletion runbook |
| NFR-054 | Media retention 24 months | Not started | S10 | S3 lifecycle policy |
| NFR-055 | Audit log 7 years | Not started | S10 | |

## Usability / Accessibility

| ID | Target | Status | Sprint | Notes |
|---|---|---|---|---|
| NFR-060 | WCAG 2.1 AA automated checks | Not started | S10 | Manual review at S10 |
| NFR-061 | 200 % font scale | Not started | S5 | |
| NFR-062 | Color contrast | Not started | S1 | |
| NFR-063 | EN + VI localization | Not started | S2 | |
| NFR-064 | Sync state visibility | Not started | S2 | |
| NFR-065 | Tap targets ≥ 44 pt | Not started | S1 | |

## Maintainability

| ID | Target | Status | Sprint | Notes |
|---|---|---|---|---|
| NFR-070 | Mobile coverage ≥ 70 % | Not started | S1 | |
| NFR-071 | Backend coverage ≥ 80 % | Not started | S1 | |
| NFR-072 | Static analysis clean | Not started | S1 | |
| NFR-073 | API versioned | In progress | S1 | `/v1/` prefix |
| NFR-074 | Sprint completion ≥ 80 % | Not started | S2 | Track in sprint review |

## Observability

| ID | Target | Status | Sprint | Notes |
|---|---|---|---|---|
| NFR-080 | Backend JSON logs | Not started | S1 | structlog |
| NFR-081 | Mobile crash reports | Not started | S6 | Sentry |
| NFR-082 | Backend RED + USE | Not started | S8 | Prometheus |
| NFR-083 | Distributed tracing | Not started | S8 | OpenTelemetry |
| NFR-084 | Sync queue dashboard | Not started | S8 | Grafana |
| NFR-085 | Alerting | Not started | S10 | PagerDuty |

## Compliance

| ID | Target | Status | Sprint | Notes |
|---|---|---|---|---|
| NFR-090 | Privacy policy | Not started | S10 | |
| NFR-091 | TOS | Not started | S10 | |
| NFR-092 | Data residency | Not started | S0 | DEC-008 |
| NFR-093 | Right to be forgotten procedure | Not started | S10 | |

## Operational

| ID | Target | Status | Sprint | Notes |
|---|---|---|---|---|
| NFR-100 | Backend deploy ≤ 10 min | Not started | S1 | |
| NFR-101 | Backend rollback ≤ 5 min | Not started | S10 | |
| NFR-102 | Mobile phased rollout | Not started | S10 | |
| NFR-103 | DB migration safety | Not started | S1 | |
| NFR-104 | Postgres backups hourly | Not started | S8 | |

## Update procedure

- Engineer updates the row when implementing or measuring.
- QA confirms at sprint review.
- PM aggregates "At risk" / "Failed" rows for risk register.
- Pilot Gate references this register; any "Failed" without waiver blocks pilot.
