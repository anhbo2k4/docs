# ADR-002: Supabase Edge Functions handle OTP issuance

**Status:** Accepted  
**Date:** 2026-05-13  
**Deciders:** Engineering Lead, Backend Lead, Security Champion  
**Consulted:** Product Owner, Sponsor  
**Informed:** All

## Context

The app uses phone + OTP as the only login mechanism. We need to send SMS quickly, throttle OTP requests, verify codes safely, and avoid building an in-house OTP service that requires PCI-style hardening. Twilio is the chosen SMS provider. We need a small server component to broker OTP because we cannot embed Twilio credentials in mobile.

## Decision

Use **Supabase Edge Functions** (Deno + TypeScript) for two endpoints:
- `POST /request_otp` — validates phone, applies rate limit, calls Twilio.
- `POST /verify_otp` — verifies code, returns a Supabase JWT.

FastAPI later exchanges the Supabase JWT for an internal JWT plus employee context (see ADR-004).

## Rationale

- Supabase Edge gives us a managed runtime close to the user (low latency).
- Built-in rate limiting and project-scoped secret storage.
- Edge functions are small and replaceable; we are not locked in.
- Pushes OTP-specific complexity (rate limit, Twilio retry, code expiry) out of FastAPI's hot path.

## Alternatives considered

| Option | Pros | Cons | Why not chosen |
|---|---|---|---|
| FastAPI handles OTP directly | Single backend | More attack surface, must implement rate limit + audit | Adds scope to FastAPI |
| Auth0 / Cognito | Mature OTP UX | Cost, vendor lock-in for our scale | Overkill for pilot |
| Firebase Phone Auth | Free tier, mature | Vendor coupling, harder Odoo employee mapping | Constrains identity model |
| Custom microservice | Full control | Build + maintain | Not a strategic asset |

## Consequences

### Positive
- OTP flow is isolated and replaceable.
- Rate limiting and abuse controls live in one place.
- FastAPI focuses on Odoo integration.

### Negative
- Two vendors in the auth path (Supabase + Twilio).
- Supabase outage blocks new logins (existing JWTs continue working).
- Edge function code lives outside the main repo's runtime — needs CI integration.

### Neutral
- Future move to a different OTP provider requires replacing two endpoints, not the whole auth model.

## Compliance / verification

- Edge functions code lives in `infra/edge-functions/` with its own deploy pipeline.
- Rate limit thresholds documented in `security/threat-model.md` and tested in TC-AUTH-041.
- No OTP code in the FastAPI codebase.

## Related

- FR-001, FR-002, FR-003
- ADR-004 (FastAPI exchanges Supabase JWT)
- DEC-005 (regional SMS provider)
- Stories: `epics/EPIC-02-auth-employee-mapping/stories/`
