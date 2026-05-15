# Security policy

**Status:** Active
**Owner:** Security Champion
**Last updated:** 2026-05-13

This file is the entry point for anyone reporting or investigating a security issue.

## Reporting a vulnerability

- **Internal:** open a `Severity: SEV-1 Security` issue in the engineering tracker and tag `@security-champion`. Do not post details in public channels.
- **External (vendors, pen-testers, pilot users):** email `security@<company>` with subject `[Field Work App] vuln report`. We acknowledge within 1 business day, triage within 3 business days.
- **Coordinated disclosure window:** 90 days from acknowledgement, or sooner if patched and deployed.

Do not include PII, real OTP codes, or production tokens in reports. Use redacted samples.

## Scope

| In scope | Out of scope |
|---|---|
| Mobile app (iOS / Android release builds) | Third-party services beyond our control (Supabase, Twilio, Odoo) |
| FastAPI backend (production + staging) | Vulnerabilities requiring physical access to a rooted/jailbroken device |
| Custom Odoo module `field_mobile_sync` | Social engineering of staff |
| CI/CD supply chain | DoS attempts (please test on staging only) |

## Severity definitions

| Severity | Definition | Response SLA |
|---|---|---|
| **SEV-1** | Active exploitation; PII leak; auth bypass; data loss | Page on-call within 15 min; mitigation < 4 h |
| **SEV-2** | Vulnerability exploitable but not active; high impact | Triage within 1 business day; fix in current sprint |
| **SEV-3** | Lower-impact issue; defense-in-depth | Triage within 3 business days; fix within 2 sprints |
| **SEV-4** | Hardening / informational | Tracked in backlog |

## Hard rules

1. PII is never logged, sent to crash reporters, or stored unencrypted on device. See `security/pii-policy.md`.
2. Mobile never calls Odoo directly. See `adr/ADR-005-no-direct-mobile-to-odoo.md`.
3. Tokens live only in Keychain (iOS) / Keystore (Android). See `security/auth-flow.md`.
4. Secrets never enter git. CI runs gitleaks on every PR. See `security/secrets.md`.
5. Photos are EXIF-stripped client-side before upload. See `security/pii-policy.md`.

## Threat model

See `security/threat-model.md` (STRIDE) — refreshed every release, owned by Security Champion.

## Pen-test cadence

- Internal: on every breaking change to auth or sync.
- External: once before pilot launch (S10), once before GA, then annually.

## Compliance posture

| Topic | Posture |
|---|---|
| GDPR / equivalent | DSAR endpoint available; logout wipes local DB; retention policy documented in `security/pii-policy.md`. |
| SOC 2 | Audit-readiness tracked; not certified at MVP. |
| App store guidelines | Apple ATS enforced; Android network-security-config bundled. |

## Contacts

| Role | Owner |
|---|---|
| Security Champion | (TBD per `governance/raci.md`) |
| Privacy / DPO | (TBD) |
| Backend Lead | (TBD) |
| Mobile Lead | (TBD) |

When a contact is missing in this file, the role is unstaffed and the Engineering Lead is the fallback.
