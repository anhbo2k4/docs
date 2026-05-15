# Code review checklist

**Status:** Active
**Owner:** Engineering Lead

Reviewers must satisfy this checklist before approving. Authors should self-review against it before requesting review.

## PR description template

```
### What
One sentence summary.

### Why
Story link (US-XXX-NNN), problem statement, or bug ticket.

### Scope
- Files touched: ...
- Behaviour change: ...
- Migrations: yes/no
- API contract change: yes/no (if yes, link contract diff)

### How tested
- Unit: ...
- Widget / integration: ...
- Manual: device(s), OS version, scenario.

### Risk
Low / Medium / High + rationale.

### Rollback plan
How to revert if this fails in production.

### Screenshots / videos
For UI changes, attach.
```

## Reviewer checks

### Correctness

- [ ] Code matches the story's acceptance criteria.
- [ ] Edge cases covered (offline, error 4xx/5xx, empty state, unauthorized).
- [ ] Idempotency preserved on any sync-related change.
- [ ] No regressions in existing tests.

### Architecture

- [ ] Fits the layering documented in `engineering/coding-standards.md`.
- [ ] Does not bypass FastAPI to call Odoo from mobile (ADR-005).
- [ ] No new third-party dependency without ADR.
- [ ] If introducing a public API or breaking change, contract updated.

### Security

- [ ] No PII logged, no secrets in commits.
- [ ] Authz check present where required.
- [ ] Input validated at the boundary.
- [ ] If touching auth or crypto, security champion tagged.

### Performance

- [ ] No accidental N+1 or unbounded loops.
- [ ] Hot paths avoid synchronous I/O.
- [ ] Mobile rebuild scope minimised.

### Testability

- [ ] New logic has tests (unit + widget/integration as relevant).
- [ ] Tests fail before fix, pass after.
- [ ] Flakiness mitigated (no `sleep`, no timing-dependent assertions).

### Observability

- [ ] Logs cover the failure modes.
- [ ] Metrics added when business outcome changes (e.g. envelope status transitions).
- [ ] Sentry breadcrumbs preserve causal info without PII.

### Documentation

- [ ] Story updated (status, link to PR).
- [ ] CHANGELOG bumped if user-visible.
- [ ] Code comments where the why is non-obvious.
- [ ] If config or env var added, `.env.example` updated.

### Release

- [ ] Feature flag in place for non-trivial flows.
- [ ] Backward compatible with the last shipped client (or contract bumped intentionally).
- [ ] Rollback plan tested mentally; risky migrations have a roll-forward path.

## When to escalate

- Authz or crypto change → security champion approval required.
- Schema migration → backend lead approval required.
- ADR-level architectural change → ADR drafted before merge.
- Breaking API change → contract updated and mobile + backend reviewers approve.
