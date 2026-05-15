# Odoo Mapping

**Status:** Draft — depends on DEC-001, DEC-002, DEC-003, DEC-004  
**Owner:** Backend Lead + Odoo Specialist

This document defines how mobile entities map to Odoo records. Sections marked **[TBD]** depend on Phase 0 decisions.

## Authoritative direction

- **Reads** (shift list, employee profile): Odoo is source of truth. FastAPI caches in Postgres for performance.
- **Writes** (PPE, forms, GPS, work results, voice): Mobile is source of truth at capture time. FastAPI persists envelope, Celery writes to Odoo, Odoo becomes long-term system of record. `client_id` is the bridge.

## Entity mapping

### Employee

| Mobile field | Odoo model | Odoo field | Notes |
|---|---|---|---|
| `id` | `hr.employee` | `id` | Primary key |
| `name` | `hr.employee` | `name` | |
| `phone` | `hr.employee` | `mobile_phone` or `work_phone` (DEC-006) | Lookup field for OTP login |
| `department` | `hr.department` | `name` | Joined via `department_id` |
| `job_title` | `hr.employee` | `job_title` | |
| `avatar_url` | `hr.employee` | `image_1920` | Resized + uploaded to S3 by FastAPI |

### Shift

The shift entity could be `project.task` or `planning.slot` depending on installed modules. **DEC-002 must be closed first.**

If using `project.task` (Project module):

| Mobile field | Odoo field | Notes |
|---|---|---|
| `id` | `id` | |
| `title` | `name` | |
| `site_name`, `site_address` | custom fields on `project.project` | Add via `field_mobile_sync` |
| `project_id` | `project_id.id` | |
| `project_name` | `project_id.name` | |
| `start_at` | `planned_date_begin` | |
| `end_at` | `planned_date_end` | |
| `actual_start_at` | custom `x_actual_start` | Added by `field_mobile_sync` |
| `actual_end_at` | custom `x_actual_end` | Added by `field_mobile_sync` |
| `status` | `state` (mapped) | DRAFT / IN_PROGRESS / COMPLETED |

If using `planning.slot` (Planning module): TBD — see DEC-002.

### PPE check

Stored in a custom Odoo model `field_mobile_sync.ppe_check`:

```python
class PpeCheck(models.Model):
    _name = 'field_mobile_sync.ppe_check'
    _description = 'PPE Check'
    _rec_name = 'external_id'

    external_id     = fields.Char(required=True, index=True, copy=False)  # mobile client_id
    employee_id     = fields.Many2one('hr.employee', required=True)
    task_id         = fields.Many2one('project.task', required=True)
    items_json      = fields.Text(required=True)
    notes           = fields.Text()
    captured_at     = fields.Datetime(required=True)
    received_at     = fields.Datetime(default=fields.Datetime.now)
```

Unique constraint on `external_id` (per employee).

### Form response (Pre / Post shift)

Stored in `field_mobile_sync.form_response`:

```python
class FormResponse(models.Model):
    _name = 'field_mobile_sync.form_response'
    _description = 'Field form response'

    external_id     = fields.Char(required=True, index=True, copy=False)
    employee_id     = fields.Many2one('hr.employee', required=True)
    task_id         = fields.Many2one('project.task', required=True)
    form_type       = fields.Selection([('PRE_SHIFT','Pre'),('POST_SHIFT','Post')], required=True)
    schema_version  = fields.Char(required=True)
    answers_json    = fields.Text(required=True)
    submitted_at    = fields.Datetime(required=True)
```

### GPS event

Stored in custom model `field_mobile_sync.gps_event` (DEC-004 default = custom model):

```python
class GpsEvent(models.Model):
    _name = 'field_mobile_sync.gps_event'

    external_id     = fields.Char(required=True, index=True, copy=False)
    employee_id     = fields.Many2one('hr.employee', required=True)
    task_id         = fields.Many2one('project.task', required=True)
    event_type      = fields.Selection([('CHECK_IN','Check in'),('CHECK_OUT','Check out'),('WAYPOINT','Waypoint')], required=True)
    latitude        = fields.Float(digits=(9,6), required=True)
    longitude       = fields.Float(digits=(9,6), required=True)
    accuracy_m      = fields.Float()
    captured_at     = fields.Datetime(required=True)
```

### Work result

