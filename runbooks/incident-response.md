# Runbook: Incident Response (first 60 minutes)

**Status:** Active  
**Owner:** Engineering Lead + On-call

## Symptom

Any SEV-1 or SEV-2 alert. Pages: P0 in PagerDuty.

## Severity

SEV-1 (data loss, security breach, pilot/prod down) or SEV-2 (major feature broken, no workaround).

## First 5 minutes — Acknowledge

- [ ] Acknowledge PagerDuty.
- [ ] Post in `#field-app-incidents`:

```
INCIDENT: <one-line summary>
Sev: <SEV-1|SEV-2>
Started: <UTC>
Lead: @<your handle>
Status: investigating
```

- [ ] Spawn an incident channel thread: `INC-YYYYMMDD-NNN: <summary>`.
- [ ] Open `runbooks/incidents/INC-YYYYMMDD-NNN.md` from template.

## 5–15 minutes — Stabilise

- [ ] Identify whether the issue is mobile, backend, Odoo, or external (Twilio/Supabase).
- [ ] Check Grafana: API RED, queue depth, sync success rate, Odoo error rate.
- [ ] Check Sentry: any new mobile crashes.
- [ ] Check status pages: Twilio, Supabase, AWS region.
- [ ] If user impact is escalating, **stabilise first, diagnose second**:
  - Roll back the latest deploy (`runbooks/rollback.md`).
  - Disable a feature flag.
  - Drain queue (`SYNC_DRAIN_ONLY=true`).
  - Throttle traffic (rate-limit harder).

## 15–30 minutes — Diagnose

- [ ] Pull recent deploy timeline: when did metrics deviate?
- [ ] Tail backend logs filtered by error codes.
- [ ] Tail Celery worker logs.
- [ ] Tail Odoo logs (request access if not already on-call).
- [ ] Check recent Phase 0 decision changes that may have shifted behaviour.
- [ ] Confirm the **scope**: how many users, which devices, which envelopes.

## 30–60 minutes — Communicate

- [ ] Post status update every 15 min in `#field-app-incidents`.
- [ ] If pilot users affected, post a single message in `#field-app-stakeholders` (PM owns this; do not let on-call engineer field stakeholder questions during firefight).
- [ ] If SEV-1: page Sponsor + PM via `governance/escalation.md`.
- [ ] If security-related: page Security Champion.

## After containment

- [ ] Update `runbooks/incidents/INC-YYYYMMDD-NNN.md` with timeline.
- [ ] Schedule postmortem within 5 business days.
- [ ] Identify follow-up items, file in backlog with `incident-followup` label.

## Common quick checks

| Symptom | First check | Likely runbook |
|---|---|---|
| Sync success rate drops | Odoo error rate | `sync-recovery.md` |
| Backend 5xx spike | Recent deploy + Postgres | `rollback.md` |
| Mobile crash spike | Sentry stack traces | `rollback.md` (mobile) |
| OTPs not arriving | Twilio status + Supabase status | `otp-fallback.md` |
| Photos not uploading | S3 region status + presign expiry logs | (no runbook yet — diagnose live) |
| Auth 401 storm | JWT key rotation done correctly? | `auth-flow.md` |

## Roles during incident

| Role | Responsibility |
|---|---|
| **Incident Lead** | Drives the response; takes decisions; updates channel |
| **Communications** | Posts updates to stakeholders; PM by default |
| **Investigator** | Reads logs, finds root cause |
| **Implementer** | Applies fix or rollback |
| **Scribe** | Maintains incident timeline doc |

For SEV-2 in business hours these can overlap. For SEV-1, separate them.

## Postmortem expectations

- Blameless.
- Timeline of what happened with timestamps.
- Root cause (5 whys).
- What worked.
- What did not.
- Action items with owners and due dates.
- Update relevant runbook if response was slower than target.
