---
id: US-POST-001
epic: EPIC-10
sprint: post-MVP
fr: [FR-NEW-001]
priority: P2
estimate: XL
status: Draft
owner: TBD
---

# US-POST-001 — Dynamic forms schema + renderer

## User story

**As** a Field Worker
**I want** the pre-shift and post-shift forms to be defined server-side
**so that** the office can change form questions without an app release.

## Acceptance criteria

1. Backend exposes `GET /v1/forms/{slug}` returning a JSON schema versioned by `schema_version`.
2. Mobile renders the schema with text, select, multi-select, number, date, photo-attach widgets.
3. Local SQLite stores the active schema version per form; offline renders from cache.
4. Submitted form payload validates against the schema both on device and on backend.
5. Schema migration plan covers older app versions: app downgrades gracefully if schema_version > supported.
6. Unit + integration tests cover ≥ 5 schemas including required, conditional, and validation rules.
7. Performance: render of a 50-field schema completes in < 600 ms cold (PB-M-002).

## Tasks

### Flutter UI
- [ ] Schema parser + widget mapper.
- [ ] Conditional visibility engine.
- [ ] Caching of last good schema per form slug.

### Domain / Backend
- [ ] Form schema CRUD (admin path).
- [ ] Versioning + migration helpers.
- [ ] Validation engine shared between mobile (Dart) and backend (Pydantic).

### Testing
- [ ] 5 schema fixtures (per `qa/test-data-strategy.md`).
- [ ] e2e: schema change in admin reflects in mobile within 1 sync cycle.

## Dependencies

- EPIC-09 closed; pilot stable.
- ADR proposing the schema language (likely JSON Schema 2020-12 subset).

## Risks / Open questions

- DEC-014 (new): form schema language and storage in Odoo.
- RISK-101: schema versioning regresses old envelopes; needs replay-safe migrations.

## FR mapping

FR-NEW-001 (post-MVP, to be added when promoted).

## Test cases

TC-FORM-201..220 (to be drafted).

## Definition of Done

- All AC pass on iOS and Android.
- Schema versioning matches `engineering/api-versioning.md`.
- Documentation updated under `engineering/`.
