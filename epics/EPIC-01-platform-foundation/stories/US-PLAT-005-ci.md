---
id: US-PLAT-005
epic: EPIC-01
sprint: S1
fr: []
priority: P0
estimate: M
status: Ready
owner: DevOps
---

# US-PLAT-005 — CI pipeline (analyze, test, build, scan)

## Acceptance criteria

1. CI pipeline (chosen platform per DEC-009) runs on every PR with stages:
   - `setup` (cache pub deps + iOS pods)
   - `analyze` (`flutter analyze`)
   - `test` (`flutter test --coverage`, fail on coverage < 70% on changed files)
   - `build` (Android APK debug, iOS unsigned)
   - `scan` (secret scan + dependency scan)
2. Status checks required for merge to `main`.
3. Pipeline runtime ≤ 12 min for hot caches, ≤ 25 min cold.
4. Artifacts (APK, IPA) uploaded for last 14 days.
5. Pre-commit hooks via `lefthook`: `dart format`, `flutter analyze --fatal-infos`, secret scan.

## Tasks

- [ ] Author pipeline YAML.
- [ ] Configure caches (pub, Pods, Gradle).
- [ ] Add `gitleaks` or equivalent secret scanner (TC-SEC-001).
- [ ] Configure required checks branch protection.
- [ ] Add `lefthook` and document setup.

## Dependencies

DEC-009 closed.

## FR mapping

Cross-cutting.

## Test cases

TC-SEC-001.

## DoD

- CI green on `main` for ≥ 3 consecutive PRs.
- Required checks enforced.
