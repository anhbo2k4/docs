# Changelog

All notable changes to this delivery blueprint are recorded here. Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) and dates use ISO-8601.

## [2.1.0] â€” 2026-05-15

### Added
- `adr/ADR-015-no-custom-module-rpc-adapter.md` â€” supersedes ADR-014. No custom Odoo module; FastAPI RPC adapter pattern with capability detection at runtime; works on Odoo Online + Enterprise + Community.
- `adr/ADR-016-gps-in-fastapi-postgres.md` â€” supersedes ADR-013. GPS history lives in FastAPI Postgres (`gps_pings` table) with 180-day retention; Odoo gets only `x_ngynapp_gps_*` summary fields.
- `data/odoo-x-fields-spec.md` â€” authoritative spec for the manually-created `x_ngynapp_*` field set on `project.task` / `fsm.task`, `hr.employee`, `ir.attachment`, `hr.attendance`, `survey.user_input`. Studio + Developer-mode setup steps included; FastAPI startup probe contract (JSON `ProbeReport`) defined.

### Changed
- `adr/ADR-013-custom-gps-model.md` â€” status flipped to **Superseded by ADR-016**.
- `adr/ADR-014-field-mobile-sync-module.md` â€” status flipped to **Superseded by ADR-015**.
- `adr/ADR-011-odoo-19-enterprise.md` â€” amended. Odoo 19 EE is now the *test target*; production contract is RPC-only across versions and editions.
- `adr/ADR-012-aws-s3-object-storage.md` â€” rewritten. Storage backend changed from AWS S3 (`ap-southeast-1`) to **DigitalOcean Spaces (`sgp1`)**; S3-compatible API kept for portability.
- `engineering/odoo-module-mapping.md` â€” full rewrite as the **RPC adapter capability spec**. Path A / Path B model removed; capability matrix replaces edition-specific paths.
- `architecture/deployment.md` â€” full rewrite. DigitalOcean topology: Droplet + managed Postgres + managed Redis + Spaces. Cost projection ~$59/mo testing/pilot; ~$148â€“168/mo prod year-1. DOKS migration deferred to capacity-plan threshold.
- `traceability/decision-log.md` â€” Phase 0 fully closed. Four `Provisional` items flipped to `Decided`:
  - DEC-002 â€” Odoo modules: capability detection at runtime.
  - DEC-008 â€” SLA: customer Odoo provider owns Odoo SLA; we own integration-layer SLA.
  - DEC-009 â€” Orchestration: DO Droplet + Compose for testing/pilot, vertical scale-up for prod, DOKS deferred.
- `traceability/decision-log.md` â€” `Decided` items amended:
  - DEC-001 â€” Odoo target: 19 EE for testing; RPC contract for production portability.
  - DEC-003 â€” flipped: **no custom module**.
  - DEC-004 â€” flipped: GPS in FastAPI Postgres.
  - DEC-007 â€” amended: DigitalOcean Spaces.
- New stretch decisions logged: DEC-014 (GPS retention 180 days), DEC-015 (`x_ngynapp_*` field prefix reserved).
- `WORKING.md` â€” focus, last decisions, open blockers updated for v2.1.0.
- `README.md` â€” version bump to 2.1.0; quick links extended (`x_ngynapp_*` spec, RPC adapter spec, DO topology).
- `adr/README.md` â€” total 16 ADRs; ADR-013 + ADR-014 marked Superseded; ADR-015 + ADR-016 added as Accepted.

### Removed
- The `field-mobile-sync` Odoo custom-module repo plan and its CI pipeline `odoo-module-ci` are cancelled. Story US-ODOO-001 (module skeleton) is rewritten in-place to "RPC adapter scaffold + capability probe" during S1 kickoff.

### Notes
- Driver: sponsor reconfirmation on 2026-05-15 â€” the solution must run on Odoo Online; Odoo Online prohibits custom modules. The platform must work across Odoo versions and editions, with Odoo 19 EE remaining as the controlled test environment.
- Compute and storage rationalised onto DigitalOcean for testing and pilot. Single vendor, predictable cost, S3-compatible storage to keep migration optional later.
- Twilio account + API key confirmed available; OTP flow unchanged from ADR-002 + ADR-010.
- Phone resolution finalised: `work_phone` â†’ `private_phone` â†’ `mobile_phone` (DEC-006 closed).

## [2.0.0] â€” 2026-05-15

