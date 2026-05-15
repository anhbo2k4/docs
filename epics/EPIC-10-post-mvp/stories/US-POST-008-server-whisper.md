---
id: US-POST-008
epic: EPIC-10
sprint: post-MVP
priority: P2
estimate: M
status: Draft
owner: TBD
---

# US-POST-008 — Server-side Whisper fallback

## User story

**As** the platform
**I want** server-side transcription when on-device transcription fails or is too slow
**so that** voice notes always end up transcribed before reaching Odoo.

## Acceptance criteria

1. Mobile uploads audio when on-device transcription confidence < threshold.
2. Backend transcribes asynchronously; result attached to the same envelope by `client_id`.
3. Cost guardrail: daily quota per tenant; over-quota requests deferred.
4. PII guardrail: transcripts stored encrypted at rest; access audited.
5. Mobile UI shows "Transcript pending" until server-side completes.

## Dependencies

- ADR for hosted Whisper choice (managed vs self-hosted).
- Cost / throughput budget added to `architecture/non-functional-requirements.md`.

## Risks

- Provider lock-in; data residency.
