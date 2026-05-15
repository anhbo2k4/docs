---
id: US-OFF-005
epic: EPIC-04
sprint: S4
fr: [FR-012]
priority: P0
estimate: M
status: Ready
owner: Mobile
---

# US-OFF-005 — Migration harness + downgrade guard

## Acceptance criteria

1. Migration registry maps `(from, to) → migration function`.
2. Forward migrations run automatically on app start, in a single transaction; failure rolls back to prior state.
3. Downgrade attempt is detected (current version > app's max known) and triggers a wipe + re-login flow.
4. Migrations have integration tests using a snapshot of the previous schema.
5. CI step replays migrations on the previous version's DB to validate forward compatibility.

## Tasks

- [ ] Implement registry and runner.
- [ ] Snapshot harness (commit fixtures under `test/fixtures/db/v1.sqlite`).
- [ ] CI replay step.

## FR mapping

FR-012.

## Test cases

TC-OFF-004.

## DoD

- One no-op migration v1→v1 exercised in CI.
- Downgrade case verified.
