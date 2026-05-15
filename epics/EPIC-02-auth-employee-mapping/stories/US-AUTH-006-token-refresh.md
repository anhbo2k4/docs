---
id: US-AUTH-006
epic: EPIC-02
sprint: S2
fr: [FR-003]
priority: P0
estimate: M
status: Ready
owner: Mobile
---

# US-AUTH-006 — Transparent token refresh on 401

## User story

**As** a field worker
**I want** the app to refresh my session in the background when my access token expires
**so that** I do not see surprise logouts.

## Acceptance criteria

1. Dio interceptor catches 401, calls `POST /v1/auth/refresh`, retries the original request once with the new access token.
2. Concurrent 401s share one in-flight refresh; subsequent callers wait on the same future.
3. Refresh failure (`401 INVALID_REFRESH`, `403 SESSION_REVOKED`) clears the session and routes to login.
4. Refresh-token rotation: every successful refresh stores the new refresh token and invalidates the previous (TC-SEC-009).
5. Re-use of an old refresh token revokes the entire session server-side and surfaces a forced re-login (TC-SEC-010).

## Tasks

### Mobile
- [ ] `AuthRefreshInterceptor` with shared-future pattern.
- [ ] Refresh queue + cancellation safe.
- [ ] Session-revocation handler.

### Backend
- [ ] `POST /v1/auth/refresh` with rotation + reuse detection.
- [ ] Sliding window for refresh family, revoke on reuse.

### Testing
- [ ] Concurrent 401 test (10 parallel calls → 1 refresh).
- [ ] Reuse-detection integration test.

## Dependencies

- US-AUTH-003 (refresh token issuance).

## FR mapping

FR-003.

## Test cases

TC-AUTH-007, TC-SEC-009, TC-SEC-010.

## Definition of Done

- Concurrent-refresh test passes deterministically.
- Reuse detection revokes all tokens in family.
- Logging shows redacted token IDs only.
