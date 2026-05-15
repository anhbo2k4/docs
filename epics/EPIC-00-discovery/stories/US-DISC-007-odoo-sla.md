---
id: US-DISC-007
epic: EPIC-00
sprint: S0
fr: []
priority: P0
estimate: M
status: Ready
owner: Sponsor + Internal IT
---

# US-DISC-007 — Confirm Odoo SLA, scaling, DR (DEC-008)

## Acceptance criteria

1. Production Odoo SLA documented: target uptime, planned-maintenance windows.
2. Scaling posture: read-replicas, worker count, queue capacity.
3. DR strategy: RPO and RTO targets, backup cadence, restore drill date.
4. DEC-008 closed (or explicitly deferred with documented pilot risk acceptance).

## Tasks

- [ ] Interview Odoo admin / hosting provider.
- [ ] Capture SLA in writing.
- [ ] Identify gaps that block pilot.

## Risks

- RISK-082 Odoo write p95 violates pilot SLA.

## DoD

- DEC-008 closed or explicitly accepted with mitigation.
