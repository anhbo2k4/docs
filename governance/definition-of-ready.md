# Definition of Ready

**Status:** Active
**Owner:** Project Manager + Product Owner

A story is **Ready** to enter a sprint when every box below is checked. PM confirms during refinement; EL confirms during planning.

## Front matter

- [ ] Story ID follows convention (`US-<area>-NNN`).
- [ ] Epic, target Sprint, Priority, Estimate (S/M/L), Status, Owner are set.
- [ ] Linked `FR-NNN` and at least one `TC-<area>-NNN`.

## Body

- [ ] User story sentence in `As a / I want / so that` form.
- [ ] Acceptance criteria are numbered, measurable, each individually pass/fail.
- [ ] Non-functional considerations called out (perf budgets, accessibility, security).
- [ ] Test plan section lists the TCs that will verify it.
- [ ] Dependencies on other stories listed and scheduled.
- [ ] References to relevant ADRs, data schemas, API contracts, runbooks.

## Pre-conditions

- [ ] No unresolved `DEC-NNN` blocks the story.
- [ ] If UI: design approved (or noted as design-light) by Designer.
- [ ] If API: contract diff merged in `api-contracts/`.
- [ ] If data: schema migration drafted in `data/`.
- [ ] If cross-cutting: pre-mortem completed (per agents-system / `protocols/pre-mortem.md`).

## Sizing

- [ ] Estimate is `S` (≤1 d), `M` (1–3 d), or `L` (3–5 d).
- [ ] `XL` is forbidden; split before entering a sprint.

## Out-of-scope

A story stating "and we'll see if X also fits" is not Ready. Split.

## Anti-patterns

- "TBD" in AC.
- AC that requires reading the conversation thread.
- "Make it work like the legacy app." Specify behaviour explicitly.
- AC that mixes UI and behaviour: split or be explicit which lines test which.
