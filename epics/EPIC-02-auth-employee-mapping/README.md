---
epic: EPIC-02
title: Authentication & Employee Mapping
sprint: S2
priority: P0
status: Ready
owner: Mobile Lead + Backend Lead
---

# EPIC-02 — Authentication & Employee Mapping

## 1. Goal

A worker enters their phone, receives an SMS code, and lands on My Shifts with a session that survives app restart. The phone resolves to an `hr.employee` in Odoo per the strategy chosen in DEC-006.

## 2. In scope

- Phone entry, country code, normalisation.
- Supabase Edge Function for `request_otp` and `verify_otp` (Twilio SMS).
- Internal JWT exchange endpoint (`/v1/auth/exchange`) issued by FastAPI.
- Employee lookup that maps `phone → hr.employee.id` per DEC-006.
- Token storage (access 15 min, refresh 30 days) via secure storage.
- Transparent refresh on 401.
- Logout that clears secure storage and SQLite (TC-SEC-008).
- OTP rate limiting (per phone + per device) per NFR-042.

## 3. Out of scope

- Biometric unlock (post-MVP).
- Multi-factor beyond OTP.
- Self-registration. Pilot users are pre-mapped.

## 4. Functional requirements covered

FR-001, FR-002, FR-003.

## 5. Stories

| ID | Title | Pri | Estimate | Status | Owner |
|---|---|---|---|---|---|
| US-AUTH-001 | Phone entry + format validation | P0 | S | Ready | Mobile |
| US-AUTH-002 | Request OTP via Supabase Edge | P0 | M | Ready | Mobile + Backend |
| US-AUTH-003 | Verify OTP and exchange JWT | P0 | M | Ready | Backend |
| US-AUTH-004 | Persist tokens in secure storage | P0 | S | Ready | Mobile |
| US-AUTH-005 | Map phone to hr.employee | P0 | M | Ready | Backend |
| US-AUTH-006 | Transparent token refresh on 401 | P0 | M | Ready | Mobile |
| US-AUTH-007 | Logout wipes data and revokes refresh | P0 | S | Ready | Mobile + Backend |

## 6. Dependencies

- EPIC-01 complete (secure storage, Dio interceptors).
- DEC-005 (SMS provider) and DEC-006 (employee mapping) closed.

## 7. Risks

- RISK-020 Twilio delivery latency in pilot region exceeds 30 s.
- RISK-021 Phone-mapping ambiguity (multiple employees share a phone).
- RISK-022 Refresh-token reuse misconfigured → silent session loss.

## 8. Definition of Done (epic-level)

- Login → My Shifts works end-to-end on iOS and Android with a real Twilio sandbox.
- All TC-AUTH cases green.
- Logs and Sentry events redact phone numbers (TC-SEC-002, TC-SEC-005).
- Auth flow documented in `security/auth-flow.md` matches code.

## 9. Open questions

DEC-005, DEC-006 (must close at S0 end). DEC-013 (crash reporter default) tracked but not blocking.
