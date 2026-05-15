# Odoo `x_ngynapp_*` Field Specification

**Status:** Authoritative
**Owners:** Backend Lead + Odoo Specialist
**Last updated:** 2026-05-15 (blueprint v2.1.0)
**Consumes:** [ADR-015](../adr/ADR-015-no-custom-module-rpc-adapter.md), [ADR-016](../adr/ADR-016-gps-in-fastapi-postgres.md)

This document is the single source of truth for the **manually-created Odoo fields** the integration depends on. The FastAPI startup probe reads this set and refuses to serve writes if any required field is missing.

The prefix `x_ngynapp_` is **reserved** for this program. Do not reuse it for unrelated customisations.

> **No custom module.** Per ADR-015, the solution must run on Odoo Online. Fields below are added by an Odoo administrator using **Studio** (Enterprise / Online) or **Developer mode → Settings → Technical → Fields** (Community).

---

## 1. Setup checklist (per Odoo tenant)

1. Login as Odoo administrator.
2. Confirm the required base modules are installed: `base`, `hr`, `project`, `contacts`. Install if missing.
3. Confirm optional modules (graceful degrade): `industry_fsm`, `planning`, `hr_attendance`, `hr_timesheet`, `survey`, `quality`, `maintenance`, `documents`. Install only those you intend to use.
4. Enter Studio (Enterprise/Online) or Developer mode (Community).
5. Open each target model below and add the listed fields with the exact names, types, and sizes.
6. Add the listed Selection options verbatim (case-sensitive technical values).
7. Run the FastAPI startup probe; review the report; remediate gaps.
8. Sign off by pasting the probe report into `traceability/decision-log.md` under the `Decided` row for DEC-002.

Estimated time on a fresh tenant: **20–30 minutes**.

---

## 2. Field set by model

### 2.1 `project.task` (or `fsm.task` if Field Service installed)

The shift entity. The adapter automatically targets `fsm.task` when `industry_fsm` is detected; both models inherit `project.task`, so the field set is identical.

| Field name | Type | Size / args | Required | Purpose |
|---|---|---|---|---|
| `x_ngynapp_client_id` | Char | 64 | Yes | Idempotency anchor (UUID v4 from mobile, ADR-007) |
| `x_ngynapp_sync_state` | Selection | options below | Yes | Sync state mirror (FastAPI is authoritative) |
| `x_ngynapp_sync_error` | Text | — | No | Last sync error summary (≤ 1 KB) |
| `x_ngynapp_synced_at` | Datetime | — | No | Last successful sync write time |
| `x_ngynapp_actual_start` | Datetime | — | Yes | Mobile check-in time (UTC) |
| `x_ngynapp_actual_end` | Datetime | — | Yes | Mobile check-out time (UTC) |
| `x_ngynapp_field_status` | Selection | options below | Yes | Mobile workflow state |
| `x_ngynapp_gps_check_in_lat` | Float | digits=(9,6) | Yes | Latitude at check-in |
| `x_ngynapp_gps_check_in_lon` | Float | digits=(9,6) | Yes | Longitude at check-in |
| `x_ngynapp_gps_check_in_at` | Datetime | — | No | Timestamp of check-in ping |
| `x_ngynapp_gps_check_out_lat` | Float | digits=(9,6) | Yes | Latitude at check-out |
| `x_ngynapp_gps_check_out_lon` | Float | digits=(9,6) | Yes | Longitude at check-out |
| `x_ngynapp_gps_check_out_at` | Datetime | — | No | Timestamp of check-out ping |
| `x_ngynapp_gps_last_lat` | Float | digits=(9,6) | No | Latest known position (interval pings) |
| `x_ngynapp_gps_last_lon` | Float | digits=(9,6) | No | Latest known position |
| `x_ngynapp_gps_last_at` | Datetime | — | No | Timestamp of latest ping |
| `x_ngynapp_gps_ping_count` | Integer | — | No | Total pings recorded for this shift |
| `x_ngynapp_gps_trail_url` | Char | 512 | No | Signed FastAPI dashboard URL (operators) |
| `x_ngynapp_evidence_count` | Integer | — | No | Photos + voice notes attached |
| `x_ngynapp_voice_note_count` | Integer | — | No | Voice notes attached |
| `x_ngynapp_ppe_passed` | Boolean | — | No | PPE checklist passed at start of shift |
| `x_ngynapp_preshift_form_id` | Char | 64 | No | FastAPI-side preshift form submission id |

**Selection options.**

`x_ngynapp_sync_state` (technical → display):
- `pending` → `Pending`
- `synced` → `Synced`
- `failed` → `Failed`
- `dead_letter` → `Dead Letter`

`x_ngynapp_field_status`:
- `draft` → `Draft`
- `in_progress` → `In Progress`
- `completed` → `Completed`
- `blocked` → `Blocked`
- `cancelled` → `Cancelled`

### 2.2 `hr.employee`

Employee identity is read-only from the integration. We add only sync-side metadata.

| Field name | Type | Size / args | Required | Purpose |
|---|---|---|---|---|
| `x_ngynapp_normalised_phone` | Char | 32 | Yes | E.164 normalised phone for OTP lookup |
| `x_ngynapp_phone_source` | Selection | options below | Yes | Which Odoo field supplied the phone |
| `x_ngynapp_app_user_id` | Char | 64 | No | Mobile app user id (FastAPI side) |
| `x_ngynapp_last_login_at` | Datetime | — | No | Last successful mobile login |

`x_ngynapp_phone_source` selection:
- `work_phone` → `Work Phone`
- `private_phone` → `Private Phone`
- `mobile_phone` → `Mobile Phone`
- `manual` → `Manual Override`

