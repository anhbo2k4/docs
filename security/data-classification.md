# Data classification

**Status:** Active
**Owner:** Security Champion + Privacy Officer

## Why

Every field, log line, and stored artifact has a sensitivity. Classifying once, here, prevents inconsistent handling across mobile, backend, Odoo, and analytics.

## Levels

| Level | Description | Examples | Storage rules |
|---|---|---|---|
| **L0 — Public** | Safe to share externally without harm. | App version, public docs, store metadata. | Anywhere. |
| **L1 — Internal** | Operational data; not secret but not public. | Sprint plans, dashboards, anonymized usage counts. | Internal repos, internal dashboards. |
| **L2 — Confidential** | Business data tied to identifiable users or operations. | Shift assignments, work results, GPS events. | Encrypted at rest; access via authn + authz. |
| **L3 — Restricted PII** | Direct identifiers or sensitive personal data. | Phone numbers, names, faces in photos, voice clips, lat/lng. | Encrypted at rest + transit; access logged; minimal retention; never in logs / Sentry / metrics. |
| **L4 — Secret** | Cryptographic secrets and access tokens. | JWT signing keys, refresh tokens, Twilio API keys, S3 credentials. | Secret manager only; never in code, never in plaintext logs, rotated on schedule. |

## Field-level classification

### Auth

| Field | Class | Notes |
|---|---|---|
| `phone` | L3 | Stored only in Odoo `hr.employee` and as a hash for backend rate-limit keying. Mobile holds it transiently during OTP entry. |
| `otp_code` | L4 | Server-side ephemeral (≤ 5 min). Never logged. |
| `supabase_jwt`, `access_token`, `refresh_token` | L4 | Mobile: Keychain/Keystore. Backend: never persisted; refresh stored hashed. |
| `device_id` | L2 | Stable random UUID; not a hardware ID. |
| `employee_id` | L2 | Numeric Odoo PK. |
| `name`, `avatar_url` | L3 | Cached for UI only; wiped on logout. |

### Capture

| Field | Class | Notes |
|---|---|---|
| `ppe_check.items` | L2 | Operational. |
| `ppe_check.notes` | L3 (potentially) | May contain free text with PII; treat as L3 for log/transport. |
| `form_response.answers` | L3 | Free-text answers; user may include identifiable info. |
| `gps_event.{lat,lng,accuracy_m}` | L3 | Precise location is sensitive. |
| `media_blob` (photos) | L3 | EXIF stripped client-side. |
| `voice_note.audio` | L3 | Voice is biometric-adjacent. |
| `voice_note.transcript` | L3 | Free text from worker; treat as PII. |
| `work_result.notes` | L3 | Free text. |

### System

| Field | Class | Notes |
|---|---|---|
| `client_id` | L2 | UUID v4, not derived from PII. Safe in metrics. |
| `sync_envelope.error_code` | L1 | Stable enum; safe to label metrics. |
| Trace IDs (`traceparent`) | L1 | Useful for debugging, no PII. |
| `app_version`, `platform` | L1 | Aggregateable. |

## Handling rules

### Logs

- L0–L1: free.
- L2: allowed with field name and value.
- L3: redacted by formatter; only first / last 2–4 chars or hashed.
- L4: never. CI fails the build if a known L4 key appears in logs.

### Metrics

- Labels limited to L0–L1 enums (see `engineering/observability.md`).
- L2+ values are forbidden as labels (cardinality + privacy risk).

### Sentry / crash reporters

- L3+ scrubbed by `before_send` hook.
- Breadcrumbs: explicit allowlist, default deny.
- Tests verify scrubbing (TC-SEC-003, TC-SEC-006).

### Storage

- Mobile SQLite: contains L2/L3. Encrypted (per ADR-003 / SQLCipher when adopted) and wiped on logout.
- Backend Postgres: contains L2 + hashed/derived L3.
- Object storage (S3): contains L3 media. Server-side encryption mandatory.
- Odoo: contains L2/L3 per business mapping.
- Backups: same class as the source; access requires DPO approval.

### Transit

- All L2+ data on TLS 1.2+.
- Pinned certificates for mobile → backend in production builds.
- Backend ↔ Odoo on internal network only; mTLS where supported.

## Retention

| Data | Retention | Trigger |
|---|---|---|
| OTP codes | ≤ 5 min | Verify or expiry. |
| Refresh tokens | 30 days, rolling | Use or revoke. |
| Sync envelopes | 90 days online + 1 yr cold | Status final. |
| Media (photos) | Per business retention (default 1 yr) | Linked work result archived. |
| Voice clips | 90 days | Replaced by transcript when audio no longer needed. |
| Audit logs (auth, security) | 1 yr minimum | Regulatory; review with DPO. |
| Crash reports | 90 days | Sentry default; PII already scrubbed. |

Retention is enforced via scheduled jobs; failures page on-call.

## Access

- Workers: their own captures, current sprint shifts.
- Supervisors: their team's captures, read-only via Odoo.
- Backend service accounts: scoped per task (read-only sessions, write-only sync, etc.).
- Support: DSAR endpoint (`TC-SEC-015`) only; no direct DB.
- Engineers in production: break-glass via privileged-access workflow; logged + reviewed.

## DSAR (Data Subject Access Request)

- Export endpoint: `GET /v1/admin/dsar/{employee_id}` (admin-only).
- Returns L2 + L3 fields belonging to the subject.
- Logged in audit; SLA 30 days.

## Right to erasure

- Logout already wipes mobile storage (TC-SEC-008 / NFR-044).
- Backend erasure: `POST /v1/admin/erase/{employee_id}` (admin-only) cascades to Odoo per `runbooks/data-deletion.md`.
- SLA per regulation in pilot region.
