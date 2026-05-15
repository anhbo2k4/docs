# CI / CD Pipeline

**Status:** Active
**Owner:** DevOps + Engineering Lead
**Last updated:** 2026-05-13

This document is the authoritative spec for what runs on every pull request, every merge to `main`, and every release tag. It complements `engineering/release-process.md` (which covers the human-driven release flow) and `engineering/git-workflow.md` (branching).

> Pipeline platform: **GitHub Actions**. Workflow files live in `.github/workflows/`.

---

## 1. Pipelines at a glance

| Trigger | Workflow file | Purpose |
|---|---|---|
| `pull_request` to `main` | `pr-mobile.yml`, `pr-backend.yml`, `pr-edge.yml`, `pr-docs.yml` | Lint + unit + integration on changed area only |
| `push` to `main` | `main-build.yml` | Full build + integration tests + artefact upload |
| `push` of tag `mobile-v*` | `release-mobile.yml` | Signed iOS + Android build, store upload, release notes |
| `push` of tag `backend-v*` | `release-backend.yml` | Container build, push, deploy to staging |
| `push` of tag `blueprint-v*` | `release-docs.yml` | Tag and archive `field_work_delivery/` blueprint snapshot |
| `schedule` nightly 02:00 UTC | `nightly-soak.yml` | Long-soak sync + load tests (LP-001..008 from `qa/load-plan.md`) |
| `workflow_dispatch` | `manual-rollback.yml` | One-click rollback per `runbooks/rollback.md` |

Each workflow MUST set `concurrency: { group: ${{ github.workflow }}-${{ github.ref }}, cancel-in-progress: true }` so superseded runs cancel.

---

## 2. PR pipeline — required checks

A PR cannot merge until **all** required checks pass. Required checks are configured in branch protection on `main`.

### 2.1 `pr-mobile.yml`

Triggered when files under `mobile/**` change.

```
jobs:
  format       → dart format --set-exit-if-changed lib/ test/
  analyze      → flutter analyze --fatal-infos
  build_runner → dart run build_runner build --delete-conflicting-outputs
                 then `git diff --exit-code` to fail if generated code stale
  test_unit    → flutter test --coverage
                 enforce coverage ≥ 70% overall, ≥ 90% on @core/sync, @core/auth, @core/db
  test_golden  → flutter test test/golden/  (UI snapshot tests)
  android_apk  → flutter build apk --debug --flavor pilot
  ios_build    → flutter build ios --debug --no-codesign --flavor pilot
                 (runs on macos-latest)
  size_check   → APK and IPA must remain ≤ 80 MB (PB-M-006); fails if exceeded
  a11y_audit   → flutter test test/accessibility/  (TC-A11Y-001..004)
```

### 2.2 `pr-backend.yml`

Triggered when files under `backend/**` change.

```
jobs:
  format     → ruff format --check .
  lint       → ruff check . && mypy app/
  test_unit  → pytest -m "not integration" --cov=app --cov-fail-under=80
  test_int   → pytest -m integration  (testcontainers-postgres + redis)
  contract   → schemathesis run api-contracts/openapi.yaml --base-url http://app
               (validates FastAPI emits the same shape declared in openapi.yaml)
  docker     → docker buildx build --target prod --load -t backend:pr-${SHA}
               docker scout cves backend:pr-${SHA} → fail on CRITICAL or HIGH unfixed
```

### 2.3 `pr-edge.yml`

Triggered when files under `edge/**` change.

```
jobs:
  deno_fmt   → deno fmt --check
  deno_lint  → deno lint
  deno_test  → deno test --allow-env
  edge_dryrun → supabase functions deploy --dry-run
```

### 2.4 `pr-docs.yml`

Triggered when files under `field_work_delivery/**`, `*.md`, `LICENSE`, `SECURITY.md` change.

```
jobs:
  link_check  → lychee --no-progress field_work_delivery/**/*.md
  spelling    → cspell "field_work_delivery/**/*.md"
  mermaid     → mmdc -i <each .md with mermaid> -o /tmp/render/  (renders fail if syntax broken)
  consistency → scripts/check-fr-coverage.py  (every FR-XXX has ≥1 epic + ≥1 TC)
                scripts/check-changelog.py    (CHANGELOG bumped if any field_work_delivery/** changed)
```

### 2.5 Common to all PR workflows

- Cache: pub-cache (mobile), pip-cache + .venv (backend), deno-cache (edge), node_modules (docs).
- Timeout: 30 minutes per job; fail fast.
- Annotations: lint findings appear inline on the PR diff.

---

## 3. Main-branch pipeline

`main-build.yml` runs on every merge to `main`:

```
jobs:
  full_test   → all PR checks repeated (no skip)
  e2e_mobile  → patrol_cli on Android emulator (TC-AUTH-*, TC-SHIFT-*, TC-OFF-*)
  e2e_sync    → mobile + backend + odoo-mock end-to-end (TC-SYNC-001..010)
  upload_apk  → upload signed-debug APK as build artefact (retention 30 days)
  upload_ipa  → upload unsigned IPA simulator build (retention 30 days)
  upload_oas  → diff backend-emitted OpenAPI vs api-contracts/openapi.yaml; warn on drift
  notify      → post Slack channel #field-builds with build status + size deltas
```