> **Phone resolution rule (DEC-006).** The adapter prefers `work_phone`; if empty, falls back to `private_phone`; finally `mobile_phone` if present. The chosen source is recorded in `x_ngynapp_phone_source`. Numbers are normalised to E.164 (`+84…`) at write time.

### 2.3 `ir.attachment`

Evidence files are attached natively. We add tagging fields so the adapter can filter sync-origin attachments.

| Field name | Type | Size / args | Required | Purpose |
|---|---|---|---|---|
| `x_ngynapp_origin` | Selection | options below | Yes | Source of the attachment |
| `x_ngynapp_client_id` | Char | 64 | Yes | Mobile-side attachment UUID for idempotency |
| `x_ngynapp_checksum_sha256` | Char | 64 | Yes | SHA-256 of payload bytes |
| `x_ngynapp_captured_at` | Datetime | — | No | When the device captured it |
| `x_ngynapp_exif_lat` | Float | digits=(9,6) | No | EXIF latitude (photos) |
| `x_ngynapp_exif_lon` | Float | digits=(9,6) | No | EXIF longitude (photos) |
| `x_ngynapp_voice_transcript` | Text | — | No | Whisper transcript (voice notes) |
| `x_ngynapp_voice_duration_s` | Float | digits=(6,2) | No | Voice note duration in seconds |

`x_ngynapp_origin` selection:
- `photo` → `Photo`
- `voice` → `Voice Note`
- `signature` → `Signature`
- `other` → `Other`

### 2.4 `hr.attendance` (optional — only if `hr_attendance` installed)

| Field name | Type | Size / args | Required if module present | Purpose |
|---|---|---|---|---|
| `x_ngynapp_client_id` | Char | 64 | Yes | Idempotency anchor |
| `x_ngynapp_source` | Selection | `mobile` / `web` / `kiosk` | Yes | Origin of the attendance record |
| `x_ngynapp_shift_id` | Many2one | `project.task` (or `fsm.task`) | No | Link back to the shift |

### 2.5 `survey.user_input` (optional — only if `survey` used for PPE / preshift)

| Field name | Type | Size / args | Required if module present | Purpose |
|---|---|---|---|---|
| `x_ngynapp_client_id` | Char | 64 | Yes | Idempotency anchor |
| `x_ngynapp_shift_id` | Many2one | `project.task` (or `fsm.task`) | Yes | Link back to the shift |
| `x_ngynapp_purpose` | Selection | `ppe` / `preshift` / `postshift` | Yes | Which mobile flow generated this |

---

## 3. Required vs optional summary

The FastAPI startup probe classifies fields by hardness:

- **Hard:** all fields on `project.task` (or `fsm.task`), `hr.employee`, and `ir.attachment` marked Required = Yes. Probe failure → service refuses to serve writes; admin alert emitted with the missing-field list and Studio steps.
- **Soft:** fields on `hr.attendance` and `survey.user_input` are required only when their host module is installed and selected for sync.
- **Best-effort:** all fields marked Required = No are advisory; missing them degrades operator UX but does not block sync.

---

## 4. Studio setup steps (Enterprise / Online)

For each field above:

1. Open the target model's list/form view.
2. Click **Studio** (top-right toolbox icon).
3. Drag a new field of the appropriate type onto the form.
4. In the field properties panel, set:
   - **Field name (technical):** the exact `x_ngynapp_*` name (Studio will prefix `x_studio_` by default — override).
   - **Type:** match the spec exactly.
   - **Required:** check if marked Yes above.
   - **Selection options:** add each `(technical, display)` pair.
   - **Size / digits:** match the spec.
5. Save. Studio creates the field via `ir.model.fields` write.

> **Tip.** For Selection fields, the technical value (left side of `→`) must match exactly — the adapter compares case-sensitively.

## 5. Developer-mode setup steps (Community)

1. Enable Developer mode: Settings → Activate Developer Mode.
2. Settings → Technical → Database Structure → Models → select target model.
3. Add fields via the **Fields** tab; use the same names and types.
4. For Selection fields, populate the **Selection Options** tab.
5. Save; Odoo regenerates the model schema.

---

## 6. Probe report contract

The FastAPI startup probe emits a JSON report:

```json
{
  "tenant_url": "https://acme.odoo.com",
  "odoo_version": "19.0",
  "edition_hint": "online",
  "modules_present": ["hr", "project", "industry_fsm", "planning", "survey"],
  "modules_absent": ["quality", "maintenance", "documents"],
  "shift_model": "fsm.task",
  "phone_field": "work_phone",
  "fields_required_present": [...],
  "fields_required_missing": [],
  "fields_optional_present": [...],
  "ready_for_writes": true,
  "warnings": [],
  "errors": []
}
```

`ready_for_writes: false` means at least one Hard field is missing; the API returns `503 Service Unavailable` with a `Retry-After` header until remediated.

## 7. Change control

- Adding a new `x_ngynapp_*` field: open an ADR amendment + update this spec + bump program version.
- Renaming or removing a field: same as a breaking schema change — see `engineering/api-versioning.md`.
- Customer-side ad-hoc additions outside this spec are tolerated as long as they do not collide with reserved names.

## 8. Related

- ADRs: ADR-007 (idempotency), ADR-011 (Odoo as system of record), ADR-015 (no custom module — RPC adapter), ADR-016 (GPS in FastAPI Postgres)
- Stories: US-DISC-001 (Odoo workshop), US-DISC-002 (employee mapping), US-DISC-003 (module scope), US-ODOO-001..006
- Decisions: DEC-001 (Odoo target), DEC-002 (modules — capability detection), DEC-003 (no custom module), DEC-004 (GPS in Postgres), DEC-006 (phone resolution)
