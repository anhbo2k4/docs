# Security

**Status:** Active  
**Owner:** Security Champion + Engineering Lead

| File | Purpose |
|---|---|
| [threat-model.md](./threat-model.md) | STRIDE threat model + mitigations |
| [secrets.md](./secrets.md) | Where secrets live, rotation policy |
| [pii-policy.md](./pii-policy.md) | What is PII, how it's handled |
| [auth-flow.md](./auth-flow.md) | Detailed login flow with security notes |

## Security checkpoints in the SDLC

| Sprint | Activity |
|---|---|
| S0 | Threat model first draft (this file is the seed) |
| S1 | Secure storage choice locked, secret scanning in CI |
| S2 | OTP flow security review (rate limit, OTP brute force) |
| S5 | Permissions audit (camera, GPS, mic) |
| S7 | Sync envelope validation review (injection, large payload) |
| S9 | Odoo write path review (privilege escalation) |
| S10 | Full security audit + penetration test |

## Non-negotiables

1. No secrets committed to source control.
2. No PII in logs (phone numbers redacted to last 4 digits).
3. TLS 1.2+ everywhere.
4. Tokens at rest only in OS-managed secure stores.
5. Mobile never stores Odoo credentials.
6. Per-action consent for camera, GPS, microphone.
7. All authenticated endpoints validate JWT + employee mapping.
8. Object storage URLs are presigned and short-lived.
9. Rate limits documented and enforced per `api-contracts/`.
10. Security findings P0/P1 trigger escalation per `governance/escalation.md`.