### Added
- 5 new ADRs closing the remaining Phase 0 decisions:
  - `adr/ADR-010-twilio-sms-only.md` â€” closes DEC-005 (SMS provider).
  - `adr/ADR-011-odoo-19-enterprise.md` â€” closes DEC-001 (Odoo version + edition).
  - `adr/ADR-012-aws-s3-object-storage.md` â€” closes DEC-007 (object storage).
  - `adr/ADR-013-custom-gps-model.md` â€” closes DEC-004 (GPS storage target).
  - `adr/ADR-014-field-mobile-sync-module.md` â€” closes DEC-003 (custom Odoo module scope).

### Changed
- `traceability/decision-log.md` â€” full rewrite. All 9 Phase 0 decisions now resolved: 5 Decided, 4 Provisional (DEC-002, DEC-006, DEC-008, DEC-009) under PM confirmation SLA.
- `WORKING.md` â€” focus shifted from Phase 0 close-out to Sprint 1 kickoff. Open blockers section cleared.
- `README.md` â€” version bump to 2.0.0; project at-a-glance reflects Odoo 19 EE + Drift; quick links re-grouped (planning / engineering / QA / architecture / repo); duplicates removed.
- `adr/README.md` â€” index extended with ADR-010..ADR-014; total 14 ADRs.
- `epics/EPIC-00-discovery/README.md` â€” status flipped to Done; story table reflects closing artefacts; risk register adjusted.
- `sprints/sprint-01-foundation/README.md` â€” status flipped to Active; preconditions met; DEC-011 reframed as confirmation rather than closing event.
- `governance/self-critique.md` â€” round 5 logged: decision-gap closed.

### Notes
- 4 Provisional defaults (DEC-002, DEC-006, DEC-008, DEC-009) ride along with PM confirmation SLA. They unblock S1 immediately; if a flip occurs, the owning doc and CHANGELOG are amended.
- ADR-010..014 are all `Accepted` (no Proposed gate); the program leadership treated the Phase 0 decisions as a single batch sign-off on 2026-05-15.

## [1.7.0] â€” 2026-05-13

### Added
- `engineering/odoo-module-mapping.md` â€” authoritative answer to "which Odoo modules do we need before building?". Two reference paths (Path A Community-only, Path B Enterprise FSM-aligned), full module decision matrix, field-level mapping per entity, custom module `field_mobile_sync` skeleton (manifest, models inventory, security groups, ops server actions), migration risk register, S0 workshop checklist for the Odoo Specialist.

### Changed
- `README.md` â€” version bump to 1.7.0; quick links extended (Odoo module mapping).
- `WORKING.md` â€” last decisions log and current focus updated.
- `data/odoo-mapping.md` referenced as the field-level companion to the new mapping doc.

### Closes self-critique gaps (round 4 partial)
- Gap AD (no consolidated answer to "which Odoo modules are required") â€” closed.
- DEC-003 (custom Odoo module scope) â€” proposed scope ratified pending S0 sign-off.
- DEC-004 (GPS storage target) â€” re-confirmed as custom model, recorded with rationale.

## [1.6.0] â€” 2026-05-13

### Added
- `LICENSE` â€” proprietary / confidential terms at root.
- `.gitignore.sample` â€” recommended `.gitignore` for the program repo (mobile/, backend/, edge/, infra/).
- `engineering/ci-pipeline.md` â€” full CI/CD spec: 7 workflows (PR per area, main, releases, nightly soak, manual rollback), required checks, caching/secrets, ownership.
- `engineering/api-versioning.md` â€” URL-major versioning, breaking-vs-additive rules, deprecation playbook, error envelope, idempotency, OpenAPI versioning, branch protection ties.
- `qa/test-data-strategy.md` â€” personas, distributions (tiny/small/pilot/xlarge), evidence corpus, sync envelope corpus, anonymisation, CI integration, pilot seeding.
- `runbooks/mobile-release.md` â€” store-release runbook (TestFlight + Play closed/open tracks), pre-flight checklist, rejection handling, demo account, phased rollout dashboard.
- `runbooks/odoo-dr-upgrade.md` â€” Odoo degraded-mode behaviour, intra-region failover, region failover, backup/restore drill, major upgrade phases, schema migrations.
- `governance/onboarding.md` â€” Day 1 / 7 / 30 ramp-up plan; reading order, setup checklist, first commit, vertical-slice deep-dive, story ownership; AI-agent per-task variant.
- `governance/comms-templates.md` â€” 11 templates: sprint review email, weekly stakeholder update, release notes, incident notice/resolution, postmortem invite, decision request, ADR proposal, joiner welcome, end-of-pilot summary.
- `backlog/capacity-plan.md` â€” team baseline, velocity formula, per-sprint capacity reservation, headroom (bug bash, NFR debt, docs, spike), velocity tracking template, risk-based adjustments, WIP limits, stop-the-line conditions.
- `epics/EPIC-10-post-mvp/stories/US-POST-001..011-*.md` â€” 11 post-MVP story stubs (dynamic forms, supervisor read-only, i18n, biometric unlock, map view, batch sync, push, server-side Whisper, OCR, white-label, SSO).

