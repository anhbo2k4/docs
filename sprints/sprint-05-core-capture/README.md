---
sprint: S5
title: Core Field Capture
duration: 2 weeks
priority: P0
status: Ready
owner: Mobile Lead + Designer
---

# Sprint 5 — Core Field Capture

## 1. Sprint goal

Workers complete PPE, pre-shift form, and GPS check-in/out fully offline, with durable SQLite writes and visible PENDING state.

## 2. Theme

First wave of capture flows. Voice notes and gallery polish land in S6. The bar: every capture writes locally first, every action is consent-respecting, and the UI never blocks on the network.

## 3. Scope (in)

- PPE checklist UI + submit + badge wiring.
- Pre-shift form UI + submit.
- GPS plugin wrapper + consent.
- Check-in / check-out captures with accuracy guard.
- Photo capture pipeline (compression, EXIF strip, checksum).
- Photo gallery review/delete.
- Work result composer.

## 4. Out of scope

- Voice notes (S6).
- Background sync (S7).
- Server ingest (S8).

## 5. Pre-conditions

- S4 done (DAOs + state machine usable).
- DEC-004 closed (GPS storage target known).
- Static schemas for PPE and pre-shift form approved by PO.

## 6. Stories committed

| ID | Title | Owner | Estimate |
|---|---|---|---|
| US-CAP-001 | PPE checklist UI | Mobile | M |
| US-CAP-002 | PPE submit → SQLite + sync_queue | Mobile | M |
| US-CAP-003 | PPE state badge wired | Mobile | S |
| US-CAP-004 | Pre-shift form UI | Mobile | M |
| US-CAP-005 | Pre-shift submit → SQLite | Mobile | S |
| US-CAP-006 | GPS plugin wrapper + consent | Mobile | M |
| US-CAP-007 | Check-in capture + accuracy guard | Mobile | M |
| US-CAP-008 | Check-out capture + reconcile timing | Mobile | M |
| US-CAP-009 | Photo capture + compression pipeline | Mobile | L |
| US-CAP-010 | EXIF strip + checksum | Mobile + Security | M |
| US-CAP-011 | Photo gallery review/delete | Mobile | M |
| US-CAP-012 | Work result composer | Mobile | M |
| US-CAP-013 | Work result link voice notes (S6 hook) | Mobile | S |

## 7. Risks for this sprint

- RISK-050 Image compression eats CPU on low-end Android.
- RISK-051 GPS accuracy threshold false-rejects valid captures.
- RISK-052 Consent prompts skipped → privacy posture weakens.

## 8. Sprint-level Definition of Done

- TC-PPE-001..003, TC-FORM-001..002, TC-GPS-001..003, TC-MEDIA-001..006, TC-SEC-007 green.
- All captures persist as PENDING with audit rows in `sync_attempt`.
- Photo p95 compression ≤ 800 ms on mid-tier (TC-PERF-005).
- Consent prompts logged in `outbox_audit`.

## 9. Demo script

- Airplane mode on, run a full shift: PPE → pre-shift → check-in → photos → work result → check-out.
- Open SQLite browser to show every row PENDING with its `client_id`.
- Force a GPS skip with reason.

## 10. Retro inputs

- Were any DAO contracts insufficient?
- Were consent prompts intrusive?
- Did the design system handle dense capture screens?
