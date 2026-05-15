---
id: US-POST-005
epic: EPIC-10
sprint: post-MVP
priority: P2
estimate: L
status: Draft
owner: TBD
---

# US-POST-005 — Map view of GPS captures

## User story

**As** a Field Supervisor
**I want** to see a map of my team's GPS check-ins and check-outs
**so that** I can confirm coverage of an area.

## Acceptance criteria

1. Map view with day / week / custom-range filter.
2. Markers cluster on zoom-out; clusters show count.
3. Tap a marker reveals shift, worker, time.
4. Anonymisation for cross-team views: GPS snapped to 100 m grid.
5. Works offline using last cached batch.

## Dependencies

- US-POST-002 (supervisor view).
- Decision: map provider (Google, Mapbox, OSM) — new ADR.

## Risks

- Map provider cost at scale.
