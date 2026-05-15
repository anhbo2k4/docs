# Test Cases

**Status:** Active  
**Owner:** QA Lead

Catalog of test cases. Each case lists **steps**, **expected**, **owner**, **automation**, **traceability**. Stored as a single file for quick lookup; consider splitting per area in S5 if it exceeds 1500 lines.

## AUTH

### TC-AUTH-001 — OTP request happy path

- Pre: phone `+84912345678` is mapped to an active employee.
- Steps: enter phone, tap Send.
- Expected: 200 from `/request_otp`, SMS arrives within 30 s, "code sent" UI.
- Automation: integration_test with mocked Twilio.
- FR: FR-001. Story: US-AUTH-001.

### TC-AUTH-002 — Invalid phone format

- Steps: enter `0912` and tap Send.
- Expected: client-side validation error; no network call.
- Automation: widget test.
- FR: FR-001.

### TC-AUTH-003 — Verify OTP happy path

- Pre: TC-AUTH-001 completed.
- Steps: enter received code.
- Expected: 200 from `/verify_otp`, then 200 from `/v1/auth/exchange`, app routes to My Shifts; tokens persisted in secure storage.
- Automation: integration_test.
- FR: FR-002.

### TC-AUTH-004 — Wrong OTP attempts

- Steps: enter wrong code 5 times.
- Expected: each rejected with `INVALID_CODE`; on 5th, `OTP_LOCKED`; UI shows 5-min cooldown.
- Automation: integration_test.
- FR: FR-001/002. NFR-042.

### TC-AUTH-005 — OTP expired

- Steps: wait 5 min after request, enter code.
- Expected: `OTP_EXPIRED`; UI suggests re-request.
- FR: FR-001.

### TC-AUTH-006 — Employee not mapped

- Pre: phone not in `hr.employee`.
- Steps: complete OTP flow.
- Expected: 403 `EMPLOYEE_NOT_MAPPED`; UI shows "contact admin".
- FR: FR-002.

### TC-AUTH-007 — Token refresh transparent

- Steps: simulate access token expiry while on My Shifts; trigger an API call.
- Expected: 401 received, refresh executed, original call retried, success.
- Automation: integration_test with clock injection.
- FR: FR-003.

### TC-AUTH-008 — Logout wipes data

- Steps: tap Logout.
- Expected: refresh revoked server-side; secure storage cleared; SQLite tables emptied; routed to login.
- Automation: integration_test.
- FR: FR-003. NFR-044.

### TC-AUTH-041 — OTP rate limit per device

- Steps: request OTP 4× within 5 min from same device.
- Expected: 4th request → 429 `RATE_LIMIT`.
- Automation: backend integration test.
- NFR-042.

## SHIFT

### TC-SHIFT-001 — List shifts happy path

- Pre: 3 shifts assigned to logged-in employee (1 SCHEDULED, 1 IN_PROGRESS, 1 COMPLETED).
- Steps: open My Shifts.
- Expected: list shows all 3, sorted by `start_at`, with sync state badges; cache TTL respected.
- FR: FR-004.

### TC-SHIFT-002 — List pagination

- Pre: 60 shifts in range.
- Steps: scroll to end.
- Expected: cursor-based pagination loads next page.
- FR: FR-004.

### TC-SHIFT-003 — Detail happy path

- Steps: tap a shift.
- Expected: detail screen shows title, site, time, instructions; CTAs to PPE / Pre-shift / Work result / Post-shift.
- FR: FR-005.

### TC-SHIFT-004 — Not assigned

- Steps: deep-link to a shift not assigned to user.
- Expected: 403 `NOT_ASSIGNED`; UI shows "shift not available".
- FR: FR-005.

## PPE

### TC-PPE-001 — Happy offline path

- Pre: airplane mode.
- Steps: complete checklist, submit.
- Expected: row inserted into `ppe_check` (PENDING), sync_queue row enqueued, badge PENDING.
- FR: FR-006.

### TC-PPE-002 — Sync after reconnect

