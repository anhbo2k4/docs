# PII Policy

**Status:** Active  
**Owner:** Security Champion + Compliance

## What is PII in this app

- Phone number (used as login identifier).
- Employee name, job title, department.
- Photo of a person's face (incidental).
- Voice note (incidental).
- Voice transcript.
- GPS coordinates with timestamp.
- Device identifier.
- Audit log entries that include any of the above.

## Where it lives

| PII | Mobile | Edge | FastAPI / Postgres | S3 | Odoo | Logs / Sentry |
|---|---|---|---|---|---|---|
| Phone | yes (employee row) | yes (OTP flow) | yes (sessions, audit) | no | yes | redacted (last 4) |
| Name / role | yes | no | yes (cache) | no | yes (SoR) | no |
| GPS | yes | no | metadata only | no | yes (SoR) | no |
| Photo bytes | yes (cache) | no | no | yes | metadata only | never |
| Audio bytes | yes (cache) | no | no | yes | metadata only | never |
| Transcript | yes | no | yes (envelope) | no | yes | redacted |
| Device ID | yes | no | yes | no | optional | yes (correlated only) |

## Rules

1. **No PII in logs.** Phone numbers are redacted: `+849****5678`. Names truncated to first letter. GPS coordinates rounded to 2 decimal places (~1 km) when logged at all.
2. **No PII in Sentry.** A scrubber rule removes `phone`, `name`, `email`, raw payloads, and `transcript` fields.
3. **No PII in metrics.** Aggregate counts only.
4. **Per-action consent** for camera, GPS, microphone.
5. **Visible recording indicator** during voice capture (NFR-051).
6. **EXIF strip** on photos server-side unless required for the work.
7. **Right to be forgotten** procedure documented in `runbooks/data-deletion.md`.
8. **Retention** per `architecture/non-functional-requirements.md` NFR-053..055.
9. **Local DB wiped on logout.**
10. **No analytics SDKs** that collect PII (Sentry is opt-in for crash data only, scrubbed).

## On-device retention

- Photos: kept 7 days after `CONFIRMED`, then deleted by janitor task. (Configurable via OQ-006.)
- Audio: kept 7 days after `CONFIRMED`.
- Transcripts: kept indefinitely on device until logout.
- Local DB encrypted by OS file-level encryption; sensitive columns optionally encrypted at app layer.

## Server-side retention

- Phone numbers + employee mapping: indefinite while employee active in Odoo.
- Sync envelopes: 24 months (NFR-054).
- Audit log: 7 years (NFR-055).
- Object storage: 24 months default.

## Right of access

- Workers can request a full export of their data. Process documented in `runbooks/data-export.md` (TBD post-MVP).
- Workers can request deletion. Process documented in `runbooks/data-deletion.md`.
- SLA: 30 days from request to delivery / completion.

## Incident response

PII exposure is automatically SEV-1. Escalation per `governance/escalation.md`. Compliance notified within 1 hour.

## Tests

- TC-SEC-005 PII scrubber on logs.
- TC-SEC-006 PII scrubber on Sentry.
- TC-SEC-007 EXIF stripped on photo upload.
- TC-SEC-008 Local DB wiped on logout.
