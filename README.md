# Field Work + Contracting — Unified Flutter App Delivery Blueprint

**Version:** 2.1.0
**Status:** Active (Phase 0 closed, Sprint 1 ready)
**Owners:** Engineering Lead, Product Owner, QA Lead
**Last updated:** 2026-05-15

---

## 1. What this document set is

This is the **production delivery blueprint** for the Unified Flutter mobile app that merges the legacy "Field Work" React/Lovable PWA with the planned contracting Flutter app. It is the **single source of truth** for engineers, QA, PM, and stakeholders during build, pilot, and rollout.

Every artifact here is structured so an autonomous agent or a new engineer can pick up a story, sprint, or runbook **without asking for clarification**.

## 2. How to navigate

```
field_work_delivery/
├── README.md                  ← you are here (entry point + reading path)
├── AGENTS.md                  ← how AI agents + new engineers consume this blueprint
├── WORKING.md                 ← current focus + open blockers + next 3 actions
├── CHANGELOG.md               ← document history
├── CONTRIBUTING.md            ← how to update this blueprint
├── SECURITY.md                ← vulnerability reporting + scope + SLAs
│
├── governance/                ← who decides what (RACI, comms, escalation, DoR/DoD, ceremonies)
├── engineering/               ← developer enablement (env, standards, git, review, release, obs, on-call, flags)
├── architecture/              ← C4 diagrams, NFR, sequence flows, deployment
├── adr/                       ← architectural decisions of record
│
├── epics/                     ← 11 epics × stories (work breakdown)
│   ├── README.md              ← epic index
│   ├── _story-template.md     ← template for new stories
│   └── EPIC-XX-*/
│       ├── README.md          ← epic charter
│       └── stories/
│           ├── README.md      ← story index
│           └── US-XXX-NNN-*.md
├── sprints/                   ← 11 sprints × plan (Sprint Planning Prompt format)
│   ├── README.md              ← sprint index
│   └── sprint-NN-*/README.md
│
├── traceability/              ← FR ↔ Epic ↔ Sprint ↔ TC, definitions, glossary, open Qs, decisions
├── data/                      ← SQLite schema, Postgres schema, Odoo mapping, sync state machine, ERD
├── api-contracts/             ← FastAPI endpoint contracts (auth, shifts, sync, media, error catalog)
│
├── security/                  ← STRIDE threat model, secrets, PII handling, auth flow
├── qa/                        ← test strategy, test cases, pilot gate, device matrix
├── runbooks/                  ← rollback, incident, sync recovery, OTP fallback, postmortem template
└── backlog/                   ← release plan, risk register, NFR register, RAID log
```

### Recommended reading order by role

| Role | Read first | Then |
|---|---|---|
| **Product Owner** | README → governance/ → backlog/release-plan.md | epics/ → traceability/decision-log.md → backlog/raid-log.md |
| **Engineering Lead** | README → architecture/ → adr/ → engineering/ | epics/ → sprints/ → data/ → api-contracts/ |
| **Mobile Engineer** | engineering/dev-environment.md → architecture/tech-stack.md → data/sqlite-schema.md → epics/EPIC-01..06 | sprints/sprint-XX → stories assigned |
| **Backend Engineer** | engineering/dev-environment.md → architecture/c4-container.md → api-contracts/ → data/odoo-mapping.md | epics/EPIC-07,08 → sprints/sprint-08,09 |
| **QA Engineer** | qa/test-strategy.md → traceability/QA-acceptance-mapping.md | qa/test-cases.md → qa/pilot-gate.md |
| **DevOps / SRE** | runbooks/ → architecture/deployment.md → engineering/observability.md | security/secrets.md → backlog/risk-register.md |
| **Stakeholder** | README → backlog/release-plan.md | governance/stakeholders.md |
| **AI Agent** | AGENTS.md → WORKING.md → README → traceability/glossary.md → engineering/coding-standards.md | task-specific path above |

## 3. Project at a glance

| Attribute | Value |
|---|---|
| **Product** | Unified Flutter mobile app for field workers and contractors |
| **Platforms** | iOS 14+, Android 8+ (API 26+) |
| **Backend** | FastAPI + Redis + Celery on DigitalOcean Droplet (Docker Compose) |
| **Auth** | Supabase Edge Functions + Twilio Verify OTP |
| **ERP** | Odoo via JSON-RPC — runs on Online / Enterprise / Community; Odoo 19 EE is the test target (ADR-011, ADR-015) |
| **Object storage** | DigitalOcean Spaces (`sgp1`), S3-compatible API (ADR-012) |
| **Local store** | SQLite via Drift (offline-first, durable-write-first) |
| **Sync model** | Local SQLite → Mobile sync queue → FastAPI → Celery → Odoo (no custom Odoo module; RPC adapter + `x_ngynapp_*` fields) |
| **Sprint length** | 2 weeks |
| **Sprints (S0–S10)** | 11 sprints, ~22 weeks total |
| **Team size** | 4–6 (1 Mobile Lead, 1–2 Mobile, 1 Backend, 1 QA, 1 PM/PO; +Designer part-time) |
| **MVP definition** | EPIC-00 through EPIC-09 (S0–S10), EPIC-10 = post-MVP |
| **Pilot target** | End of Sprint 10 (Sprint 10 = Hardening & Pilot) |