- Pre: TC-PPE-001 completed.
- Steps: enable network.
- Expected: queue drains, row CONFIRMED within 30 s.
- FR: FR-006.

### TC-PPE-003 — Idempotent retry

- Steps: force network failure mid-sync, then succeed.
- Expected: only one Odoo record created.
- FR: FR-006/014. NFR-024.

## FORM

### TC-FORM-001 — Pre-shift submit happy path

- Steps: fill all fields, submit.
- Expected: row inserted, queued, eventually CONFIRMED.
- FR: FR-007.

### TC-FORM-002 — Post-shift submit happy path

- Steps: at shift end, fill post-shift form.
- Expected: same as above; shift `actual_end_at` set on CONFIRMED.
- FR: FR-007.

## GPS

### TC-GPS-001 — Check-in capture happy path

- Steps: tap "Start shift" (consent prompt → grant), GPS captured.
- Expected: `gps_event` row PENDING with lat/lon/accuracy; shift moves to IN_PROGRESS locally.
- FR: FR-008.

### TC-GPS-002 — Permission denied

- Steps: deny location permission.
- Expected: UI explains; cannot start shift; user can re-grant in settings.
- FR: FR-008. NFR-050.

### TC-GPS-003 — Low accuracy warning

- Steps: indoor where accuracy > 50 m.
- Expected: UI warns user, asks to retry; if confirmed, captures with accuracy field.
- FR: FR-008.

## MEDIA

### TC-MEDIA-001 — Capture and compress

- Steps: take photo on mid device.
- Expected: compressed file ≤ 500 KB; SHA-256 stored; row PENDING.
- NFR-005.

### TC-MEDIA-002 — Multiple photos in work result

- Steps: capture 5 photos, submit work result.
- Expected: all uploaded via presign, then envelope CONFIRMED.
- FR: FR-009/010.

### TC-MEDIA-003 — Upload retry on 5xx

- Steps: simulate S3 5xx on first attempt.
- Expected: retry up to 6 times; eventually success.

### TC-MEDIA-004 — Bytes mismatch

- Steps: spoof bytes count in finalize call.
- Expected: 409 `BYTES_MISMATCH`.

### TC-MEDIA-005 — SHA-256 mismatch

- Expected: 409 `SHA256_MISMATCH`.

### TC-MEDIA-006 — Work result rejected when media not finalized

- Expected: 422 `MEDIA_NOT_FINALIZED`.

## VOICE

### TC-VOICE-001 — Record happy path

- Steps: hold record, release after 30 s.
- Expected: audio file persisted, transcript starts processing.

### TC-VOICE-002 — Transcript accuracy on benchmark

- Pre: 10-clip benchmark set.
- Expected: WER ≤ 25 % on tiny.en (acceptable for MVP).

### TC-VOICE-003 — Long recording (60 s)

- Expected: completes within NFR-006 budget on mid device.

### TC-VOICE-004 — Permission denied

- Expected: UI explains; cannot record.

### TC-VOICE-005 — Whisper latency NFR-006

- Expected: ≤ 25 s for 60 s audio on Pixel 6a.

## OFF

### TC-OFF-001 — Survives app kill mid-capture

- Steps: kill app while pre-shift form half-filled.
- Expected: drafts (if implemented) restored; SQLite consistent.

### TC-OFF-002 — Survives 72 h offline

- Steps: collect 30 envelopes over 72 h offline, then connect.
- Expected: all CONFIRMED within 10 min.
- NFR-025.

### TC-OFF-003 — DB integrity after force-kill

- Expected: no SYNCING rows left after recovery; all reset to FAILED then drained.

### TC-OFF-004 — Capacity test

- Steps: 1000 sync_queue rows.
- Expected: no UI lag; queue drains in order.

### TC-OFF-005 — Disk full

- Steps: simulate disk-full on photo write.
- Expected: friendly error; no DB corruption.

## SYNC

### TC-SYNC-001 — PENDING after first POST

