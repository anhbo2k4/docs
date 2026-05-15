---
id: US-DISC-004
epic: EPIC-00
sprint: S0
fr: []
priority: P0
estimate: S
status: Ready
owner: Backend Lead
---

# US-DISC-004 — Pick SMS provider and capacity plan (DEC-005)

## Acceptance criteria

1. Provider chosen (Twilio only or Twilio + regional fallback).
2. Capacity plan confirmed for pilot: messages/day, burst limit, cost/SMS.
3. DLR (delivery receipt) sampling agreed.
4. DEC-005 closed.

## Tasks

- [ ] Compare Twilio vs regional fallback for pilot region.
- [ ] Forecast pilot volume (workers × logins × retries).
- [ ] Sign capacity contract or sandbox agreement.

## Risks

- RISK-002 Provider not approved in pilot region.

## DoD

- DEC-005 closed.
- Sandbox credentials handed to Backend Lead.
