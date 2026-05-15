---
sprint: S2
title: Auth & Employee Context
duration: 2 weeks
priority: P0
status: Ready
owner: Mobile Lead + Backend Lead
---

# Sprint 2 — Auth & Employee Context

## 1. Sprint goal

A worker logs in via OTP, lands on My Shifts (stub), and the session survives app restart. Phone resolves to one `hr.employee` per DEC-006.

## 2. Theme

End-to-end authentication, mobile + edge + FastAPI. Every cross-cutting concern from S1 is exercised under real load.

## 3. Scope (in)

- Phone entry + format validation.
- Supabase Edge `request_otp` + `verify_otp`.
- FastAPI `/v1/auth/exchange` + employee mapping.
- Token persistence (secure storage) + transparent refresh.
- Logout + revoke + DB wipe.
- OTP rate limiting + refresh-token rotation.

## 4. Out of scope

- Shifts list rendering (S3 — only stub here).
- Real Odoo data (mock first; Odoo lookup behind a feature flag if needed).

## 5. Pre-conditions

- DEC-005 (SMS provider) and DEC-006 (employee mapping) closed.
- S1 done.
- Twilio sandbox creds available.

## 6. Stories committed

| ID | Title | Owner | Estimate |
|---|---|---|---|
| US-AUTH-001 | Phone entry + format validation | Mobile | S |
| US-AUTH-002 | Request OTP via Supabase Edge | Mobile + Backend | M |
| US-AUTH-003 | Verify OTP + JWT exchange | Backend | M |
| US-AUTH-004 | Persist tokens in secure storage | Mobile | S |
| US-AUTH-005 | Map phone to hr.employee | Backend | M |
| US-AUTH-006 | Transparent token refresh | Mobile | M |
| US-AUTH-007 | Logout wipes data + revokes refresh | Mobile + Backend | S |

## 7. Risks for this sprint

- RISK-020 Twilio delivery latency.
- RISK-021 Phone-mapping ambiguity surfaces in real data.
- RISK-022 Refresh-token reuse misconfigured.

## 8. Sprint-level Definition of Done

- Login → My Shifts (stub) on iOS and Android with real Twilio sandbox.
- TC-AUTH-001..008, TC-AUTH-041, TC-SEC-002, TC-SEC-005, TC-SEC-009, TC-SEC-010 green.
- p95 SMS delivery ≤ 30 s in staging.
- No PII in logs or Sentry.

## 9. Demo script

- Live login on a real device.
- Force a 401, observe transparent refresh.
- Logout, confirm DB file wiped.

## 10. Retro inputs

- Did Twilio behave as expected in pilot region?
- How clean was the auth contract between mobile and backend?
- Did rate limiting lock anyone out unexpectedly?
