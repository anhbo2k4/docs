# Self-critique log

**Status:** Active
**Owner:** Engineering Lead
**Purpose:** Record gaps found in the blueprint and what was done about them. This is meta — it's how we know the blueprint is improving rather than just growing.

## Round 1 — 2026-05-13 (after v1.1.0)

### Gaps identified

| # | Gap | Severity |
|---|---|---|
| 1 | `qa/test-cases.md` had stub headers for SYNC-006..010, ODOO-001..006, SEC-*, PERF-* with no steps/expected/automation. An agent could not execute. | High |
| 2 | API contracts narrative-style; mostly OK but error-code coverage thin. | Med |
| 3 | Many stories <1 KB; AC and DoD not always sufficient for autonomous execution. | Med |
| 4 | No `engineering/` hub. Devs (and agents) lacked a single place for env setup, coding standards, git, review, release. | High |
| 5 | DoR / DoD scattered between `traceability/definitions.md` and per-sprint READMEs. No authoritative version. | Med |
| 6 | No ER diagram. Schema files were column-only; visual relationships had to be reconstructed mentally. | Med |
| 7 | No RAID log; risk register existed but cross-cutting view (assumptions, issues, dependencies) was missing. | Med |
| 8 | No `SECURITY.md` at root for inbound vulnerability reports. | High |
| 9 | No postmortem template; incident-response file exists but template was implicit. | Med |
| 10 | No formal observability spec (metric naming, label cardinality, dashboards, SLOs). | High |
| 11 | No on-call doc separate from runbooks; pager criteria implicit. | Med |
| 12 | No code-review checklist or PR template. | Med |
| 13 | No feature-flag policy. Cross-cutting changes had no documented rollout safety net. | Med |
| 14 | No dev-environment doc with pinned versions. | High |
| 15 | No release-process doc separate from rollback runbook. | Med |
| 16 | Sprint ceremonies, cadence, and stakeholder review rhythm not documented. | Med |
| 17 | No A11Y or OBS test cases. | Med |

### Actions taken — v1.2.0

- Expanded `qa/test-cases.md` to 25 SYNC + 11 ODOO + 15 SEC + 13 PERF + 4 A11Y + 4 OBS cases, each with steps / expected / automation / FR-NFR mapping. Added coverage targets table for the S10 gate.
- Added `engineering/` with `dev-environment.md`, `coding-standards.md`, `git-workflow.md`, `code-review-checklist.md`, `feature-flags.md`, `release-process.md`, `observability.md`, `oncall.md`, `README.md`.
- Added `governance/definition-of-ready.md`, `governance/definition-of-done.md`, `governance/sprint-ceremonies.md`.
- Added `data/erd.md` (SQLite ERD, Postgres ERD, sync state machine diagram).
- Added `backlog/raid-log.md`.
- Added `SECURITY.md` at root.
- Added `runbooks/_postmortem-template.md`.
- README and AGENTS routing updated to surface the new docs.
- CHANGELOG bumped to 1.2.0; WORKING.md updated.

### Gaps still open after v1.2.0

| # | Gap | Plan | Status after v1.3.0 |
|---|---|---|---|
| A | Story bodies still vary in depth. Sub-1KB stories should be expanded with explicit AC, DoD, test plan. | Sprint-by-sprint; PM expands the next sprint's stories during refinement. | **Process closed v1.4.0** — `governance/story-expansion-plan.md` defines exemplars + per-sprint expansion calendar. Tracking remains open per sprint. |
| B | OpenAPI YAML is not generated. Contracts are markdown-only. | Add `api-contracts/openapi.yaml` once backend scaffold lands in S1. | **Closed v1.4.0** — `api-contracts/openapi.yaml` (OpenAPI 3.1) ships ahead of S1; CI diff target deferred to S1. |
| C | Architecture diagrams are Mermaid-only; no rendered images. | Render in CI and commit `docs/img/` once `mermaid-cli` is on the build agent. | Open. Deferred to S1 CI. |
| D | No data classification matrix. | Add `security/data-classification.md` in the first security review. | **Closed v1.3.0** — `security/data-classification.md`. |
| E | No `engineering/dependency-policy.md`. | Add when first new dep is proposed; for now, ADR for any new dep is required. | **Closed v1.3.0** — `engineering/dependency-policy.md`. |
| F | No `engineering/i18n.md` despite a multilingual user base. | Add when the first non-English locale story enters refinement. | **Closed v1.3.0** — `engineering/i18n.md`. |
| G | Test-case file is monolithic; will exceed 1500 lines mid-pilot. | Split per area in S5 per the file's own note. | Open. Deferred to S5. |
| H | No load-test plan in `qa/`. | Add `qa/load-plan.md` referencing TC-PERF-008 and TC-SYNC-024. | **Closed v1.3.0** — `qa/load-plan.md`. |

### How to run the next round

When v1.3.0 is being considered:

