# Threat Model (STRIDE)

**Status:** Active  
**Owner:** Security Champion  
**Last reviewed:** 2026-05-13

## Scope

The unified Flutter mobile app, Supabase Edge functions for OTP, FastAPI integration backend, Celery workers, PostgreSQL, Redis, object storage, and Odoo integration boundary. Out of scope: Odoo internals, Twilio internals.

## Trust boundaries

```
[ Untrusted: Internet / mobile device ]
    │
    ▼
[ DMZ: Supabase Edge, ALB ]
    │
    ▼
[ Internal: FastAPI, Celery, Postgres, Redis ]
    │
    ▼
[ Internal: Odoo, Object Storage ]
```

## Asset inventory

| Asset | Sensitivity | Where stored |
|---|---|---|
| OTP codes | High (short TTL) | Supabase Edge memory + Twilio |
| Supabase JWT | Medium (5 min) | Mobile memory only |
| Internal access JWT | High (15 min) | flutter_secure_storage |
| Internal refresh JWT | High (30 days) | flutter_secure_storage |
| Phone numbers | PII | Mobile + Edge + Postgres + Odoo |
| Employee profile | PII | Mobile + Postgres cache + Odoo |
| GPS coordinates | PII | Mobile + S3 metadata + Odoo |
| Photos | PII (may include faces) | Mobile + S3 + Odoo (refs) |
| Voice notes (audio) | PII | Mobile + S3 + Odoo (refs) |
| Voice transcripts | PII | Mobile + Postgres + Odoo |
| `client_id` UUIDs | Low | Everywhere |
| Odoo service credentials | Critical | Backend secret manager |
| Twilio API key | Critical | Supabase secret store |
| S3 access keys | Critical | Backend secret manager |
| Postgres credentials | Critical | Backend secret manager |

## STRIDE per component

### Mobile app

| Threat | Description | Likelihood | Impact | Mitigation |
|---|---|---|---|---|
| **S** Spoofing user | Attacker steals phone, opens app | Medium | High | OS biometric / PIN required (post-MVP); access JWT TTL 15 min; remote logout via session revocation |
| **T** Tampering of local DB | Rooted/jailbroken device modifies SQLite | Low | Medium | Encrypt sensitive columns; treat client as untrusted server-side; idempotency dedup |
| **R** Repudiation of submission | Worker denies submitting | Low | Low | Audit log keyed by `client_id`, `device_id`, server timestamps |
| **I** Information disclosure (logs) | Verbose logs include phone | Medium | High | Logger redacts PII; no Sentry breadcrumbs with raw payload |
| **D** Denial of service | App spams sync POSTs | Low | Low | Rate limit per employee; mobile-side debounce |
| **E** Elevation of privilege | Compromised app calls Odoo directly | None | — | ADR-005 forbids direct Odoo; mobile has no Odoo creds |

### Supabase Edge (OTP)

| Threat | Description | Likelihood | Impact | Mitigation |
|---|---|---|---|---|
| **S** OTP brute force | Attacker tries codes | Medium | High | 5 wrong attempts → 5 min lock per phone; codes have 5 min TTL |
| **S** Phone enumeration | Asking who is registered | Medium | Low | Generic 200 response regardless of registration |
| **T** Replay of OTP | Re-using used code | Low | High | Codes single-use, marked consumed atomically |
| **I** SMS interception | SIM swap, SS7 attack | Low (regional) | High | TFA via OTP only is acceptable for MVP; document in risk register; consider TOTP later |
| **D** SMS bombing | Triggering many sends to same phone | Medium | Medium (cost) | 3 requests / 5 min per phone; daily cap per device |
| **D** Twilio outage | OTP delivery breaks | Low | High | Document fallback (manual override by admin via Odoo) — see runbooks |

### FastAPI

| Threat | Description | Likelihood | Impact | Mitigation |
|---|---|---|---|---|
| **S** Forged JWT | Attacker crafts token | Low | Critical | RS256 + JWKS; rotate keys 90 days |
| **T** Tampered envelope | Modified payload after capture | Low | Medium | Server validates against schema; idempotency by `client_id` |
| **R** Lost audit | Cannot trace who did what | Low | Medium | `audit_log` for every state-changing call |
| **I** PII leak via error message | Stacktraces in 500 | Medium | High | Generic error envelope; details only on 4xx with safe fields |
| **D** Sync flood | Many clients overwhelm queue | Medium | High | Rate limit + queue depth alerts + autoscale |
| **E** Privilege escalation | Worker writes for another employee | Low | High | Server checks `employee_id == jwt.employee_id` for every write |

### Celery / Odoo

| Threat | Description | Likelihood | Impact | Mitigation |
|---|---|---|---|---|
| **T** Duplicate Odoo records | Retry races create multiples | Medium | Medium | `external_id = client_id` unique per employee; upsert pattern |
| **I** Odoo schema leak in errors | Odoo validation errors return raw | Medium | Medium | Map Odoo errors to internal error codes |
| **D** Odoo unavailable | Whole sync stalls | Medium | Medium | Circuit breaker, queue grows, ops alert; envelopes stay PENDING |
| **E** Worker compromised | Could write anywhere in Odoo | Low | Critical | Service user has minimal model permissions; secret rotation |

### Postgres / Redis / S3

| Threat | Description | Likelihood | Impact | Mitigation |
|---|---|---|---|---|
| **I** Backup leak | DB snapshot exposed | Low | Critical | Encrypted snapshots, IAM-scoped access |
| **T** Redis injection | Malicious cache key | Low | Low | Treat broker as trusted; restrict network |
| **I** Public S3 bucket | Misconfiguration | Low | Critical | Bucket policy denies public; access only via presigned URLs |

## Top 10 prioritized risks

1. PII leak via logs / Sentry. *Mitigation: logger redaction + Sentry scrubber, tested in CI.*
2. OTP brute force. *Mitigation: rate limits + lockout; tested.*
3. Forged JWT. *Mitigation: RS256 + JWKS rotation; tested.*
4. Duplicate Odoo records. *Mitigation: idempotency by `client_id`; tested.*
5. Lost evidence after app kill. *Mitigation: SQLite first durable write; tested.*
6. Privilege escalation in sync intake. *Mitigation: server enforces ownership; tested.*
7. SMS bombing. *Mitigation: daily caps + monitoring.*
8. Misconfigured S3 (public). *Mitigation: IaC linting + nightly check.*
9. Pre-shift form schema drift between mobile and Odoo. *Mitigation: `schema_version` on every form_response.*
10. Stale presigned URLs leading to lost photos. *Mitigation: 10-min expiry + retry on `EXPIRED`.*

## Dependencies and supply chain

- Renovate / Dependabot for both mobile and backend.
- CVE feed monitored. SLA per `architecture/non-functional-requirements.md` NFR-047.
- Mobile dependencies must be on pubspec.lock; backend on poetry.lock or pip-compile.
- No transitive packages from unverifiable sources.

## Review cadence

- Quarterly review during build.
- Monthly review during pilot.
- After every SEV-1.

## Open security questions

- Should we require biometric auth on every app open during pilot? (deferred, post-MVP)
- Should photos blur faces by default? (post-MVP)
