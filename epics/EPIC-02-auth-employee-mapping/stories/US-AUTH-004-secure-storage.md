---
id: US-AUTH-004
epic: EPIC-02
sprint: S2
fr: [FR-002]
priority: P0
estimate: S
status: Ready
owner: Mobile
---

# US-AUTH-004 — Persist tokens in secure storage

## User story

**As** a field worker
**I want** my session to survive app restart and OS suspension
**so that** I do not have to re-enter my phone every time I open the app.

## Acceptance criteria

1. `access_token`, `refresh_token`, `employee_id`, and `device_id` are persisted via `flutter_secure_storage` with platform options:
   - iOS: `accessibility=first_unlock`, no iCloud sync.
   - Android: AES-GCM keystore-backed; `EncryptedSharedPreferences`.
2. App boot reads tokens once and seeds `AuthSessionNotifier`.
3. If access is expired but refresh is not, the bootstrap exchange occurs before routing to My Shifts.
4. If both tokens are missing or invalid, app routes to `LoginPhoneScreen`.
5. Logout (US-AUTH-007) clears every key listed.
6. Secure storage round-trip verified on both platforms (TC-SEC-004).

## Tasks

### Mobile
- [ ] `SecureSessionStore` wrapper with read/write/clear.
- [ ] Bootstrap auth in `application/bootstrap.dart`.
- [ ] Riverpod `AuthSessionNotifier` consuming bootstrap.

### Testing
- [ ] Round-trip integration test on simulator and emulator.
- [ ] Cold-start test: tokens valid → My Shifts.
- [ ] Cold-start test: tokens missing → Login.

## Dependencies

- US-AUTH-003 (tokens to persist).
- EPIC-01 secure storage wrapper (US-PLAT-003).

## FR mapping

FR-002.

## Test cases

TC-SEC-004, TC-AUTH-003.

## Definition of Done

- Round-trip test green on iOS and Android.
- No keys stored in plain `SharedPreferences` or `UserDefaults`.
- Security Champion review approved.
