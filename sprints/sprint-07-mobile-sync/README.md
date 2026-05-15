---
sprint: S7
title: Mobile Sync Engine
duration: 2 weeks
priority: P0
status: Ready
owner: Mobile Lead
---

# Sprint 7 — Mobile Sync Engine

## 1. Sprint goal

Drain the local sync queue reliably under flaky networks, OS lifecycle changes, and battery saver. Make sync state visible.

## 2. Theme

The reliability sprint on mobile side. After this sprint, no capture can quietly disappear.

## 3. Scope (in)

- Sync queue manager + state transitions.
- WorkManager (Android) and BGTaskScheduler (iOS) wiring.
- Backoff + jitter + circuit breaker.
- Sync Center UI.
- Dead-letter UI + diagnostics export.

## 4. Out of scope

- Server-side ingest (S8 — mobile points at staging mock until then).
- Odoo writes (S9).

## 5. Pre-conditions

- S6 done.
- Staging FastAPI mock available with `/v1/sync/envelope` and `/v1/sync/status/{client_id}` returning representative responses.

## 6. Stories committed

| ID | Title | Owner | Estimate |
|---|---|---|---|
| US-SYNC-001 | Sync queue manager + transitions | Mobile Lead | L |
| US-SYNC-002 | Background scheduling Android | Mobile | M |
| US-SYNC-003 | Background scheduling iOS | Mobile | M |
| US-SYNC-004 | Backoff + jitter + circuit breaker | Mobile | M |
| US-SYNC-005 | Sync Center UI | Mobile + Designer | M |
| US-SYNC-006 | Dead-letter UI + diagnostics export | Mobile | M |

## 7. Risks for this sprint

- RISK-060 Battery saver suspends background work indefinitely.
- RISK-062 OEM background quotas vary (Xiaomi, Huawei).
- RISK-063 Concurrent retry storm post-reconnect overwhelms backend.

## 8. Sprint-level Definition of Done

- TC-SYNC-001..010 + TC-SYNC-024 green.
- 24-hour soak: 1k envelopes pass through end-to-end with zero loss.
- Dead-letter UI surfaces every backend 4xx with usable error message.
- Storm test (50 concurrent retries) yields exactly one persisted envelope.

## 9. Demo script

- Capture 20 envelopes offline.
- Toggle network on; watch Sync Center counts go from PENDING → CONFIRMED.
- Force a 422 from staging; observe DEAD_LETTER + diagnostics export.

## 10. Retro inputs

- Did OEM background quotas surprise us?
- Was the dead-letter UI usable for support?
- Did the circuit breaker over- or under-react?
