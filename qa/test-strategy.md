# Test Strategy

**Status:** Active  
**Owner:** QA Lead

Defines how we test, what we test, who owns each layer, and how we report.

## Goals

- Catch regressions before merge.
- Ensure offline-first and idempotency promises hold under chaos.
- Keep mobile crash-free sessions ≥ 99.5 % during pilot.
- Provide a verifiable Pilot Gate.

## Pyramid

```
                    Manual / UAT          ~5 %
                    integration_test     ~10 %
              Widget tests / contract    ~25 %
        Unit tests (domain + services)   ~60 %
```

## Tooling

| Layer | Tool |
|---|---|
| Mobile unit | `flutter_test`, `mocktail` |
| Mobile widget | `flutter_test` (golden tests for critical screens) |
| Mobile integration | `integration_test` + `patrol` |
| Mobile DB | `sqflite_common_ffi` for in-memory SQLite |
| Backend unit | `pytest`, `respx` for HTTP mocks |
| Backend integration | `pytest` + `testcontainers` (Postgres) |
| Backend e2e | Spins up FastAPI + Postgres + fake Odoo + fake S3 |
| Contract | JSON Schemas in `api-contracts/` consumed by both sides |
| Performance | k6 against staging |
| Security | `gitleaks`, `bandit`, `safety`, `snyk` (post-MVP) |
| Accessibility | `flutter_test` semantics + manual screen reader |
| Device matrix | Firebase Test Lab + AWS Device Farm |

## Coverage targets

| Area | Target |
|---|---|
| Mobile `domain/` | ≥ 70 % line |
| Mobile `@core/` | ≥ 70 % line |
| Backend overall | ≥ 80 % line |
| Sync engine paths | 100 % branch |
| Auth paths | 100 % branch |

## Categories

### Functional tests
Cover every AC of every story. Linked via `TC-AREA-NNN` IDs in story files. Indexed in `qa/test-cases/`.

### Non-functional tests
- **Performance:** k6 scripts hit `/v1/sync/envelope` and `/v1/shifts` with realistic load. Pass when NFR-007, NFR-008 hit p95 targets.
- **Reliability / chaos:**
  - Force-kill mobile during SYNCING → confirm row recovers to FAILED.
  - Force 5xx storm → confirm DEAD_LETTER after 6 retries.
  - Power loss during photo capture → confirm SQLite consistent.
- **Security:** scanner suite + targeted tests (TC-SEC-NNN).
- **Privacy:** PII scrub assertions in unit tests.

### Manual / UAT
Pilot site supervisor + 5 field workers exercise full flows. Driven from `qa/uat-plan.md` (drafted at S9).

## Branch policy

- All tests must pass on the PR branch.
- Coverage must not drop more than 1 % from main.
- Any new public API needs at least one unit and one integration test.

## Test data

- Synthetic employees and shifts seeded via fixtures.
- No production PII used.
- Anonymized sample audio for Whisper benchmark.

## Reporting

- CI publishes per-PR coverage and test-count delta.
- Sprint review shows test count, pass rate, bug trend.
- Pilot Gate produces a single document with full TC pass matrix and bug list.

## Test environments

| Env | When | Data |
|---|---|---|
| Local | Engineer | Synthetic |
| CI | Per PR | Synthetic |
| Staging | Per merge | Sanitised |
| Pilot | Pre-release | Live (limited) |
| Prod | Smoke only | Live |

## Flaky test policy

- A test that fails non-deterministically is quarantined within 24 h.
- Quarantined tests do not block CI but must be fixed within 5 business days.
- A second occurrence after fix is treated as SEV-3.

## Owners

| Test type | Owner |
|---|---|
| Mobile unit / widget | Mobile engineers |
| Mobile integration | Mobile + QA |
| Backend unit / integration | Backend engineers |
| Backend e2e | Backend + QA |
| NFR / load | QA + DevOps |
| Security | QA + Security Champion |
| UAT | QA + PO + Stakeholders |
