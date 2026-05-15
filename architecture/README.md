# Architecture Overview

**Status:** Active  
**Owner:** Engineering Lead  
**Last updated:** 2026-05-13

This folder contains the architecture documentation for the unified Flutter app. Diagrams use the [C4 model](https://c4model.com/) and [Mermaid](https://mermaid.js.org/).

## Contents

| File | Purpose |
|---|---|
| [c4-context.md](./c4-context.md) | C4 Level 1 — system in its environment |
| [c4-container.md](./c4-container.md) | C4 Level 2 — runtime containers and protocols |
| [c4-component-mobile.md](./c4-component-mobile.md) | C4 Level 3 — Flutter app internal components |
| [c4-component-backend.md](./c4-component-backend.md) | C4 Level 3 — FastAPI service internal components |
| [tech-stack.md](./tech-stack.md) | Authoritative dependency list with versions |
| [non-functional-requirements.md](./non-functional-requirements.md) | NFRs (performance, reliability, security, usability) |
| [sequence-flows.md](./sequence-flows.md) | Sequence diagrams for critical flows |
| [deployment.md](./deployment.md) | Environments and deployment topology |

## Reading guide

1. Start with **c4-context.md** to understand the system boundaries.
2. Then **c4-container.md** for the runtime layout.
3. Then your role-specific component file (mobile or backend).
4. Cross-reference **tech-stack.md** for versions and **non-functional-requirements.md** for constraints.
5. Use **sequence-flows.md** when implementing a specific user flow.
6. Use **deployment.md** when working on infra or release.

## Architectural style

- **Mobile:** layer-first Flutter project, offline-first, durable-write-first.
- **Backend:** FastAPI with Celery workers, Redis broker, PostgreSQL (FastAPI-side metadata only).
- **ERP:** Odoo accessed only via FastAPI through Odoo JSON-RPC.
- **Auth:** Supabase Edge Functions handle OTP issuance via Twilio. Mobile receives a Supabase JWT plus an internal session token from FastAPI.
- **Sync:** Eventual consistency. Mobile's local SQLite is the durable record of intent. Idempotency by `client_id` (UUID v4).

## Key constraints (from non-negotiable principles)

- Mobile **never** calls Odoo directly. (ADR-005)
- Local SQLite is the **first** durable write. (ADR-003)
- Every sync envelope carries `client_id`. (ADR-007)
- All evidence (photo, voice) requires explicit per-action consent.

See ADRs in `../adr/` for the rationale behind each decision.
