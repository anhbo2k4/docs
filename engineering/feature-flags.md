# Feature flags

**Status:** Active
**Owner:** Mobile Lead + Backend Lead

## Why

- Decouple deploy from release.
- Ship dark; enable per cohort.
- Reduce blast radius for cross-cutting changes.

## Tooling (MVP)

- Flag service: simple `feature_flags` table in Postgres + cache; resolved at `/v1/auth/exchange` and refreshed every 30 min.
- Mobile: `FlagsRepository` (Riverpod-provided) with last-known-good cache in SQLite. Falls back to defaults offline.
- Backend: `Flags` service injected into routers and Celery tasks.

## Flag taxonomy

| Type | Lifespan | Owner |
|---|---|---|
| **release** | Days to a sprint. Removed after rollout. | Story author. |
| **ops** | Indefinite. Toggleable in incident. | Backend Lead. |
| **experiment** | Sprint or two. Removed after readout. | PO. |
| **kill-switch** | Permanent. Disables a feature in emergency. | Security Champion. |

## Naming

`<area>.<feature>.<modifier>`:

- `sync.background.enabled`
- `auth.refresh.rotation`
- `media.compression.aggressive`
- `voice.whisper.bundled` (linked to DEC-010)

## Defaults

- Defaults must be safe-when-offline. Mobile assumes flag = false unless explicitly cached otherwise.
- New release flags default off in production until a sprint review approves rollout.

## Lifecycle

1. Author adds flag to `feature_flags` seed migration with default off.
2. Code reads flag at the boundary (router or screen state).
3. Story for cleanup added to next sprint after full rollout.
4. CI lint warns on flags older than 90 days without a cleanup story.

## Forbidden patterns

- Reading the same flag in multiple layers; resolve once at the boundary.
- Branching deeply on a flag; gate at composition root.
- Using flags as configuration. Configuration goes in env or a config table.