### Changed
- `README.md` â€” version bump to 1.6.0; quick links extended (CI, API versioning, store/Odoo runbooks, onboarding, comms, capacity, test-data, LICENSE, .gitignore).
- `WORKING.md` â€” last decisions log and current focus updated.

### Closes self-critique gaps (round 3)
- Gap R (no LICENSE) â€” closed.
- Gap S (no `.gitignore` sample) â€” closed.
- Gap T (no CI pipeline spec) â€” closed.
- Gap U (no API versioning policy) â€” closed.
- Gap V (no test-data strategy) â€” closed.
- Gap W (no store-release runbook) â€” closed.
- Gap X (no Odoo DR / upgrade runbook) â€” closed.
- Gap Y (no onboarding plan) â€” closed.
- Gap Z (no comms templates) â€” closed.
- Gap AA (no capacity / velocity plan) â€” closed.
- Gap AB (EPIC-10 had no story files) â€” closed (11 Draft stubs).

## [1.5.0] â€” 2026-05-13

### Added
- `adr/ADR-009-drift-over-sqflite.md` â€” adopts Drift 2.18+ for the local SQLite layer, closes DEC-011.
- `engineering/accessibility.md` â€” single source of truth for A11Y rules; per-story checklist; covers WCAG 2.1 AA targets, semantics, contrast, focus, live regions, locale rules.
- `engineering/performance-budget.md` â€” consolidated perf budget IDs (PB-M-*, PB-B-*, PB-N-*, PB-D-*) so stories cite IDs instead of restating numbers.
- `engineering/code-owners.md` â€” CODEOWNERS table, PR template, issue templates (bug, NFR debt, feature request), review SLAs.
- `qa/uat-plan.md` â€” pilot UAT scripts (S-UAT-01..15), cadence, sign-off.
- `backlog/releases/_template.md` â€” release-notes template referenced by `engineering/release-process.md`.

### Changed
- `traceability/decision-log.md` â€” DEC-011 marked **Decided** with linked ADR-009; entry added under "Closed decisions".
- `adr/README.md` â€” ADR-009 listed; tally now 9 ADRs.
- `engineering/README.md` â€” index lists accessibility, performance-budget, code-owners, dependency-policy, i18n.
- `qa/README.md` â€” index lists load-plan and uat-plan.
- `backlog/README.md` â€” index lists raid-log and releases template.
- `README.md` â€” version bump to 1.5.0; quick links updated; quick-link list deduped (single RAID entry); ADR count updated to 9.
- `WORKING.md` â€” last-decisions log notes v1.5.0 release; current focus updated; DEC-011 removed from open blockers.

### Closes self-critique gaps
- Round 2 gap I (DEC-011 inconsistency) â€” closed by ADR-009.
- Round 2 gap J (no accessibility doc) â€” closed.
- Round 2 gap K (no performance-budget doc) â€” closed.
- Round 2 gap L (UAT plan referenced but missing) â€” closed.
- Round 2 gap M (no release-notes template) â€” closed.
- Round 2 gap N (no CODEOWNERS / PR-template doc) â€” closed.
- Round 2 gap O (README quick-link RAID duplication) â€” closed.

## [1.4.0] â€” 2026-05-13

### Added
- `api-contracts/openapi.yaml` â€” OpenAPI 3.1 spec covering auth, shifts, sync, media, admin endpoints with shared error envelope and named schemas. Markdown contracts remain authoritative until backend scaffolding lands in S1; after S1 CI diffs FastAPI-emitted spec against this file.
- `governance/story-expansion-plan.md` â€” sprint-by-sprint plan to bring every story up to the full template, with exemplar references and an appendix tracking expansion status.

### Changed
- `api-contracts/README.md` lists `openapi.yaml`.
- Expanded `epics/EPIC-06-sync-engine/stories/US-SYNC-001-queue-manager.md` from 0.9 KB stub to 4.2 KB full template (background, 11 numbered AC, mobile/tests task split, dependencies, risks, FR + TC mapping, story-specific DoD). Reference exemplar for sync-area expansion.

### Closes self-critique gaps
- Gap B (OpenAPI YAML) â€” closed.
- Gap A (story body depth) â€” process closed; per-sprint expansion now scheduled in `governance/story-expansion-plan.md`. Two exemplars in place (US-AUTH-001 UI-leaning, US-SYNC-001 infrastructure).

