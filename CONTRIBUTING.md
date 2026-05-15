# Contributing to the Delivery Blueprint

This blueprint is a living document. Use these rules to keep it coherent and traceable.

## 1. Who can change what

| Area | Owner | Approval needed |
|---|---|---|
| `README.md`, `CHANGELOG.md` | Engineering Lead | PO + EL |
| `governance/` | Project Manager | Sponsor |
| `architecture/`, `adr/` | Engineering Lead | EL + Backend Lead + Mobile Lead |
| `epics/EPIC-XX/README.md` | Epic Owner | PO |
| `epics/EPIC-XX/stories/*.md` | Story Author (any contributor) | Epic Owner |
| `sprints/sprint-NN/` | Scrum Master | EL |
| `traceability/` | QA Lead | EL |
| `data/`, `api-contracts/` | Backend Lead | EL + Mobile Lead |
| `security/` | Security Champion | EL + Sponsor |
| `qa/` | QA Lead | EL |
| `runbooks/` | SRE / DevOps | EL |
| `backlog/` | PM | PO |

EL = Engineering Lead. PO = Product Owner. PM = Project Manager.

## 2. Change workflow

1. Open a branch named `docs/<area>-<short-slug>` (e.g. `docs/epic-05-add-voice-quality-story`).
2. Make the change. Update `CHANGELOG.md` with a short bullet under the next version.
3. If the change affects an ADR or a Phase 0 decision, also update `traceability/decision-log.md` and bump the ADR status.
4. Open a PR. The owner above is the required reviewer.
5. Reviewer checks:
   - Does it preserve traceability (FR ↔ Epic ↔ Sprint ↔ TC)?
   - Is the ID scheme respected?
   - Are links between files still correct?
   - Does it conflict with an existing ADR? If yes, supersede the ADR formally.
6. On merge, bump the document version in any file that has a `Version:` header.

## 3. Versioning

- Blueprint follows [SemVer](https://semver.org/).
  - **Patch** (1.0.x): typo, link fix, minor wording.
  - **Minor** (1.x.0): new story, new test case, new runbook section, new ADR.
  - **Major** (x.0.0): change to scope, change to a non-negotiable principle, supersede multiple ADRs at once.

## 4. ID conventions

- Epic: `EPIC-NN` (zero-padded, NN = 00–10).
- Story: `US-<EPIC-SHORT>-NNN` where `<EPIC-SHORT>` is a 3–5 letter code (e.g. `US-AUTH-001`, `US-CAP-014`).
- Sprint: `S-NN` (00–10).
- Test case: `TC-<AREA>-NNN`. Areas: `AUTH`, `SHIFT`, `GPS`, `PPE`, `FORM`, `MEDIA`, `VOICE`, `OFF`, `SYNC`, `ODOO`, `SEC`, `PERF`.
- Functional requirement: `FR-NNN`.
- Non-functional requirement: `NFR-NNN`.
- Decision (Phase 0 / open): `DEC-NNN`.
- ADR: `ADR-NNN`.
- Risk: `RISK-NNN`.

## 5. Story file checklist

Before marking a story `Ready`:

- [ ] Front matter has Epic, Sprint, Priority, Estimate, Status, Owner.
- [ ] User story sentence in `As a / I want / so that` form.
- [ ] Acceptance criteria (AC) numbered, each measurable.
- [ ] Tasks grouped by `Flutter UI` / `Backend` / `Integration` / `Testing`.
- [ ] Definition of Done (3–5 measurable bullets).
- [ ] Dependencies (other stories, decisions, contracts).
- [ ] Risks / Open Questions reference DEC-NNN if applicable.
- [ ] FR mapping (which FR-NNN this story serves).
- [ ] Test case mapping (which TC-NNN verify it).

## 6. ADR workflow

New ADRs use `adr/_template.md`. Status transitions:

```
Proposed → Accepted → (later) Deprecated | Superseded by ADR-NNN
```

Never edit an Accepted ADR's decision text. Supersede instead.

## 7. Definition of "Ready" (DoR) and "Done" (DoD)

See `traceability/definitions.md`. PRs that change DoR/DoD require Sponsor approval.

## 8. Style

- Prose: short sentences, active voice, present tense.
- Tables for any list with 3+ structured fields.
- Code blocks with language hint (` ```dart `, ` ```python `, ` ```sql `).
- Diagrams: Mermaid only (no images committed unless screenshot evidence).
- Filenames: kebab-case, ASCII only.
