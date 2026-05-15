# Onboarding Plan — Day 1 / 7 / 30

**Status:** Active
**Owner:** Engineering Lead + Project Manager
**Audience:** new engineers, QAs, designers, PMs joining mid-program; new AI agents adopting the blueprint.

This is the structured ramp-up. Every new person follows the same checklist; AI agents reuse the **Day 1** checklist on every fresh task.

---

## Goals

- Day 1: read enough to be safe; know who decides what; have a working dev environment.
- Day 7: shipped at least one P2 or doc PR; understands one full vertical slice (capture → sync → Odoo).
- Day 30: owns at least one story end-to-end; participates in incident drill; gives feedback on the blueprint.

---

## Day 1 — orient and set up

### Reading order (90 minutes)

1. `README.md` — full read.
2. `WORKING.md` — current focus + open blockers.
3. `AGENTS.md` — routing rules.
4. `traceability/glossary.md` — terms.
5. `governance/raci.md` — who decides.
6. `architecture/c4-context.md` and `architecture/c4-container.md` — system shape.
7. `engineering/coding-standards.md` and `engineering/git-workflow.md`.

### Setup (½ day)

| Task | Owner | Done when |
|---|---|---|
| Repo access (read + write to feature branches) | DevOps | `git clone` works |
| 1Password vault membership | DevOps | Demo account credentials visible |
| Slack channels | PM | Joined #field-program, #field-builds, #field-incidents |
| Local dev environment | Self, with buddy | `make doctor` (or `flutter doctor` + backend smoke) prints green; per `engineering/dev-environment.md` |
| Pilot device handout | Mobile Lead | Has 1 Android + 1 iOS test device, both side-loaded with latest build |
| Pager rotation acknowledgment (engineers) | Engineering Lead | Read `engineering/oncall.md` and `runbooks/incident-response.md` |
| Mentor pairing | PM | Buddy assigned for Day 1–14 |

### First commit

Pick one from the **Good first PRs** list (kept in pinned issue `gfi-board`). Tasks are typically:

- Story typo fix
- Mermaid diagram improvement
- Adding a missing TC mapping in `traceability/QA-acceptance-mapping.md`
- Adding a `Notes` row in `backlog/raid-log.md`

PR must follow `engineering/code-review-checklist.md`.

### Definition of "Day 1 complete"

- [ ] Local environment runs `flutter test` and `pytest -m "not integration"` green.
- [ ] One PR opened (even doc-only).
- [ ] Stand-up time noted; first stand-up attended.
- [ ] Knows the on-call rotation and how to escalate (`governance/escalation.md`).

---

## Day 7 — first vertical slice

### Goal

Ship one story end-to-end OR demo a deep understanding of one vertical slice (sync) by walking the on-call through it.

### Pick one slice

| Slice | Path |
|---|---|
| Auth | `epics/EPIC-02-auth-employee-mapping/` → `security/auth-flow.md` → `api-contracts/auth.md` → run TC-AUTH-001..010 manually |
| Capture | `epics/EPIC-05-field-capture/` → `data/sqlite-schema.md` → run TC-PPE / TC-FORM / TC-GPS manually |
| Sync | `epics/EPIC-06-sync-engine/` → `data/sync-state-machine.md` → walk the envelope under TC-SYNC-001..025 |
| Odoo | `epics/EPIC-08-odoo-integration/` → `data/odoo-mapping.md` → run TC-ODOO-001..010 with mock |

### Activities

- Pair on a real story for 2–3 days.
- Read all 9 ADRs (`adr/`). Take notes on anything unclear; raise it in your buddy 1:1.
- Attend backlog refinement; expand one story per `governance/story-expansion-plan.md`.
- Shadow on-call for one day.
- Submit at least one PR with passing CI; merge before end of week.

### Definition of "Day 7 complete"

- [ ] One non-trivial PR merged.
- [ ] One story expanded (story-expansion plan).
- [ ] Walked through one slice with the buddy.
- [ ] Raised at least one improvement to the blueprint (open `nfr-debt` or `feature-request` issue).

---

## Day 30 — own a story and ship it

### Goal

Carry a story from `Ready` to `Done` per Definition of Done. Participate in a sprint review demo.

### Activities

| Activity | Reference |
|---|---|
| Pick a story marked `Ready` for the current sprint, sized M | `sprints/sprint-XX/README.md` |
| Implement, write tests, raise PR, address review | `engineering/code-review-checklist.md` |
| Demo at sprint review | `governance/sprint-ceremonies.md` |
| Participate in one incident drill (table-top or live) | `runbooks/incident-response.md` |
| Submit a self-critique entry: what was unclear in the blueprint when you joined? | `governance/self-critique.md` |
| (Optional) Draft an ADR if you've found an architectural gap | `adr/_template.md` |

### Definition of "Day 30 complete"

- [ ] Owns at least 1 merged story (`US-XXX-NNN`).
- [ ] Participated in 1 incident drill or postmortem.
- [ ] Filed at least one self-critique entry.
- [ ] Buddy + Engineering Lead 1:1 retro completed.

---

## For AI agents (refresh on every task)

AI agents do not have a "30 days" — they have a per-task ramp. The Day-1 reading list is the per-task checklist. Specifically:

1. Read `WORKING.md` (current state) → `AGENTS.md` (routing) → glossary.
2. Read the assigned story in full + linked epic + linked sprint.
3. Read each referenced ADR.
4. Check `traceability/decision-log.md` for any DEC-NNN cited as a blocker.
5. Only then start coding.

Skipping any of these is a red flag in code review.

---

## Ramp-up red flags

If any of these are true, the new joiner needs intervention:

- After 5 days, `make doctor` still fails.
- After 7 days, no PR opened.
- After 14 days, no story claimed.
- After 30 days, no demo at sprint review.

The buddy raises the flag in 1:1 with PM + Engineering Lead.

---

## Cross-references

- Dev environment: `engineering/dev-environment.md`
- Coding standards: `engineering/coding-standards.md`
- Stand-up + ceremonies: `governance/sprint-ceremonies.md`
- RACI: `governance/raci.md`
- Escalation: `governance/escalation.md`
- On-call: `engineering/oncall.md`
- Self-critique log: `governance/self-critique.md`
