# Auth API

**Status:** Active  
**Owner:** Backend Lead

Maps to FR-001, FR-002, FR-003. ADR-002, ADR-004.

## Flow

```
1. Mobile → Supabase Edge: POST /request_otp
2. Supabase Edge → Twilio: send SMS
3. Mobile → Supabase Edge: POST /verify_otp → returns Supabase JWT
4. Mobile → FastAPI:        POST /v1/auth/exchange (Bearer Supabase JWT)
                            ← returns internal access + refresh tokens + employee
5. Mobile → FastAPI:        POST /v1/auth/refresh   (when access expired)
6. Mobile → FastAPI:        POST /v1/auth/logout    (revokes refresh)
```

---

## 1. POST /v1/auth/request_otp (Supabase Edge)

Initiates an OTP send.

### Request

```http
POST /functions/v1/request_otp
Content-Type: application/json
apikey: <supabase_anon>
```

```json
{ "phone": "+84912345678" }
```

### Response (200)

```json
{ "data": { "expires_in": 300 } }
```

### Errors

| HTTP | Code | When |
|---|---|---|
| 400 | `INVALID_PHONE_FORMAT` | E.164 validation failed |
| 429 | `RATE_LIMIT` | More than 3 requests / 5 min |

---

## 2. POST /v1/auth/verify_otp (Supabase Edge)

### Request

```json
{ "phone": "+84912345678", "code": "123456" }
```

### Response (200)

```json
{
  "data": {
    "supabase_jwt": "eyJhbGciOi...",
    "expires_in": 300
  }
}
```

### Errors

| HTTP | Code |
|---|---|
| 400 | `INVALID_CODE` |
| 410 | `OTP_EXPIRED` |
| 429 | `OTP_LOCKED` (after 5 wrong attempts) |

---

## 3. POST /v1/auth/exchange (FastAPI)

Exchange a Supabase JWT for an internal access + refresh token. Maps phone → `hr.employee` in Odoo.

### Request

```http
POST /v1/auth/exchange
Authorization: Bearer <supabase_jwt>
X-Device-ID: <uuid>
X-App-Version: 1.0.0+42
X-Platform: ios
Content-Type: application/json
```

```json
{ "device_name": "Pixel 6a", "locale": "en-VN" }
```

### Response (200)

```json
{
  "data": {
    "access_token":  "eyJhbGciOi...",
    "refresh_token": "eyJhbGciOi...",
    "access_expires_in": 900,
    "refresh_expires_in": 2592000,
    "employee": {
      "id": 42,
      "name": "Nguyen Van A",
      "phone": "+84912345678",
      "department": "Field Ops",
      "job_title": "Technician",
      "avatar_url": "https://..."
    },
    "min_app_version": "1.0.0"
  }
}
```

### Errors

| HTTP | Code | When |
|---|---|---|
| 401 | `INVALID_SUPABASE_JWT` | JWKS verification failed or expired |
| 403 | `EMPLOYEE_NOT_MAPPED` | Phone not found in `hr.employee` |
| 409 | `EMPLOYEE_INACTIVE` | Employee record disabled |
| 503 | `ODOO_UNAVAILABLE` | Cannot reach Odoo to look up employee |

---

## 4. POST /v1/auth/refresh

### Request

```json
{ "refresh_token": "eyJhbGciOi..." }
```

### Response (200)

```json
{
  "data": {
    "access_token":  "eyJhbGciOi...",
    "refresh_token": "eyJhbGciOi...",
    "access_expires_in": 900,
    "refresh_expires_in": 2592000
  }
}
```

### Errors

| HTTP | Code |
|---|---|
| 401 | `REFRESH_INVALID` |
| 401 | `REFRESH_REVOKED` |
| 401 | `REFRESH_EXPIRED` |

Refresh tokens rotate on every use.

---

## 5. POST /v1/auth/logout

### Request

Bearer access token.

### Response (204)

No body.

### Side effects

- Refresh token revoked.
- Mobile must clear all secure storage.
- Local SQLite is wiped (security policy).

---

## Token shape (informational)

```json
{
  "iss": "fieldwork-api",
  "sub": "42",
  "aud": "fieldwork-mobile",
  "exp": 1717000000,
  "iat": 1716999100,
  "jti": "...",
  "employee_id": 42,
  "device_id": "..."
}
```

Signing algorithm: RS256. Key rotation policy: 90 days, JWKS endpoint at `/v1/auth/.well-known/jwks.json`.

## Tests

- TC-AUTH-001 valid OTP request
- TC-AUTH-002 invalid phone format
- TC-AUTH-003 valid OTP code
- TC-AUTH-004 wrong OTP code (3 attempts)
- TC-AUTH-005 OTP expired
- TC-AUTH-006 employee mapped
- TC-AUTH-007 token refresh happy path
- TC-AUTH-008 logout revokes refresh
- TC-AUTH-041 OTP rate limit