## 4. Build order (locked)

```
S0  Discovery & Build Authorization
S1  Project Foundation (Flutter scaffold, CI, secure storage, SQLite skeleton)
S2  Auth & Employee Context (OTP login, employee mapping)
S3  Shifts Read Workflow (My Shifts dashboard, Shift Detail)
S4  SQLite Offline Foundation (DAOs, sync_queue table, sync states)
S5  Core Field Capture (PPE, pre-shift form, GPS check-in)
S6  Evidence Capture (photos with compression, voice note + Whisper)
S7  Mobile Sync Engine (WorkManager / BGTaskScheduler, retry, dead-letter)
S8  Backend Sync Intake (FastAPI ingest, Celery enqueue, idempotency)
S9  Odoo Write Path (Odoo JSON-RPC via RPC adapter; no custom module — capabilities detected at runtime)
S10 Hardening & Pilot (perf, security audit, device matrix, pilot launch)
```

This order is **not negotiable**. Cross-cutting concerns (SQLite in S1, secure storage in S1, sync states UI in S2, image compression in S6) are woven into earlier sprints to prevent late discovery of blockers.

## 5. Core principles (non-negotiable)

1. **Offline-first.** Local SQLite is the **first durable write**. Never block UX on network.
2. **No direct mobile-to-Odoo.** Mobile only talks to FastAPI. (See ADR-005.)
3. **Idempotency by `client_id`.** Every sync envelope carries a UUID v4 `client_id` so retries are safe.
4. **Durable evidence.** Photos and voice notes are stored locally with checksums, then uploaded with resumable strategy.
5. **Privacy by design.** GPS, voice, and photos require explicit per-action consent. No silent capture.
6. **Sync states are visible.** `PENDING → SYNCING → CONFIRMED / FAILED` is shown to the user, not hidden.
7. **All decisions are recorded.** ADRs for architecture; decision-log for Phase 0 open questions.
8. **Test-first for sync, auth, and Odoo writes.** These three areas are where bugs cost the most.

## 6. Phase 0 status — fully closed (2026-05-15, v2.1.0)

S0 Discovery decisions (DEC-001..009) are all `Decided`. Sponsor reconfirmation on 2026-05-15 flipped the four previously `Provisional` items.

- **Decided (architectural pivot — supersedes earlier ADRs):**
  - DEC-003 → **No custom Odoo module** ([ADR-015](./adr/ADR-015-no-custom-module-rpc-adapter.md))
  - DEC-004 → **GPS history in FastAPI Postgres** ([ADR-016](./adr/ADR-016-gps-in-fastapi-postgres.md))
  - DEC-001 → **Odoo 19 EE for testing; RPC contract for production** ([ADR-011 amended](./adr/ADR-011-odoo-19-enterprise.md))
  - DEC-007 → **DigitalOcean Spaces (`sgp1`)** ([ADR-012 amended](./adr/ADR-012-aws-s3-object-storage.md))
- **Decided (operational):**
  - DEC-002 → Capability detection at runtime; install only modules in use.
  - DEC-005 → Twilio Verify only for MVP; key issued.
  - DEC-006 → `work_phone` → `private_phone` → `mobile_phone` fallback; E.164 normalised.
  - DEC-008 → Odoo SLA delegated to customer's Odoo provider; we own integration-layer SLA.
  - DEC-009 → DigitalOcean Droplet + Compose for testing/pilot; vertical scale-up for prod; DOKS deferred.
- **New stretch decisions:** DEC-014 (GPS retention 180 days), DEC-015 (`x_ngynapp_*` field prefix reserved).

Sprint 1 is unblocked. Two pieces of S1-adjacent work were cancelled (custom Odoo module repo + Odoo CI pipeline). EPIC-08 story US-ODOO-001 is rewritten in-place as "RPC adapter scaffold + capability probe".

## 7. Quick links

