# ADR-008: Whisper tiny.en runs on-device for voice notes

**Status:** Accepted  
**Date:** 2026-05-13  
**Deciders:** Mobile Lead, Engineering Lead, Product Owner  
**Consulted:** Security Champion, Backend Lead  
**Informed:** All

## Context

Field workers use voice notes when typing is impractical (gloves, dust, bad signal). We need short transcripts (<= 60 s) attached to work results. Options were:
- On-device transcription (latency: device-bound).
- Server transcription (latency: includes upload of audio).
- Hybrid (try on-device, fallback server).

Constraints:
- Offline-first: voice notes must transcribe with no connectivity.
- Privacy: voice may capture incidental third-party speech in the field.
- Cost: server-side ASR per minute is non-trivial at scale.

## Decision

Run **Whisper tiny.en** (English) **on-device** via a Flutter binding to whisper.cpp. The model is bundled into the app or downloaded on first launch (decision on bundling vs download deferred to Mobile Lead at S1).

For non-English support, defer to post-MVP (EPIC-10). Server-side transcription is **not** in MVP.

## Rationale

- Offline-first is a non-negotiable principle.
- Privacy: audio never leaves the device for transcription.
- No server cost per minute.
- Whisper tiny.en is good enough for typical 30–60 s field notes.
- Acceptable latency on mid-range Android (≤ 25 s for 60 s audio per NFR-006).

## Alternatives considered

| Option | Pros | Cons | Why not chosen |
|---|---|---|---|
| Server Whisper (large) | Higher accuracy, multilingual | Requires upload, breaks offline, ongoing cost | Fails offline-first |
| Cloud STT (Google / AWS) | Mature | Cost, vendor lock-in, network dependency | Same |
| No transcription, audio only | Simplest | Field admins want searchable text in Odoo | Loses key product value |
| Local model + server fallback | Best of both | Complexity in pilot | Defer to post-MVP |

## Consequences

### Positive
- Voice notes work fully offline.
- Lower data usage and cost.
- Privacy-preserving by default.

### Negative
- App size grows by tens of MB if model is bundled (~75 MB for tiny.en).
- Transcription quality is below cloud baselines; users see "draft transcript, please review" UX.
- Multilingual support deferred.
- whisper.cpp Flutter bindings are evolving; we accept some integration risk.

### Neutral
- Open question: bundle the model in the app or download on first run? Decision at S1, recorded in S1 sprint review.

## Compliance / verification

- Microphone permission is per-action, with a visible recording indicator.
- Audio file is stored locally with a checksum and never auto-uploaded for transcription.
- Transcription cannot leave the device unless the user submits the work result envelope, in which case the *transcript text* travels with the envelope; the audio file uploads via presigned URL only on user submit.
- Tests TC-VOICE-001..005 cover accuracy on a benchmark set, latency, and offline mode.

## Related

- FR-011
- NFR-006, NFR-051
- Stories: `epics/EPIC-05-field-capture/stories/US-VOICE-*.md`
