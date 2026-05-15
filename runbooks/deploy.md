# Runbook: Standard Deploy

**Status:** Active  
**Owner:** DevOps

## Pre-flight

- [ ] CI green on main for the target SHA.
- [ ] CHANGELOG bumped.
- [ ] Migrations reviewed (if any).
- [ ] Sponsor / EL approval (for pilot or prod).
- [ ] Communication posted in `#field-app-dev` 30 min before.
- [ ] Freeze window respected (see `governance/change-management.md`).

## Backend deploy

```
1. Tag release: git tag v1.2.3 && git push --tags
2. CI pipeline triggers:
   - Build container images (api + worker).
   - Push to registry.
   - Run smoke tests on staging.
3. Manual approval gate in CI for pilot/prod.
4. Apply Postgres migrations:
   - Alembic upgrade head, dry-run first.
   - Then commit.
5. Rolling deploy:
   - kubectl set image deploy/fastapi  fastapi=registry/...:v1.2.3
   - kubectl set image deploy/celery   celery=registry/...:v1.2.3
   - Watch rollout: kubectl rollout status ...
6. Run smoke: scripts/smoke.sh <env>
7. Monitor for 30 min: API RED, queue depth, sync success rate.
```

## Mobile deploy

```
1. Tag mobile release: git tag mobile-v1.2.3
2. CI builds signed APK + IPA, uploads to Play Internal + TestFlight.
3. Internal testers (team) verify happy path.
4. Promote to closed track (pilot users):
   - Play: Internal → Closed.
   - App Store: TestFlight external testers.
5. Phased rollout to production:
   - Day 1: 5 %.
   - Day 2: 25 %.
   - Day 3: 50 %.
   - Day 5: 100 %.
6. Monitor crash-free sessions, sync success rate per cohort.
```

## Force update

If a release fixes a critical bug:

```
1. Bump backend min_app_version to the new version.
2. /v1/auth/exchange returns header X-Min-App-Version: x.y.z.
3. Mobile shows blocking update screen with store deep-link.
```

## Verify

- `/healthz` 200.
- Smoke tests pass.
- Metrics steady for 30 min.
- No new SEV-1 / SEV-2 alerts.

## Rollback

If verify fails or new alerts fire, follow `rollback.md` immediately.

## After

- Tag release notes in GitHub Releases.
- Post in `#field-app-stakeholders` summary.
- Update CHANGELOG version section if "Unreleased" → version.
