# ADR-010: Twilio as the sole SMS / OTP provider for MVP

**Status:** Accepted
**Date:** 2026-05-15
**Deciders:** Sponsor, Backend Lead, Engineering Lead
**Consulted:** Mobile Lead, DevOps, Security Champion
**Informed:** PO, QA Lead

## Context

DEC-005 asked whether the MVP should use Twilio only or Twilio plus a regional fallback (e.g. Vietnamese SMS gateway, eSMS, Stringee, Speedchat) for OTP delivery.

Constraints:

- Pilot region: Vietnam (single country) for the first 6 months.
- Auth flow already designed around Supabase Edge Functions calling Twilio Verify (ADR-002).
- We do not have an internal SMS provider abstraction; adding a fallback now means a second integration, second secret rotation path, and a routing decision tree.
- Pilot user count is bounded (under ~200 field workers). OTP volume is low.
- Twilio's Vietnam delivery rate has been within acceptable bounds in prior projects; SLAs are documented.

## Decision

Use **Twilio Verify** as the sole SMS / OTP provider for MVP. No regional fallback in v1.0. A fallback evaluation is scheduled as `DEC-005-FOLLOWUP` once pilot delivery telemetry is available.

## Rationale

- Single provider keeps the auth flow simple and removes a routing decision.
- Twilio Verify handles retries, rate limiting, and code generation server-side, reducing custom code surface in Supabase Edge.
- Pilot scale does not justify the engineering cost of dual-provider routing.
- A fallback can be added without breaking the OTP API contract (the swap is internal to the Edge Function), so this is a reversible decision.

## Alternatives considered

| Option | Pros | Cons | Why not chosen |
|---|---|---|---|
| Twilio + regional fallback now | Higher delivery rate in edge cases; vendor risk diversification | Second integration, doubled secret rotation, routing rules, more on-call burden | Cost not justified at pilot scale |
| Regional provider only (e.g. eSMS) | Lower per-SMS cost in VN; faster delivery in some carriers | Less mature verify API, must hand-roll code generation, weaker observability | Engineering cost too high for pilot |
| Twilio only (chosen) | Simple, mature API, strong tooling, already aligned with ADR-002 | Vendor lock-in until fallback added; per-SMS cost slightly higher | Best fit for pilot scope |

## Consequences

### Positive

- Auth flow remains as designed in ADR-002.
- One secret to rotate, one dashboard to monitor.
- Faster path to S2 completion.

### Negative

- Single point of failure for OTP delivery during pilot. Mitigated by `runbooks/otp-fallback.md` which describes manual support workflow.
- If Twilio raises Vietnam pricing, no negotiation leverage at pilot scale.

### Neutral

- A follow-up evaluation (`DEC-005-FOLLOWUP`) will revisit fallback before national rollout.

## Compliance / verification

- `engineering/dependency-policy.md` lists Twilio as the approved SMS vendor; PR introducing any other SMS SDK requires ADR amendment.
- `qa/test-cases.md` TC-AUTH-007 covers Twilio outage simulation via mocked 5xx response.
- DataDog dashboard `auth-otp-delivery` tracks Twilio success rate; alert at <97% rolling 24h.

## Related

- FR / NFR: FR-001, FR-002 (OTP login), NFR-AUTH-01 (OTP delivery latency p95 < 8s)
- Stories: US-DISC-004, US-AUTH-001, US-AUTH-002
- Other ADRs: ADR-002 (Supabase Edge OTP)
- Decisions closed: DEC-005
