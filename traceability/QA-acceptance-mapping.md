# QA Acceptance Mapping

**Status:** Active  
**Owner:** QA Lead

Maps acceptance criteria categories to test areas, owners, and pilot-gate weighting. Use this together with `FR-matrix.md` and `qa/pilot-gate.md`.

## Test areas

| Area code | Area | Owner | TCs (count target) |
|---|---|---|---|
| **AUTH** | Authentication, OTP, session | QA + Backend | 10 |
| **SHIFT** | Shift list and detail | QA + Mobile | 6 |
| **PPE** | PPE check-in | QA | 4 |
| **FORM** | Pre/Post shift forms | QA | 4 |
| **GPS** | Location capture | QA + Mobile | 6 |
| **MEDIA** | Photo capture, compression, upload | QA + Mobile | 8 |
| **VOICE** | Voice note + Whisper | QA + Mobile | 5 |
| **OFF** | Offline / SQLite durability | QA + Mobile | 8 |
| **SYNC** | Sync queue, retry, idempotency | QA + Backend + Mobile | 12 |
| **ODOO** | Odoo write path | QA + Backend + Odoo Specialist | 8 |
| **SEC** | Security (auth, secrets, PII) | QA + Security Champion | 8 |
| **PERF** | Performance NFRs | QA + Mobile | 11 |

Total target test cases at MVP: **90**.

## Severity policy for QA

| Severity | Definition | Pilot gate impact |
|---|---|---|
| **Blocker** | Crash, data loss, security flaw, P0 FR not working | Block pilot |
| **Critical** | Major feature broken, no workaround | Block pilot |
| **Major** | Feature degraded, workaround exists | Block production, allow pilot with caveat |
| **Minor** | Cosmetic, edge case | Tracked, do not block |
| **Trivial** | Nit, polish | Tracked |

## Pilot gate weighting

A release is **pilot-ready** when:

- 100 % of P0 FRs have all P0 TCs passing on at least one iOS and one Android device.
- 0 Blocker, 0 Critical bugs open against MVP scope.
- ≤ 3 Major bugs open, each with documented workaround.
- All NFR targets met or accepted-with-waiver by PO + EL.
- Security audit complete, no P0 / P1 finding.
- Pilot training material reviewed by Stakeholder.

See `qa/pilot-gate.md` for the full checklist.

## Test pyramid

```
                    ┌───────────────┐
                    │  Manual & UAT │  ~5 %
                    └───────────────┘
              ┌──────────────────────────┐
              │   E2E (integration_test) │  ~10 %
              └──────────────────────────┘
        ┌──────────────────────────────────────┐
        │  Widget / API contract tests         │  ~25 %
        └──────────────────────────────────────┘
   ┌──────────────────────────────────────────────────┐
   │  Unit tests (domain, use cases, services)        │  ~60 %
   └──────────────────────────────────────────────────┘
```

## Devices in matrix

See `qa/device-matrix.md`. At minimum:

- iOS: iPhone 12 (mid), iPhone SE 2 (low), iPhone 15 (high).
- Android: Pixel 6a (mid), Galaxy A13 (low), Pixel 8 (high).
- All on supported OS major + previous (iOS 17, 18; Android 13, 14).

## Reporting

QA reports per sprint review:
- # TCs executed, passed, failed, blocked.
- Trend of bug count by severity.
- Coverage delta against `FR-matrix.md`.
- Any NFR target that drifted.

QA reports at pilot gate:
- Full TC pass matrix.
- Open bug list with severity and triage.
- Device matrix coverage.
- NFR results vs target.
- Recommendation: `Go` / `No-go` / `Go with caveat`.
