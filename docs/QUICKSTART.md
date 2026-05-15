# Developer Quickstart

> Get oriented in 5 minutes. Start contributing in 15.

---

## TL;DR

We're building an **offline-first Flutter app** that lets field workers capture shift data, photos, and voice notes — then syncs everything to Odoo via a FastAPI backend.

```
Mobile (Flutter + SQLite) → FastAPI (Python) → Odoo (JSON-RPC)
```

---

## Tech Stack

| Layer | Tech | Key detail |
|-------|------|------------|
| Mobile | Flutter (Dart) | iOS 14+ / Android 8+ |
| Local DB | SQLite via Drift | Offline-first, all writes go here first |
| State mgmt | Riverpod | Recommended, not mandated |
| Auth | Supabase Edge + Twilio OTP | Phone-based, no passwords |
| Backend | FastAPI + Celery + Redis | Async job processing |
| Database | PostgreSQL (managed) | On DigitalOcean |
| Storage | DigitalOcean Spaces | S3-compatible, `sgp1` region |
| ERP | Odoo 19 via JSON-RPC | No custom module — RPC adapter pattern |

---

## Repo Structure (what matters)

```
├── README.md              ← Start here (project overview)
├── WORKING.md             ← Current sprint focus + blockers
│
├── engineering/           ← YOUR FIRST STOP
│   ├── dev-environment.md    ← Setup instructions
│   ├── coding-standards.md   ← Style guide
│   └── ci-pipeline.md        ← How CI works
│
├── architecture/          ← System design
│   ├── tech-stack.md         ← Full stack decisions
│   └── deployment.md         ← Infra topology
│
├── data/                  ← Schemas
│   ├── sqlite-schema.md      ← Mobile DB schema
│   ├── erd.md                ← Entity relationships
│   └── odoo-mapping.md       ← How data maps to Odoo
│
├── api-contracts/         ← API specs
│   └── openapi.yaml          ← Full OpenAPI spec
│
├── epics/                 ← Work breakdown
│   └── EPIC-XX/stories/      ← Individual user stories
│
└── sprints/               ← Sprint plans
    └── sprint-NN/            ← What's in each sprint
```

---

## Your First Day

### Mobile Engineer

1. Read [engineering/dev-environment.md](../engineering/dev-environment.md) — setup Flutter, emulators
2. Read [architecture/tech-stack.md](../architecture/tech-stack.md) — understand the stack
3. Read [data/sqlite-schema.md](../data/sqlite-schema.md) — the local DB you'll work with daily
4. Check [WORKING.md](../WORKING.md) — see what sprint we're in
5. Pick your assigned story from `epics/` → read it → code it

### Backend Engineer

1. Read [engineering/dev-environment.md](../engineering/dev-environment.md) — setup Python, Docker
2. Read [api-contracts/openapi.yaml](../api-contracts/openapi.yaml) — the API you're building
3. Read [data/odoo-mapping.md](../data/odoo-mapping.md) — how we talk to Odoo
4. Read [engineering/odoo-module-mapping.md](../engineering/odoo-module-mapping.md) — RPC adapter spec
5. Check [WORKING.md](../WORKING.md) → pick your story

### QA Engineer

1. Read [qa/test-strategy.md](../qa/test-strategy.md) — overall approach
2. Read [qa/test-cases.md](../qa/test-cases.md) — existing test cases
3. Read [traceability/FR-matrix.md](../traceability/FR-matrix.md) — requirements ↔ tests mapping
4. Check [qa/pilot-gate.md](../qa/pilot-gate.md) — what "done" looks like

---

## Core Rules (don't break these)

1. **Offline-first** — SQLite is the first write. Never block UI on network.
2. **No direct mobile → Odoo** — Always go through FastAPI.
3. **Idempotency** — Every sync payload has a `client_id` (UUID v4). Retries are safe.
4. **Privacy** — GPS/photo/voice require explicit user consent per action.
5. **Sync is visible** — Users see PENDING → SYNCING → CONFIRMED/FAILED status.

---

## How to pick up a story

```
1. Check WORKING.md → current sprint
2. Go to sprints/sprint-NN/ → see story list
3. Open epics/EPIC-XX/stories/US-XXX-NNN.md
4. Read acceptance criteria
5. Check linked data/ or api-contracts/ files
6. Code it, test it, PR it
```

---

## Key Links

| What | Where |
|------|-------|
| Current focus | [WORKING.md](../WORKING.md) |
| Coding standards | [engineering/coding-standards.md](../engineering/coding-standards.md) |
| API spec | [api-contracts/openapi.yaml](../api-contracts/openapi.yaml) |
| Architecture decisions | [adr/](../adr/) |
| Glossary | [traceability/glossary.md](../traceability/glossary.md) |
| PR/contribution rules | [CONTRIBUTING.md](../CONTRIBUTING.md) |

---

## Questions?

- Check [traceability/glossary.md](../traceability/glossary.md) for terminology
- Check [adr/](../adr/) for "why did we choose X?"
- Full blueprint details: [docs/FULL-BLUEPRINT.md](./FULL-BLUEPRINT.md)

---

*Last updated: 2026-05-15 | Blueprint v2.1.0*
