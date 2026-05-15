---
id: US-AUTH-001
epic: EPIC-02
sprint: S2
fr: [FR-001]
priority: P0
estimate: S
status: Ready
owner: Mobile
---

# US-AUTH-001 — Phone entry with format validation

## User story

**As** a field worker
**I want** to enter my phone number with country code on the login screen
**so that** I can request an OTP without typing extra characters or hitting a server with garbage input.

## Acceptance criteria

1. The login screen exposes a phone field with a country-code selector defaulted to the pilot region (DEC-005).
2. Input is normalised to E.164 (`+<country><subscriber>`); spaces, dashes, and parens are stripped.
3. Invalid entries (too short, non-numeric, missing leading `+` after normalisation) display an inline error and disable the **Send code** button.
4. Pressing **Send code** with a valid number triggers `request_otp` (US-AUTH-002).
5. The screen renders correctly on the smallest supported device (iPhone SE, 4.7"), no overflow at 200% text scale.
6. Phone number is never logged at any level. (TC-SEC-002, TC-SEC-005.)

## Tasks

### Flutter UI
- [ ] Build `LoginPhoneScreen` under `screen/auth/login_phone_screen.dart`.
- [ ] Country code picker widget with default region from `--dart-define=PILOT_REGION`.
- [ ] Inline validation messages.
- [ ] A11y labels on field and button.

### Domain
- [ ] `PhoneNumberValidator` in `domain/use_cases/auth/`.
- [ ] `NormalizePhoneToE164` use case.

### Testing
- [ ] Widget tests for valid/invalid input.
- [ ] Golden test for two locales.
- [ ] Test that no log lines contain raw phone digits.

## Dependencies

- EPIC-01 done (theme + router + Riverpod scaffolding).
- DEC-005 closed (knowing pilot region drives default country code).

## Risks / Open questions

- DEC-005 (SMS provider region).

## FR mapping

FR-001.

## Test cases

TC-AUTH-002 (invalid format), TC-SEC-002, TC-SEC-005.

## Definition of Done

- Acceptance criteria pass on iOS and Android.
- Coverage on validator unit ≥ 95%.
- No PII in logs verified by automated test.
- Reviewed by Mobile Lead and Security Champion.
