---
id: US-POST-003
epic: EPIC-10
sprint: post-MVP
priority: P2
estimate: L
status: Draft
owner: TBD
---

# US-POST-003 — i18n + first non-English locale

## User story

**As** a Vietnamese field worker
**I want** the entire app in Vietnamese
**so that** I can work without translating English in my head.

## Acceptance criteria

1. App ships EN + VN locales selectable in Settings; persists across launches.
2. All UI strings sourced from ARB files; no hard-coded strings remain (lint enforces).
3. Date, time, number, currency formatting follows locale.
4. Pseudo-locale runs in CI to detect missing strings (per `engineering/i18n.md`).
5. Error codes from backend map to localised messages on device.
6. RTL preview validated (no RTL locale shipping yet, but layout passes).

## Tasks

- [ ] ARB files: en.arb + vi.arb.
- [ ] Locale-aware formatters in shared util.
- [ ] CI lint rule: forbid `Text('...')` outside of allowlisted files.

## Dependencies

- `engineering/i18n.md`.

## Risks

- Translation quality for technical terms.
