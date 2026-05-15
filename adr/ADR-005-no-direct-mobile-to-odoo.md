# ADR-005: Mobile never calls Odoo directly

**Status:** Accepted  
**Date:** 2026-05-13  
**Deciders:** Engineering Lead, Backend Lead, Security Champion  
**Consulted:** Mobile Lead, Odoo Specialist, Sponsor  
**Informed:** All

## Context

Odoo exposes JSON-RPC. Mobile could in theory call it directly, but:

- Odoo session model is not designed for thousands of mobile clients.
- Odoo error semantics are heavy and ERP-coupled.
- Schema changes in Odoo would break the mobile app immediately.
- Authentication would require embedding Odoo credentials or building a token bridge.
- Audit, rate limiting, abuse controls would have to be built inside Odoo.
- Mobile networks are flaky; Odoo writes need queue + retry semantics that an ERP server should not own.

## Decision

The mobile app **only** talks to:
- Supabase Edge Functions (OTP request / verify).
- FastAPI (everything else).
- Object Storage (presigned URLs issued by FastAPI).

Mobile **never** calls Odoo JSON-RPC directly, **never** holds Odoo credentials, **never** parses Odoo error envelopes.

## Rationale

- Encapsulates Odoo behind a stable, versioned contract (`/v1/...`).
- Lets us version, evolve, or even replace Odoo without recompiling and republishing the app.
- Centralises retry, idempotency, audit, and rate limiting in FastAPI / Celery.
- Reduces attack surface — Odoo stays internal.
- Simplifies the mobile error model: one error envelope, one retry policy.

## Alternatives considered

| Option | Pros | Cons | Why not chosen |
|---|---|---|---|
| Direct mobile → Odoo | One less hop | Brittle, leaks Odoo schema, hard to retry safely | Violates layering |
| Mobile → BFF for reads, direct for writes | Simpler reads | Mixed model; idempotency hard | Confusing rules |
| GraphQL gateway | Flexible queries | Adds complexity for our scope | Overkill |

## Consequences

### Positive
- Clear contract surface for mobile (`api-contracts/`).
- Backend can change Odoo version, swap modules, or migrate to a different ERP without changing the mobile app's contract.
- Strong control over rate limiting, audit, retries.

### Negative
- One more service to operate.
- Slight added latency on read paths (mitigated by Postgres cache + presigned uploads).
- Backend team owns the integration roadmap.

### Neutral
- All Odoo schema knowledge concentrates on the backend team and the Odoo specialist.

## Compliance / verification

- Mobile dependency list (`pubspec.yaml`) must not contain any Odoo client.
- CI lint rule: forbid `xmlrpc`, `jsonrpc`, or any Odoo-specific package in mobile.
- Threat model treats Odoo as internal-only; firewall rules enforce.
- API contracts under `api-contracts/` are the only documented surface for mobile to call.

## Related

- FR-004 to FR-015
- ADR-004
- Stories: all backend epics
