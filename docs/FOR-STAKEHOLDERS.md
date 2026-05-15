# Project Overview — For Stakeholders

> Non-technical summary of what we're building, when it ships, and what you need to know.

---

## What is this?

A **mobile app** (iOS + Android) for field workers and contractors to:

- Clock in/out of shifts with GPS verification
- Complete safety checklists (PPE, pre-shift forms)
- Capture photo evidence and voice notes on-site
- Work **offline** — everything syncs automatically when back online
- Push all data into your existing **Odoo ERP** system

No more paper forms. No more manual data entry. Workers use their phone, data flows to Odoo.

---

## How it works (simplified)

```
┌─────────────┐       ┌─────────────┐       ┌─────────┐
│  Mobile App │ ───── │  Our Server │ ───── │  Odoo   │
│  (offline)  │  sync │  (FastAPI)  │  push │  (ERP)  │
└─────────────┘       └─────────────┘       └─────────┘
```

- App works **without internet** — saves everything locally first
- When connected, syncs to our server automatically
- Server pushes verified data into Odoo — no manual step

---

## Timeline

| Phase | Duration | What happens |
|-------|----------|--------------|
| Discovery (S0) | 2 weeks | Requirements confirmed ✅ Done |
| Foundation (S1–S2) | 4 weeks | App skeleton + login system |
| Core Features (S3–S6) | 8 weeks | Shifts, forms, photos, voice notes |
| Sync + Integration (S7–S9) | 6 weeks | Offline sync + Odoo connection |
| Hardening + Pilot (S10) | 2 weeks | Testing, security, pilot launch |
| **Total** | **~22 weeks** | **MVP ready for pilot** |

Current status: **Sprint 1 ready to start** (all decisions finalized).

---

## What you (the customer) need to provide

1. **Odoo access** — service account + API key for your Odoo instance
2. **Custom fields setup** — ~30 min one-time setup in Odoo Studio (we provide instructions)
3. **Employee phone numbers** — must be in your Odoo HR module (we auto-match workers by phone)
4. **Pilot group** — 5–10 field workers for initial testing

---

## Key decisions already made

| Decision | What it means for you |
|----------|----------------------|
| Works with any Odoo (Online/Enterprise/Community) | No vendor lock-in |
| No custom Odoo module needed | Nothing to install on your Odoo server |
| DigitalOcean hosting (Singapore) | Fast for APAC, ~$59/month for pilot |
| SMS login (OTP) | Workers don't need passwords |
| GPS + photo evidence | Verifiable proof of on-site work |
| 180-day GPS data retention | Compliant, auto-deleted after |

---

## What it costs

| Tier | Team | Timeline | Estimate |
|------|------|----------|----------|
| Lean MVP | 3–4 people | 10–14 weeks | $40k–90k |
| Standard MVP | 4–6 people | 14–22 weeks | $90k–180k |
| Enterprise Pilot | 6–8 people | 22–36 weeks | $180k–350k+ |

*These are non-binding brackets. Final scope determines final cost.*

---

## Your involvement

| When | What you do | Time needed |
|------|-------------|-------------|
| Sprint reviews (every 2 weeks) | See demo, give feedback | 30 min |
| Odoo setup (once) | Create custom fields per our guide | 30 min |
| Pilot launch | Nominate test users, observe | 1–2 hours |
| Go/No-go decision | Approve production rollout | 15 min |

---

## Questions?

- Full technical details: [docs/FULL-BLUEPRINT.md](./FULL-BLUEPRINT.md)
- Release plan: [backlog/release-plan.md](../backlog/release-plan.md)
- Risk register: [backlog/risk-register.md](../backlog/risk-register.md)

---

*Last updated: 2026-05-15 | Blueprint v2.1.0*
