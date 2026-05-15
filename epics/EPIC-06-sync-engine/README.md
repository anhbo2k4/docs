---
epic: EPIC-06
title: Sync Engine & Background Work
sprint: S6, S7
priority: P0
status: Ready
owner: Mobile Lead
---

# EPIC-06 — Sync Engine & Background Work

## 1. Goal

Drain the local sync queue reliably under flaky networks, OS lifecycle changes, and battery saver. Make sync state visible so workers trust the system.

## 2. In scope

- Background scheduling: WorkManager (Android), BGTaskScheduler (iOS).
- Foreground drain on connectivity change.
- Retry with capped exponential backoff and jitter.
- Dead-letter queue UI: list, reason, retry, copy diagnostics.
- Sync Center screen: per-shift counts, last sync, retry/all-retry.
- Voice note slice: record audio, transcribe with Whisper tiny.en on-device, link transcript to voice envelope.
- Outbox audit log for every transition.
- Cancel-on-logout ensures no leftover jobs after wipe.

## 3. Out of scope

- Server-side ingest (EPIC-07).
- Odoo write semantics (EPIC-08).

## 4. Functional requirements covered

FR-011 (voice + transcription), FR-012, FR-013, FR-015.

## 5. Stories

| ID | Title | Pri | Estimate | Status | Owner |
|---|---|---|---|---|---|
| US-VOICE-001 | Audio recorder UI + storage | P0 | M | Ready | Mobile |
| US-VOICE-002 | Whisper tiny.en bundling + warmup | P0 | L | Ready | Mobile Lead |
| US-VOICE-003 | Transcribe + edit transcript | P0 | M | Ready | Mobile |
| US-VOICE-004 | Voice envelope assembly + link to work result | P0 | M | Ready | Mobile |
| US-SYNC-001 | Sync queue manager + state transitions | P0 | L | Ready | Mobile Lead |
| US-SYNC-002 | Background scheduling Android (WorkManager) | P0 | M | Ready | Mobile |
| US-SYNC-003 | Background scheduling iOS (BGTaskScheduler) | P0 | M | Ready | Mobile |
| US-SYNC-004 | Backoff + jitter + circuit breaker | P0 | M | Ready | Mobile |
| US-SYNC-005 | Sync Center UI | P0 | M | Ready | Mobile + Designer |
| US-SYNC-006 | Dead-letter UI + diagnostics export | P0 | M | Ready | Mobile |

## 6. Dependencies

- EPIC-04 storage live.
- EPIC-07 ingest endpoints reachable in staging by S7 mid-sprint.
- DEC-010 (Whisper bundling vs first-launch download).

## 7. Risks

- RISK-060 Battery saver suspends background work indefinitely.
- RISK-061 Whisper bundle pushes IPA over budget.
- RISK-062 OS background quotas vary across Android OEMs (Xiaomi, Huawei).
- RISK-063 Concurrent retry storm post-reconnect overwhelms backend.

## 8. Definition of Done (epic-level)

- TC-SYNC-001..010, TC-SYNC-024, TC-VOICE-001..005 green.
- 24-hour soak: 1k envelopes pass through end-to-end with zero loss.
- Dead-letter UI surfaces every backend 4xx with usable error message.
- Whisper transcription runs offline within memory budget on mid-tier device.

## 9. Open questions

DEC-010 closed before S6 start.
