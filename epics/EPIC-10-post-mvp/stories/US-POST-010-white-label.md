---
id: US-POST-010
epic: EPIC-10
sprint: post-MVP
priority: P2
estimate: L
status: Draft
owner: TBD
---

# US-POST-010 — White-label theming

## User story

**As** a contracting partner
**I want** my logo, primary colour, and app name on the app
**so that** field workers see my brand.

## Acceptance criteria

1. Build flavor `partner-{slug}` consumes a `theme.json` with allowed customisations (logo, primary colour, accent, app name).
2. Asset packs validated by CI (image dimensions, contrast ratios per `engineering/accessibility.md`).
3. Light + dark mode supported per partner.
4. No code branches per partner — configuration only.

## Dependencies

- New ADR for build-flavor matrix.

## Risks

- Apple / Play account ownership; one app or per-partner apps.
