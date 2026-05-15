# Odoo RPC Adapter — Capability Spec

**Status:** Active (rewritten in v2.1.0)
**Owners:** Backend Lead + Odoo Specialist
**Last updated:** 2026-05-15
**Version:** 2.0 (replaces Path A / Path B model)
**Consumes:** [ADR-011](../adr/ADR-011-odoo-19-enterprise.md), [ADR-015](../adr/ADR-015-no-custom-module-rpc-adapter.md), [ADR-016](../adr/ADR-016-gps-in-fastapi-postgres.md)
**See also:** [`data/odoo-x-fields-spec.md`](../data/odoo-x-fields-spec.md), [`data/odoo-mapping.md`](../data/odoo-mapping.md)

This document specifies how the FastAPI integration layer (ADR-004) talks to Odoo across versions and editions (Community / Enterprise / Online). The previous Path A / Path B division is **superseded** — it assumed a single edition choice plus a custom module, both of which were retired in v2.1.0.

> **Build rule.** No code in EPIC-08 may be written until the FastAPI startup probe reports `ready_for_writes: true` against the target Odoo tenant. The probe contract is defined in `data/odoo-x-fields-spec.md` §6.

---

## 1. Why an adapter, not a fixed contract

Sponsor binding constraints (2026-05-15):

1. **Must run on Odoo Online.** No custom Python modules; only Studio-level customisation (`x_*` fields).
2. **Must work across Odoo versions and editions.** Production targets are not pinned to one Odoo version.
3. **Manual Odoo setup is acceptable.** Field set is small and stable.
4. **Twilio + Odoo + Storage credentials are tenant-supplied.** The integration is multi-tenant friendly.

A fixed-contract integration cannot satisfy (1) + (2). The adapter pattern probes the tenant at startup, classifies its capabilities, and routes writes accordingly.

## 2. Architectural picture

```
┌─────────┐  HTTPS   ┌────────────────────────────────────────┐  JSON-RPC   ┌──────────┐
│ Mobile  │────────▶ │ FastAPI integration layer              │────────────▶│  Odoo    │
└─────────┘          │  ├─ Auth + sync envelope               │             │ (any)    │
                     │  ├─ Idempotency (Redis SETNX)          │             └──────────┘
                     │  ├─ Sync state (Postgres)              │
                     │  ├─ GPS history (Postgres, ADR-016)    │
                     │  └─ OdooAdapter                        │
                     │       ├─ probe() at startup            │
                     │       ├─ capability map                │
                     │       └─ per-capability write paths    │
                     └────────────────────────────────────────┘
```

## 3. Capability matrix

Detected at startup by reading `ir.module.module` and `ir.model.fields`. The matrix governs every write path.

| Capability key | Detection | Required? | Effect when present | Effect when absent (degrade) |
|---|---|---|---|---|
| `hr` | `hr.employee` model exists | **Hard** | Used for identity, phone, department | Probe fails → 503 |
| `project` | `project.task` model exists | **Hard** | Default shift entity | Probe fails → 503 |
| `contacts` | `res.partner` exists | **Hard** | Site / customer linkage | Probe fails → 503 |
| `x_ngynapp_*` field set | `ir.model.fields` lookup per `data/odoo-x-fields-spec.md` | **Hard (Required = Yes rows)** | Idempotency, sync state, GPS summary, etc. | Probe fails with missing-field list |
| `industry_fsm` | module installed | Soft | Shift entity = `fsm.task`, native FSM workflows used | Shift entity = `project.task`; FSM-specific fields skipped |
| `planning` | module installed | Soft | Read `planning.slot` for shift calendar | Calendar view falls back to `project.task.date_deadline` |
| `hr_attendance` | module installed | Soft | Mobile check-in/out also writes `hr.attendance` | Attendance writes skipped; presence inferred from shift fields |
| `hr_timesheet` | module installed | Soft | Auto-create `account.analytic.line` on check-out | Timesheet writes skipped |
| `survey` | module installed | Soft | PPE / preshift forms map to `survey.survey` + `survey.user_input` | Forms stored in FastAPI Postgres only; mirrored as JSON in `x_ngynapp_preshift_form_id` |
| `quality` | module installed | Soft | Post-shift inspections map to `quality.check` | Inspection JSON stored alongside preshift |
| `maintenance` | module installed | Soft | Asset issues link to `maintenance.request` | Asset issues stored in our DB; surfaced as text on the task |
| `documents` | module installed | Soft | Photos filed under `documents.document` with tags | Photos remain as `ir.attachment` only |

