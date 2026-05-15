# Release v<MAJOR>.<MINOR>.<PATCH> — <YYYY-MM-DD>

**Type:** <Mobile | Backend | Both>
**Track:** <internal | TestFlight | Play Internal | pilot | prod>
**Risk:** <low | medium | high>
**Engineering Lead:** <name>
**On-call:** <name + pager>

## Highlights (for stakeholders)

- 

## What's new (user-facing)

- FR-XXX — <one-line>
- ...

## What's changed (engineering)

### Mobile

- 

### Backend / Edge / Odoo

- 

## Migrations

- DB: <none | reversible | one-way>
- Mobile schema bump: <vN → vN+1, see `data/sqlite-schema.md`>
- API contract: <no break | versioned bump>

## Breaking changes

- <none | list with mitigation>

## Risk and rollback

- Risk areas: 
- Rollback path: <runbooks/rollback.md anchor>
- Kill-switch flag(s): 
- `min_app_version` enforced: <yes/no>

## Test summary

- TC pass count: 
- Coverage delta vs main: 
- Soak / chaos: <pass | partial | n/a>
- Manual smoke matrix: <devices tested>

## Telemetry to watch (24 h)

- Crash-free sessions ≥ 99.5 %
- Sync success ≥ 99 % within 1 h
- 5xx rate < 1 %
- p95 latency within budget (`engineering/performance-budget.md`)

## Comms

- Stakeholder note: <link>
- In-app changelog snippet: <link>

## Stories closed

- US-XXX-NNN
- ...

## Known issues

- <ID> — <one-liner + workaround>

## Sign-off

- Sponsor: <name>
- PO: <name>
- EL: <name>
- QA: <name>
- DevOps / SRE: <name>
