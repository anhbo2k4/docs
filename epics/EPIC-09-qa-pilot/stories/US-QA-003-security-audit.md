---
id: US-QA-003
epic: EPIC-09
sprint: S10
fr: []
priority: P0
estimate: L
status: Ready
owner: Security Champion
---

# US-QA-003 — Security audit + dependency scan

## Acceptance criteria

1. STRIDE walkthrough vs `security/threat-model.md`; new findings logged.
2. Dependency scan in CI is green; no high/critical vulnerabilities outstanding.
3. Secret scan over the full repo passes; no plaintext secrets in history (TC-SEC-001).
4. Pen-test ticket raised if scope justifies (decision recorded with sponsor).
5. PII flow review: confirm logs, Sentry, and outbox audit do not leak phone or location data.

## Tasks

- [ ] STRIDE workshop.
- [ ] Run vuln scan + SBOM export.
- [ ] PII spot-check on logs / Sentry / DB.
- [ ] Update threat model with closed mitigations.

## FR mapping

Cross-cutting.

## Test cases

TC-SEC-001..011.

## DoD

- Findings list closed or accepted with mitigations.
- Updated threat model committed.
