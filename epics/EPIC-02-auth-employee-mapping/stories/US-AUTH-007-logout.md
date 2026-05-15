---
id: US-AUTH-007
epic: EPIC-02
sprint: S2
fr: [FR-003]
priority: P0
estimate: S
status: Ready
owner: Mobile + Backend
---

# US-AUTH-007 — Logout wipes data and revokes refresh

## User story

**As** a field worker
**I want** to log out and have all my data removed from the device
**so that** my privacy is protected when I hand the device back.

## Acceptance criteria

1. Tap **Logout** invokes `POST /v1/auth/logout`; backend revokes the active refresh family.
2. Mobile clears `flutter_secure_storage` keys, deletes the SQLite database file, and removes local media.
3. Sync queue is drained or canceled per current state: `SYNCING` finishes; `PENDING` is dropped with audit; `FAILED` and `DEAD_LETTER` are dropped after diagnostic export prompt (optional).
4. Crash reporter session is closed; user-id is detached from Sentry.
5. App routes to `LoginPhoneScreen`.
6. After logout, attempting any authenticated API with the old refresh token returns 403 `SESSION_REVOKED`.

## Tasks

### Mobile
- [ ] `LogoutUseCase`.
- [ ] DB wipe + secure storage clear + media wipe.
- [ ] Sentry user detach.

### Backend
- [ ] `POST /v1/auth/logout` revokes refresh family.

### Testing
- [ ] TC-AUTH-008.
- [ ] TC-SEC-008 (DB wiped on logout).

## Dependencies

- US-AUTH-003 (token family).
- EPIC-04 (DB to wipe).

## FR mapping

FR-003.

## Test cases

TC-AUTH-008, TC-SEC-008, TC-SEC-009.

## Definition of Done

- DB file size = 0 post-logout (check via OS file API).
- Old refresh token rejected.
- Sentry user-id removed.
