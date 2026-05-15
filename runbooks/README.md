# Runbooks

**Status:** Active  
**Owner:** DevOps / SRE + Engineering Lead

Operational playbooks. Each runbook is self-contained: read top-to-bottom during an incident, follow steps, decide.

| File | Purpose |
|---|---|
| [rollback.md](./rollback.md) | Rolling back a bad release (backend or mobile) |
| [incident-response.md](./incident-response.md) | First 60 minutes of any SEV-1 / SEV-2 |
| [sync-recovery.md](./sync-recovery.md) | Stuck queues, dead-letter triage, replay |
| [otp-fallback.md](./otp-fallback.md) | OTP delivery failure (Twilio / Supabase outage) |
| [data-deletion.md](./data-deletion.md) | Right-to-be-forgotten procedure |
| [deploy.md](./deploy.md) | Standard backend + mobile release |

## On-call basics

- Primary on-call: 1 engineer / week, rotated.
- Secondary: Engineering Lead (always).
- Channels: Slack `#field-app-incidents`, PagerDuty.
- Severity definitions: see `governance/escalation.md`.

## Conventions

- Every runbook starts with **Symptom**, **Severity**, **Decision**, **Steps**, **Verify**, **After-action**.
- Runbooks are tested at least quarterly during pilot, monthly during prod year-1.
- Untested runbooks are documented as such — do not rely on them blind.

## Incident artefacts

Each incident produces `runbooks/incidents/INC-YYYYMMDD-NNN.md` capturing timeline, RCA, and follow-ups.
