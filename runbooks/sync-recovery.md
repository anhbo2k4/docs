# Runbook: Sync Recovery

**Status:** Active  
**Owner:** Backend Lead + DevOps

## Symptom

One or more of:

- Sync success rate < 95 % over 6 hours.
- Backend `sync_envelopes` table has growing `PENDING` or `PROCESSING` count.
- DEAD_LETTER count rising.
- Mobile users report rows stuck in `SYNCING` indefinitely.

## Severity

SEV-2 by default. SEV-1 if pilot site reports inability to submit.

## Diagnose

```sql
-- Backend: queue depth by state
SELECT state, count(*) FROM sync_envelopes GROUP BY state;

-- Recent envelopes that failed
SELECT id, type, error_code, error_message, received_at
FROM sync_envelopes
WHERE state IN ('DEAD_LETTER')
ORDER BY received_at DESC
LIMIT 50;

-- Stuck PROCESSING (should never exceed worker concurrency × 2)
SELECT count(*) FROM sync_envelopes WHERE state = 'PROCESSING'
AND received_at < now() - interval '5 minutes';
```

```bash
# Celery queue depth
celery -A app.workers.celery_app inspect active
celery -A app.workers.celery_app inspect reserved
celery -A app.workers.celery_app inspect stats
```

```bash
# Odoo health
curl -s https://odoo.internal/web/health
```

## Decision tree

```
Are envelopes piling up in PENDING?
├─ Yes: workers stalled.
│        → check Celery; restart workers; check Redis health.
└─ No → are they piling in PROCESSING?
         ├─ Yes: workers running but Odoo write hangs.
         │        → see Odoo unresponsive section.
         └─ No → check DEAD_LETTER trend.
                  ├─ Spike on one type: schema mismatch likely.
                  │        → see Validation storm section.
                  └─ Generic: investigate per-envelope.
```

## Workers stalled

```
1. Check Celery pod status:
   kubectl get pods -l app=celery
2. If pods OOMKilled or crashlooping:
   - Increase memory; investigate memory leak.
3. If pods running but no tasks consumed:
   - Check Redis connectivity from worker.
   - Check broker queue is correctly named.
4. Restart:
   kubectl rollout restart deploy/celery
5. Watch queue drain.
```

## Odoo unresponsive

```
1. Open circuit breaker manually:
   - Set ODOO_CIRCUIT_OPEN=true (env or feature flag).
   - All Odoo writes pause; envelopes remain PROCESSING.
2. Wait for Odoo to recover (or escalate to ERP team).
3. Close circuit; workers resume.
4. Verify exponential backoff prevented thundering herd.
```

## Validation storm (single type DEAD_LETTER spike)

```
1. Identify the type:
   SELECT type, error_code, count(*) FROM sync_envelopes
   WHERE state='DEAD_LETTER' AND received_at > now() - interval '1 hour'
   GROUP BY type, error_code;
2. Inspect a sample envelope payload:
   SELECT payload FROM sync_envelopes WHERE id = <id>;
3. If schema drifted (e.g. mobile sent extra field):
   - Patch backend validator to accept (best for emergency).
   - Or block the offending mobile build via min_app_version.
4. Replay valid DEAD_LETTER rows once root cause fixed.
```

## Replay DEAD_LETTER

Ops endpoint (admin-only):

```
POST /v1/admin/sync/replay
{ "envelope_ids": [123, 124, 125] }
```

Or SQL helper (use cautiously):

```sql
UPDATE sync_envelopes SET state='PENDING', attempts=0, error_code=NULL, error_message=NULL
WHERE id = ANY('{123,124,125}'::bigint[]);
-- Then poke worker to pick up.
```

## Mobile-side stuck SYNCING

If mobile shows rows stuck in SYNCING but backend has no record:

- Mobile has the recovery rule on resume:
  ```sql
  UPDATE sync_queue SET state='FAILED', last_error='INTERRUPTED' WHERE state='SYNCING';
  ```
- If user reports persistent stuck state, ask them to:
  1. Open Sync Center → tap Retry.
  2. Force quit, reopen — recovery rule fires on `AppLifecycleState.resumed`.

## Verify resolved

- Queue depth in `PENDING` and `PROCESSING` returns to baseline (< 100 for pilot).
- DEAD_LETTER additions stop or revert to noise floor.
- Sync success rate ≥ 99 % over last 30 min.
- No new alerts firing.

## After-action

- Open backlog item if a code defect caused the storm.
- Update this runbook if a step was missing or wrong.
- Add a metric / alert if the issue was hard to detect.
