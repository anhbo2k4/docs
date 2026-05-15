---
epic: EPIC-00
title: Discovery & Build Authorization
sprint: S0
priority: P0
status: Done
owner: Project Manager
closed: 2026-05-15
---

# EPIC-00 — Discovery & Build Authorization

> **Status:** Done (2026-05-15). All 9 Phase 0 decisions resolved in Blueprint v2.0.0. 5 are `Decided` (sponsor-confirmed via ADR-010..014); 4 are `Provisional` defaults under PM confirmation SLA. See `traceability/decision-log.md`.

## 1. Goal

Close every Phase 0 unknown that would otherwise block S1, and obtain a signed Statement of Work from the sponsor so the build can begin.

## 2. In scope

- Workshops with sponsor, IT, HR, and Odoo admin.
- Closing the **9 Phase 0 decisions** (DEC-001 through DEC-009) tracked in `traceability/decision-log.md`.
- Confirming Odoo version, modules, and access path.
- Confirming SMS provider and rate plan.
- Confirming object storage target and region.
- Confirming container orchestration target.
- Producing the build SOW: scope, milestones, cost bracket, exit criteria.
- Scheduling kickoff for S1.

## 3. Out of scope

- Any production code.
- Any Odoo customisation. (Custom module `field_mobile_sync` is **planned** in S0 but **built** in S8/S9.)
- Pilot user recruitment (occurs in S9/S10).

## 4. Functional requirements covered

None directly. EPIC-00 is the gate that all FRs depend on.

## 5. Stories — all closed

| ID | Title | Pri | Estimate | Status | Closed by |
|---|---|---|---|---|---|
| US-DISC-001 | Run Odoo discovery workshop (DEC-001, DEC-002) | P0 | M | Done | ADR-011 + provisional DEC-002 |
| US-DISC-002 | Confirm employee phone-mapping strategy (DEC-006) | P0 | S | Done | Provisional DEC-006 (work_phone, E.164) |
| US-DISC-003 | Decide custom Odoo module scope (DEC-003, DEC-004) | P0 | M | Done | ADR-013, ADR-014 |
| US-DISC-004 | Pick SMS provider and capacity plan (DEC-005) | P0 | S | Done | ADR-010 |
| US-DISC-005 | Pick object storage and region (DEC-007) | P0 | S | Done | ADR-012 |
| US-DISC-006 | Pick container orchestration target (DEC-009) | P0 | S | Done | Provisional DEC-009 (single-VM Compose for pilot) |
| US-DISC-007 | Confirm Odoo SLA, scaling, DR (DEC-008) | P0 | M | Done | Provisional DEC-008 (degraded SLA documented) |
| US-DISC-008 | Produce SOW + cost bracket | P0 | M | Done | SOW v1.0 stored with project record |
| US-DISC-009 | Sponsor sign-off + kickoff schedule | P0 | S | Done | Sponsor email captured 2026-05-15 |

## 6. Dependencies

- Sponsor availability for sign-off — met.
- Odoo admin availability for live walkthrough — met.

## 7. Risks (resolved or carried forward)

- RISK-001 Odoo decisions slip → **mitigated**, decisions closed.
- RISK-002 SMS provider not approved in pilot region → **mitigated** (ADR-010 Twilio approved).
- RISK-003 Custom Odoo module timeline underestimated → **carried**: tracked in `backlog/risk-register.md`; re-assessed at S8 entry.

## 8. Definition of Done (epic-level) — all met

- ✅ All 9 Phase 0 decisions resolved (5 Decided, 4 Provisional under PM SLA).
- ✅ SOW signed and stored with the project record.
- ✅ Kickoff date for S1 fixed on the calendar.
- ✅ Sponsor email confirmation captured.

## 9. Open questions

None. Provisional items (DEC-002, DEC-006, DEC-008, DEC-009) are tracked in `traceability/decision-log.md` and ride along with PM confirmation SLA — they do not gate S1 entry.
