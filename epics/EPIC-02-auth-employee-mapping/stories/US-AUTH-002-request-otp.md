---
id: US-AUTH-002
epic: EPIC-02
sprint: S2
fr: [FR-001]
priority: P0
estimate: M
status: Ready
owner: Mobile + Backend
---

# US-AUTH-002 — Request OTP via Supabase Edge

## User story

**As** a field worker
**I want** to receive a 6-digit SMS code within 30 seconds of pressing **Send code**
**so that** I can complete login without delay.

## Acceptance criteria

1. Tapping **Send code** invokes the Supabase Edge function `request_otp` with `{phone}` and a request id.
2. On HTTP 200 the app routes to OTP entry; on any error, an inline message shows.
3. OTP rate limiting: more than 3 requests in 5 minutes per phone returns `RATE_LIMIT`; UI displays a cooldown.
4. SMS is delivered within 30 s p95 in the pilot region (gated on DEC-005).
5. Phone is never logged in plaintext on either side.
6. Telemetry counter increments on success/failure with a hashed phone bucket only.

## Tasks

### Mobile
- [ ] `RequestOtpUseCase` calls Supabase client.
- [ ] OTP entry screen with 6-digit auto-fill and 30 s "request again" timer.
- [ ] Error mapping for `RATE_LIMIT`, `INVALID_PHONE`, `NETWORK_ERROR`.

### Backend (Edge function)
- [ ] `supabase/functions/request_otp/index.ts` validates input.
- [ ] Calls Twilio Verify (or equivalent per DEC-005).
- [ ] Persists rate-limit counter in Redis.
- [ ] Returns generic success regardless of mapping (avoid user enumeration).

### Testing
- [ ] Integration test: rate limit boundary.
- [ ] Mock Twilio for CI.
- [ ] Manual real-SMS smoke in staging.

## Dependencies

- DEC-005 (SMS provider).
- US-AUTH-001 (phone normalised).

## Risks / Open questions

- RISK-020 Twilio delivery latency.
- DEC-005 unresolved means provider client may change.

## FR mapping

FR-001.

## Test cases

TC-AUTH-001, TC-AUTH-005, TC-AUTH-041.

## Definition of Done

- 200/4xx behaviour matches spec.
- p95 SMS delivery ≤ 30 s in staging proof.
- Logs and Sentry events redact phone (TC-SEC-002).
- Edge function deployed via CI in staging.
