# Secrets Handling

**Status:** Active  
**Owner:** Security Champion + DevOps

## Where secrets live

| Secret | Location | Access |
|---|---|---|
| Supabase anon key | App config (public) | Bundled in mobile build (anon, OK to be public) |
| Supabase service role key | Supabase Edge env | Edge functions only |
| Twilio API key + secret | Supabase Edge env | Edge functions only |
| FastAPI JWT signing keys (RSA) | AWS Secrets Manager (or Vault) | FastAPI runtime via OIDC |
| Odoo service user credentials | AWS Secrets Manager | Celery workers only |
| Postgres credentials | AWS Secrets Manager | FastAPI + workers |
| Redis credentials | AWS Secrets Manager | FastAPI + workers |
| S3 access keys | IAM role on pods | Pod identity, no static keys |
| Sentry DSN (mobile) | Bundled (semi-public) | OK to be public |
| Sentry DSN (backend) | Secrets Manager | Backend runtime |

## Forbidden

- Secrets in `.env` files committed to git.
- Secrets in CI logs.
- Secrets in Sentry breadcrumbs.
- Long-lived AWS access keys in mobile or CI scripts.

## CI

- GitHub Actions uses **OIDC** to obtain short-lived AWS credentials. No static keys in repository secrets except a single OIDC role ARN.
- Secret scanning: gitleaks runs on every PR. PRs are blocked on detection.
- `.gitignore` includes `.env*` and `secrets.*`.

## Rotation policy

| Secret class | Rotation | Owner |
|---|---|---|
| FastAPI JWT signing keys | 90 days | DevOps |
| Odoo service user | 90 days | DevOps + Odoo Specialist |
| Postgres user | 180 days | DevOps |
| Redis password | 180 days | DevOps |
| Twilio API key | 180 days | DevOps |
| Sentry DSN | On compromise only | DevOps |
| OTP HMAC secret (Edge) | 90 days | DevOps |

## Loading at runtime

- FastAPI: `pydantic-settings` reads from env. CI / runtime injects from Secrets Manager via container env.
- Mobile: `--dart-define` for the Supabase anon key and base URL. No build-time PII.
- Edge functions: Supabase project secrets, accessed via `Deno.env.get`.

## Mobile-side secure storage

- iOS: Keychain via `flutter_secure_storage`, accessibility `whenUnlockedThisDeviceOnly`.
- Android: Keystore-backed via `flutter_secure_storage`. AndroidOptions: `keyCipherAlgorithm = RSA_ECB_PKCS1Padding`, `storageCipherAlgorithm = AES_GCM_NoPadding`, `encryptedSharedPreferences = true`.

Stored items:
- Internal access JWT
- Internal refresh JWT
- `device_id` (UUID, generated at first launch)
- Optional: app-level encryption key for sensitive SQLite columns.

## Compromise response

If a key leaks:

1. Declare SEV-1 per `governance/escalation.md`.
2. Rotate the secret immediately.
3. Invalidate dependent tokens (e.g. revoke all sessions for JWT key compromise).
4. Audit access logs for misuse.
5. Postmortem within 5 business days.

## Tests

- TC-SEC-001 secret scanning blocks PR on test fixture.
- TC-SEC-002 logger redacts phone numbers.
- TC-SEC-003 Sentry scrubber strips PII from breadcrumbs.
- TC-SEC-004 secure storage roundtrip on iOS + Android.