- Pre: authenticated session, valid `client_id` (UUID v4), shift assigned.
- Steps: POST `/v1/sync/envelope` with PPE_CHECK envelope.
- Expected: 202; body `{status: "PENDING"}`; Postgres `sync_envelope` row exists with `status=PENDING`; Celery job enqueued in Redis.
- Automation: backend integration test (pytest + httpx + Redis fakeredis).
- Owner: Backend QA. FR: FR-012/014. NFR: NFR-024.

### TC-SYNC-002 — Idempotent retry returns existing status

- Pre: TC-SYNC-001 produced `client_id=X` with status PENDING.
- Steps: POST same envelope (identical body) twice within 5 s.
- Expected: both calls return same `client_id`; second response includes current status (`PENDING` or `CONFIRMED`); only one Postgres row; only one Celery job dispatched.
- Automation: backend integration test.
- FR: FR-014. ADR-007.

### TC-SYNC-003 — 409 on payload change for same client_id

- Steps: POST envelope with `client_id=X, payload={notes:"a"}`, then POST again with `client_id=X, payload={notes:"b"}`.
- Expected: second call returns 409 `CLIENT_ID_CONFLICT`; original payload preserved; Sentry breadcrumb logged on mobile side.
- Automation: backend integration test + mobile widget test asserting Sentry capture.
- FR: FR-014.

### TC-SYNC-004 — DEAD_LETTER on validation 4xx

- Pre: envelope with malformed `payload.items[0].key` (>64 chars).
- Steps: POST envelope; wait for Celery to process.
- Expected: HTTP 202 initially; after Celery validation fails, status moves to `DEAD_LETTER` with `error_code=VALIDATION_FAILED`; mobile sees DEAD_LETTER badge after status poll.
- Automation: backend e2e test driving real Celery worker against test broker.
- FR: FR-015.

### TC-SYNC-005 — Status poll happy path

- Pre: `client_id=X` is CONFIRMED.
- Steps: GET `/v1/sync/status/X`.
- Expected: 200; `status=CONFIRMED`; `odoo_ref` populated; `received_at <= processed_at`.
- Automation: backend integration test.
- FR: FR-015.

### TC-SYNC-006 — Envelope type PPE_CHECK end-to-end

- Steps: capture PPE check, sync, observe Odoo.
- Expected: row in `field_mobile_sync.ppe_check` with all items, `external_id=client_id`, linked to correct `hr.employee`.
- Automation: backend e2e + Odoo fixture.
- FR: FR-006/014.

### TC-SYNC-007 — Envelope type FORM_RESPONSE

- Steps: submit pre-shift form (10 questions answered).
- Expected: Odoo `field_mobile_sync.form_response` row with answers JSON intact; schema_version stored.
- Automation: backend e2e.
- FR: FR-007/014.

### TC-SYNC-008 — Envelope type GPS_EVENT

- Steps: submit GPS check-in envelope (lat/lng/accuracy).
- Expected: row in target Odoo model (per DEC-004); Sync Center shows CONFIRMED.
- Automation: backend e2e.
- FR: FR-008/014/015.

### TC-SYNC-009 — Envelope type WORK_RESULT

- Steps: submit work result envelope referencing 3 finalized media + 1 voice note.
- Expected: Odoo `field_mobile_sync.work_result` row links all media via `client_id`; envelope CONFIRMED.
- Automation: backend e2e.
- FR: FR-010/014.

### TC-SYNC-010 — Envelope type VOICE_NOTE

- Steps: submit voice note envelope (transcript + duration_ms + media_client_id of finalized audio).
- Expected: Odoo `field_mobile_sync.voice_note` row created with transcript and audio reference.
- Automation: backend e2e.
- FR: FR-011/014.

### TC-SYNC-011 — Backoff respects retry-after

- Pre: backend returns 503 `ODOO_UNAVAILABLE` on first 2 attempts.
- Steps: capture envelope.
- Expected: mobile uses exponential backoff (2 s, 4 s, 8 s caps at 60 s); 3rd attempt succeeds; sync_queue retry counter logged.
- Automation: integration test with mock backend.
- FR: FR-013.

