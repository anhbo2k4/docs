---
id: US-QA-004
epic: EPIC-09
sprint: S10
fr: []
priority: P0
estimate: M
status: Ready
owner: QA + Mobile
---

# US-QA-004 — Accessibility audit on critical flows

## Acceptance criteria

1. Critical flows tested with screen readers (TalkBack, VoiceOver): login, OTP, shifts list, shift detail, PPE, photo capture, sync center.
2. Text scaling 200% does not break layouts on critical screens.
3. Contrast ratios meet AA on critical text (verified with tooling).
4. Tap targets ≥ 44x44 pt (iOS) / 48x48 dp (Android).

## Tasks

- [ ] Audit checklist run.
- [ ] Defects logged with severity.
- [ ] Re-test after fixes.

## DoD

- Critical flows pass; cosmetic issues tracked post-pilot.
- Accessibility report attached to pilot gate.

## Notes

Full WCAG validation requires manual testing with assistive technologies and accessibility expert review. This story covers the practical subset for the MVP critical path.
