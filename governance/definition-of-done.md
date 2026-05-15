# Definition of Done

**Status:** Active
**Owner:** QA Lead + Engineering Lead

This file is the canonical DoD. `traceability/definitions.md` summarises but defers here for the authoritative version.

## Story DoD

- [ ] Code merged to `main` via squash; PR linked from story.
- [ ] All AC pass on at least one device per OS (iOS + Android) for mobile work; on staging for backend.
- [ ] Unit tests added for new domain logic.
  - Mobile coverage ≥ 70 % on touched packages.
  - Backend coverage ≥ 80 % on touched packages.
- [ ] Widget / integration tests cover happy path + 1 error path minimum.
- [ ] All linked `TC-<area>-NNN` test cases executed and passing.
- [ ] No new P0/P1 security findings (lint, gitleaks, dependency audit).
- [ ] Logs and metrics updated where the story changes observable behaviour.
- [ ] Docs updated: relevant blueprint files (epic README, ADR, contract), code comments, CHANGELOG bullet.
- [ ] Story status set to `Done`; PR link recorded in front matter.
- [ ] Demoed at sprint review or recorded async to PO.

## Sprint DoD

- [ ] All committed P0 stories meet story DoD.
- [ ] Sprint Goal validated by PO.
- [ ] Burndown updated; carry-over reasons recorded.
- [ ] Risk register updated (`backlog/risk-register.md`).
- [ ] Retro action items captured.
- [ ] CHANGELOG bumped (sprint-end version).
- [ ] Pilot Health dashboard reviewed at sprint review.

## Release DoD (pilot)

See `qa/pilot-gate.md`. Highlights:

- Acceptance test pass rate ≥ 95 % across device matrix.
- Crash-free sessions ≥ 99 % over the last 7 days of staging.
- All P0 security findings resolved.
- Runbooks for top 5 risks present and rehearsed.
- On-call rotation staffed.
- Rollback rehearsed within last 7 days.
- Sign-off recorded from EL, PO, Sponsor, Security Champion, QA Lead.

## Documentation DoD

A change to this blueprint is Done when:

- [ ] Affected files updated.
- [ ] Cross-references checked (epics → stories, FR → TC, ADR → impacted areas).
- [ ] CHANGELOG entry added under the next version with `### Added/Changed/Removed`.
- [ ] Sponsor or EL approval obtained for governance / contract / ADR changes.
