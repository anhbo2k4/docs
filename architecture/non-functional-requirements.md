# Non-Functional Requirements (NFR)

**Status:** Active  
**Owner:** Engineering Lead  
**Last updated:** 2026-05-13

NFRs are committed targets. Each is testable. Each maps to an `NFR-NNN` ID and to test cases in `qa/test-cases/`.

## 1. Performance

| ID | Requirement | Target | Measurement |
|---|---|---|---|
| NFR-001 | Cold start (login screen visible) | ≤ 2.5 s p95 | TC-PERF-001 |
| NFR-002 | Warm start | ≤ 1.0 s p95 | TC-PERF-002 |
| NFR-003 | My Shifts list render (100 items) | ≤ 500 ms p95 | TC-PERF-003 |
| NFR-004 | Photo capture → preview | ≤ 300 ms p95 | TC-PERF-004 |
| NFR-005 | Photo compression (5 MP → ≤ 500 KB) | ≤ 800 ms p95 on mid-range device | TC-PERF-005 |
| NFR-006 | Whisper transcription of 60 s audio | ≤ 25 s on mid-range Android | TC-VOICE-005 |
| NFR-007 | Sync envelope POST (single) | ≤ 800 ms p95 over 4G | TC-SYNC-007 |
| NFR-008 | Backend `/sync/envelope` server time | ≤ 200 ms p95 | TC-PERF-008 |
| NFR-009 | Odoo write per envelope | ≤ 2 s p95 | TC-ODOO-009 |
| NFR-010 | App APK size (Android) | ≤ 80 MB | TC-PERF-010 |
| NFR-011 | iOS IPA size | ≤ 120 MB | TC-PERF-011 |

Mid-range device baseline: Pixel 6a, iPhone 12. Define "low-end" as Galaxy A13, iPhone SE 2.

## 2. Reliability

| ID | Requirement | Target | Measurement |
|---|---|---|---|
| NFR-020 | Backend uptime (pilot) | 99.5 % monthly | Synthetic check, Prometheus |
| NFR-021 | Sync envelope durability | 0 lost envelopes per 10 000 | TC-OFF-021 |
| NFR-022 | Sync success rate after 24 h | ≥ 99 % | Logs + metrics |
| NFR-023 | App crash-free sessions | ≥ 99.5 % | Sentry |
| NFR-024 | Idempotent retry safety | 100 % (no duplicates in Odoo) | TC-SYNC-024 |
| NFR-025 | Offline duration supported | ≥ 72 h continuous capture | TC-OFF-025 |
| NFR-026 | Local DB integrity after force-kill | 100 % (no corruption) | TC-OFF-026 |

## 3. Scalability

| ID | Requirement | Target | Notes |
|---|---|---|---|
| NFR-030 | Concurrent active users (pilot) | 50 | |
| NFR-031 | Concurrent active users (prod year-1) | 1 000 | Horizontal scaling design |
| NFR-032 | Sync envelopes / hour (pilot) | 5 000 | |
| NFR-033 | Sync envelopes / hour (prod year-1) | 50 000 | |
| NFR-034 | Object storage growth | ≤ 100 GB / month at year-1 | Lifecycle policy after 12 months |

## 4. Security

| ID | Requirement | Target | Reference |
|---|---|---|---|
| NFR-040 | TLS for all external traffic | TLS 1.2+ enforced | security/secrets.md |
| NFR-041 | Tokens at rest | iOS Keychain / Android Keystore via flutter_secure_storage | ADR-001 |
| NFR-042 | OTP rate limiting | 3 requests / phone / 5 min, lockout after 5 fails | TC-AUTH-041 |
| NFR-043 | JWT lifetime | Access ≤ 15 min, refresh ≤ 30 days | api-contracts/auth.md |
| NFR-044 | Local DB encryption | At-rest: rely on OS file encryption + Keystore-protected key for sensitive columns | security/threat-model.md |
| NFR-045 | PII minimization | No PII in logs; phone numbers redacted (last 4 digits only) | logging policy |
| NFR-046 | Secret scanning in CI | Block PR on secret leak | governance/change-management.md |
| NFR-047 | Dependency vulnerability SLA | P0 ≤ 72 h, P1 ≤ sprint | tech-stack.md |
| NFR-048 | Sentry / observability data | No PII, scrubbed before send | logging policy |

## 5. Privacy

| ID | Requirement | Target |
|---|---|---|
| NFR-050 | GPS capture | Per-action consent, never silent |
| NFR-051 | Voice recording | Visible recording indicator + consent |
| NFR-052 | Photo capture | Per-action consent, location stripped from EXIF unless required |
| NFR-053 | Right to erasure | Documented procedure, ≤ 30 days SLA |
| NFR-054 | Data retention (media) | 24 months default, configurable per stakeholder agreement |
| NFR-055 | Audit log retention | 7 years for shift submissions |

## 6. Usability and accessibility

| ID | Requirement | Target |
|---|---|---|
| NFR-060 | WCAG 2.1 AA target | All key flows pass automated checks; manual review at S10 |
| NFR-061 | Dynamic type / large fonts | Layouts must not break at 200 % text scale |
| NFR-062 | Color contrast | ≥ 4.5:1 for body, ≥ 3:1 for large text |
| NFR-063 | Localization | EN + VI from S2 onward |
| NFR-064 | Offline UI clarity | Sync state badges always visible on capture screens |
| NFR-065 | Tap targets | ≥ 44 × 44 pt |

WCAG full validation requires manual testing with assistive tech and expert review; we verify automated checks here and schedule expert review at S10.

## 7. Maintainability

| ID | Requirement | Target |
|---|---|---|
| NFR-070 | Mobile test coverage | ≥ 70 % line coverage on `domain/` and `@core/` |
| NFR-071 | Backend test coverage | ≥ 80 % line coverage |
| NFR-072 | Static analysis | `flutter analyze` clean; `ruff` + `mypy --strict` clean |
| NFR-073 | API breaking changes | Versioned (`/v1/`); breaking change requires ADR |
| NFR-074 | Sprint completion rate | ≥ 80 % story points planned |

## 8. Observability

| ID | Requirement | Target |
|---|---|---|
| NFR-080 | Backend structured logs | JSON, request_id correlation |
| NFR-081 | Mobile crash reports | Sentry pilot opt-in, prod opt-in |
| NFR-082 | Backend metrics | RED + USE per service |
| NFR-083 | Distributed tracing | End-to-end across mobile → FastAPI → Celery → Odoo |
| NFR-084 | Sync queue dashboard | Pending / in-flight / failed visible to ops |
| NFR-085 | Alerting | P0 within 5 min, P1 within 30 min |

## 9. Compliance

| ID | Requirement | Notes |
|---|---|---|
| NFR-090 | Privacy policy in app | Available on login screen and in-app settings |
| NFR-091 | Terms of service | Same |
| NFR-092 | Data residency | Per pilot agreement (TBD at Phase 0) |
| NFR-093 | Right to be forgotten | Procedure in `runbooks/data-deletion.md` |

## 10. Operational

| ID | Requirement | Target |
|---|---|---|
| NFR-100 | Deploy time (backend) | ≤ 10 min |
| NFR-101 | Rollback time (backend) | ≤ 5 min |
| NFR-102 | Mobile rollout | Phased (5 % → 25 % → 100 %) over 5 days |
| NFR-103 | DB migration safety | Reversible or dual-write strategy mandatory |
| NFR-104 | Backup frequency (Postgres) | Hourly snapshots, 30-day retention |

## Tracking

NFRs are tracked in `backlog/nfr-register.md`. Failures to meet a target trigger a backlog story labelled `nfr-debt`.
