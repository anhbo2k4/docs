# Engineering — developer enablement

**Status:** Active
**Owner:** Engineering Lead

This folder is the developer-onboarding entry point. Anyone (human or agent) joining a build sprint should read these files in order.

## Contents

| File | Purpose |
|---|---|
| `dev-environment.md` | Tools, versions, and local setup for mobile + backend. |
| `coding-standards.md` | Lint, format, naming, layering, error-handling conventions. |
| `git-workflow.md` | Branching, commits, PR flow, review etiquette. |
| `code-review-checklist.md` | What reviewers must check before approving. |
| `feature-flags.md` | When and how to gate code behind flags. |
| `release-process.md` | Versioning, release notes, store submission, rollback. |
| `observability.md` | Logging, metrics, traces, dashboards. |
| `oncall.md` | Rotation, escalation, runbook entry rules. |
| `dependency-policy.md` | Vetting, pinning, supply-chain integrity. |
| `i18n.md` | Locale strategy (EN + VI), ARB workflow, pseudolocalisation. |
| `accessibility.md` | A11Y targets and per-story checklist (WCAG 2.1 AA). |
| `performance-budget.md` | Consolidated mobile + backend perf budgets. |
| `code-owners.md` | CODEOWNERS table, PR template, issue templates, review SLAs. |

## Why this exists

The blueprint in `epics/` and `sprints/` tells engineers **what** to build. This folder tells them **how to operate** while building.

## How agents should use it

1. Read `dev-environment.md` and confirm the local setup is valid before opening a story.
2. Apply `coding-standards.md` to every change — these override personal preference.
3. Follow `git-workflow.md` for branch naming and PR titling.
4. When introducing observability, `observability.md` is the authority on naming + cardinality.
