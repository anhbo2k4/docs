# On-call

**Status:** Active
**Owner:** Engineering Lead

## Rotation

- Pilot phase: 2 engineers, 7-day weeks, follow-the-sun within local TZ.
- Primary handles pages; secondary backs up if primary is unreachable for 15 min.
- Out-of-hours pages only for SEV-1 / SEV-2.

## Tooling

- Pager: PagerDuty (or vendor); routes per `Backend Service Health` and `Mobile Release Health` alerts.
- Status page: internal-only during pilot; external after GA.
- Incident commander tool: a shared doc using `runbooks/incident-response.md` template.

## When pager fires

1. Acknowledge within 5 min.
2. Open the linked runbook. If missing, treat the page itself as a defect.
3. Mitigate first; root-cause later.
4. If SEV-1, declare incident, assemble responders per `governance/escalation.md`.
5. Update affected stakeholders per `governance/communication-plan.md`.
6. Write postmortem within 5 business days using `runbooks/_postmortem-template.md`.

## Handoff

- End-of-shift handoff covers: open incidents, error-budget burn, anomalies seen, deploys overnight.
- Documented in the on-call channel.

## Page-worthy criteria

| Symptom | Page? |
|---|---|
| 5xx > 1 % sustained 5 min | yes |
| Crash-free < 99 % rolling 1 h | yes |
| OTP success < 90 % for 15 min | yes |
| Dead-letter spike > 50/h | yes |
| Single user reports issue | no (ticket) |
| Cosmetic UI bug | no |
| Capacity alert without user impact | no (ticket) |

## What on-call cannot do

- Deploy without peer review; CI gates remain in force.
- Modify Odoo data directly; recovery flows are in `runbooks/sync-recovery.md`.
- Disable security controls; if needed, follow `runbooks/incident-response.md`.

## Health of on-call

- Track pages per week; > 3 unactionable pages triggers a fix-it sprint slot.
- After SEV-1, give responders the next morning off.