## [1.3.0] â€” 2026-05-13

### Added
- `engineering/dependency-policy.md` â€” vetting checklist, pinning, updates, supply-chain integrity, anti-patterns.
- `engineering/i18n.md` â€” locale strategy (EN + VN at MVP), ARB workflow, error-code mapping, pseudolocalisation plan.
- `security/data-classification.md` â€” L0â€“L4 levels, field-level table, handling rules for logs/metrics/storage/transit, retention, DSAR + erasure paths.
- `qa/load-plan.md` â€” 8 load scenarios (LP-001..008) covering sync steady-state, burst, idempotency storm, auth, media, backpressure, refresh storm, mobile e2e harness; cadence and capacity assumptions.

### Changed
- `governance/self-critique.md` round 1 closes gaps D, E, F, H from the residual list.

## [1.2.0] â€” 2026-05-13

### Added
- `SECURITY.md` at root: vulnerability reporting policy, scope, SLAs.
- `engineering/` folder: `dev-environment.md`, `coding-standards.md`, `git-workflow.md`, `code-review-checklist.md`, `feature-flags.md`, `release-process.md`, `observability.md`, `oncall.md`, `README.md`.
- `governance/definition-of-ready.md`, `governance/definition-of-done.md`, `governance/sprint-ceremonies.md` â€” split out from `traceability/definitions.md` for authoritative versions.
- `data/erd.md` â€” Mermaid ER diagrams for SQLite, Postgres, and the sync state machine.
- `backlog/raid-log.md` â€” running Risks / Assumptions / Issues / Dependencies log.
- `runbooks/_postmortem-template.md` â€” blameless postmortem skeleton.
- Expanded `qa/test-cases.md`: SYNC (TC-SYNC-001..025), ODOO (TC-ODOO-001..024), SEC (TC-SEC-001..015), PERF (TC-PERF-001..013), A11Y (TC-A11Y-001..004), OBS (TC-OBS-001..004) with full steps / expected / automation notes per case.
- Coverage targets table in `qa/test-cases.md` for the S10 pilot gate.

### Changed
- `README.md` reading paths updated to surface `engineering/` and the new governance docs.
- `README.md` quick links extended (RAID log, ERD, DoR/DoD, security policy, engineering hub).

### Notes
- Test-case body grew from ~8 KB stub to a fully-specified suite usable by an autonomous agent without further clarification.

## [1.1.0] â€” 2026-05-13

### Added
- Epic READMEs for all 11 epics (`epics/EPIC-00..EPIC-10/README.md`).
- 70 user-story files across all MVP epics under `epics/EPIC-XX/stories/`.
- 11 sprint plans in Sprint Planning Prompt format under `sprints/sprint-NN-*/README.md`.
- `epics/README.md` index for epics.
- `sprints/README.md` index for sprints.
- Per-epic `stories/README.md` index files.
- `AGENTS.md` at project root: how AI agents and engineers should consume this blueprint.
- `WORKING.md`: current focus + open blockers + next 3 actions.
- `epics/_story-template.md`: copy-paste template for new stories.

### Changed
- Aligned story references to schema names in `data/sqlite-schema.md` (replaced `outbox_audit` and `media_blob` and `shifts_cache` references with `sync_attempt`, `media`, `shift`).
- Tightened EPIC-04 README to reference the canonical schema as source of truth.

## [1.0.0] â€” 2026-05-13

### Added
- Initial production blueprint generated from Field Work + Contracting v2.7.0 source brief
- Root: README, CHANGELOG, CONTRIBUTING
- Governance: RACI, communication plan, escalation, stakeholders, change management
- Architecture: C4 (context, container, component), tech stack, NFR, sequence flows, deployment topology
- ADRs: 8 records (ADR-001 â†’ ADR-008) plus template
- Epics: 11 epic READMEs with stories
- Sprints: 11 sprint plans (S0 â†’ S10) in Sprint Planning Prompt format
- Traceability: FR matrix, QA acceptance mapping, definitions, decision log, glossary, open questions
- Data: SQLite schema, Odoo mapping, sync state machine, ERD
- API contracts: auth, shifts, sync, media, error catalog
- Security: STRIDE threat model, secrets handling, PII policy, auth flow
- QA: test strategy, test cases, pilot gate, device matrix
- Runbooks: rollback, incident, sync recovery, OTP fallback
- Backlog: release plan, risk register, NFR register

### Notes
- Phase 0 has 9 open decisions blocking S1 entry. See `traceability/decision-log.md`.
- ADR-006, ADR-007, ADR-008 added beyond the 5 originally identified to cover Riverpod, idempotency model, and Whisper on-device strategy.
