---
id: US-PLAT-003
epic: EPIC-01
sprint: S1
fr: []
priority: P0
estimate: S
status: Ready
owner: Mobile
---

# US-PLAT-003 — Add secure storage wrapper

## Acceptance criteria

1. `SecureSessionStore` API: `get(key)`, `put(key, value)`, `delete(key)`, `clear()`.
2. iOS options: `accessibility=first_unlock`, no iCloud.
3. Android options: `EncryptedSharedPreferences`, AES-GCM, hardware-backed where available.
4. Error mapping: `KeystoreUnavailableException` triggers a one-time recovery flow that wipes and re-asks the user to log in.
5. Round-trip integration test (TC-SEC-004) green on simulator and emulator.

## Tasks

- [ ] Add `flutter_secure_storage` dependency.
- [ ] Implement wrapper with platform-specific options.
- [ ] Author recovery flow trigger.
- [ ] Integration test with cold-start.

## Dependencies

US-PLAT-001.

## FR mapping

Foundation for FR-002, FR-003.

## Test cases

TC-SEC-004.

## DoD

- Round-trip test green.
- Wrapper documented in code with usage examples.
