---
id: US-VOICE-004
epic: EPIC-06
sprint: S6
fr: [FR-011]
priority: P0
estimate: M
status: Ready
owner: Mobile
---

# US-VOICE-004 — Voice envelope assembly + link to work result

## Acceptance criteria

1. Submission creates a `VOICE_NOTE` envelope with `transcript`, `duration_ms`, and `media_client_id` for the audio file.
2. Audio uploaded via `/v1/media/upload-url` then finalized.
3. Linked work result references the voice envelope `client_id`.
4. Sync state visible at the work result level.

## Tasks

- [ ] `SubmitVoiceNoteUseCase`.
- [ ] Linkage to work result via repository.

## FR mapping

FR-011, FR-010.

## Test cases

TC-VOICE-005.

## DoD

- Envelope round-trip verified end-to-end.
