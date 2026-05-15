---
sprint: S10
title: Hardening & Pilot
duration: 2 weeks
priority: P0
status: Ready
owner: QA Lead + PM
---

# Sprint 10 — Hardening & Pilot

## 1. Sprint goal

Pass the pilot gate and ship to a small pilot cohort with active monitoring and runbooks rehearsed.

## 2. Theme

No new features. Harden, measure, audit, and launch.

## 3. Scope (in)

- Device matrix regression run.
- Perf budget verification across NFR-100..NFR-115.
- Security audit + dependency scan + secret scan.
- Accessibility audit on critical flows.
- Dashboards + alerting + on-call rotation.
- Runbook walkthroughs (incident, sync recovery, OTP fallback, rollback, data deletion).
- Pilot recruitment, onboarding, feedback channel.
- Force-update plumbing (DEC-012) + crash reporter default (DEC-013) closed.

## 4. Out of scope

- Any new feature.
- General availability rollout (post-pilot).

## 5. Pre-conditions

- S9 done.
- DEC-008 closed.
- Staging is feature-frozen.
- TestFlight / Play internal track configured.

## 6. Stories committed

| ID | Title | Owner | Estimate |
|---|---|---|---|
| US-QA-001 | Device matrix regression | QA | L |
| US-QA-002 | Perf budget verification | QA + Mobile Lead | M |
| US-QA-003 | Security audit + dep scan | Security Champion | L |
| US-QA-004 | Accessibility audit | QA + Mobile | M |
| US-QA-005 | Dashboards + alerting + on-call | DevOps | M |
| US-QA-006 | Pilot launch | PM | M |

## 7. Risks for this sprint

- RISK-090 Pilot region SMS rates too low.
- RISK-091 Pilot users blocked by MDM.
- RISK-092 Late-discovered NFR breach forces re-baseline.
- RISK-093 Sentry quotas exceed budget under pilot load.

## 8. Sprint-level Definition of Done

- `qa/pilot-gate.md` shows all rows green.
- Pilot cohort actively using the app.
- On-call rotation in place; incident drill executed.
- DEC-012 and DEC-013 closed.
- One rehearsed rollback executed in staging.

## 9. Demo script

- Walk through pilot gate row by row.
- Show dashboards live during a synthetic load test.
- Have a pilot worker do a real shift on a real device.

## 10. Retro inputs

- Did we under- or over-rely on automation?
- Were runbooks usable under stress?
- What pilot signal do we trust before expansion?