The adapter **never branches on edition strings**. Online does not expose a clean edition signal; module presence is the only durable signal across SaaS and self-host.

## 4. The `OdooAdapter` interface

```python
class OdooAdapter:
    # Lifecycle
    def probe(self) -> ProbeReport: ...
    def refresh(self) -> None: ...           # call on schedule + on admin action

    # Capability access
    def supports(self, capability: str) -> bool: ...
    @property
    def shift_model(self) -> str: ...        # 'fsm.task' or 'project.task'
    @property
    def phone_field(self) -> str: ...        # 'work_phone' | 'private_phone' | 'mobile_phone'

    # Read paths
    def get_shifts(self, employee_id: int, since: datetime) -> list[Shift]: ...
    def get_employee_by_phone(self, phone_e164: str) -> Employee | None: ...

    # Write paths (idempotent on x_ngynapp_client_id)
    def upsert_shift_state(self, envelope: ShiftEnvelope) -> SyncAck: ...
    def upsert_attendance(self, envelope: AttendanceEnvelope) -> SyncAck: ...
    def upsert_gps_summary(self, shift_id: int, summary: GpsSummary) -> SyncAck: ...
    def upsert_ppe_result(self, envelope: PpeEnvelope) -> SyncAck: ...
    def upsert_preshift_form(self, envelope: FormEnvelope) -> SyncAck: ...
    def upload_attachment(self, envelope: AttachmentEnvelope) -> SyncAck: ...
```

`ProbeReport` follows the JSON contract in `data/odoo-x-fields-spec.md` §6. `SyncAck` carries `{odoo_id, write_date, sync_state, correlation_id}` — the mobile sees this as confirmation the write committed.

## 5. Probe behaviour

Triggered at:

- **Service startup.** Blocks readiness until probe completes or fails.
- **Every 15 minutes** via Celery beat (`tasks.odoo.refresh_capabilities`).
- **On admin action** via `POST /admin/odoo/refresh-capabilities`.

Probe steps:

1. Authenticate to Odoo via `xmlrpc/2/common authenticate` (or JSON-RPC equivalent).
2. `search_read` on `ir.module.module` for `state = 'installed'` to enumerate modules.
3. Choose `shift_model = 'fsm.task' if 'industry_fsm' installed else 'project.task'`.
4. `search_read` on `ir.model.fields` filtered by the model + `x_ngynapp_*` prefix; cross-reference against `data/odoo-x-fields-spec.md`.
5. Resolve `phone_field` per DEC-006: prefer `work_phone`, fallback `private_phone`, then `mobile_phone`. Record selection in cache.
6. Emit `ProbeReport`; cache for 15 minutes.

## 6. Write-path patterns

### 6.1 Idempotent upsert

Every write carries `x_ngynapp_client_id` (UUID v4 from mobile, ADR-007).

```
search_read on (shift_model, [('x_ngynapp_client_id','=', client_id)], limit=1)
if exists:
    write(id, payload)
else:
    create(payload)
```

The Redis SETNX layer in FastAPI prevents concurrent duplicate creates within a 15-second window; the `x_ngynapp_client_id` lookup is the durable backstop.

### 6.2 Read-after-write sync ack

After every mutating call, the adapter immediately reads the record back to fetch `id`, `write_date`, and `x_ngynapp_sync_state`. The combined value is returned as the `SyncAck` to the mobile envelope handler.

### 6.3 Capability-gated extras

Example check-out flow:

```
upsert_shift_state(client_id, end_at, gps_summary)
if supports('hr_attendance'):
    upsert_attendance(client_id, employee_id, check_out=end_at)
if supports('hr_timesheet'):
    create_analytic_line(task_id, hours_decimal)
upsert_gps_summary(shift_id, gps_summary)   # always runs
```

Each sub-write is independently idempotent; failures degrade gracefully — primary shift state always wins.

### 6.4 Attachment writes

Photos and voice notes are uploaded to DO Spaces directly via presigned URL (ADR-012). FastAPI then registers an `ir.attachment` record with metadata pointing to the object key:

