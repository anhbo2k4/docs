# ADR-006: Riverpod 2.x for state management

**Status:** Accepted  
**Date:** 2026-05-13  
**Deciders:** Mobile Lead, Engineering Lead  
**Consulted:** Mobile engineers  
**Informed:** All

## Context

The unified app needs a state management layer that:
- Survives complex async flows (sync, capture, transcription).
- Plays well with offline-first patterns and stream-driven DB queries.
- Has compile-time safety and good testability.
- Has a low barrier for new engineers joining the project.

Source brief listed Riverpod as recommended but not mandated. Without an explicit decision, we risk drift across modules.

## Decision

Standardise on **flutter_riverpod 2.5+** with code-generation (`riverpod_generator`) for typed providers. Place provider definitions under `application/providers/` grouped by feature.

## Rationale

- Riverpod separates UI from state with no `BuildContext` coupling.
- Code-gen providers are typed, testable, and refactor-safe.
- Async providers (`FutureProvider`, `StreamProvider`) match Drift / DAO usage.
- Excellent overrides for tests.
- Active community and stable API in 2.x.

## Alternatives considered

| Option | Pros | Cons | Why not chosen |
|---|---|---|---|
| flutter_bloc | Mature, predictable | More boilerplate; teams diverge between Cubit and Bloc styles | Slower to deliver under our timeline |
| Provider 6.x | Simple | Less ergonomic for async; not as testable | Replaced by Riverpod for new projects |
| GetIt + Cubits hybrid | Lightweight | Mixes paradigms; bug-prone as project grows | Inconsistent |
| MobX | Reactive, terse | Less common in Flutter community | Talent pool concern |

## Consequences

### Positive
- Single, consistent state pattern across screens.
- Strong testability via overrides.
- Clear separation of feature providers.

### Negative
- Code generation step in build pipeline (`build_runner watch` for DX).
- Engineers new to Riverpod need a half-day ramp-up.

### Neutral
- Future move to another library would require touching every provider; cost is non-trivial but acceptable given expected stability.

## Compliance / verification

- Lint rule: forbid `flutter_bloc`, `provider`, and `get_it` in mobile dependencies.
- Code review must reject UI files that read from DAOs directly.
- All providers must have a test that overrides at least one dependency.

## Related

- ADR-001
- Stories: `epics/EPIC-01-platform-foundation/`