Stored in `field_mobile_sync.work_result`:

```python
class WorkResult(models.Model):
    _name = 'field_mobile_sync.work_result'

    external_id     = fields.Char(required=True, index=True, copy=False)
    employee_id     = fields.Many2one('hr.employee', required=True)
    task_id         = fields.Many2one('project.task', required=True)
    notes           = fields.Text()
    completion_pct  = fields.Integer()
    submitted_at    = fields.Datetime(required=True)
    media_ids       = fields.One2many('field_mobile_sync.media', 'work_result_id')
    voice_ids       = fields.One2many('field_mobile_sync.voice_note', 'work_result_id')
```

### Media (photos)

Stored in `field_mobile_sync.media`:

```python
class Media(models.Model):
    _name = 'field_mobile_sync.media'

    external_id     = fields.Char(required=True, index=True, copy=False)
    work_result_id  = fields.Many2one('field_mobile_sync.work_result')
    employee_id     = fields.Many2one('hr.employee', required=True)
    task_id         = fields.Many2one('project.task', required=True)
    kind            = fields.Selection([('PHOTO','Photo'),('VIDEO','Video')], required=True)
    object_key      = fields.Char(required=True)        # S3 key
    sha256          = fields.Char(required=True)
    bytes           = fields.Integer()
    mime_type       = fields.Char()
    captured_at     = fields.Datetime(required=True)
```

The byte content lives in S3, not in Odoo. We store metadata only.

### Voice note

Stored in `field_mobile_sync.voice_note`:

```python
class VoiceNote(models.Model):
    _name = 'field_mobile_sync.voice_note'

    external_id     = fields.Char(required=True, index=True, copy=False)
    work_result_id  = fields.Many2one('field_mobile_sync.work_result')
    employee_id     = fields.Many2one('hr.employee', required=True)
    task_id         = fields.Many2one('project.task', required=True)
    object_key      = fields.Char(required=True)
    duration_ms     = fields.Integer()
    transcript      = fields.Text()
    captured_at     = fields.Datetime(required=True)
```

## Idempotency at the Odoo layer

Every custom model has `external_id` (the mobile `client_id`) with a unique constraint per employee. Celery's write code:

```python
existing = self.env['field_mobile_sync.ppe_check'].search([
    ('employee_id', '=', emp_id),
    ('external_id', '=', client_id),
], limit=1)
if existing:
    existing.write(payload)
    return existing.id
return self.env['field_mobile_sync.ppe_check'].create(payload | {'external_id': client_id, 'employee_id': emp_id}).id
```

## Read paths

| Mobile request | FastAPI action | Odoo call |
|---|---|---|
| `GET /v1/shifts` | Cache lookup (TTL 60 s); on miss, fetch from Odoo | `project.task.search_read([('user_ids','in', [user_id])], fields=[...])` |
| `GET /v1/shifts/{id}` | Cache lookup; on miss, fetch from Odoo | `project.task.read([id], fields=[...])` |
| `GET /v1/employees/me` | From session | `hr.employee.read([emp_id], fields=[...])` |

## Write paths (Celery)

Each envelope type has a Celery task. Pseudocode:

```python
@celery.task(bind=True, max_retries=6, default_retry_delay=60)
def write_ppe_check(self, envelope_id: int):
    env = SyncEnvelope.get(envelope_id)
    try:
        odoo_id = odoo_client.upsert_ppe_check(env.payload, external_id=env.client_id)
        env.mark_confirmed(odoo_ref=f'field_mobile_sync.ppe_check,{odoo_id}')
    except OdooValidationError as e:
        env.mark_dead_letter(error=e)
    except OdooTransientError as e:
        raise self.retry(exc=e, countdown=backoff(self.request.retries))
```

## Custom Odoo module: `field_mobile_sync`

Status: **proposed, must be confirmed via DEC-003**.

Scope:
- Custom models above.
- Helper actions for ops (re-process dead-letter, dedup).
- Read views in Odoo backend so admins can audit submissions.
- Endpoint stubs (if any RPC custom methods are needed).

Owner: Odoo Specialist (vendor or internal).

## Open questions

- DEC-001 Odoo version
- DEC-002 Modules installed
- DEC-003 `field_mobile_sync` build / scope
- DEC-004 GPS storage model
- DEC-006 Phone-to-employee mapping field
