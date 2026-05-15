---
id: US-SYNC-001
epic: EPIC-06
sprint: S7
fr: [FR-012, FR-013, FR-015]
priority: P0
estimate: L
status: Ready
owner: Mobile Lead
---

# US-SYNC-001 — Sync queue manager + state transitions

## User story

**As** the mobile sync engine
**I want** a single coordinator that drives every envelope from PENDING to a terminal state
**so that** offline captures reach Odoo exactly once with visible progress and no UX-blocking work.

## Background

Sync is the single most error-prone area of the app. ADR-007 (idempotency by `client_id`) and ADR-005 (no direct mobile-to-Odoo) anchor the contract. The state machine is normative in `data/sync-state-machine.md` and reflected visually in `data/erd.md`.

## Acceptance criteria

1. `SyncQueueManager` is a singleton owned by `application/sync_orchestrator.dart`. It is the only writer of `sync_queue.state` outside the SQLite layer.
2. It reads `sync_queue` rows in FIFO order by `next_attempt_at` and dispatches each to a per-type `EnvelopeUploader` registered at startup.
3. Concurrency:
   - At most **2 envelopes in-flight** for non-media types, configurable via `kSyncMaxInFlight`.
   - Media envelopes (`MEDIA_FINALIZE`, voice audio uploads) are serialised: 1 in-flight at a time.
4. State transitions follow `data/sync-state-machine.md`:
   - `PENDING → SYNCING` on attempt start.
   - `SYNCING → CONFIRMED` on backend 2xx + Odoo `odoo_ref` populated.
   - `SYNCING → FAILED` on retryable error (network, 5xx); reset to `PENDING` after backoff.
   - `SYNCING → DEAD_LETTER` on non-retryable 4xx (validation).
5. Cancellation safety: on app start, any row in `SYNCING` left behind from a previous process is reset to `PENDING` with `attempt_count` unchanged.
6. Idempotency: every uploader sends `Idempotency-Key` header equal to the body `client_id`. Reusing a `client_id` for a different payload is a programming error (asserted in debug, surfaced to Sentry in release).
7. Backoff: exponential 2 s → 60 s, jittered ±20 %, capped. Jitter is deterministic per `client_id` to make tests reproducible.
8. Circuit breaker: after 5 consecutive failures across all types, manager pauses for 60 s; one probe envelope half-opens; success closes the circuit.
9. Observability:
   - All transitions emit a Riverpod event via `syncEventBusProvider`.
   - Each transition writes a `sync_attempt` row.
   - Metrics counter `sync_envelope_attempt_total{type,outcome}` is incremented (sent to backend on next health beacon; no per-event network call).
10. Backpressure: if `sync_queue` count > 200, manager surfaces a warning state read by the Sync Center UI; does not stop draining.
11. Battery: when device on cellular and battery < 15 %, pause non-essential uploads (photos/audio) until conditions improve; user can override via Sync Center "Send now".

## Tasks

### Mobile

- [ ] `@core/sync/sync_queue_manager.dart` with a finite-state internal scheduler.
- [ ] `@core/sync/uploader_registry.dart` mapping envelope type → uploader.
- [ ] Per-type uploaders for `PPE_CHECK`, `FORM_RESPONSE`, `GPS_EVENT`, `WORK_RESULT`, `VOICE_NOTE`; media uploaders live in their own module but plug through the same registry.
- [ ] Boot reconciliation in `application/bootstrap/sync_bootstrap.dart` (resets stuck `SYNCING` rows, schedules a drain).
- [ ] Backoff utility with deterministic jitter under test seed.
- [ ] Circuit breaker.
- [ ] Riverpod event bus + selectors for Sync Center UI.

### Tests

- [ ] Unit: scheduler picks oldest `next_attempt_at` first.
- [ ] Unit: backoff is bounded and deterministic given a seed.
- [ ] Unit: circuit breaker opens/half-opens/closes per spec.
- [ ] Integration: `TC-SYNC-001`, `TC-SYNC-002`, `TC-SYNC-013` (background resume), `TC-SYNC-018` (concurrent capture).
- [ ] Soak: 1k envelopes mixed types, no leaks, queue drains in order.
- [ ] Chaos: random network errors and process kills; final state always reachable from any intermediate state.

## Dependencies

- US-OFF-001 (schema v1).
- US-OFF-002 (DAOs + repositories).
- US-OFF-003 (state machine).
- API contract `api-contracts/sync.md` finalised; backend stub available in S7 by US-API-004.

## Risks / Open questions

- RISK-006 (idempotency contract violation by client editing rows). Mitigation: assertion + Sentry breadcrumb.
- DEC-007 (object storage) affects media uploaders, not this story directly.

## FR mapping

FR-012 (durability), FR-013 (background sync), FR-015 (visibility + retry).

## Test cases

`TC-SYNC-001`, `TC-SYNC-002`, `TC-SYNC-003`, `TC-SYNC-011`, `TC-SYNC-012`, `TC-SYNC-013`, `TC-SYNC-018`, `TC-SYNC-024`.

## Definition of Done

- All AC pass on iOS + Android.
- 1k-envelope soak passes with zero loss and stable memory (RSS < 250 MB peak — TC-PERF-012).
- 1 chaos test green (process kill mid-flight; queue resumes deterministically).
- No PII in logs (verified by automated test).
- Mobile Lead + Backend Lead reviewed contract assumptions.
- Updated `data/sync-state-machine.md` if any transition diverged.
- Coverage on `@core/sync` ≥ 80 % line.
- Story status set to `Done` with PR link.
