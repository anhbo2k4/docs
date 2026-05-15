---
id: US-POST-011
epic: EPIC-10
sprint: post-MVP
priority: P2
estimate: XL
status: Draft
owner: TBD
---

# US-POST-011 — SSO federation per partner

## User story

**As** an enterprise contracting partner
**I want** my workers to sign in with our IdP (OIDC)
**so that** we don't manage a separate phone-OTP path.

## Acceptance criteria

1. Backend supports OIDC providers per tenant (issuer, client_id, JWKS URL).
2. Mobile login screen offers "Continue with <Partner>" alongside phone-OTP.
3. Token exchange yields the same internal session model as OTP.
4. Account linking: an existing OTP-based employee can attach an OIDC identity.
5. SCIM provisioning optional (separate story).

## Dependencies

- New ADR superseding the OTP-only auth model where federation is enabled.
- Updates to `security/auth-flow.md`.

## Risks

- Audit + compliance per partner; data residency for tokens.
