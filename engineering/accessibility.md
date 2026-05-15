# Accessibility (Mobile)

**Status:** Active
**Owner:** Mobile Lead + Designer
**Last updated:** 2026-05-13

This is the single source of truth for what every UI story must satisfy. Stories cite this file in their AC instead of restating rules.

## Targets

| ID | Target | Source |
|---|---|---|
| A11Y-01 | WCAG 2.1 **AA** for all in-scope flows; full validation requires manual testing with assistive tech and expert review at S10. | NFR-060 |
| A11Y-02 | Layouts must not break at **200 % text scale**. | NFR-061 |
| A11Y-03 | Color contrast ≥ **4.5:1** body, ≥ **3:1** large text. | NFR-062 |
| A11Y-04 | EN + VI from S2 onward (`engineering/i18n.md`). | NFR-063 |
| A11Y-05 | Sync state badges always visible on capture screens (offline UX clarity). | NFR-064 |
| A11Y-06 | Tap targets ≥ **44 × 44 pt**. | NFR-065 |
| A11Y-07 | Every interactive widget has a non-empty `Semantics.label` in the active locale. | this doc |
| A11Y-08 | No information conveyed by color alone (icon + text or label). | this doc |
| A11Y-09 | Focus order matches visual order; focus visible on every focusable element. | this doc |
| A11Y-10 | Form errors are announced via live region (TalkBack / VoiceOver). | this doc |

## Implementation rules

### Semantics

- Use Flutter `Semantics` widget for non-text widgets and custom controls.
- Replace icon-only buttons with `IconButton(tooltip: ..., onPressed: ...)` so the tooltip becomes the screen-reader label.
- Wrap groups (e.g. PPE checklist) with `MergeSemantics` only when grouping increases clarity.
- For status chips (PENDING / SYNCING / CONFIRMED / FAILED / DEAD_LETTER) the `Semantics.label` reads the localized status, not the color.

### Color and contrast

- Use only color tokens from `resource/theme/colors.dart`. No inline colors.
- Audit dark and light themes against AA at PR time using `flutter_contrast` or equivalent fixture.
- Status colors carry an icon companion (`✓ CONFIRMED`, `↻ SYNCING`, `! FAILED`) — never color alone.

### Text and scale

- Always use `MediaQuery.textScaler` (do not clamp it). Layouts must adapt.
- Critical CTA buttons must remain reachable when text doubles.
- Long labels truncate with ellipsis only when the full text is in a tooltip / Semantics label.

### Tap targets

- Minimum 44 × 44 pt for any interactive widget. `Material` ripple area must extend to the same size.
- Adjacent buttons separated by at least 8 pt.

### Focus

- All routes use `FocusTraversalGroup` with `OrderedTraversalPolicy` if the natural order is wrong.
- Modal dialogs trap focus and restore it on dismiss.

### Live regions and announcements

- Use `SemanticsService.announce` for transient events: capture submitted, sync state change, OTP delivered.
- Long lists (My Shifts) announce the count after refresh.

### Forms

- Every input has a visible label and a `helperText` for non-obvious format.
- Errors render below the field, are announced, and never rely on color alone.

### Localization

- All user-visible strings live in ARB files under `mobile/lib/l10n/`.
- No hard-coded strings in widgets. Lint rule enforces.

## Assistive tech matrix

| Platform | Tested with |
|---|---|
| iOS | VoiceOver |
| Android | TalkBack |
| iOS | Dynamic Type |
| Android | Font Size scaling |
| Both | Switch Control / Switch Access (smoke only at S10) |

## Test cases

| TC | Coverage |
|---|---|
| TC-A11Y-001 | Semantics labels present on key screens (capture, login, sync center). |
| TC-A11Y-002 | 200 % text scale renders without overflow on smallest device. |
| TC-A11Y-003 | Color contrast meets AA on light + dark theme. |
| TC-A11Y-004 | Status chips convey state without color alone. |

## Per-story checklist (paste into UI stories)

- [ ] All interactive widgets carry a non-empty `Semantics.label`.
- [ ] Tap targets ≥ 44 × 44 pt.
- [ ] AA contrast verified in light + dark.
- [ ] No hard-coded strings; ARB updated for EN + VI.
- [ ] No information conveyed by color alone.
- [ ] Manual TalkBack + VoiceOver pass on key paths.

## Out of scope (post-MVP)

- Sign language video alternatives.
- Custom assistive gestures beyond OS defaults.
- Voice-control navigation beyond what the OS provides.

## Review cadence

- Per-PR by Designer + Mobile Lead.
- Sprint demo includes one A11Y walk-through.
- Pre-pilot manual review (TC-A11Y-001..004 + assistive tech smoke) gates `qa/pilot-gate.md`.
