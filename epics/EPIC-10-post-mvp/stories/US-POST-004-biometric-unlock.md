---
id: US-POST-004
epic: EPIC-10
sprint: post-MVP
priority: P2
estimate: M
status: Draft
owner: TBD
---

# US-POST-004 — Biometric unlock

## User story

**As** a Field Worker
**I want** to unlock the app with biometrics after first OTP login
**so that** I don't repeat OTP every shift.

## Acceptance criteria

1. After successful OTP login, user can enable Face ID / Touch ID / Android biometrics.
2. Re-auth via biometrics issues a new short-lived session without contacting the OTP path.
3. Falling back to OTP is always available.
4. After 30 days of inactivity OR explicit logout, biometrics are revoked and OTP is required.
5. PII / token storage uses `flutter_secure_storage` per `security/auth-flow.md`.

## Tasks

- [ ] Biometric prompt UX.
- [ ] Token re-issuance endpoint scoped to "biometric refresh".
- [ ] Failure paths: locked-out, no enrolled biometric, hardware missing.

## Dependencies

- ADR proposing the biometric session model.

## Risks

- Replay attacks if device is compromised; mitigated by hardware-backed keystore.
