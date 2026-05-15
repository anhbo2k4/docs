---
id: US-POST-007
epic: EPIC-10
sprint: post-MVP
priority: P2
estimate: L
status: Draft
owner: TBD
---

# US-POST-007 — Push notifications

## User story

**As** a Field Worker
**I want** a push notification when a new shift is assigned or an existing shift changes
**so that** I don't have to refresh the app.

## Acceptance criteria

1. iOS APNs + Android FCM integrated; per-OS opt-in flow.
2. Server emits notifications on shift create / update / cancel.
3. Notifications carry no PII in body; deep link to in-app shift detail.
4. Offline: notifications buffer on the OS; opening the app triggers a sync.
5. Test devices receive notifications within 30 s p95.

## Dependencies

- DEC-016 (push provider config).
- New ADR for notification payload privacy.

## Risks

- Token rotation / revocation when employee changes phone.
