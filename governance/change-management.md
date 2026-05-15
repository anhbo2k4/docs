# Change Management

**Status:** Active  
**Owner:** Project Manager

Any change to scope, architecture, or non-negotiable principles follows this process.

## What counts as a change

| Type | Examples |
|---|---|
| **Scope change** | Adding / removing an epic, story, or FR; changing pilot definition |
| **Architecture change** | New service, new dependency, change to a non-negotiable principle |
| **Schedule change** | Sprint reordering, timeline shift > 1 week |
| **Budget change** | > 5% variance |
| **ADR supersession** | Replacing or deprecating an Accepted ADR |

What does **not** count: clarifying wording, fixing typos, adding test cases, adding stories that don't change MVP scope, refining acceptance criteria within an existing story.

## Process

```
1. Submit Change Request (CR-NNN)
   ↓
2. PM logs in change-log.md, assigns owner
   ↓
3. Impact analysis (technical, schedule, budget, risk)
   ↓
4. Approval per RACI:
   - Scope:        PO + Sponsor
   - Architecture: EL (+ Sponsor if breaks principle)
   - Schedule:     PM + PO + Sponsor (if > 1 week)
   - Budget:       Sponsor
   ↓
5. Update affected docs:
   - CHANGELOG.md (always)
   - Relevant epic / sprint / ADR / decision log
   - traceability/FR-matrix.md if FR changed
   ↓
6. Communicate via weekly digest + Slack
```

## Change Request template

Create at `governance/change-requests/CR-NNN.md`:

```markdown
# CR-NNN: <Title>

**Submitted by:** <name>  
**Date:** YYYY-MM-DD  
**Type:** Scope | Architecture | Schedule | Budget | ADR
**Status:** Draft | Under Review | Approved | Rejected | Implemented

## Description
What is changing and why.

## Impact
- Technical: ...
- Schedule: ...
- Budget: ...
- Risk: ...

## Affected artifacts
- [ ] CHANGELOG.md
- [ ] EPIC-XX/...
- [ ] sprints/sprint-NN/...
- [ ] ADR-XXX
- [ ] traceability/...
- [ ] backlog/risk-register.md

## Approvers
- [ ] <role 1>
- [ ] <role 2>

## Decision
<after review>
```

## Change log

A running log lives at `governance/change-log.md`. Every CR (approved or rejected) is recorded with date, type, decision, and link.

## Freeze windows

- **Pilot freeze:** Last 3 days of S10. Only SEV-1 / SEV-2 fixes allowed.
- **Production release freeze:** 24 h before release. Only SEV-1 fixes allowed.
- **Holiday freezes:** Defined by org calendar, communicated 2 weeks in advance.
