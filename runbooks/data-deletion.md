# Runbook: Data Deletion (Right to be Forgotten)

**Status:** Active  
**Owner:** Compliance + DevOps

## When triggered

A field worker, employee, or compliance team requests deletion of personal data. SLA: **30 days** from request to completion.

## Authority

PO + Compliance authorise the request. DevOps executes. Audit log entries record actor and target.

## Scope

What gets deleted:

- Mobile local DB on the user's device (only if the user still has the app and logs out).
- Backend Postgres records: `sessions`, `audit_log` (older than legal minimum), `sync_envelopes`, `media_uploads` (rows referencing this employee).
- Object storage: photo + audio bytes referenced by this employee.
- Odoo: per ERP team's procedure (we file a ticket; Odoo team executes).
- Sentry: scrubbed automatically; manual remove on request.
- Logs: rotation policy will purge over time; we do not perform targeted log scrubs unless legally required.

What is **not** deleted:

- Audit log entries within legal retention (currently 7 years).
- Aggregate analytics (no PII).
- Records required for tax / regulatory retention.

## Steps

```
1. Open ticket: TKT-DEL-NNN with employee_id, requester, justification.
2. Verify identity (out of band) of the requester.
3. PO + Compliance sign off.
4. Mark employee in Odoo as "deletion-pending" (custom flag, custom module).
5. Mobile: next time the user opens the app, force-logout occurs (server returns SESSION_EXPIRED with reason=DELETION).
6. Backend cleanup:
   - DELETE sessions WHERE employee_id = <id>;
   - DELETE sync_envelopes WHERE employee_id = <id>;
   - DELETE media_uploads WHERE employee_id = <id>;
   - For each object_key in step above: delete from S3 (versioned bucket: delete all versions).
   - UPDATE audit_log SET actor_id=NULL, metadata = jsonb_set(metadata, '{phone}', '"REDACTED"')
        WHERE actor_id = <id>;  -- conservative: keep auditability
7. Odoo deletion ticket filed; ERP team executes.
8. Confirm completion in writing.
```

## Verify

- `SELECT count(*) FROM sync_envelopes WHERE employee_id = <id>` → 0.
- S3 listing under media prefix for this employee → empty.
- Audit log no longer contains direct PII.
- Confirmation email to requester.

## Audit trail

- Ticket TKT-DEL-NNN preserved indefinitely.
- A redacted entry in `audit_log` records: who requested, when authorised, when executed.

## Failure modes

- Odoo cannot delete (e.g. records referenced by closed time entries) → mark in Odoo as anonymised (replace fields with placeholders).
- S3 versioned object cannot delete (legal hold) → escalate to compliance.

## Post-deletion

- Cannot restore. Document this prominently in privacy policy.
- If requester later returns to the platform, they enrol fresh.
