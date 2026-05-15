# RAID log — Risks, Assumptions, Issues, Dependencies

**Status:** Active
**Owner:** Project Manager
**Last updated:** 2026-05-13

This is the live operational log for the program. Granular risks live in `backlog/risk-register.md`; this file is the running cross-cutting view.

## Legend

- **Severity:** Low / Med / High / Critical
- **Status:** Open / In Progress / Mitigated / Closed
- **Owner:** named role per `governance/raci.md`

## Risks (top-level only; full register in `backlog/risk-register.md`)

| ID | Risk | Severity | Status | Owner | Mitigation |
|---|---|---|---|---|---|
| R-001 | Odoo version incompatible with chosen module set | High | Open | Backend Lead | Discovery workshop S0 (US-DISC-001). Linked DEC-001/002. |
| R-002 | SMS deliverability unstable in pilot region | Med | Open | Backend Lead | Identify regional fallback (DEC-005). |
| R-003 | Whisper on-device perf misses NFR-006 on low-end Android | Med | Open | Mobile Lead | Benchmark on Pixel 6a + budget device pre-S6; fallback to server transcription if exceeded. |
| R-004 | Object storage residency / cost surprises | Med | Open | DevOps | Resolve DEC-007. |
| R-005 | Pilot users without stable cell coverage | Low | Open | PM | Validate "30 envelopes / 72 h offline" scenario in S7. |
| R-006 | Idempotency contract violation by client editing rows | Med | Open | Mobile Lead | Lint at envelope builder; TC-SYNC-003 enforces. |
| R-007 | Refresh token compromise | High | Open | Security Champion | Rotation + re-use detection (TC-SEC-009/010). |
| R-008 | Vendor outage (Twilio / Supabase) blocks login | High | Open | Backend Lead | Document fallback per `runbooks/otp-fallback.md`. |
| R-009 | App store rejection on permission rationale | Med | Open | Mobile Lead | Pre-submission privacy manifest review at S10. |

## Assumptions

| ID | Assumption | Confidence | Owner | Validation |
|---|---|---|---|---|
| A-001 | Pilot region has 3G+ coverage 90 %+ of the time | Med | PM | Confirm with Sponsor before S0 close. |
| A-002 | `hr.employee.work_phone` will be the mapping field | Med | Odoo Specialist | DEC-006. |
| A-003 | All pilot devices run iOS 14+ / Android 8+ | High | Mobile Lead | Confirmed by Sponsor at S0. |
| A-004 | Service-account auth into Odoo is acceptable for pilot | Med | Security Champion | Reassess at GA. |
| A-005 | Daily envelope volume per user ≤ 50 | Med | PO | Used for capacity sizing; revisit after first sprint of pilot. |

## Issues (active)

| ID | Issue | Severity | Status | Owner | Action |
|---|---|---|---|---|---|
| I-001 | Phase 0 decisions DEC-001..009 still open | High | Open | EL + PM | Close in S0; track in WORKING.md. |

## Dependencies (external)

| ID | Dependency | Owner | Needed by | Status |
|---|---|---|---|---|
| D-001 | Odoo environment provisioned by Internal IT | Internal IT | S1 | Open |
| D-002 | Twilio account + sender provisioned | Sponsor + Backend Lead | S2 | Open |
| D-003 | S3 / object storage account provisioned | DevOps | S6 | Open |
| D-004 | Pilot device list from operations | Sponsor | S10 | Open |
| D-005 | Pilot user shortlist + comms plan | PO | S10 | Open |
| D-006 | App store accounts (Apple, Google) | Sponsor | S10 | Open |

## Cadence

- PM updates this file weekly and at every sprint review.
- New risks promoted to `backlog/risk-register.md` with full mitigation plan.
- Closed items move to a "Closed" section at the bottom for audit.

## Closed

(none yet)
