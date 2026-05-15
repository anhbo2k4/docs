---
id: US-VOICE-003
epic: EPIC-06
sprint: S6
fr: [FR-011]
priority: P0
estimate: M
status: Ready
owner: Mobile
---

# US-VOICE-003 — Transcribe + edit transcript

## Acceptance criteria

1. After recording, transcription kicks off automatically; UI shows progress.
2. User can edit the transcript before submission.
3. Edited transcript and original transcript are both stored (audit trail).
4. Failures surface a retry option; the audio is preserved.

## Tasks

- [ ] Transcription service.
- [ ] Editor widget.
- [ ] Persistence with audit fields.

## FR mapping

FR-011.

## Test cases

TC-VOICE-003, TC-VOICE-004.

## DoD

- Edit roundtrip verified.
- Failure retry verified.