1. Re-read all root-level files (README, AGENTS, WORKING, CHANGELOG, SECURITY).
2. Walk every directory's README to verify it still describes contents accurately.
3. Sample 3 stories per epic; verify they meet DoR with no extra context.
4. Ask: "Could a fresh agent build this with only this blueprint?" For each `no`, file a self-critique entry above and resolve in the next version.


## Round 2 — 2026-05-13 (after v1.4.0)

### Gaps identified

| # | Gap | Severity |
|---|---|---|
| I | DEC-011 (Drift vs sqflite) was "default Drift" in WORKING.md but still `Open` in `decision-log.md`, with no ADR. Architectural decision without an ADR. | High |
| J | No `engineering/accessibility.md`. NFR-060..065 were spread across NFR + per-story AC; UI stories had no single page to consume on expansion. | High |
| K | No consolidated performance budget. Targets were scattered across NFR-001..011, individual stories, and load plan; risk of drift. | Med |
| L | `qa/uat-plan.md` was referenced by `qa/test-strategy.md` but did not exist. | Med |
| M | `engineering/release-process.md` mandated a `releases/<version>.md` format but no template existed. | Med |
| N | `engineering/observability.md` mandated CODEOWNER review for new metric labels but the repo had no CODEOWNERS doc, PR template, or issue templates documented in the blueprint. | Med |
| O | README §7 quick links contained two `RAID log` lines (duplicate). | Low |

### Actions taken — v1.5.0

- ADR-009 adopts Drift; `decision-log.md` updated to **Decided** with ADR link; "Closed decisions" populated.
- `engineering/accessibility.md` consolidates A11Y-01..10 with implementation rules, assistive-tech matrix, test-case mapping, and a paste-in per-story checklist.
- `engineering/performance-budget.md` introduces PB-M-*, PB-B-*, PB-N-*, PB-D-* IDs that stories cite by reference; verification cadence per PR / sprint / S10 / pilot.
- `qa/uat-plan.md` defines pilot cohort, 15 S-UAT scripts, pass/fail rules, daily cadence, sign-off.
- `backlog/releases/_template.md` provides the release-notes structure referenced by the release process.
- `engineering/code-owners.md` lists CODEOWNERS by path, the PR template, three issue templates (bug, NFR debt, feature request), and review SLAs.
- README quick-link block deduplicated and re-ordered; ADR count updated to 9; version bumped to 1.5.0.
- WORKING.md, CHANGELOG.md, and folder index READMEs (`engineering/`, `backlog/`, `qa/`, `adr/`) updated to surface the new docs.

### Gaps still open after v1.5.0

| # | Gap | Plan | Status |
|---|---|---|---|
| A | Story bodies still vary in depth. | Sprint-by-sprint expansion via `governance/story-expansion-plan.md`. | Process closed v1.4.0; per-sprint expansion still tracked. |
| C | Architecture diagrams remain Mermaid-only. | Render in CI once `mermaid-cli` lands on the build agent. | Open. Deferred to S1 CI. |
| G | `qa/test-cases.md` will exceed practical size mid-pilot. | Split per area in S5. | Open. Deferred to S5. |
| P | OpenAPI 3.1 spec not yet diffed against FastAPI-emitted spec. | Add CI job once backend scaffold lands in S1. | Open. Deferred to S1. |
| Q | DEC-001..009 still open. | Close in S0 per `epics/EPIC-00-discovery/`. | Open. Owner: PM + Sponsor. |

### How to run round 3 (post-S0)

1. Re-read root files (README, AGENTS, WORKING, CHANGELOG, SECURITY).
2. Verify each DEC closure has either an ADR (architectural) or a `decision-log.md` entry (operational).
3. Re-sample 3 stories per epic; confirm DoR readiness for the next sprint per `governance/story-expansion-plan.md`.
4. Confirm every NFR has a current pass status in `backlog/nfr-register.md`.
5. Ask: "Could a fresh agent build the next sprint with only this blueprint?" Each `no` becomes a round-3 entry.


## Round 3 — 2026-05-13 (after v1.6.0)

### Gaps identified

| # | Gap | Severity |
|---|---|---|
| R | No `LICENSE` at root. Confidential delivery blueprint shared to vendors needed explicit terms. | Med |
| S | No `.gitignore` template for the future program repo. New scaffold sprint (S1) would have to invent it. | Low |
| T | CI/CD references scattered across many docs (release-process, mobile-release, code-owners, observability) but no single pipeline spec with required-check matrix. | High |
| U | No API versioning policy. Backwards-compat rules implicit. Risky once mobile pins to a major. | High |
| V | No test-data strategy. Fixtures, personas, anonymisation rules absent → CI flakiness + privacy risk. | High |
| W | No store-release runbook (TestFlight + Play Console). Release happens once a sprint; needs a checklist. | High |
| X | No Odoo DR / upgrade runbook. Single biggest external dependency had no degraded-mode story. | High |
| Y | No onboarding plan. New joiners (and AI agents) had to derive Day-1 reading order from README. | Med |
| Z | No comms templates. PM had to draft from scratch each time. | Med |
| AA | No capacity / velocity plan tying team size → sprint targets. | Med |
| AB | EPIC-10 had a README + index but no story files. Promotion process undefined in practice. | Med |

