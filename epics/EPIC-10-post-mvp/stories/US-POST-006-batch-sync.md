---
id: US-POST-006
epic: EPIC-10
sprint: post-MVP
priority: P2
estimate: M
status: Draft
owner: TBD
---

# US-POST-006 — POST /v1/sync/batch

## User story

**As** the mobile sync engine
**I want** to send up to 50 envelopes in one HTTP call
**so that** I save battery and bandwidth.

## Acceptance criteria

1. New endpoint `POST /v1/sync/batch` accepts an array of envelopes (≤ 50).
2. Server processes envelopes individually; per-item success / failure returned.
3. Idempotency by `client_id` per item; whole-batch failure does not re-process succeeded items.
4. Mobile prefers batch when ≥ 5 envelopes pending and bandwidth is reasonable; falls back to per-item.
5. Backwards compatible: per-item endpoint stays for older clients.

## Dependencies

- ADR-007 (idempotency).
- `engineering/api-versioning.md` (additive change within `/v1`).

## Test cases

TC-SYNC-201..210 (to draft).
