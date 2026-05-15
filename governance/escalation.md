# Escalation Path

**Status:** Active  
**Owner:** Project Manager

## Severity definitions

| Sev | Definition | Examples |
|---|---|---|
| **SEV-1** | Pilot or production unusable; data loss; security breach | Sync engine corrupts SQLite; OTP service down; PII leaked |
| **SEV-2** | Major feature broken, no workaround | Cannot sync any shift; cannot log in on Android; Odoo writes silently fail |
| **SEV-3** | Feature degraded, workaround exists | One photo upload retries excessively; shift list shows stale data |
| **SEV-4** | Minor issue, cosmetic, low impact | Typo, layout glitch, non-blocking warning |

## Response targets (during pilot)

| Sev | Acknowledge | First update | Resolution target | Postmortem required |
|---|---|---|---|---|
| SEV-1 | 15 min | 1 h | 4 h | Yes |
| SEV-2 | 30 min | 2 h | 1 business day | Yes |
| SEV-3 | 4 h | 1 business day | 5 business days | Optional |
| SEV-4 | 1 business day | — | Next sprint | No |

## Escalation chain

```
Reporter → On-call Engineer → Engineering Lead → Project Manager → Sponsor
                              ↓                   ↓
                          Security Champion (if security)
                          Backend Lead / Mobile Lead (per area)
```

### When to escalate

| Trigger | Escalate to |
|---|---|
| SEV-1 detected | On-call → EL → PM (immediately, parallel) |
| SEV-2 with no progress in 2 h | EL → PM |
| Phase 0 decision (DEC-NNN) blocked > 5 business days | PM → Sponsor |
| Scope change request | PO → PM → Sponsor |
| Budget overrun > 10% | PM → Sponsor |
| Pilot gate failure | QA → EL → PO → Sponsor |
| Security finding (P0 or P1) | Anyone → SEC → EL → Sponsor |

## On-call rotation (pilot phase)

- Primary on-call: 1 engineer per week, rotates across mobile + backend.
- Secondary: EL (always).
- Hours: 24/7 during pilot; business hours pre-pilot.
- Tooling: PagerDuty integration with Slack `#field-app-incidents`.

## After-action

Every SEV-1 and SEV-2 incident must produce:

1. Incident timeline (`runbooks/incidents/INC-YYYYMMDD-NNN.md`).
2. Root cause analysis (5 whys minimum).
3. Action items added to backlog with owners.
4. Postmortem meeting within 5 business days.
5. Update to relevant runbook if response was slower than target.
