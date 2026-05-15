# Release Plan

**Status:** Active  
**Owner:** Project Manager + Engineering Lead

Phased rollout strategy from build to general availability.

## Stages

```
Stage 0  Discovery (S0)               → Phase 0 decisions closed
Stage 1  Internal Alpha (S1-S5)       → Team-only builds, daily Internal Testing track
Stage 2  Closed Beta (S6-S9)          → 5-10 internal stakeholders, controlled environment
Stage 3  Pilot Soak (S10)             → 50 real users at one site, 2-week soak
Stage 4  Pilot Expansion (post-S10)   → 200 users across 2-3 sites, 4 weeks
Stage 5  GA Phased (post-pilot)       → 5% → 25% → 50% → 100% over 5 days
Stage 6  Steady State                 → Monthly releases, hotfix as needed
```

## Stage entry criteria

### Stage 1 — Internal Alpha
- [ ] S1 sprint complete (foundation + scaffold).
- [ ] Mobile builds signed and uploaded to TestFlight + Play Internal.
- [ ] Backend deployed to staging.
- [ ] Smoke test passes.

### Stage 2 — Closed Beta
- [ ] S5 functional capture working end-to-end (PPE, forms, GPS).
- [ ] Backend ingest + Postgres dedup live.
- [ ] At least 5 stakeholders on internal Build.
- [ ] Crash-free ≥ 99 % over 7 days.

### Stage 3 — Pilot Soak (THE big gate)
- [ ] Pilot Gate (`qa/pilot-gate.md`) all green.
- [ ] All P0 FRs working on at least 1 iOS + 1 Android device.
- [ ] 50 pilot users invited, devices enrolled.
- [ ] Pilot supervisor trained.
- [ ] Rollback runbook tested.
- [ ] Incident response on-call rotation live.

### Stage 4 — Pilot Expansion
- [ ] Stage 3 ran 2 weeks with no SEV-1.
- [ ] Sync success rate ≥ 99 % over the 2 weeks.
- [ ] PO + Stakeholder approval to expand.
- [ ] Capacity plan refreshed for new user count.

### Stage 5 — GA Phased
- [ ] Stage 4 ran 4 weeks.
- [ ] Pilot retro held.
- [ ] Compliance sign-off.
- [ ] Sponsor approval.

### Stage 6 — Steady State
- [ ] One full release in production with no rollback.
- [ ] Operational runbooks all tested.
- [ ] DR drill completed.

## Rollout cadence (Stage 5 → 6)

| Day | Cohort | Action |
|---|---|---|
| Day 0 | 5 % | Phased Release on App Store; Staged rollout 5 % on Play |
| Day 1 | 5 % | Monitor; halt if crash-free < 99 % |
| Day 2 | 25 % | Promote |
| Day 3 | 25 % | Monitor |
| Day 4 | 50 % | Promote |
| Day 5 | 100 % | Promote |

Halt criteria at any step:
- Crash-free sessions < 99 %.
- Sync success rate < 95 % over 1 hour.
- Two SEV-2 within 24 hours.

## Backend release cadence

- Minor releases: every 2 weeks (post-sprint).
- Patches: as needed.
- Coordinated with mobile minor when API changes.
- Backwards compatibility: backend supports current and previous mobile minor for 30 days.

## Communication

| Audience | When | Channel |
|---|---|---|
| Pilot users | T-1 day | In-app banner + email |
| Stakeholders | T-3 days | Email + Slack `#field-app-stakeholders` |
| Steering | At each stage gate | Steering meeting |
| Sponsor | Before Stage 4 / Stage 5 | Direct |

## Rollback authority per stage

| Stage | Authority |
|---|---|
| 1, 2 | EL or on-call |
| 3 (Pilot) | EL + PM |
| 4 | EL + PM + PO |
| 5, 6 | Sponsor + EL + PM |

## Backout windows

If we need to pull back:

- Stage 1, 2: instant via TestFlight / Play Internal.
- Stage 3, 4: announce to pilot users; new app version pushed; old version `min_app_version` enforced.
- Stage 5, 6: phased rollback; force-update in worst case.

## Post-release reviews

- After each stage gate, hold a brief retro.
- Capture in `backlog/release-retros/STAGE-X-YYYYMMDD.md`.
- Adjust this plan if recurring issues surface.

## Out of scope for MVP

- Push notifications (post-MVP, EPIC-10).
- Admin web portal (post-MVP).
- Video capture (post-MVP).
- Dynamic forms (post-MVP).
- Activity log / audit UI (post-MVP).
- Billing integration (post-MVP).
- Team list / chat (post-MVP).
