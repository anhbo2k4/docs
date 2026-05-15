# Field Work + Contracting — Unified Mobile App

> Offline-first Flutter app for field workers. Capture shifts, photos, voice notes → sync to Odoo.

**Version:** 2.1.0 | **Status:** Sprint 1 Ready | **Last updated:** 2026-05-15

---

## What is this?

A mobile app (iOS + Android) that replaces paper forms for field workers:

```
┌──────────────────┐         ┌──────────────┐         ┌─────────┐
│   📱 Mobile App   │  sync   │  ⚡ FastAPI   │  push   │  🏢 Odoo │
│   (works offline) │ ──────▶ │  + Celery    │ ──────▶ │  (ERP)  │
└──────────────────┘         └──────────────┘         └─────────┘
```

Workers clock in, fill safety checklists, snap photos, record voice notes — all offline. Data syncs automatically when connected.

---

## Quick Navigation

| You are... | Start here |
|------------|-----------|
| 🏢 **Stakeholder / PM** | [docs/FOR-STAKEHOLDERS.md](./docs/FOR-STAKEHOLDERS.md) |
| 👨‍💻 **Developer (new)** | [docs/QUICKSTART.md](./docs/QUICKSTART.md) |
| 🤖 **AI Agent** | [AGENTS.md](./AGENTS.md) → [WORKING.md](./WORKING.md) |
| 📋 **Full blueprint** | [docs/FULL-BLUEPRINT.md](./docs/FULL-BLUEPRINT.md) |

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Mobile | Flutter + SQLite (Drift) + Riverpod |
| Auth | Supabase Edge + Twilio OTP (SMS login) |
| Backend | FastAPI + Celery + Redis + PostgreSQL |
| ERP | Odoo 19 via JSON-RPC (no custom module) |
| Hosting | DigitalOcean (Droplet + Spaces + Managed DB) |

---

## Timeline (22 weeks)

```
S0  ✅ Discovery (done)
S1  ▶️ Project Foundation        ← WE ARE HERE
S2     Auth & Login
S3     Shifts Dashboard
S4     Offline Database
S5     Safety Forms & GPS
S6     Photos & Voice Notes
S7     Sync Engine (mobile)
S8     Sync Engine (backend)
S9     Odoo Integration
S10    Hardening & Pilot Launch
```

---

## Core Principles

1. **Offline-first** — SQLite is the first write. UI never blocks on network.
2. **No direct mobile → Odoo** — Always through FastAPI.
3. **Idempotent sync** — Every payload has a UUID. Retries are safe.
4. **Privacy by design** — GPS/photo/voice require explicit consent.
5. **Visible sync status** — Users see PENDING → SYNCING → CONFIRMED/FAILED.

---

## Repo Structure

```
├── docs/                  ← 📖 Start here (stakeholder + dev guides)
├── architecture/          ← System design, C4 diagrams, deployment
├── adr/                   ← Architecture Decision Records (16 ADRs)
├── engineering/           ← Dev environment, coding standards, CI
├── epics/                 ← 11 epics with user stories
├── sprints/               ← 11 sprint plans
├── data/                  ← DB schemas, ERD, Odoo field mapping
├── api-contracts/         ← OpenAPI spec
├── qa/                    ← Test strategy, test cases, pilot gate
├── security/              ← Threat model, data classification
├── runbooks/              ← Incident response, rollback procedures
├── backlog/               ← Release plan, risk register, RAID log
├── governance/            ← RACI, ceremonies, DoR/DoD
└── traceability/          ← Requirements ↔ Epics ↔ Tests mapping
```

---

## Key Links

| Resource | Link |
|----------|------|
| Current focus & blockers | [WORKING.md](./WORKING.md) |
| All architecture decisions | [adr/](./adr/) |
| API specification | [api-contracts/openapi.yaml](./api-contracts/openapi.yaml) |
| Release plan | [backlog/release-plan.md](./backlog/release-plan.md) |
| Contributing rules | [CONTRIBUTING.md](./CONTRIBUTING.md) |
| Security policy | [SECURITY.md](./SECURITY.md) |
| Changelog | [CHANGELOG.md](./CHANGELOG.md) |

---

## Contributing

See [CONTRIBUTING.md](./CONTRIBUTING.md) for branch naming, PR workflow, and review rules.

---

*For the complete detailed blueprint (all sections, conventions, agent instructions), see [docs/FULL-BLUEPRINT.md](./docs/FULL-BLUEPRINT.md).*