### TC-SYNC-012 — Circuit breaker opens after N failures

- Pre: backend returns 5xx 5× in a row.
- Steps: enqueue 10 envelopes.
- Expected: after 5 failures, mobile circuit opens for 60 s; no further requests sent during open state; Sync Center shows "Backing off"; circuit half-opens, single probe, succeeds, closes.
- Automation: integration test.
- FR: FR-013/015.

### TC-SYNC-013 — Background sync resumes after app kill

- Pre: 5 PENDING envelopes; force-quit app.
- Steps: WorkManager (Android) / BGTaskScheduler (iOS) fires next scheduled window.
- Expected: queue drains; final state CONFIRMED; UI on next launch reflects state.
- Automation: device test (Patrol or Maestro).
- FR: FR-013.

### TC-SYNC-014 — Manual retry from Sync Center

- Pre: 1 envelope in `FAILED` (transient 5xx, exhausted retries).
- Steps: open Sync Center, tap Retry.
- Expected: envelope re-enters PENDING; new attempt succeeds; row CONFIRMED.
- Automation: widget test + integration test.
- FR: FR-015.

### TC-SYNC-015 — Sync Center shows accurate counts

- Pre: 4 PENDING, 2 SYNCING, 1 FAILED, 10 CONFIRMED, 1 DEAD_LETTER.
- Steps: open Sync Center.
- Expected: tile counts match SQLite query; tapping each tile filters list correctly.
- Automation: widget test.
- FR: FR-015.

### TC-SYNC-016 — Network change triggers drain

- Steps: airplane mode on with 5 PENDING; toggle off.
- Expected: connectivity listener fires; queue drains within 30 s.
- Automation: integration test with mock connectivity stream.
- FR: FR-013.

### TC-SYNC-017 — DEAD_LETTER survives logout

- Pre: 1 DEAD_LETTER row with PII redacted.
- Steps: logout, login as same user.
- Expected: per security policy, SQLite is wiped on logout; DEAD_LETTER row gone; backend retains record (queryable by support).
- Automation: integration test.
- FR: FR-003/015. NFR-044.

### TC-SYNC-018 — Concurrent capture from two screens

- Steps: PPE submit + GPS check-in within 100 ms of each other.
- Expected: both rows enqueued; no race on `sync_queue` PK; both reach CONFIRMED.
- Automation: integration test with concurrent futures.
- FR: FR-012/013.

### TC-SYNC-019 — Envelope >256 KB rejected at client

- Steps: attempt to enqueue envelope with payload 300 KB (e.g. base64 image inline).
- Expected: client guard rejects with friendly error; envelope not stored; user prompted to use media upload path.
- Automation: unit test on envelope builder.
- FR: FR-014.

### TC-SYNC-020 — Clock skew tolerated

- Pre: device clock skewed by +90 s vs NTP.
- Steps: capture and sync envelope.
- Expected: backend `received_at` is server clock; `captured_at` preserved; no rejection.
- Automation: backend integration test with skewed Date header.
- FR: FR-014.

### TC-SYNC-021 — schema_version forward-compat reject

- Pre: backend supports schema_version 1; client posts schema_version 99.
- Steps: POST envelope.
- Expected: 422 `UNSUPPORTED_SCHEMA_VERSION`; client logs and surfaces upgrade hint.
- Automation: backend integration test.
- FR: FR-014.

### TC-SYNC-022 — Token expired mid-sync

- Pre: access token expires while envelope in flight.
- Steps: POST envelope; receive 401 `TOKEN_EXPIRED`.
- Expected: mobile auto-refreshes once, retries POST, succeeds; no user prompt.
- Automation: integration test with clock injection.
- FR: FR-003/013.

### TC-SYNC-023 — Logout clears in-flight queue

