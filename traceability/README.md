# Traceability Layer

**Status:** Active  
**Owner:** Engineering Lead + QA Lead

Cross-cutting layer that links FRs, NFRs, decisions, glossary, and acceptance mapping. Update this layer whenever the matrix relationships change.

## Files

| File | Purpose |
|---|---|
| [glossary.md](./glossary.md) | Authoritative term definitions |
| [definitions.md](./definitions.md) | DoR, DoD, WIP limits, branch / PR rules |
| [FR-matrix.md](./FR-matrix.md) | FR ↔ Epic ↔ Sprint ↔ Story ↔ TC matrix |
| [QA-acceptance-mapping.md](./QA-acceptance-mapping.md) | Test areas, severity policy, test pyramid |
| [decision-log.md](./decision-log.md) | Phase 0 + ongoing decisions |
| [open-questions.md](./open-questions.md) | Pre-DEC question tracker |

## How traceability works

```
FR-NNN
  ├─ implemented by → Epic-NN
  │                    └─ scheduled in → Sprint-NN
  │                                       └─ delivered by → US-XXX-NNN
  └─ verified by → TC-AREA-NNN
                    └─ executed in → qa/test-cases/
```

Every story file's front matter must list `FR`, `Epic`, `Sprint`, and `TestCases`. CI script (planned for S1) regenerates the FR matrix from front matter.

## NFRs

Tracked separately in `architecture/non-functional-requirements.md` and `backlog/nfr-register.md`. Each NFR has at least one TC.

## Decisions

Phase 0 decisions block sprint entry. ADRs promote a decision to architectural status. Both link from story "Open Questions" sections.