```
res_model = shift_model
res_id    = task_id
name      = original_filename
mimetype  = ...
type      = 'binary'
db_datas  = NULL   # bytes live in Spaces, not Odoo
url       = signed Spaces URL (15 min) at write time; refreshed on read
x_ngynapp_origin, x_ngynapp_client_id, x_ngynapp_checksum_sha256, ...
```

Operators inside Odoo see the attachment row; clicking opens a presigned read URL via FastAPI proxy.

## 7. Field-level mapping by entity

The detail mapping for one tenant lives in `data/odoo-mapping.md`. Below is the version-portable contract used by every adapter implementation. Path-specific differences (`fsm.task` vs `project.task`) are flagged inline.

### 7.1 Employee

| Mobile field | Odoo source | Notes |
|---|---|---|
| `id` | `hr.employee.id` | Primary key |
| `name` | `hr.employee.name` | |
| `phone_e164` | `hr.employee.x_ngynapp_normalised_phone` | Pre-computed by setup; adapter validates format |
| `phone_source` | `hr.employee.x_ngynapp_phone_source` | Audit only |
| `department_name` | `hr.employee.department_id.name` | Joined via search_read |
| `job_title` | `hr.employee.job_title` | |
| `avatar_url` | derived from `hr.employee.image_1920` | FastAPI hydrates and uploads to Spaces |
| `active` | `hr.employee.active` | If false, app forces logout |

### 7.2 Shift

`shift_model` = `fsm.task` if `supports('industry_fsm')` else `project.task`.

| Mobile field | Odoo source | Notes |
|---|---|---|
| `id` | `<shift_model>.id` | |
| `client_id` | `x_ngynapp_client_id` | Mobile-supplied UUID |
| `title` | `name` | |
| `project_id`, `project_name` | `project_id.id`, `project_id.name` | Both models share this |
| `site_name` | `partner_id.name` (FSM) or fallback to manual address | FSM has native partner link |
| `site_address` | `partner_id.contact_address` (FSM) or `description` extract | |
| `start_at` | `planned_date_begin` | UTC |
| `end_at` | `planned_date_end` | UTC |
| `actual_start_at` | `x_ngynapp_actual_start` | |
| `actual_end_at` | `x_ngynapp_actual_end` | |
| `field_status` | `x_ngynapp_field_status` | Selection: draft / in_progress / completed / blocked / cancelled |
| `sync_state` | `x_ngynapp_sync_state` | Mirror of FastAPI authoritative state |
| `gps_check_in_*`, `gps_check_out_*` | `x_ngynapp_gps_*` summary fields | Full trail in FastAPI Postgres (ADR-016) |
| `evidence_count`, `voice_note_count` | `x_ngynapp_evidence_count`, `x_ngynapp_voice_note_count` | Maintained by adapter on attachment writes |

### 7.3 Attendance (only when `hr_attendance` present)

| Mobile field | Odoo source | Notes |
|---|---|---|
| `client_id` | `hr.attendance.x_ngynapp_client_id` | |
| `employee_id` | `hr.attendance.employee_id` | |
| `check_in` | `hr.attendance.check_in` | UTC |
| `check_out` | `hr.attendance.check_out` | UTC |
| `shift_id` | `hr.attendance.x_ngynapp_shift_id` | M2O to shift_model |
| `source` | `hr.attendance.x_ngynapp_source` | `mobile` |

### 7.4 PPE / preshift form (when `survey` present, else FastAPI-only)

When `supports('survey')`:

- PPE template = `survey.survey` configured by Odoo admin (one survey per template type).
- Each submission = `survey.user_input` + child `survey.user_input.line` rows.
- `x_ngynapp_purpose` set to `ppe` / `preshift` / `postshift`.
- `x_ngynapp_shift_id` links back to the shift.

When `survey` absent:

- Submission JSON stored in FastAPI Postgres (`form_submissions` table).
- `x_ngynapp_preshift_form_id` on the shift carries the FastAPI submission id (UUID).
- Operator opens the FastAPI dashboard to view contents (read-through link).

### 7.5 GPS

Per ADR-016: history in FastAPI Postgres, summary in Odoo. Full mapping in `data/odoo-x-fields-spec.md` §2.1 and `data/postgres-schema.md`.

### 7.6 Attachments

