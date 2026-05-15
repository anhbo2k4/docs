---
epic: EPIC-10
title: Post-MVP Enhancements
sprint: post-MVP
priority: P2
status: Draft
owner: Product Owner
---

# EPIC-10 — Post-MVP Enhancements

## 1. Goal

Capture the post-pilot work that increases scale, autonomy, and ergonomics without changing the core architecture. This epic is **not part of MVP** and is sequenced after pilot stabilisation.

## 2. In scope (proposed)

- Dynamic forms (server-driven schema) replacing static MVP forms.
- Supervisor mobile view (read-only) with team roll-up.
- Multi-language UI (i18n hooks already present).
- Biometric unlock for re-auth.
- Map view of GPS captures.
- Batch sync `POST /v1/sync/batch` (deferred from EPIC-07).
- Server-side voice transcription fallback.
- Push notifications for shift updates.
- Offline OCR for forms.
- White-label theming for contracting partners.
- Federated identity (SSO) per partner.

## 3. Out of scope

Anything that contradicts an existing ADR without first superseding it.

## 4. Functional requirements covered

None of FR-001..FR-015 directly. New FRs will be added as items here graduate to a sprint.

## 5. Stories

| ID | Title | Pri | Estimate | Status | Owner |
|---|---|---|---|---|---|
| US-POST-001 | Dynamic forms schema + renderer | P2 | XL | Draft | TBD |
| US-POST-002 | Supervisor mobile read-only | P2 | XL | Draft | TBD |
| US-POST-003 | i18n + first non-English locale | P2 | L | Draft | TBD |
| US-POST-004 | Biometric unlock | P2 | M | Draft | TBD |
| US-POST-005 | Map view of GPS captures | P2 | L | Draft | TBD |
| US-POST-006 | `POST /v1/sync/batch` | P2 | M | Draft | TBD |
| US-POST-007 | Push notifications | P2 | L | Draft | TBD |
| US-POST-008 | Server-side Whisper fallback | P2 | M | Draft | TBD |
| US-POST-009 | OCR for forms | P2 | XL | Draft | TBD |
| US-POST-010 | White-label theming | P2 | L | Draft | TBD |
| US-POST-011 | SSO federation per partner | P2 | XL | Draft | TBD |

## 6. Dependencies

- Pilot stabilisation done (EPIC-09 closed).
- Re-baselined NFRs after pilot data.

## 7. Risks

- RISK-100 Scope creep into MVP.
- RISK-101 SSO drives auth model change incompatible with current JWT design.

## 8. Definition of Done (epic-level)

Each story enters MVP-equivalent rigor (DoR/DoD per `traceability/definitions.md`) before promotion to a real sprint.

## 9. Open questions

None blocking. Items are pre-prioritised; sequencing is decided post-pilot.
