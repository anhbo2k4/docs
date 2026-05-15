---
id: US-API-002
epic: EPIC-07
sprint: S8
fr: [FR-002, FR-003]
priority: P0
estimate: M
status: Ready
owner: Backend
---

# US-API-002 — `/v1/auth/exchange` + JWT + refresh + revoke

## Acceptance criteria

1. `POST /v1/auth/exchange` validates Supabase JWT, calls employee mapping (US-AUTH-005), and issues `{access_token, refresh_token}`.
2. Tokens are JWT with `kid` for key rotation; signing key fetched from secrets manager.
3. `POST /v1/auth/refresh` rotates refresh tokens (US-AUTH-006); reuse triggers revocation of the family.
4. `POST /v1/auth/logout` revokes the active refresh family.
5. Tokens carry: `sub=employee_id`, `device_id`, `iat`, `exp`, `kid`.
6. All token writes recorded in `auth_sessions` with `device_id`, `created_at`, `revoked_at`.

## Tasks

- [ ] Endpoints + schemas.
- [ ] Token issuance and validation utils.
- [ ] Refresh family table + rotation logic.
- [ ] Tests for rotation + reuse + revocation.

## Dependencies

US-API-001, US-AUTH-005.

## FR mapping

FR-002, FR-003.

## Test cases

TC-AUTH-003, TC-AUTH-007, TC-SEC-009, TC-SEC-010.

## DoD

- All tests green.
- Key rotation runbook in `runbooks/`.