- Pre: 3 SYNCING envelopes.
- Steps: tap Logout.
- Expected: in-flight requests cancelled; SQLite wiped; backend retains envelopes (idempotent on next login from same device unlikely but harmless).
- Automation: integration test.
- FR: FR-003.

### TC-SYNC-024 — Retry storm (50 concurrent retries) yields exactly one Odoo record

- Pre: 50 mobile clients with the same `client_id` (test harness simulates resend storm).
- Steps: each posts the envelope concurrently.
- Expected: exactly 1 Postgres row, 1 Celery dispatch, 1 Odoo record; all callers receive consistent status.
- Automation: backend load test (locust + assertion).
- FR: FR-014. NFR-024.

### TC-SYNC-025 — Dead-letter inspector available to support

- Pre: 5 DEAD_LETTER rows for an employee.
- Steps: support uses backend admin endpoint `GET /v1/admin/dead_letters?employee_id=42`.
- Expected: returns rows with masked PII (phone partial, no full payload to non-admin); admin role returns full payload.
- Automation: backend integration test.
- FR: FR-015. NFR-051.

## ODOO

### TC-ODOO-001 — PPE write idempotent

- Pre: `field_mobile_sync.ppe_check` empty.
- Steps: Celery processes 1 envelope; reprocesses same envelope.
- Expected: 1 row in Odoo; `external_id=client_id`; second invocation no-ops.
- Automation: backend integration with Odoo container.
- FR: FR-006/014.

### TC-ODOO-002 — Form response write

- Steps: Celery handles FORM_RESPONSE; writes `field_mobile_sync.form_response`.
- Expected: answers JSON exactly preserved; `form_type` and `schema_version` stored.
- FR: FR-007/014.

### TC-ODOO-003 — GPS event write

- Steps: Celery writes GPS_EVENT to target model (per DEC-004).
- Expected: lat/lng with 6 decimals; `accuracy_m` integer; `event_type` ∈ {`CHECK_IN`,`CHECK_OUT`}.
- FR: FR-008/014.

### TC-ODOO-004 — Work result write

- Pre: 3 finalized media rows with `external_id` populated in Odoo.
- Steps: Celery handles WORK_RESULT.
- Expected: `field_mobile_sync.work_result` references 3 media via M2M; voice note linked if present.
- FR: FR-010/014.

### TC-ODOO-005 — Voice note write

- Steps: Celery handles VOICE_NOTE.
- Expected: transcript stored; audio reference resolves to S3 URL or Odoo attachment per DEC-007.
- FR: FR-011/014.

### TC-ODOO-006 — Media finalize creates ir.attachment

- Pre: media uploaded to S3 with valid SHA-256.
- Steps: Celery on MEDIA_FINALIZE creates `ir.attachment` with `res_model` and `res_id` per envelope target.
- Expected: 1 attachment per media; reusing same client_id is no-op.
- FR: FR-009.

### TC-ODOO-007 — Odoo down → exponential backoff

- Pre: Odoo container stopped.
- Steps: Celery retries 3× with backoff 30s/2m/10m.
- Expected: after 3 fails, envelope status remains PROCESSING; alert fires (`OdooUnreachable`); on Odoo recovery, envelope succeeds without re-enqueue from mobile.
- Automation: backend integration test.
- NFR: NFR-026.

### TC-ODOO-008 — Schema mismatch surfaces clearly

- Pre: Odoo missing `field_mobile_sync.ppe_check`.
- Steps: Celery attempts write.
- Expected: envelope marked DEAD_LETTER with `error_code=ODOO_MODEL_MISSING`; runbook `runbooks/sync-recovery.md` referenced.
- FR: FR-015. ADR-005.

### TC-ODOO-009 — Odoo write p95 ≤ 2 s

- Pre: load profile 10 envelopes/min sustained for 30 min.
- Steps: measure Celery → Odoo write latency.
- Expected: p95 ≤ 2.0 s, p99 ≤ 5.0 s; failure rate < 0.1 %.
- Automation: load test in nightly CI.
- NFR: NFR-013.

### TC-ODOO-010 — RPC timeout enforced

