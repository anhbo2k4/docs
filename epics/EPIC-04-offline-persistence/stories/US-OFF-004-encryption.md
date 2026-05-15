---
id: US-OFF-004
epic: EPIC-04
sprint: S4
fr: [FR-012]
priority: P0
estimate: M
status: Ready
owner: Mobile + Security Champion
---

# US-OFF-004 — DB encryption (SQLCipher / Keychain key)

## Acceptance criteria

1. Android: SQLCipher via `sqlcipher_flutter_libs`; key stored in Keystore-backed secure storage.
2. iOS: SQLCipher with key stored in Keychain (`first_unlock`).
3. Key rotation procedure documented: re-encrypt + verify + atomic swap; manual trigger for now.
4. Disk inspection of the file with `sqlite3` returns garbage (test harness verifies signature is not `SQLite format 3`).
5. Test boot when keystore is unavailable: graceful failure → wipe + re-login.

## Tasks

- [ ] Wire SQLCipher.
- [ ] Implement key bootstrap on first launch.
- [ ] Author key rotation playbook in `runbooks/`.
- [ ] Cross-platform integration test (TC-SEC-004).

## Dependencies

US-PLAT-003, US-PLAT-004, US-OFF-001.

## FR mapping

FR-012, NFR-040 (data at rest).

## Test cases

TC-SEC-004, TC-SEC-008.

## DoD

- Encrypted file verified on both platforms.
- Wipe-on-keystore-loss path tested.
