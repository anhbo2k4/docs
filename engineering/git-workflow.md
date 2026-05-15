# Git workflow

**Status:** Active
**Owner:** Engineering Lead

## Branching model

Trunk-based with short-lived feature branches.

- `main` is always deployable to staging.
- Release tags (`vX.Y.Z`) cut releases to production.
- No long-lived `develop` branch.

## Branch names

```
feat/US-AUTH-001-otp-request
fix/SEV2-sync-race
chore/upgrade-flutter-3.22
docs/blueprint-v1.2
```

`<type>/<scope>-<slug>`. `<type>` ∈ `feat`, `fix`, `chore`, `docs`, `test`, `refactor`, `perf`, `build`, `ci`. Scope is the story ID or severity tag.

## Commit messages

Conventional Commits enforced via commitlint:

```
feat(auth): add OTP request screen (US-AUTH-001)

Implements phone entry, format validation, and rate-limit error UI.

Refs: US-AUTH-001
```

Body wraps at 72 cols. Footer references stories or issues.

## PR flow

1. Open PR against `main` from your feature branch.
2. PR title: `[US-AUTH-001] OTP request screen with rate-limit messaging`.
3. PR body: see `code-review-checklist.md` for the required template.
4. CI must be green. Required checks:
   - lint, format, unit, widget, backend tests, integration smoke, security scan, build artifacts.
5. At least 1 approval from the area owner per `governance/raci.md`.
6. Squash merge with the PR title as the commit subject.
7. Delete the branch on merge.

## Review etiquette

- Reviewers respond within 1 business day.
- Comments are specific and actionable. "This is wrong" without why is not acceptable.
- Authors mark threads `Resolved` only when the reviewer's question is answered.
- Disagreements escalate to the area lead, not Slack pile-ons.

## Hotfix

- Branch from the latest production tag: `hotfix/SEV1-otp-bypass`.
- Land on `main`, then cherry-pick to the release branch.
- Tag a patch release (`vX.Y.Z+1`).
- Fill out a postmortem (`runbooks/incident-response.md` template) within 5 business days.

## Forbidden actions

- Force-push to `main` or any release branch.
- `git rebase` of pushed branches owned by others.
- Bypassing CI required checks. Admins use the `Override` action only with logged justification.
- Committing to `main` directly.

## Tags

- `v1.0.0` → first pilot build.
- `v1.0.0-rc.N` → release candidates.
- Annotated tags only.

## Keeping branches fresh

- Rebase on `main` daily during active development.
- Resolve conflicts locally; never force-push during a review.
- If a branch falls behind by more than a week, recreate it.