- Pre: Odoo simulated to hang on RPC.
- Steps: Celery worker invokes JSON-RPC.
- Expected: timeout at 30 s; raises retryable error; envelope stays PROCESSING.
- Automation: backend integration test.
- NFR: NFR-013.

### TC-ODOO-011 — Auth via service account

- Pre: Celery uses Odoo service account (no per-user impersonation in MVP).
- Expected: writes attribute `created_by_employee_id` field; service account permissions audited.
- Automation: security review checklist.
- NFR: NFR-046.

### TC-ODOO-024 — Re-process same client_id no duplicate

- Pre: 1 envelope already CONFIRMED.
- Steps: replay Celery message.
- Expected: handler observes `external_id` already exists; no-op; status remains CONFIRMED.
- Automation: backend integration test.
- FR: FR-014.

## SEC

### TC-SEC-001 — Secret scanning blocks PR

- Steps: open a PR introducing `SUPABASE_SERVICE_KEY=...` literal.
- Expected: pre-commit hook fails locally (gitleaks); CI fails on PR with link to remediation.
- Automation: CI pipeline test.
- NFR: NFR-040.

### TC-SEC-002 — Logger redacts phone numbers

- Steps: log line `phone=+84912345678`.
- Expected: emitted as `phone=+84***5678` in log sink (Sentry, Datadog, stdout).
- Automation: unit test on logger formatter.
- NFR: NFR-041.

### TC-SEC-003 — Sentry scrubber removes PII

- Steps: throw exception carrying user phone, OTP, lat/lng in extra context.
- Expected: Sentry payload after `before_send` hook contains `***` for those keys; original values absent.
- Automation: unit test against Sentry SDK in mocked transport mode.
- NFR: NFR-041.

### TC-SEC-004 — Secure storage roundtrip iOS+Android

- Steps: store JWT, restart app, read.
- Expected: same JWT value returned; on Android backed by Keystore (StrongBox if available); on iOS backed by Keychain with `kSecAttrAccessibleAfterFirstUnlockThisDeviceOnly`.
- Automation: integration_test on real devices.
- NFR: NFR-043.

### TC-SEC-005 — PII scrub on logs (mobile)

- Steps: enable verbose mobile logging, perform OTP flow.
- Expected: no full phone, no OTP code, no JWT body in logs (only `jti` last-4).
- Automation: integration_test with log sink capture.
- NFR: NFR-041.

### TC-SEC-006 — PII scrub on Sentry (mobile)

- Steps: trigger crash carrying phone in breadcrumbs.
- Expected: Sentry `before_send` removes phone from breadcrumbs.
- Automation: unit test.
- NFR: NFR-041.

### TC-SEC-007 — EXIF stripped on photo upload

- Steps: capture photo with GPS+camera+orientation EXIF; upload via `/v1/media/finalize`.
- Expected: stored object has only `Orientation` retained; GPS, Make, Model, DateTime stripped at client before upload.
- Automation: integration test reading uploaded object EXIF.
- NFR: NFR-045.

### TC-SEC-008 — Local DB wiped on logout

- Steps: capture data, logout.
- Expected: all SQLite tables truncated; database file size shrinks; secure storage cleared.
- Automation: integration_test asserting row counts and file size.
- NFR: NFR-044.

### TC-SEC-009 — Refresh token rotation

- Steps: call `/v1/auth/refresh`; capture new refresh.
- Expected: new refresh token issued; old refresh marked `rotated_at`; reusing old refresh returns 401 `REFRESH_INVALID`.
- Automation: backend integration test.
- NFR: NFR-046.

### TC-SEC-010 — Refresh re-use revokes session

- Steps: replay an already-rotated refresh token.
- Expected: 401; session marked `revoked_reason=REUSE`; all sibling refreshes for that session also revoked; security alert fires.
- Automation: backend integration test.
- NFR: NFR-046.

### TC-SEC-011 — Device binding mismatch rejects

