# Capacity & Velocity Plan

**Status:** Active
**Owner:** Project Manager + Engineering Lead
**Last updated:** 2026-05-13

This plan ties team capacity to the 11-sprint roadmap (`backlog/release-plan.md`) and gives the PM a single place to compute velocity, plan headroom, and flag overcommit risks.

---

## 1. Team baseline (MVP team, sprints S0–S10)

| Role | FTE | Focus areas |
|---|---|---|
| Engineering Lead | 1.0 | Architecture, ADRs, reviews, on-call rotation owner |
| Mobile Lead | 1.0 | Mobile delivery, sync engine, ADRs ADR-001/006/007/009 |
| Mobile Engineer A | 1.0 | Auth + capture |
| Mobile Engineer B | 1.0 | Sync + offline persistence |
| Backend Engineer | 1.0 | FastAPI, Celery, Odoo bridge |
| QA Engineer | 1.0 | Test cases, automation, pilot gate |
| Designer | 0.5 | UI/UX, accessibility checks |
| Project Manager / PO | 1.0 | Backlog, decisions, stakeholders |
| Odoo Specialist | 0.4 | DEC-001..004; review of Odoo writes |
| DevOps | 0.4 | CI, release pipeline, infra |
| Security Champion | 0.2 | Threat model, PII reviews, on-call rotation |

Effective build capacity: ~6.5 FTE.

---

## 2. Sprint capacity formula

Per 2-week sprint, per FTE:

```
working_days  = 10
focus_factor  = 0.8   # meetings, reviews, ceremonies
velocity_pts  = working_days * focus_factor * pts_per_day
              = 10 * 0.8 * 1.5  ≈ 12 points/FTE/sprint
```

Total team velocity target: **6.5 × 12 ≈ 78 points/sprint** at steady state.

Story-point scale (Fibonacci):

| Points | Sized for | Rough effort |
|---|---|---|
| 1 | XS / typo / config tweak | < 0.5 day |
| 2 | S | 0.5–1 day |
| 3 | S–M | 1–2 days |
| 5 | M | 2–3 days |
| 8 | L (must split if possible) | 3–5 days |
| 13 | XL | mandatory split |

A story sized 13 cannot enter a sprint per Definition of Ready.

---

## 3. Per-sprint capacity reservation

| Sprint | Theme | Focus split (mobile / backend / qa / discovery) | Notes |
|---|---|---|---|
| S0 | Discovery | 0 / 10 / 10 / 80 % | All capacity on Phase 0 decisions; no implementation work |
| S1 | Foundation | 50 / 30 / 10 / 10 % | Bootstraps both mobile + backend skeletons |
| S2 | Auth + employee context | 55 / 30 / 15 / 0 | Tight contract between mobile + Edge + backend |
| S3 | Shifts read | 60 / 25 / 15 / 0 | Mobile-heavy; Odoo specialist reviews mapping |
| S4 | SQLite foundation | 75 / 10 / 15 / 0 | Mobile-heavy; backend pulls SQLite shape into FastAPI mirror |
| S5 | Core capture (PPE / form / GPS) | 65 / 20 / 15 / 0 | UI-heavy with backend stubs |
| S6 | Evidence (photo + voice) | 60 / 25 / 15 / 0 | Whisper + media; performance budget tight |
| S7 | Mobile sync engine | 60 / 25 / 15 / 0 | Most complex sprint; protect from feature creep |
| S8 | Backend sync intake | 25 / 60 / 15 / 0 | Backend-heavy; mobile catches up on hardening |
| S9 | Odoo write path | 15 / 65 / 20 / 0 | Backend + Odoo specialist heavy; QA full pilot prep |
| S10 | Hardening + pilot | 35 / 30 / 35 / 0 | Stabilisation; lots of test, dashboards, runbook drills |

---

## 4. Headroom and reserve

Each sprint reserves capacity for non-feature work:

| Reserve | % of sprint | Used for |
|---|---|---|
| Bug bash + reactive | 10 % | Defects, escaped bugs |
| Tech debt / NFR debt | 10 % | Items from `backlog/nfr-register.md`, `backlog/raid-log.md` |
| Documentation | 5 % | Story expansion, blueprint upkeep |
| Spike / discovery | 5 % | Open questions in stories |

Net feature delivery: ~70 % of nominal velocity.

---

## 5. Velocity tracking

- Updated by PM at sprint review.
- Tracked in `backlog/raid-log.md` (sub-section `Velocity history`) and a dashboard.
- Use rolling average of last 3 sprints to forecast next sprint.

| Sprint | Planned pts | Done pts | Carry pts | Flaky CI cost | Notes |
|---|---|---|---|---|---|
| S0 | n/a (discovery) | n/a | n/a | n/a | |
| S1 | 70 | TBD | TBD | TBD | First implementation sprint |
| ... | | | | | |

A sprint with > 20 % carry over is a red flag → PM raises in retro.

---

## 6. Risk-based capacity adjustments

| Trigger | Adjustment |
|---|---|
| 2+ P0 incidents in a sprint | Halt new story intake; capacity goes to fixes |
| Velocity drops > 25 % vs rolling 3-sprint avg | Replan in retro; check WIP and external blockers |
| Pilot gate (`qa/pilot-gate.md`) flags red | Reserve 50 % of next sprint for hardening |
| New joiner (Day 0–14) | Treat as 0.4 FTE for 2 sprints, then 1.0 |

---

## 7. WIP limits

- Per engineer: at most 1 story `In Progress` and 1 `In Review`.
- Per sprint: at most `headcount + 2` stories `In Progress` simultaneously.

Exceeding WIP requires PM approval and a note in the daily stand-up.

---

## 8. Stop-the-line conditions

The team stops new story work when any of:

- A P0 incident is open longer than 4 h.
- The pilot dashboard breaches more than 1 NFR concurrently.
- CI red on `main` for > 2 h.

Recovery is the only WIP until cleared.

---

## 9. Cross-references

- Release plan: `backlog/release-plan.md`
- Risk register: `backlog/risk-register.md`
- RAID log: `backlog/raid-log.md`
- NFR register: `backlog/nfr-register.md`
- Definition of Ready / Done: `governance/definition-of-ready.md`, `governance/definition-of-done.md`
- Sprint ceremonies: `governance/sprint-ceremonies.md`
- On-call: `engineering/oncall.md`
