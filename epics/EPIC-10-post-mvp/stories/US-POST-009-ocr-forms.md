---
id: US-POST-009
epic: EPIC-10
sprint: post-MVP
priority: P2
estimate: XL
status: Draft
owner: TBD
---

# US-POST-009 — OCR for forms

## User story

**As** a Field Worker
**I want** to photograph a paper form and have fields auto-filled
**so that** I don't retype site sign-in sheets.

## Acceptance criteria

1. OCR runs on-device for one supported template.
2. Extracted fields are editable before submission.
3. Confidence per field exposed; low-confidence fields highlighted.
4. Privacy: raw photo retained only until submission; thumbnails redacted.
5. Accuracy: ≥ 85 % field accuracy on the pilot template set.

## Dependencies

- ADR for OCR engine choice (MLKit, Tesseract, custom).

## Risks

- Template drift; long tail of paper forms.