- Steps: copy refresh token to a different device-id; attempt refresh.
- Expected: 401 `DEVICE_MISMATCH`; original session unaffected; alert fires.
- Automation: backend integration test.
- NFR: NFR-046.

### TC-SEC-012 — TLS only

- Steps: attempt http (non-TLS) request to backend.
- Expected: connection refused at LB level; HSTS header present in production responses.
- Automation: smoke test.
- NFR: NFR-040.

### TC-SEC-013 — JWT with `none` algorithm rejected

- Steps: forge token with `alg:none`.
- Expected: 401 `INVALID_TOKEN`.
- Automation: backend integration test.
- NFR: NFR-046.

### TC-SEC-014 — RBAC: non-admin cannot read other employee data

- Steps: employee A queries shifts for employee B by ID.
- Expected: 403 `FORBIDDEN`.
- Automation: backend integration test.
- NFR: NFR-047.

### TC-SEC-015 — DSAR export available

- Steps: support invokes `GET /v1/admin/dsar/{employee_id}`.
- Expected: structured export of employee's captures; logged in audit.
- Automation: backend integration test.
- NFR: NFR-051.

## PERF

### TC-PERF-001 — Cold start ≤ 2.5 s p95

- Pre: mid device (Pixel 6a, iPhone 12), release build, app evicted.
- Steps: launch app, measure to first interactive frame on My Shifts (offline cache present).
- Expected: p95 ≤ 2.5 s across 30 runs.
- Automation: integration_test with `IntegrationTestWidgetsFlutterBinding.reportData`.
- NFR: NFR-001.

### TC-PERF-002 — Warm start ≤ 1.0 s p95

- Steps: foreground app from background.
- Expected: p95 ≤ 1.0 s.
- Automation: integration_test.
- NFR: NFR-001.

### TC-PERF-003 — Shift list render 100 items ≤ 500 ms

- Pre: 100 cached shifts in SQLite.
- Steps: open My Shifts.
- Expected: p95 first paint of list ≤ 500 ms.
- Automation: integration_test.
- NFR: NFR-002.

### TC-PERF-004 — Photo capture preview ≤ 300 ms

- Steps: tap shutter; measure time to capture-confirmed UI.
- Expected: p95 ≤ 300 ms on mid device.
- Automation: integration_test.
- NFR: NFR-004.

### TC-PERF-005 — Photo compression ≤ 800 ms p95 mid device

- Pre: 12 MP source photo.
- Steps: compress to JPEG ≤ 500 KB.
- Expected: p95 ≤ 800 ms on mid device.
- Automation: integration_test.
- NFR: NFR-005.

### TC-PERF-006 — Whisper transcribe 30 s clip ≤ 12 s

- Steps: feed 30 s audio to on-device Whisper tiny.en.
- Expected: transcript returned within 12 s on mid device.
- Automation: integration_test.
- NFR: NFR-006.

### TC-PERF-007 — Battery cost ≤ 6 % per 8h shift

- Pre: typical workday simulation (10 captures, 2 photos each, GPS check-in/out).
- Steps: charge device to 100 %, run 8 h, read battery delta.
- Expected: ≤ 6 % attributable to app.
- Automation: manual measurement; tracked in pilot dashboard.
- NFR: NFR-007.

### TC-PERF-008 — Backend `/sync/envelope` server time ≤ 200 ms p95

- Steps: 10 RPS sustained for 10 min.
- Expected: p95 server-time ≤ 200 ms; p99 ≤ 500 ms; error rate < 0.1 %.
- Automation: load test (k6 / locust).
- NFR: NFR-011.

### TC-PERF-009 — Backend horizontal scale verified

- Steps: scale FastAPI from 2 → 8 pods under load.
- Expected: p95 latency does not regress; throughput scales linearly to 80 RPS.
- Automation: nightly load test.
- NFR: NFR-012.

### TC-PERF-010 — Android APK ≤ 80 MB

- Steps: build release APK + Play asset delivery.
- Expected: install size ≤ 80 MB (excluding Whisper model if downloaded on-demand per DEC-010).
- Automation: CI artifact size check.
- NFR: NFR-008.

