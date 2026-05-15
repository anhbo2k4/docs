---
id: US-PLAT-002
epic: EPIC-01
sprint: S1
fr: []
priority: P0
estimate: M
status: Ready
owner: Mobile
---

# US-PLAT-002 — Wire Riverpod + go_router + theme tokens

## Acceptance criteria

1. `ProviderScope` mounts at `main.dart`.
2. `go_router` configured with auth-guard placeholder (real guard in EPIC-02) and routes for `/login`, `/shifts`, `/shifts/:id`.
3. Theme tokens live under `resource/theme/` (colors, spacing, typography); light/dark variants.
4. One demo screen renders using only theme tokens (no inline magic numbers).
5. Riverpod observers wired to log provider lifecycle in `dev` flavour only.

## Tasks

- [ ] Add `flutter_riverpod`, `go_router`, `intl`.
- [ ] Author theme tokens and `AppTheme.light/dark`.
- [ ] Build `RouterConfig` with placeholder guards.
- [ ] Implement debug logger observer for dev only.

## Dependencies

US-PLAT-001.

## DoD

- Router transitions verified in widget tests.
- Theme switch verified at runtime.
