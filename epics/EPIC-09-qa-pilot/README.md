---
epic: EPIC-09
title: QA, Security & Pilot Readiness
sprint: S10
priority: P0
status: Ready
owner: QA Lead + Security Champion
---

# EPIC-09 — QA, Security & Pilot Readiness

## 1. Goal

Harden the system. Run the device matrix, perf budget, security audit, and accessibility audit. Close the pilot gate. Ship to a small pilot cohort with active monitoring.

## 2. In scope

- Full regression on the device matrix (`qa/device-matrix.md`).
- Perf budget verification against NFR-100..NFR-115.
- Security audit: STRIDE walkthrough vs `security/threat-model.md`, secrets review, third-party dependency scan, pen-test ticket if scope warrants.
- Accessibility audit: text scaling, contrast, screen reader on critical flows.
- Force-update plumbing (DEC-012) and crash reporter default (DEC-013) decided.
- Pilot recruitment, comms plan, and feedback channel.
- Dashboards: SLO + sync queue health.
- Incident playbooks rehearsed (`runbooks/incident-response.md`, `runbooks/sync-recovery.md`, `runbooks/otp-fallback.md`, `runbooks/rollback.md`).
- Data-deletion runbook tested (`runbooks/data-deletion.md`) for a pilot user.

## 3. Out of scope

- New features. EPIC-09 is hardening only.
- General availability rollout (post-pilot).

## 4. Functional requirements covered

Validation of all FR-001..FR-015 plus all NFRs.

## 5. Stories

| ID | Title | Pri | Estimate | Status | Owner |
|---|---|---|---|---|---|
| US-QA-001 | Device matrix regression run | P0 | L | Ready | QA |
| US-QA-002 | Perf budget verification | P0 | M | Ready | QA + Mobile Lead |
| US-QA-003 | Security audit + dependency scan | P0 | L | Ready | Security Champion |
| US-QA-004 | Accessibility audit on critical flows | P0 | M | Ready | QA + Mobile |
| US-QA-005 | Dashboards + alerting + on-call | P0 | M | Ready | DevOps |
| US-QA-006 | Pilot launch (recruit, comms, feedback) | P0 | M | Ready | PM |

## 6. Dependencies

- All prior epics done.
- DEC-008 closed (Odoo SLA / DR posture acceptable for pilot).

## 7. Risks

- RISK-090 Pilot region SMS rates too low (DEC-005 unmet).
- RISK-091 Pilot users unable to install due to MDM constraints.
- RISK-092 Late-discovered NFR breach forces re-baseline.
- RISK-093 Sentry quotas exceed budget under pilot load.

## 8. Definition of Done (epic-level)

- `qa/pilot-gate.md` shows all rows green.
- Pilot users can install via TestFlight / Play internal track.
- On-call rotation in place; runbook walkthrough completed; one rehearsed rollback executed in staging.
- Decision-log entries for DEC-012 and DEC-013 closed.

## 9. Open questions

DEC-012 (force-update threshold), DEC-013 (crash reporter default) decided here.
