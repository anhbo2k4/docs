# Definitions: DoR, DoD, WIP

**Status:** Active  
**Owner:** Project Manager + QA Lead

## Definition of Ready (DoR)

A story is **Ready** to enter a sprint when:

- [ ] Story has an ID following the convention.
- [ ] Front matter complete: Epic, Sprint (target), Priority, Estimate, Status, Owner.
- [ ] User story sentence in `As a / I want / so that` form.
- [ ] Acceptance criteria numbered and measurable (each is a clear pass/fail).
- [ ] Dependencies resolved or scheduled (other stories, contracts, decisions).
- [ ] FR mapping: which `FR-NNN` does this serve.
- [ ] Test case mapping: which `TC-NNN` will verify.
- [ ] Estimate is `S/M/L`. `XL` must be split.
- [ ] No unresolved `DEC-NNN` blocks the story.
- [ ] Designs (if UI) approved by Designer.
- [ ] API contract changes (if any) merged to `api-contracts/`.

If a story enters a sprint without meeting DoR, the sprint capacity does not count it.

## Definition of Done (DoD)

A story is **Done** when:

- [ ] Code merged to main behind feature flag if cross-cutting.
- [ ] All AC pass on at least one device per OS (iOS + Android).
- [ ] Unit tests cover new domain logic (≥ 70 % line coverage in mobile, ≥ 80 % in backend).
- [ ] Integration / widget tests cover happy path + key error paths.
- [ ] Linked test cases (`TC-NNN`) executed and passing.
- [ ] Logs and metrics added where applicable.
- [ ] No P0 / P1 security findings.
- [ ] Docs updated: this blueprint, code comments, and `CHANGELOG.md`.
- [ ] Story status set to `Done` with link to merged PR.
- [ ] Demo'd at sprint review or asynchronously to PO.

## Definition of Done — Sprint

A sprint is **Done** when:

- [ ] All committed P0 stories pass DoD.
- [ ] Sprint review held with PO, EL, QA, Stakeholder.
- [ ] Sprint retro held; action items recorded.
- [ ] Burndown updated.
- [ ] Backlog refined for next sprint (≥ 70 % of next-sprint capacity is Ready).
- [ ] Risks updated in `backlog/risk-register.md`.
- [ ] CHANGELOG bumped.

## Definition of Done — Pilot release

See `qa/pilot-gate.md`.

## WIP limits (per engineer)

| Role | Max in-progress stories |
|---|---|
| Mobile engineer | 1 story `In Progress` + 1 in `In Review` |
| Backend engineer | 1 + 1 |
| QA engineer | 2 stories `In Test` |
| Mobile Lead / Backend Lead | 1 `In Progress` + reviews |

When WIP is exceeded, the oldest story takes precedence; new work blocks until the older one moves.

## Sync state contract (mobile-wide)

- A capture row's `sync_state` field is one of: `PENDING`, `SYNCING`, `CONFIRMED`, `FAILED`, `DEAD_LETTER`.
- UI must show the badge on every capture screen and on the Sync Center.
- Transitions are logged.
- See `data/sync-state-machine.md` for the formal automaton.

## Branch and PR conventions

- Branch: `feat/US-AUTH-001-otp-request` or `fix/SEV2-sync-race`.
- PR title: `[US-AUTH-001] OTP request screen with rate limit messaging`.
- PR body: link to story, what changed, how tested, screenshots / videos for UI.
- Required reviewers per `governance/raci.md`.
- CI must be green before merge.
