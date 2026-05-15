---
id: US-POST-002
epic: EPIC-10
sprint: post-MVP
priority: P2
estimate: XL
status: Draft
owner: TBD
---

# US-POST-002 — Supervisor mobile read-only

## User story

**As** a Field Supervisor
**I want** a read-only mobile view of my team's shifts and submissions
**so that** I can review without leaving the field.

## Acceptance criteria

1. Login as a user mapped to an Odoo `hr.employee` with the `field_supervisor` group lands on a Team Dashboard.
2. Dashboard shows shifts grouped by worker, status (planned, in-progress, completed, late).
3. Tapping a shift opens read-only detail with PPE, GPS, photos, voice transcripts, work result.
4. No write actions are possible.
5. PII rules in `security/data-classification.md` honoured (lat/lng to 100 m grid for non-owners).
6. Push notifications opt-in for "shift completed" events.

## Tasks

### Flutter UI
- [ ] Team Dashboard scaffold.
- [ ] Shift detail (read-only).
- [ ] Role-based routing.

### Backend
- [ ] `GET /v1/team/shifts` with supervisor scope.
- [ ] Authorisation rule using Odoo group membership.

### Testing
- [ ] Cross-role tests: worker login cannot access supervisor routes.

## Dependencies

- EPIC-09 closed.
- ADR for role-scoped API responses.

## Risks / Open questions

- DEC-015: how supervisor → team mapping resolves in Odoo (project member, manager_id, custom).

## Definition of Done

Same as MVP DoD.