### Planning + governance
- 11 Epics: [epics/](./epics/)
- 11 Sprint plans: [sprints/](./sprints/)
- 16 ADRs: [adr/](./adr/)
- 15 Functional Requirements: [traceability/FR-matrix.md](./traceability/FR-matrix.md)
- Decision log: [traceability/decision-log.md](./traceability/decision-log.md)
- Glossary: [traceability/glossary.md](./traceability/glossary.md)
- Definition of Ready / Done: [governance/definition-of-ready.md](./governance/definition-of-ready.md), [governance/definition-of-done.md](./governance/definition-of-done.md)
- Story expansion plan: [governance/story-expansion-plan.md](./governance/story-expansion-plan.md)
- Self-critique log: [governance/self-critique.md](./governance/self-critique.md)
- Onboarding (Day 1/7/30): [governance/onboarding.md](./governance/onboarding.md)
- Comms templates: [governance/comms-templates.md](./governance/comms-templates.md)

### Engineering
- Engineering hub: [engineering/README.md](./engineering/README.md)
- CI pipeline: [engineering/ci-pipeline.md](./engineering/ci-pipeline.md)
- API versioning: [engineering/api-versioning.md](./engineering/api-versioning.md)
- Accessibility: [engineering/accessibility.md](./engineering/accessibility.md)
- Performance budget: [engineering/performance-budget.md](./engineering/performance-budget.md)
- CODEOWNERS + templates: [engineering/code-owners.md](./engineering/code-owners.md)
- Odoo RPC adapter spec: [engineering/odoo-module-mapping.md](./engineering/odoo-module-mapping.md)
- Odoo `x_ngynapp_*` field spec: [data/odoo-x-fields-spec.md](./data/odoo-x-fields-spec.md)
- DigitalOcean deployment topology: [architecture/deployment.md](./architecture/deployment.md)

### QA + ops
- Pilot gate: [qa/pilot-gate.md](./qa/pilot-gate.md)
- Load test plan: [qa/load-plan.md](./qa/load-plan.md)
- UAT plan: [qa/uat-plan.md](./qa/uat-plan.md)
- Test data strategy: [qa/test-data-strategy.md](./qa/test-data-strategy.md)
- RAID log: [backlog/raid-log.md](./backlog/raid-log.md)
- Risk register: [backlog/risk-register.md](./backlog/risk-register.md)
- Release plan: [backlog/release-plan.md](./backlog/release-plan.md)
- Capacity & velocity: [backlog/capacity-plan.md](./backlog/capacity-plan.md)
- Release-notes template: [backlog/releases/_template.md](./backlog/releases/_template.md)
- Mobile store release runbook: [runbooks/mobile-release.md](./runbooks/mobile-release.md)
- Odoo DR / upgrade runbook: [runbooks/odoo-dr-upgrade.md](./runbooks/odoo-dr-upgrade.md)

### Architecture + data + APIs
- ERD: [data/erd.md](./data/erd.md)
- OpenAPI: [api-contracts/openapi.yaml](./api-contracts/openapi.yaml)
- Security policy: [SECURITY.md](./SECURITY.md)
- Data classification: [security/data-classification.md](./security/data-classification.md)

### Repo metadata
- License: [LICENSE](./LICENSE)
- `.gitignore` template: [.gitignore.sample](./.gitignore.sample)

## 8. Document conventions

- **Language.** English for all artifacts (epics, stories, ADRs, runbooks). Vietnamese is allowed in `traceability/decision-log.md` comments where stakeholders prefer it.
- **IDs.** `EPIC-NN`, `US-<EPIC-SHORT>-NNN`, `TC-<AREA>-NNN`, `FR-NNN`, `DEC-NNN`, `ADR-NNN`, `RISK-NNN`, `NFR-NNN`.
- **Story estimates.** `S` ≤ 1 day, `M` 1–3 days, `L` 3–5 days, `XL` >5 days (must split).
- **Priority.** P0 = blocker for MVP, P1 = MVP target, P2 = post-MVP.
- **Status.** `Draft → Ready → In Progress → In Review → Done`.

## 9. How agents should consume this

If you are an autonomous agent assigned to implement a story:

1. Read `traceability/glossary.md` (terms used everywhere).
2. Read the story file (`epics/EPIC-XX/stories/US-XXX-NNN.md`).
3. Follow links from the story to the epic README, sprint plan, and any referenced data/ or api-contracts/ file.
4. Check `traceability/decision-log.md` for any DEC-NNN listed under the story's "Open Questions". If unresolved, do **not** implement; raise it.
5. Use `qa/test-cases/` to derive verification steps.
6. Update story status in the file front matter when starting and finishing.
7. Never modify ADRs or governance files without escalation per `governance/change-management.md`.

---

For document history see [CHANGELOG.md](./CHANGELOG.md). To propose a change to this blueprint see [CONTRIBUTING.md](./CONTRIBUTING.md).
