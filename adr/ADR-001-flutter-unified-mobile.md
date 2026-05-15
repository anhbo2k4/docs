# ADR-001: Use Flutter for the unified mobile app

**Status:** Accepted  
**Date:** 2026-05-13  
**Deciders:** Engineering Lead, Mobile Lead, Product Owner  
**Consulted:** Backend Lead, Sponsor  
**Informed:** All

## Context

We need to unify two systems: a legacy "Field Work" React/Lovable PWA and a planned Flutter contracting app. Field workers operate in low-connectivity environments and need durable offline behaviour, native camera/GPS/audio access, and background sync that survives app kill. The team has Flutter expertise; building two native apps is not feasible within the timeline and budget.

Constraints:
- Single codebase for iOS and Android
- Native access to camera, GPS, microphone, background tasks
- Offline-first architecture
- Strong type system for sync envelope correctness
- 22-week pilot timeline, team of 4–6

## Decision

Build the unified app in **Flutter 3.22+** with **Dart 3.4+** as the single mobile codebase for iOS 14+ and Android 8+ (API 26+).

## Rationale

- One codebase, two platforms: matches team size and timeline.
- Mature plugin ecosystem for camera, geolocator, record, workmanager.
- Strong null safety and type system reduce sync-layer bugs.
- Native iOS BGTaskScheduler and Android WorkManager have stable Flutter bindings.
- Layer-first project structure scales to multi-feature app without re-architecture.
- Hot reload accelerates UI iteration.

## Alternatives considered

| Option | Pros | Cons | Why not chosen |
|---|---|---|---|
| Two native apps (Swift + Kotlin) | Best per-platform polish | Doubles team size and timeline | Out of budget |
| React Native | Reuse some Lovable code | Bridge perf for camera/audio, weaker offline DB story | Sync engine reliability concerns |
| Continue PWA + new RN app | Fastest short-term | Two codebases, PWA cannot do reliable background sync on iOS | Fails offline-first principle |
| Kotlin Multiplatform Mobile | Native UI + shared logic | Smaller talent pool, less mature for our use | Team has Flutter, not KMM |

## Consequences

### Positive
- Single codebase, single team.
- Predictable build and test pipeline.
- Plugin maturity for our hardware needs.

### Negative
- Larger app binary than fully native (~80 MB Android target).
- iOS background task quirks require careful BGTask scheduling.
- Engineers shipping Flutter must know platform channels for edge cases.

### Neutral
- Team must standardise on a state management library (see ADR-006).

## Compliance / verification

- All mobile code lives in `mobile/` in the project repo.
- CI runs `flutter analyze` and `flutter test` on every PR.
- No native iOS or Android app may be created without superseding ADR.

## Related

- FR-001 to FR-015 (all mobile-facing FRs)
- ADR-006 (Riverpod), ADR-008 (Whisper on-device)
- Stories: all under `epics/EPIC-01-platform-foundation/`
