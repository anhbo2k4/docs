---
sprint: S1
title: Project Foundation
duration: 2 weeks
priority: P0
status: Active
owner: Mobile Lead + DevOps
unblocked: 2026-05-15
---

# Sprint 1 — Project Foundation

> **Status:** Active. Phase 0 closed in Blueprint v2.0.0; all preconditions met. CI runner target = single-VM Docker on AWS EC2 `ap-southeast-1` per provisional DEC-009.

## 1. Sprint goal

Land a runnable Flutter scaffold with CI, secure storage, Drift skeleton, and observability so any feature work in S2+ has zero infrastructure friction.

## 2. Theme

Cross-cutting plumbing only. No business UI. The exit signal is "any new engineer can clone, run, and ship a small change in under 30 minutes."

## 3. Scope (in)

- Flutter project + folder layout.
- Riverpod, go_router, theme tokens.
- Secure storage wrapper.
- Drift skeleton + migration harness (no business tables yet).
- CI: analyze, test, build, secret scan.
- Pre-commit hooks (`lefthook`).
- Sentry init + redaction filter.

## 4. Out of scope

- Authentication screens (S2).
- Business tables in SQLite (S4).
- Any sync logic (S6/S7).

## 5. Pre-conditions — all met

- ✅ S0 closed (Blueprint v2.0.0, 2026-05-15). DEC-009 provisional answer = single-VM Docker Compose on AWS EC2 `ap-southeast-1`.
- iOS code-signing identity available.
- Android keystore (debug at minimum) available.

## 6. Stories committed

| ID | Title | Owner | Estimate |
|---|---|---|---|
| [US-PLAT-001](../../epics/EPIC-01-platform-foundation/stories/US-PLAT-001-bootstrap.md) | Bootstrap Flutter project | Mobile Lead | M |
| [US-PLAT-002](../../epics/EPIC-01-platform-foundation/stories/US-PLAT-002-riverpod-router-theme.md) | Riverpod + go_router + theme | Mobile | M |
| [US-PLAT-003](../../epics/EPIC-01-platform-foundation/stories/US-PLAT-003-secure-storage.md) | Secure storage wrapper | Mobile | S |
| [US-PLAT-004](../../epics/EPIC-01-platform-foundation/stories/US-PLAT-004-drift-skeleton.md) | Drift skeleton + migration harness | Mobile Lead | M |
| [US-PLAT-005](../../epics/EPIC-01-platform-foundation/stories/US-PLAT-005-ci.md) | CI pipeline | DevOps | M |
| [US-PLAT-006](../../epics/EPIC-01-platform-foundation/stories/US-PLAT-006-sentry.md) | Sentry init + redaction | Mobile | S |

## 7. Risks for this sprint

- RISK-010 CI runner provisioning slips.
- RISK-011 iOS code signing not coordinated.
- DEC-011 closed via ADR-009 (Drift adopted) — re-confirmable at S1 review.

## 8. Sprint-level Definition of Done

- `flutter run` works on iOS and Android from a fresh checkout.
- CI green on `main` ≥ 3 consecutive PRs.
- Secret scanner active, demo PR triggers a block.
- Coverage gate set (≥ 70% on changed files).
- Sentry sees a test crash without PII.
- DEC-011 confirmation captured in S1 retro notes.

## 9. Demo script

- Clone repo, run `flutter run`.
- Show CI green pipeline.
- Trigger test crash → Sentry event without phone/PII.

## 10. Retro inputs

- Were tooling decisions thrashed?
- How long is fresh-clone-to-run actually taking?
- Any pain points in the layer-first folder layout that need adjustment?
