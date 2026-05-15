---
id: US-VOICE-001
epic: EPIC-06
sprint: S6
fr: [FR-011]
priority: P0
estimate: M
status: Ready
owner: Mobile
---

# US-VOICE-001 — Audio recorder UI + storage

## Acceptance criteria

1. Recorder screen with start/stop/pause/cancel and a visible duration counter.
2. Audio is saved as M4A (AAC, 32 kbps mono) under app sandbox.
3. Per-recording max duration 5 min; soft warning at 4 min.
4. Mic permission consent shown on first use; recorded in `sync_attempt`.
5. Saved audio creates a `voice_note` row with `sync_state = PENDING_TRANSCRIPTION`.

## Tasks

- [ ] `AudioRecorderPlugin` wrapper.
- [ ] Recorder screen widget.
- [ ] Permission flow.
- [ ] Domain entity + DAO.

## FR mapping

FR-011.

## Test cases

TC-VOICE-001.

## DoD

- Recorder works on both platforms.
- File size and bitrate verified.
