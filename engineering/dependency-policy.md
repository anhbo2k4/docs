# Dependency policy

**Status:** Active
**Owner:** Engineering Lead

## Principle

Every dependency is a long-term liability. Add slowly, prune actively, audit regularly.

## When you may add a dependency

- The capability is non-trivial to build (cryptography, parsing, native bridges).
- A widely-adopted, actively-maintained option exists.
- The licence is compatible (see below).
- Removal in 12 months would still be feasible.

If any of the above is no, write the code.

## Forbidden additions without ADR

- New state-management library (Riverpod is the chosen one — ADR-006).
- New HTTP client beyond `dio` (mobile) / `httpx` (backend).
- New database / ORM beyond Drift (mobile, defaulting per DEC-011) / SQLAlchemy (backend).
- New crypto primitives. Use platform Keychain/Keystore + standard libraries.
- A package with overlapping responsibility to one already in use.

## Vetting checklist (per new dependency)

- [ ] Licence is one of: MIT, BSD-2/3, Apache-2.0, MPL-2.0. Strong copyleft (GPL/AGPL) requires legal review.
- [ ] Last commit within the last 6 months OR a clear maintenance statement.
- [ ] At least one of: > 100 GitHub stars, listed on `pub.dev`/`pypi.org` with high pub points, used by another reputable project we can name.
- [ ] No CVEs in the last 12 months without patch.
- [ ] No known typosquatting variants — verify the exact spelling and publisher.
- [ ] Bundle size impact < 100 KB on mobile, otherwise justify.
- [ ] Native dependencies disclosed (Android NDK / iOS frameworks).
- [ ] Reviewed for telemetry / phone-home behaviour. None permitted by default.

## Pinning

- Mobile (`pubspec.yaml`): exact version (`==`) for direct dependencies during MVP. Lockfile committed.
- Backend (`pyproject.toml`): caret ranges (`^x.y.z`) on direct deps, exact for security-critical (`pyjwt`, `cryptography`). Lockfile (`poetry.lock`) committed.
- TS edge: `package-lock.json` committed; `npm ci` in CI.

## Updates

- Renovate bot opens weekly PRs grouped by ecosystem.
- Patch / minor updates: auto-merged after green CI for non-security-critical packages.
- Major updates: require an ADR or a story.
- Security advisories: page on-call if HIGH/CRITICAL on a production dependency; fix or mitigate within 7 days.

## Forking

- Allowed only when upstream is dead and the function is critical.
- Mark forked deps in `engineering/forks.md` (created when first fork lands) with: reason, upstream URL, last sync, exit plan.

## Removal

- Quarterly audit (`make deps-audit`):
  - List packages used in zero locations.
  - List packages superseded by stdlib / framework features.
  - Remove with a `chore/dep-prune-YYYY-Q?` PR.

## Supply-chain integrity

- All CI-built artifacts are signed.
- SBOM generated per release (`syft` for backend, `cyclonedx` for mobile).
- Verify checksums on tooling installs in CI.
- Pre-commit `gitleaks` blocks accidental secret commits.

## Vendor lock-in

When the dependency is a hosted vendor (Sentry, Twilio, Supabase):

- Wrap in an internal interface so swapping is a single-PR change.
- Document the swap path in the relevant ADR.
- Avoid using vendor-only features that have no standard equivalent.

## Anti-patterns to flag in review

- Two libraries doing the same job.
- A direct dependency only used in tests (move to dev-deps).
- A heavy dep used for one helper function — vendor the function instead.
- A dep with last commit > 18 months and no patches for known CVEs.
