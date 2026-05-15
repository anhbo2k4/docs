# Release process

**Status:** Active
**Owner:** Engineering Lead + Mobile Lead

## Cadence

- **Backend:** continuous deploy to staging; gated promotion to production once per sprint or on demand.
- **Mobile:** release candidate at end of each sprint; pilot deploy after S10 gate (`qa/pilot-gate.md`).

## Versioning

- Backend: `v<major>.<minor>.<patch>`. Major bumps for breaking API. Minor for additive features. Patch for fixes.
- Mobile: same scheme with build number suffix `+<N>` (`1.0.0+42`).
- Both follow Semantic Versioning where the API contract is the public surface.

## Release branches

- Backend: cut `release/v1.0` from `main` for the pilot; further fixes cherry-picked.
- Mobile: same; aligned with backend minor when the contract changes.

## Release notes

- Auto-generated draft from Conventional Commits via `release-please` or equivalent.
- Engineering Lead curates: visible changes for PO, internal changes folded.
- File: `releases/<version>.md`. Linked from CHANGELOG and store metadata.

## Mobile store submission

1. Build signed artifacts via `make build-mobile-apk` and `make build-mobile-ipa`.
2. Run pre-submission checklist:
   - App size ≤ NFR limits (TC-PERF-010, TC-PERF-011).
   - Crash-free sessions ≥ 99.5 % over last 24 h staging.
   - Privacy manifest accurate (iOS).
   - Required runtime permissions listed in store metadata.
3. Submit to TestFlight (iOS) / Play Internal Testing (Android).
4. Smoke test on device matrix subset.
5. Promote to closed pilot track only after `qa/pilot-gate.md` is signed.

## Backend deploy

1. CI builds image; signs and pushes to registry.
2. Staging auto-deploys via GitOps (Argo CD or equivalent target per DEC-009).
3. Smoke tests run against staging.
4. Production promotion requires:
   - Engineering Lead approval.
   - On-call acknowledged.
   - Rollback plan reviewed.
5. Deploy uses rolling update; canary 10 % for 30 min; then full.

## Rollback

- See `runbooks/rollback.md` for backend.
- Mobile rollback is NOT instant; rely on:
  - Server-side flags (kill-switch).
  - `min_app_version` returned from `/v1/auth/exchange` to force update.
- Plan rollbacks at design time; never assume client-side recovery.

## Post-release

- Monitor for 24 h: error rate, crash-free sessions, sync success rate, p95 latencies.
- File post-release notes in `runbooks/incident-response.md` if issues arose.
- Update CHANGELOG with the released version's date.

## Pilot launch criteria

See `qa/pilot-gate.md`. Pilot launch is a release with extra ceremony:

- Comms plan executed (`governance/communication-plan.md`).
- On-call rotation staffed.
- Dashboards visible to Sponsor.
- Rollback rehearsed within last 7 days.
