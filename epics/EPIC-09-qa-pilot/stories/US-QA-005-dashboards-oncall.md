---
id: US-QA-005
epic: EPIC-09
sprint: S10
fr: []
priority: P0
estimate: M
status: Ready
owner: DevOps
---

# US-QA-005 — Dashboards + alerting + on-call

## Acceptance criteria

1. SLO dashboard: API success rate, p95 latency, queue lag, DLQ depth, Odoo errors.
2. App dashboard: crash-free sessions, ANR rate, sync success rate.
3. Alert routes: SEV-1 → on-call pager, SEV-2 → channel notification.
4. On-call rotation set up for the pilot window with playbooks (`runbooks/incident-response.md`).
5. One rehearsed incident drill executed in staging.

## Tasks

- [ ] Dashboards.
- [ ] Alert rules.
- [ ] On-call rotation.
- [ ] Drill.

## DoD

- Dashboards link from `qa/pilot-gate.md`.
- Drill notes attached.
