---
id: US-AUTH-003
epic: EPIC-02
sprint: S2
fr: [FR-002]
priority: P0
estimate: M
status: Ready
owner: Backend
---

# US-AUTH-003 — Verify OTP and exchange JWT

## User story

**As** a field worker
**I want** my code to be verified and translated into an internal session
**so that** the rest of the app can prove who I am without re-entering anything.

## Acceptance criteria

1. `verify_otp` Edge function accepts `{phone, code}`, calls Twilio Verify, returns a short-lived Supabase JWT (5 min).
2. The mobile client immediately calls `POST /v1/auth/exchange` with the Supabase JWT.
3. FastAPI verifies the Supabase JWT signature and looks up the employee per US-AUTH-005; on success returns `{access_token, refresh_token, employee}`.
4. Access token TTL = 15 min; refresh token TTL = 30 days; both signed with the rotated FastAPI signing key.
5. Wrong code returns `INVALID_CODE`; 5 wrong codes within 5 min returns `OTP_LOCKED` with cool-down (NFR-042).
6. Expired code returns `OTP_EXPIRED` with a clear message to request again.

## Tasks

### Edge
- [ ] `supabase/functions/verify_otp/index.ts`.
- [ ] Counter for wrong codes per phone.

### FastAPI
- [ ] `POST /v1/auth/exchange` with employee lookup gate.
- [ ] Token signing key in secrets manager; rotation playbook.
- [ ] Refresh-token rotation on use; old token blacklisted (TC-SEC-009/010).

### Mobile
- [ ] `VerifyOtpUseCase` and `ExchangeJwtUseCase`.
- [ ] On success persist via secure storage (US-AUTH-004).

### Testing
- [ ] TC-AUTH-003, TC-AUTH-004, TC-AUTH-005.
- [ ] Negative test for tampered Supabase JWT.

## Dependencies

- DEC-006 closed (employee mapping strategy).
- US-AUTH-005 implemented.

## Risks / Open questions

- RISK-021 phone-mapping ambiguity.
- RISK-022 refresh-token reuse misconfigured.

## FR mapping

FR-002.

## Test cases

TC-AUTH-003, TC-AUTH-004, TC-AUTH-005.

## Definition of Done

- Tokens issued and validated end-to-end.
- Refresh rotation verified by integration test.
- Failed-attempt lock behaves per AC#5.
- Security review of token handling complete.