Failures on `main` page the on-call (see `engineering/oncall.md`).

---

## 4. Release pipelines

### 4.1 Mobile release — `release-mobile.yml`

Trigger: tag matches `mobile-v[0-9]+.[0-9]+.[0-9]+(-rc[0-9]+)?`.

Steps:

1. Verify tag is on `main` and signed.
2. Decrypt signing keys from secrets (Apple App Store Connect API key, Android keystore base64).
3. `flutter build appbundle --release --flavor prod --build-name=$VERSION --build-number=$BUILD_NUM`.
4. `flutter build ipa --release --flavor prod --export-options-plist=ios/exportOptions.plist`.
5. Upload `.aab` to Play Console internal testing track.
6. Upload `.ipa` to TestFlight.
7. Generate release notes from `backlog/releases/$VERSION.md` using the template.
8. Post to #field-releases with the artefact links.
9. Update `runbooks/rollback.md` last-good entry.

See `runbooks/mobile-release.md` for the human checklist.

### 4.2 Backend release — `release-backend.yml`

Trigger: tag matches `backend-v[0-9]+.[0-9]+.[0-9]+`.

Steps:

1. Build multi-arch Docker image, push to registry with tags `:$VERSION` and `:sha-$SHA`.
2. Run `alembic upgrade --sql head > migration.sql` and attach as artefact.
3. Deploy to staging via Argo CD / kubectl set image (DEC-009).
4. Run smoke tests against staging.
5. On success, gate manual approval for production deploy.
6. Production deploy: blue/green per `runbooks/deploy.md`.
7. Post to #field-releases.

### 4.3 Blueprint release — `release-docs.yml`

Trigger: tag matches `blueprint-v[0-9]+.[0-9]+.[0-9]+`.

Steps:

1. Verify `CHANGELOG.md` has an entry matching the tag.
2. Verify `README.md` `Version:` field matches the tag.
3. Render every Mermaid diagram to `docs/img/`.
4. Bundle a zip of `field_work_delivery/` and attach to the GitHub release.
5. Post to #field-program with the changelog excerpt.

---

## 5. Nightly soak — `nightly-soak.yml`

Runs LP-001..008 from `qa/load-plan.md`:

| Time slot (UTC) | Scenario | Duration |
|---|---|---|
| 02:00 | LP-001 sync steady-state | 30 min |
| 02:30 | LP-002 burst 100→1000 envelopes | 15 min |
| 02:45 | LP-003 idempotency storm | 15 min |
| 03:00 | LP-004 auth surge | 10 min |
| 03:10 | LP-005 media presigned-URL throughput | 20 min |
| 03:30 | LP-006 backpressure / circuit-breaker | 15 min |
| 03:45 | LP-007 refresh storm | 15 min |
| 04:00 | LP-008 mobile e2e harness | 60 min |

Failures open an issue with `nfr-debt` label and ping the engineering lead.

---

## 6. Caching, secrets, and artefact retention

| Concern | Rule |
|---|---|
| Build cache | `actions/cache` keyed on lockfile hashes. Bust on `main` weekly. |
| Secrets | Stored in GitHub Encrypted Secrets and AWS Secrets Manager (DEC-007 informs which). Never echo. |
| Artefact retention | PR builds 7 days, `main` 30 days, releases 1 year. |
| Build minutes | Reserved for `main` and release runs; PR runs spill to standard runner pool. |

---

## 7. Required status checks (branch protection)

`main` requires:

- `pr-mobile / format`, `pr-mobile / analyze`, `pr-mobile / test_unit`, `pr-mobile / size_check`
- `pr-backend / lint`, `pr-backend / test_unit`, `pr-backend / contract`
- `pr-edge / deno_test`
- `pr-docs / link_check`, `pr-docs / consistency`
- 1 reviewer (CODEOWNERS-required for files under owned paths; see `engineering/code-owners.md`)
- Linear history (no merge commits — squash only)
- Force pushes disabled
- `main-build / e2e_sync` reported as expected check (warn-only until S5; required from S7)

---

## 8. Pipeline ownership

| Pipeline | Primary owner | Backup |
|---|---|---|
| `pr-mobile.yml` | Mobile Lead | DevOps |
| `pr-backend.yml` | Backend Lead | DevOps |
| `pr-edge.yml` | Backend Lead | Security Champion |
| `pr-docs.yml` | Engineering Lead | PM |
| `release-*.yml` | DevOps | Engineering Lead |
| `nightly-soak.yml` | QA Lead | Backend Lead |

A failing nightly is a P2; a failing `main-build` is a P1; a failing release is a P0 → page on-call.

---

## 9. Verification cadence

- Per PR: full PR pipeline must pass.
- Per sprint review: dashboard `CI Health` must show < 5 % flaky rate; flakiness counts as NFR debt.
- Per release: release pipeline must complete green within 30 min target (PB-D-002).

## 10. Cross-references

- Branching: `engineering/git-workflow.md`
- Release process: `engineering/release-process.md`
- Mobile store release: `runbooks/mobile-release.md`
- Backend deploy: `runbooks/deploy.md`
- Rollback: `runbooks/rollback.md`
- Observability: `engineering/observability.md`
- On-call: `engineering/oncall.md`
- Load tests: `qa/load-plan.md`