| Mobile field | Odoo source | Notes |
|---|---|---|
| `client_id` | `ir.attachment.x_ngynapp_client_id` | |
| `kind` | `ir.attachment.x_ngynapp_origin` | `photo` / `voice` / `signature` / `other` |
| `checksum_sha256` | `ir.attachment.x_ngynapp_checksum_sha256` | |
| `captured_at` | `ir.attachment.x_ngynapp_captured_at` | |
| `exif_lat`, `exif_lon` | `ir.attachment.x_ngynapp_exif_*` | Photos only |
| `voice_transcript` | `ir.attachment.x_ngynapp_voice_transcript` | Whisper output |
| `voice_duration_s` | `ir.attachment.x_ngynapp_voice_duration_s` | |
| `storage_url` | `ir.attachment.url` | Re-signed by FastAPI on each read |

## 8. Error handling and degradation

| Failure | Adapter behaviour |
|---|---|
| Probe cannot reach Odoo | Service unhealthy; mobile sync queue holds; alert raised |
| Hard field missing | `503` on writes; admin alert with remediation steps |
| Soft module missing | Silent degrade per matrix; warning in probe report |
| Odoo write returns RPC error | Retry with exponential backoff (max 5); after that → dead-letter in FastAPI; mobile sees `failed` state |
| Network flap during read-after-write | Cache the write intent; reconcile on next probe; mobile receives provisional ack with `pending` state |
| Attachment URL expired | Re-sign on read; never embed long-lived URLs in Odoo |

## 9. Security and credentials

- One Odoo service account per tenant. Username + API key (preferred) or password.
- Service account assigned the minimum groups required: `Internal User` + `HR Officer` + `Project User` + (`Field Service User` if FSM installed).
- API key rotated per `security/secrets.md` (90 days).
- Never commit credentials; FastAPI loads them from secrets manager (DO Vault / AWS Secrets Manager / HashiCorp Vault per environment).

## 10. Migration risks

| Risk | Mitigation |
|---|---|
| Customer upgrades Odoo major version (e.g., 19 → 20) | Adapter probes on next start; no code change unless module ids change. CI runs upgrade matrix yearly. |
| Customer installs Field Service mid-program | Adapter detects on next probe; switches `shift_model` to `fsm.task`; existing `project.task` rows continue to work via inheritance. Tracked under DEC-008. |
| Customer renames an `x_ngynapp_*` field via Studio | Probe detects mismatch; service refuses writes; admin alert fires with the missing-field list. |
| Customer disables `survey` after PPE went live there | Probe drops capability; PPE writes route to FastAPI Postgres; operator UX degrades but no data lost. |
| Customer migrates Online → self-host | API contract unchanged; adapter swaps endpoint + credentials. |

## 11. Decision-log impact

| Decision | Status after this doc |
|---|---|
| DEC-001 Odoo target | Decided (amended) — Odoo 19 EE for testing; RPC contract for production portability |
| DEC-002 Modules | Decided — capability detection at runtime; install only what you use |
| DEC-003 Custom module | Decided — **NO custom module**; ADR-014 superseded by ADR-015 |
| DEC-004 GPS storage | Decided — FastAPI Postgres + Odoo summary; ADR-013 superseded by ADR-016 |
| DEC-006 Phone resolution | Decided — `work_phone` → `private_phone` → `mobile_phone` fallback; recorded in `x_ngynapp_phone_source` |

## 12. S0 workshop checklist (Odoo Specialist)

For each candidate tenant:

1. Identify version (Online / Enterprise / Community + version number).
2. Enumerate installed modules; cross-check against §3 capability matrix.
3. Walk the admin through `data/odoo-x-fields-spec.md` Studio steps.
4. Run FastAPI probe; capture probe report; resolve any `errors` / `warnings`.
5. Provision Odoo service account + API key; store in environment vault.
6. Sign off `traceability/decision-log.md` rows for DEC-001/002.

## 13. Related

- ADRs: ADR-004 (FastAPI), ADR-005 (no direct mobile-to-Odoo), ADR-007 (idempotency), ADR-011 (Odoo target), ADR-012 (DO Spaces), ADR-013 (superseded), ADR-014 (superseded), ADR-015 (no custom module), ADR-016 (GPS in Postgres)
- Specs: `data/odoo-x-fields-spec.md`, `data/odoo-mapping.md`, `data/postgres-schema.md`
- Stories: US-DISC-001/002/003, US-ODOO-001..006, US-API-001..005
