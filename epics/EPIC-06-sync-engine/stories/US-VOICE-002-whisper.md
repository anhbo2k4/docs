---
id: US-VOICE-002
epic: EPIC-06
sprint: S6
fr: [FR-011]
priority: P0
estimate: L
status: Ready
owner: Mobile Lead
---

# US-VOICE-002 — Whisper tiny.en bundling + warmup

## Acceptance criteria

1. Whisper tiny.en model packaged per DEC-010 (bundled or first-launch download).
2. Model warm-up runs once after install on a background isolate; success cached.
3. APK / IPA size budget respected (NFR-110, NFR-111).
4. Inference runs off the UI isolate; UI never blocks.
5. Memory footprint ≤ 350 MB peak on the mid-tier device.

## Tasks

- [ ] FFI binding or `flutter_whisper` integration.
- [ ] Background isolate setup.
- [ ] Bundling vs download switch (per DEC-010).
- [ ] Memory probe in QA harness.

## Dependencies

DEC-010.

## FR mapping

FR-011.

## Test cases

TC-VOICE-002.

## DoD

- Cold transcribe under 30 s for a 60 s clip.
- Memory under budget.
