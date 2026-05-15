---
epic: EPIC-01
title: Platform Foundation
sprint: S1
priority: P0
status: Ready
owner: Engineering Lead
---

# EPIC-01 — Platform Foundation

## 1. Goal

Land a Flutter scaffold that any new engineer can run in under 30 minutes and that ships every cross-cutting concern (CI, secure storage, SQLite skeleton, observability, theming) before any feature work begins.

## 2. In scope

- Flutter project init (iOS + Android) at the repo root with the layer-first folder layout from `architecture/tech-stack.md`.
- Riverpod, Dio, Drift (SQLite), `flutter_secure_storage`, `logger`, `sentry_flutter`, `intl`.
- App router (`go_router`) with auth guard scaffolding.
- Theme tokens (`resource/theme/`) + light/dark.
- CI pipeline: lint → unit test → integration smoke build → artifact.
- Pre-commit hooks: `dart format`, `flutter analyze`, secret scan.
- Secure storage wrapper with platform-specific options.
- SQLite skeleton (Drift) with migrations harness; tables added in S4.
- Sentry initialisation gated by build flavour.
- `.env` strategy via build flavors (dev / staging / prod).

## 3. Out of scope

- Any auth screen logic (S2).
- Any business tables in SQLite (S4).
- Sync engine (S6/S7).

## 4. Functional requirements covered

Cross-cutting; underpins **all** FRs but does not satisfy any user-visible FR alone.

## 5. Stories

| ID | Title | Pri | Estimate | Status | Owner |
|---|---|---|---|---|---|
| US-PLAT-001 | Bootstrap Flutter project + folder layout | P0 | M | Ready | Mobile Lead |
| US-PLAT-002 | Wire Riverpod + go_router + theme tokens | P0 | M | Ready | Mobile |
| US-PLAT-003 | Add secure storage wrapper (iOS + Android) | P0 | S | Ready | Mobile |
| US-PLAT-004 | Add Drift skeleton + migration harness | P0 | M | Ready | Mobile Lead |
| US-PLAT-005 | CI pipeline (analyze, test, build, scan) | P0 | M | Ready | DevOps |
| US-PLAT-006 | Sentry init + redaction filter scaffolding | P0 | S | Ready | Mobile |

## 6. Dependencies

- DEC-009 (orchestration target) for CI runner choice.
- DEC-011 (Drift vs sqflite) — defaulted to Drift in this epic.

## 7. Risks

- RISK-010 CI runner provisioning slips → blocks merges.
- RISK-011 iOS code signing setup not coordinated with sponsor → delays IPA build.

## 8. Definition of Done (epic-level)

- `flutter run` succeeds on a fresh checkout for both iOS and Android.
- CI green on `main` for at least 3 consecutive PRs.
- Test coverage gate set (≥ 70% on changed files).
- Secret scanner active in CI and in pre-commit (TC-SEC-001 passes).
- Theme tokens consumed by at least one demo screen.

## 9. Open questions

DEC-011 closes inside this epic.
