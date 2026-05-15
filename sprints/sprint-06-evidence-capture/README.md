---
sprint: S6
title: Evidence Capture
duration: 2 weeks
priority: P0
status: Ready
owner: Mobile Lead
---

# Sprint 6 — Evidence Capture

## 1. Sprint goal

Voice notes ship: record, transcribe on-device with Whisper tiny.en, edit, and link to a work result. Capture polish wraps up.

## 2. Theme

Local AI inference plus the last bits of evidence capture. Bandwidth is precious — we transcribe on-device and ship text.

## 3. Scope (in)

- Audio recorder + storage.
- Whisper tiny.en bundling and warm-up.
- Transcription + transcript editor.
- Voice envelope assembly + linkage to work result.

## 4. Out of scope

- Background scheduling (S7).
- Server ingest (S8).

## 5. Pre-conditions

- S5 done.
- DEC-010 closed (Whisper bundling).
- Mid-tier device available for memory profiling.

## 6. Stories committed

| ID | Title | Owner | Estimate |
|---|---|---|---|
| US-VOICE-001 | Audio recorder UI + storage | Mobile | M |
| US-VOICE-002 | Whisper tiny.en bundling + warmup | Mobile Lead | L |
| US-VOICE-003 | Transcribe + edit transcript | Mobile | M |
| US-VOICE-004 | Voice envelope assembly + linkage | Mobile | M |

## 7. Risks for this sprint

- RISK-061 Whisper bundle pushes IPA over budget.
- RISK-060 Battery saver suspends background transcription on Android (deferred-OK in S6).

## 8. Sprint-level Definition of Done

- TC-VOICE-001..005 green.
- Transcribe a 60 s clip cold within 30 s on mid-device.
- Memory peak under 350 MB during inference.
- Audio + transcript linked to work result and visible in Sync Center stub.

## 9. Demo script

- Record a 30 s note in the field UI.
- Watch transcript appear, edit a word, attach to a work result.
- Show the linked envelope state.

## 10. Retro inputs

- Did Whisper warm-up impact cold start?
- Was the editor ergonomic in landscape and noisy environments?
- What surprised us about audio file sizes?
