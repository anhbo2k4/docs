# Functional Requirements Traceability Matrix

**Status:** Active  
**Owner:** QA Lead + Engineering Lead  
**Last updated:** 2026-05-13

Maps each Functional Requirement (FR) to Epic(s), Sprint(s), Story(ies), and Test Case(s). Single source of truth for traceability.

## Authentication (FR-001 — FR-003)

| FR | Title | Description | Epic | Sprint | Stories | Test Cases | Priority |
|---|---|---|---|---|---|---|---|
| **FR-001** | Phone OTP request | User enters phone, receives 6-digit SMS code within 30 s | EPIC-02 | S2 | US-AUTH-001, US-AUTH-002 | TC-AUTH-001, TC-AUTH-002, TC-AUTH-041 | P0 |
| **FR-002** | OTP verification + session | Verify code, exchange Supabase JWT for internal JWT, persist in secure storage | EPIC-02 | S2 | US-AUTH-003, US-AUTH-004, US-AUTH-005 | TC-AUTH-003, TC-AUTH-004, TC-AUTH-005, TC-AUTH-006 | P0 |
| **FR-003** | Token refresh + logout | Refresh access token transparently; logout clears all secure storage | EPIC-02 | S2 | US-AUTH-006, US-AUTH-007 | TC-AUTH-007, TC-AUTH-008 | P0 |

## Shifts (FR-004 — FR-005)

| FR | Title | Description | Epic | Sprint | Stories | Test Cases | Priority |
|---|---|---|---|---|---|---|---|
| **FR-004** | My Shifts list | Display shifts assigned to authenticated employee, sorted by start time, with sync state | EPIC-03 | S3 | US-SHIFT-001, US-SHIFT-002, US-SHIFT-003 | TC-SHIFT-001, TC-SHIFT-002 | P0 |
| **FR-005** | Shift detail | Show full shift details, navigate to capture screens, show sync states | EPIC-03 | S3 | US-SHIFT-004, US-SHIFT-005 | TC-SHIFT-003, TC-SHIFT-004 | P0 |

## Capture (FR-006 — FR-011)

| FR | Title | Description | Epic | Sprint | Stories | Test Cases | Priority |
|---|---|---|---|---|---|---|---|
| **FR-006** | PPE check-in | Worker completes PPE checklist before starting shift; durable offline | EPIC-05 | S5 | US-CAP-001, US-CAP-002, US-CAP-003 | TC-PPE-001, TC-PPE-002, TC-PPE-003 | P0 |
| **FR-007** | Pre-shift form | Worker fills pre-shift form (static schema for MVP) | EPIC-05 | S5 | US-CAP-004, US-CAP-005 | TC-FORM-001, TC-FORM-002 | P0 |
| **FR-008** | GPS check-in / check-out | Capture GPS coordinates with consent at start and end of shift | EPIC-05 | S5 | US-CAP-006, US-CAP-007, US-CAP-008 | TC-GPS-001, TC-GPS-002, TC-GPS-003 | P0 |
| **FR-009** | Photo capture | Take 1–N photos, compress to ≤ 500 KB, store with checksum | EPIC-05/06 | S5/S6 | US-CAP-009, US-CAP-010, US-CAP-011 | TC-MEDIA-001, TC-MEDIA-002, TC-MEDIA-003, TC-MEDIA-004 | P0 |
| **FR-010** | Work result | Compose work result with notes, photos, GPS, voice notes | EPIC-05/06 | S5/S6 | US-CAP-012, US-CAP-013 | TC-MEDIA-005, TC-MEDIA-006 | P0 |
| **FR-011** | Voice note + transcription | Record audio, transcribe on-device with Whisper tiny.en | EPIC-06 | S6 | US-VOICE-001, US-VOICE-002, US-VOICE-003, US-VOICE-004 | TC-VOICE-001..005 | P0 |

## Sync (FR-012 — FR-015)

| FR | Title | Description | Epic | Sprint | Stories | Test Cases | Priority |
|---|---|---|---|---|---|---|---|
| **FR-012** | Sync queue durability | Captures persist to SQLite first; sync_queue rows survive app kill | EPIC-04/06 | S4/S7 | US-OFF-001, US-OFF-002, US-SYNC-001 | TC-OFF-001..005, TC-SYNC-001 | P0 |
| **FR-013** | Background sync | Drain queue on connectivity / OS schedule; show progress in UI | EPIC-06 | S7 | US-SYNC-002, US-SYNC-003, US-SYNC-004 | TC-SYNC-002, TC-SYNC-003, TC-SYNC-004, TC-SYNC-005 | P0 |
| **FR-014** | Idempotent backend ingest | FastAPI dedupes by `client_id`; Celery writes to Odoo idempotently | EPIC-07/08 | S8/S9 | US-API-001..005, US-ODOO-001..006 | TC-SYNC-006..010, TC-ODOO-001..006, TC-SYNC-024 | P0 |
| **FR-015** | Sync visibility + retry | Show PENDING / SYNCING / CONFIRMED / FAILED / DEAD_LETTER; manual retry from Sync Center | EPIC-06 | S2/S7 | US-SYNC-005, US-SYNC-006 | TC-SYNC-008, TC-SYNC-009 | P0 |

## Coverage check

Every FR has at least one story and one test case. Every story belongs to exactly one epic and one sprint. The matrix is regenerated from story front matter at `qa/pilot-gate.md` time and again at S10.

## How to use this matrix

- **Engineer:** find your story, follow links to AC and TC.
- **QA:** run all TCs for an FR before marking that FR complete.
- **PO:** filter by Priority to see MVP scope.
- **PM:** track FR completion by sprint to gauge velocity.
