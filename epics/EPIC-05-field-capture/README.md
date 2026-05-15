---
epic: EPIC-05
title: Field Capture Workflow
sprint: S5, S6
priority: P0
status: Ready
owner: Mobile Lead + Designer
---

# EPIC-05 — Field Capture Workflow

## 1. Goal

Workers complete every capture (PPE, pre-shift form, GPS check-in/out, photos, work result) safely offline, with consent prompts and durable writes to SQLite.

## 2. In scope

- PPE checklist screen with configurable items (static schema for MVP).
- Pre-shift form screen (static schema for MVP).
- GPS check-in / check-out with consent prompts and accuracy threshold.
- Photo capture with on-device compression (≤ 500 KB target) and EXIF strip.
- Work result composition: notes + linked photos + linked voice notes (voice arrives in S6).
- Per-action consent flows for camera, GPS, microphone.
- Local persistence first; sync envelope creation deferred to EPIC-06/07.

## 3. Out of scope

- Voice notes (covered in EPIC-06 voice slice).
- Sync to backend (EPIC-06 sync engine + EPIC-07 backend).
- Dynamic forms (post-MVP).

## 4. Functional requirements covered

FR-006, FR-007, FR-008, FR-009, FR-010 (voice slice in EPIC-06).

## 5. Stories

| ID | Title | Pri | Estimate | Status | Owner |
|---|---|---|---|---|---|
| US-CAP-001 | PPE checklist UI | P0 | M | Ready | Mobile |
| US-CAP-002 | PPE submit → SQLite + sync_queue | P0 | M | Ready | Mobile |
| US-CAP-003 | PPE state badge wired | P0 | S | Ready | Mobile |
| US-CAP-004 | Pre-shift form UI | P0 | M | Ready | Mobile |
| US-CAP-005 | Pre-shift submit → SQLite | P0 | S | Ready | Mobile |
| US-CAP-006 | GPS plugin wrapper + consent | P0 | M | Ready | Mobile |
| US-CAP-007 | Check-in capture + accuracy guard | P0 | M | Ready | Mobile |
| US-CAP-008 | Check-out capture + reconcile timing | P0 | M | Ready | Mobile |
| US-CAP-009 | Photo capture + compression pipeline | P0 | L | Ready | Mobile |
| US-CAP-010 | EXIF strip + checksum | P0 | M | Ready | Mobile + Security |
| US-CAP-011 | Photo gallery review/delete | P0 | M | Ready | Mobile |
| US-CAP-012 | Work result composer | P0 | M | Ready | Mobile |
| US-CAP-013 | Work result link voice notes (S6 hook) | P0 | S | Ready | Mobile |

## 6. Dependencies

- EPIC-04 (durable storage).
- DEC-004 closed (GPS storage target).
- ADR-008 (Whisper) accepted for the voice hook.

## 7. Risks

- RISK-050 Image compression eats CPU budget on low-end Android.
- RISK-051 GPS accuracy threshold false-rejects valid captures.
- RISK-052 Consent prompts skipped by users → privacy posture weakens.

## 8. Definition of Done (epic-level)

- TC-PPE-001..003, TC-FORM-001..002, TC-GPS-001..003, TC-MEDIA-001..006 green.
- All captures persist to SQLite with sync state `PENDING`.
- Consent prompts logged with timestamp in `sync_attempt`.
- Photo p95 compression ≤ 800 ms on the mid-tier device (TC-PERF-005).

## 9. Open questions

DEC-004 must be closed before US-CAP-007 starts.