### Actions taken — v1.6.0

- Added `LICENSE` (proprietary, confidential).
- Added `.gitignore.sample`.
- Added `engineering/ci-pipeline.md` (7 workflows, required checks, ownership).
- Added `engineering/api-versioning.md` (URL-major, breaking-vs-additive, deprecation playbook, OpenAPI versioning).
- Added `qa/test-data-strategy.md` (personas, corpora, anonymisation, pilot seeding).
- Added `runbooks/mobile-release.md` (TestFlight + Play, rejection handling, phased rollout).
- Added `runbooks/odoo-dr-upgrade.md` (degraded mode, failover, drill cadence, major upgrade phases).
- Added `governance/onboarding.md` (Day 1/7/30 + AI-agent per-task variant).
- Added `governance/comms-templates.md` (11 templates).
- Added `backlog/capacity-plan.md` (team baseline, velocity formula, per-sprint reservation, WIP limits).
- Added 11 EPIC-10 story stubs (US-POST-001..011).
- Bumped README version, CHANGELOG entry, WORKING last-decisions log.

### Gaps still open after v1.6.0

| # | Gap | Plan | Status |
|---|---|---|---|
| A | Story body depth varies. | Per-sprint expansion via `governance/story-expansion-plan.md`. | Process closed v1.4.0; tracked per sprint. |
| C | Architecture diagrams remain Mermaid-only; no rendered images. | Render in CI (`pr-docs / mermaid`) once `mermaid-cli` lands on the build agent. | Open. Deferred to S1 CI. |
| G | `qa/test-cases.md` size will exceed practical limit mid-pilot. | Split per area in S5. | Open. Deferred to S5. |
| P | OpenAPI 3.1 not yet diffed against FastAPI-emitted spec. | `pr-backend / contract` job spec'd in `engineering/ci-pipeline.md`; activate when backend scaffold lands. | Open. Deferred to S1. |
| Q | DEC-001..009 still open. | Close in S0 per `epics/EPIC-00-discovery/`. | Open. Owner: PM + Sponsor. |
| AC | EPIC-10 story stubs are short (≤ 1 KB each). | Promote-on-need: each post-MVP item expands to full template before entering a real sprint, per `governance/story-expansion-plan.md`. | Open by design. |

### How to run round 4

1. Re-read root files (README, AGENTS, WORKING, CHANGELOG, SECURITY, LICENSE).
2. Walk every directory's README to verify it still describes contents accurately (esp. `engineering/`, `runbooks/`, `governance/` after v1.6.0).
3. Verify each new doc has reciprocal cross-references (link-check pipeline).
4. Sample 2 stories per epic; verify they meet DoR for the next available sprint.
5. Confirm every NFR has a current pass status in `backlog/nfr-register.md`.
6. Ask: "Could a fresh agent build the next sprint with only this blueprint?" Each `no` becomes a round-4 entry.


## Round 5 — 2026-05-15 (post v2.0.0)

### Closes

- **Gap Q (DEC-001..009 still open)** — closed. v2.0.0 resolved all nine Phase 0 decisions: 5 Decided via ADR-010..ADR-014, 4 Provisional under PM confirmation SLA. EPIC-00 status flipped to Done; S1 unblocked.

### New gaps observed during round 5

| # | Gap | Plan | Status |
|---|---|---|---|
| AE | 4 Provisional items (DEC-002, 006, 008, 009) need a tracking SLA visible to sponsors. | Captured in `WORKING.md` "Provisional confirmations" section + `traceability/decision-log.md` confirm checklist. PM walks the list at S0 workshop. | Closed by tracking. |
| AF | Odoo Community path (Path A) referenced in `engineering/odoo-module-mapping.md` is now dead. | Out of scope per ADR-011; document still references both paths for historical context. Mark Path A as deprecated in next minor revision. | Open (low priority). |
| AG | Custom Odoo module repo `field-mobile-sync` doesn't exist yet. | Created in S1 alongside CI pipeline `odoo-module-ci`; tracked under DevOps owner. | Open, scheduled S1. |
| AH | `data/odoo-mapping.md` not yet aligned to Odoo 19 EE field names. | Amend during S8 entry once Odoo Specialist validates against staging tenant. | Open, scheduled S8. |
| AI | `docs/index.html` (entry-point dashboard) added in v2.0.0 but not rendered through the md2html skill (template files missing). | Manual hand-rolled HTML for now; revisit when md2html template lands. | Open, low priority. |

### How to run round 6

1. Run after the S0 confirmation workshop closes the 4 Provisional items.
2. Verify `data/odoo-mapping.md` reflects DEC-006 default until confirmed (E.164 normalisation rule documented).
3. Walk every doc that still references "9 open decisions" or "Phase 0 awaiting sign-off" — should now read "closed" or "Provisional under SLA".
4. Confirm `governance/story-expansion-plan.md` lists S1 stories as the next expansion batch.
