# Error Codes

**Status:** Active  
**Owner:** Backend Lead

Canonical error catalogue. Every backend error response uses one of these codes. New codes require a PR with a justification entry below.

## Format

```json
{
  "error": {
    "code": "VALIDATION_FAILED",
    "message": "Field 'phone' has invalid format",
    "request_id": "8f...",
    "details": [ { "field": "phone", "issue": "INVALID_FORMAT" } ]
  }
}
```

## Catalogue

### Auth

| Code | HTTP | Meaning | Mobile action |
|---|---|---|---|
| `INVALID_PHONE_FORMAT` | 400 | Not E.164 | Show validation message |
| `INVALID_CODE` | 400 | OTP wrong | Show "wrong code, try again" |
| `OTP_EXPIRED` | 410 | OTP TTL elapsed | Re-request OTP |
| `OTP_LOCKED` | 429 | Too many wrong attempts | Show 5-min cooldown |
| `RATE_LIMIT` | 429 | Generic rate limit | Honour `Retry-After` |
| `INVALID_SUPABASE_JWT` | 401 | JWKS verify failed | Re-login |
| `EMPLOYEE_NOT_MAPPED` | 403 | Phone not in `hr.employee` | Show "contact admin" |
| `EMPLOYEE_INACTIVE` | 409 | Employee disabled | Show "contact admin" |
| `TOKEN_EXPIRED` | 401 | Internal access JWT expired | Refresh; retry once |
| `REFRESH_INVALID` | 401 | Refresh malformed / signature wrong | Re-login |
| `REFRESH_EXPIRED` | 401 | Refresh past expiry | Re-login |
| `REFRESH_REVOKED` | 401 | Logout was issued elsewhere | Re-login |
| `SESSION_EXPIRED` | 401 | Session no longer valid (admin revocation) | Re-login |

### Sync

| Code | HTTP | Meaning | Mobile action |
|---|---|---|---|
| `MISSING_CLIENT_ID` | 400 | Header / body lacks `client_id` | Programming error → Sentry |
| `INVALID_ENVELOPE` | 400 | Schema validation failed | DEAD_LETTER + show error |
| `CLIENT_ID_CONFLICT` | 409 | Same `client_id`, different payload | DEAD_LETTER + Sentry |
| `INVALID_SHIFT_STATE` | 422 | Capturing into a wrong-state shift | DEAD_LETTER + show context |
| `NOT_ASSIGNED` | 403 | Shift not assigned to user | DEAD_LETTER |
| `ODOO_UNAVAILABLE` | 503 | Backend cannot reach Odoo (read paths only) | Retry later |
| `ENVELOPE_NOT_FOUND` | 404 | Status query for unknown envelope | Treat as PENDING and retry POST |

### Media

| Code | HTTP | Meaning | Mobile action |
|---|---|---|---|
| `INVALID_MIME` | 400 | Disallowed MIME | Programming error |
| `BYTES_TOO_LARGE` | 400 | Above per-kind cap | Compress harder, retry |
| `OBJECT_NOT_FOUND` | 404 | Finalize before PUT landed | Retry PUT |
| `SHA256_MISMATCH` | 409 | Hash differs | Re-upload |
| `BYTES_MISMATCH` | 409 | Size differs | Re-upload |
| `MEDIA_NOT_FINALIZED` | 422 | Envelope refers to non-finalized media | Finalize all, retry envelope |

### Shifts

| Code | HTTP | Meaning |
|---|---|---|
| `SHIFT_NOT_FOUND` | 404 | id unknown |

### Generic

| Code | HTTP | Meaning |
|---|---|---|
| `INTERNAL_ERROR` | 500 | Unexpected server error |
| `BAD_REQUEST` | 400 | Generic bad request not covered above |

## Mobile mapping

Mobile keeps a single `apiErrorMapper` that converts these codes to:

- A `DomainError` enum.
- A user-facing message (localized).
- A retry policy hint (`retryable | non_retryable | refresh-and-retry-once`).

The mapper lives in `mobile/lib/@core/network/error_mapper.dart`.

## Adding a new error code

1. Update this file.
2. Update `mobile/lib/@core/network/error_mapper.dart`.
3. Update relevant tests.
4. Bump CHANGELOG (minor version).

## Logs

Server logs include `code`, `request_id`, `employee_id`, route, `client_id` (if present). PII is redacted.
