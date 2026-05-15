# Risk Register

**Status:** Active  
**Owner:** Project Manager  
**Last reviewed:** 2026-05-13

Living register. Reviewed bi-weekly during build, weekly during pilot. Severity = Likelihood × Impact.

## Scoring

| Likelihood | Score | Impact | Score |
|---|---|---|---|
| Rare | 1 | Insignificant | 1 |
| Unlikely | 2 | Minor | 2 |
| Possible | 3 | Moderate | 3 |
| Likely | 4 | Major | 4 |
| Almost certain | 5 | Catastrophic | 5 |

Severity = L × I. Tiers: 1–6 Low, 7–14 Medium, 15–25 High.

## Active risks

| ID | Risk | Category | L | I | Sev | Owner | Mitigation | Trigger | Contingency |
|---|---|---|---|---|---|---|---|---|---|
| RISK-001 | Phase 0 decisions slip past S0 | Schedule | 4 | 4 | 16 | PM | Daily Phase 0 standup; Sponsor escalation at day 3 of slippage | Any DEC-001..009 still Open at end of S0 | Push S1 by 1 week; reduce S1 scope to scaffolding only |
| RISK-002 | Odoo `field_mobile_sync` module not built in time | Schedule + Tech | 3 | 5 | 15 | Backend Lead | Engage Odoo Specialist at S0; spike at S2 | Module not deployed by end of S7 | Stub the module; Celery writes minimal viable subset; defer GPS to post-MVP |
| RISK-003 | iOS background sync unreliable on low-end devices | Tech | 4 | 3 | 12 | Mobile Lead | Use BGTaskScheduler best practices; foreground sync on resume covers gaps | Sync success rate < 95 % on iPhone SE 2 | Add user-initiated sync from Sync Center; document trade-off |
| RISK-004 | Whisper tiny.en accuracy unacceptable for VN accents | Product | 3 | 3 | 9 | Mobile Lead | Benchmark in S6; compare with multilingual model | WER > 40 % on VN accent benchmark | Allow audio-only attach (no transcript); revisit STT vendor post-MVP |
| RISK-005 | Twilio outage during pilot | Operational | 2 | 4 | 8 | DevOps | OTP fallback runbook; consider regional fallback per DEC-005 | OTP delivery rate < 90 % over 30 min | Manual override per `otp-fallback.md` |
| RISK-006 | Pilot site network worse than expected | Operational | 4 | 3 | 12 | PM + Stakeholder | Test on 3G in S7; document offline soak (72 h) in NFR-025 | Pilot users report sync stuck > 24 h | Increase mobile retry budget; offer manual sync from Sync Center |
| RISK-007 | Mobile binary size exceeds NFR-010 due to Whisper bundle | Tech | 3 | 2 | 6 | Mobile Lead | Track size every PR; consider download-on-first-launch (DEC-010) | APK > 80 MB | Switch model bundling strategy |
| RISK-008 | Custom Odoo module deployment requires Odoo restart | Operational | 4 | 2 | 8 | Backend Lead + Odoo Specialist | Schedule deploy windows; coordinate with ERP team | Each release blocks Odoo for > 5 min | Move to add-on module that doesn't require restart |
| RISK-009 | PII leak via crash report or log | Security | 2 | 5 | 10 | Security Champion | Sentry scrubber + log redactor; CI test for redaction | PII appears in any non-test environment | Immediate purge + SEV-1 incident |
| RISK-010 | Idempotency bug allows duplicate Odoo records | Tech | 2 | 4 | 8 | Backend Lead | TC-SYNC-024, TC-ODOO-024; chaos tests | Duplicate records found in Odoo audit | Hot-fix release; backend migration to dedup historical data |
| RISK-011 | SQLite migration corruption on upgrade | Tech | 2 | 5 | 10 | Mobile Lead | Migration tests in CI; dual-write strategy for risky columns | User reports app crash after upgrade | Rollback mobile; emergency hotfix; ask users to reinstall (data loss accepted as last resort) |
| RISK-012 | Pilot users abandon app due to UX friction | Product | 3 | 3 | 9 | PO + Designer | Sprint reviews include pilot user feedback from S5 onwards | NPS < 20 at week 2 of pilot | Reduce friction; defer non-essential forms; iterate |
| RISK-013 | Data retention non-compliance in pilot region | Compliance | 2 | 5 | 10 | Compliance + DevOps | DEC-008 closes data residency at S0 | Compliance audit finding | Migrate region; offer deletion |
| RISK-014 | Background tasks killed by aggressive battery savers (Xiaomi, Huawei) | Tech | 4 | 3 | 12 | Mobile Lead | Document per-OEM tweaks; foreground sync covers most cases | Sync success rate worse on specific OEMs | Provide on-screen instructions to whitelist app |
| RISK-015 | Scope creep from stakeholder requests | Schedule | 4 | 3 | 12 | PM + PO | Strict change-management; bi-weekly steering | Backlog grows > 30 % between sprints | Refuse via change-management; defer to EPIC-10 |
| RISK-016 | Designer not staffed full-time in S5/S6 | Resource | 3 | 3 | 9 | PM | Schedule designer for capture screens at S5; contingency contractor list ready | UI design blocked > 5 days | Use stock material; refine post-pilot |
| RISK-017 | App Store / Play review delays | Schedule | 3 | 3 | 9 | DevOps | Submit early; familiar patterns; accessibility checks pre-submit | First submission rejected | Address feedback; resubmit; phased rollout absorbs delay |
| RISK-018 | Pilot dataset too small to validate edge cases | Quality | 3 | 3 | 9 | QA | Plan 50 active users in pilot; chaos test in staging | Bugs only surface in prod scale | Extend pilot; add more sites |

## Closed risks

(none yet)

## Process

1. Anyone may file a risk via PR adding a row.
2. PM reviews bi-weekly with EL + QA.
3. Each risk has an owner who reports status at sprint review.
4. When mitigation reduces severity below 7, risk moves to "Closed" with rationale.
5. SEV ≥ 15 risks escalate to Sponsor.