### TC-PERF-011 — iOS IPA ≤ 120 MB

- Steps: build release IPA.
- Expected: install size ≤ 120 MB.
- Automation: CI artifact size check.
- NFR: NFR-008.

### TC-PERF-012 — Memory ceiling on capture screen

- Pre: capture 5 photos (12 MP each).
- Steps: monitor RSS during capture.
- Expected: peak RSS ≤ 250 MB; no OOMs on Android Go-class device.
- Automation: integration_test with memory probe.
- NFR: NFR-009.

### TC-PERF-013 — Cache eviction after 30 days

- Steps: simulate 31-day-old cached shift entries.
- Expected: eviction job removes records; SQLite vacuum reclaims space.
- Automation: integration test with clock injection.
- NFR: NFR-010.

## A11Y

### TC-A11Y-001 — All actionable controls have semantic labels

- Steps: dump semantics tree for My Shifts, PPE, Capture, Sync Center.
- Expected: every Tappable has non-empty `Semantics.label`; touch target ≥ 44×44 dp.
- Automation: widget test using `tester.semantics`.
- NFR: NFR-060.

### TC-A11Y-002 — TalkBack / VoiceOver flow

- Steps: enable screen reader, complete OTP login + capture PPE.
- Expected: focus order logical; all critical content read aloud; no traps.
- Automation: manual on iOS+Android per release.
- NFR: NFR-060.

### TC-A11Y-003 — Dynamic type up to 200 %

- Steps: set system font scale to 2.0.
- Expected: no clipped text on auth, shift list, capture screens; layout adapts.
- Automation: golden tests at multiple scales.
- NFR: NFR-061.

### TC-A11Y-004 — Color contrast ≥ 4.5:1

- Steps: audit theme tokens.
- Expected: all body text and primary CTAs meet WCAG AA contrast.
- Automation: contrast lint in CI.
- NFR: NFR-062.

## OBS

### TC-OBS-001 — Mobile crash reaches Sentry within 60 s

- Steps: trigger uncaught exception in test build.
- Expected: event in Sentry within 60 s of network availability; PII scrubbed.
- Automation: smoke after each release.
- NFR: NFR-070.

### TC-OBS-002 — Backend trace propagates to Odoo client

- Steps: capture `traceparent` header from `/sync/envelope`; verify Celery + Odoo RPC client logs same trace.
- Expected: trace spans show end-to-end path with timing breakdown.
- Automation: backend integration test reading OTel exporter.
- NFR: NFR-071.

### TC-OBS-003 — Sync success rate metric exported

- Steps: drive 100 envelopes; query `sync_envelope_total{status="confirmed"}` from Prometheus.
- Expected: metric matches; dashboard shows current rate.
- Automation: backend integration test.
- NFR: NFR-071.

### TC-OBS-004 — Pager fires on dead-letter spike

- Steps: produce 50 DEAD_LETTER in 5 min in staging.
- Expected: PagerDuty incident created; runbook link present in alert.
- Automation: chaos rehearsal each quarter.
- NFR: NFR-072.

## Mapping back

Each TC's "FR" / "NFR" line is the link back to the FR matrix. CI script (planned for S1) regenerates the FR matrix from this file plus story front matter.

## Coverage targets at S10 pilot gate

| Suite | Target |
|---|---|
| Unit (mobile) | ≥ 70 % line coverage on `@core/*` and `screen/*/state/*` |
| Unit (backend) | ≥ 80 % line coverage on `app/sync`, `app/auth`, `app/odoo` |
| Widget tests | All capture screens + Sync Center + Auth |
| Integration tests | All TC-*-001 happy paths automated |
| E2E (device matrix) | At least 1 device per row in `qa/device-matrix.md` |
| Load (k6) | TC-PERF-008 + TC-SYNC-024 weekly |
| Security | All TC-SEC-* automated where feasible; manual pen-test once before pilot |
