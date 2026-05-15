# Code owners + PR / issue templates

**Status:** Active
**Owner:** Engineering Lead

## CODEOWNERS

The repository's `.github/CODEOWNERS` file is generated from this table. Update both together.

| Path | Owners |
|---|---|
| `mobile/lib/@core/sync/**` | @mobile-lead @backend-lead |
| `mobile/lib/@core/db/**` | @mobile-lead |
| `mobile/lib/screen/**` | @mobile-lead @designer |
| `mobile/lib/l10n/**` | @mobile-lead @designer |
| `backend/app/auth/**` | @backend-lead @security-champion |
| `backend/app/sync/**` | @backend-lead |
| `backend/app/odoo/**` | @backend-lead @odoo-specialist |
| `edge/**` | @backend-lead @security-champion |
| `infra/odoo/field_mobile_sync/**` | @odoo-specialist @backend-lead |
| `infra/terraform/**` | @devops |
| `infra/ci/**` | @devops |
| `field_work_delivery/adr/**` | @engineering-lead |
| `field_work_delivery/architecture/**` | @engineering-lead |
| `field_work_delivery/security/**` | @security-champion @engineering-lead |
| `field_work_delivery/governance/**` | @project-manager @engineering-lead |
| `field_work_delivery/qa/**` | @qa-lead |

Anything matched by multiple lines requires approval from all owners.

## Pull request template

Saved at `.github/pull_request_template.md`. Reproduced here so it stays under change control.

```markdown
## Summary
<!-- What and why, 1–3 sentences. Link to story ID(s). -->

Story: US-XXX-NNN

## Changes
- 

## How tested
- [ ] Unit tests added / updated
- [ ] Integration test added / updated
- [ ] Manual: <device, OS, scenario>

## Risk / blast radius
- Areas affected:
- Reversible? <yes/no, how>

## Checklist
- [ ] All AC in the story pass
- [ ] No PII in logs (verified by test)
- [ ] No new secrets introduced
- [ ] Coverage on changed files ≥ threshold
- [ ] Updated docs in `field_work_delivery/` if behavior changed
- [ ] Ran `make lint` and `make test` locally
- [ ] CHANGELOG entry (if behavior visible to users / API)

## Screenshots / recordings (UI changes)

## Related
- ADR-XXX, FR-XXX, TC-XXX
```

## Issue templates

Saved at `.github/ISSUE_TEMPLATE/`.

### `bug_report.md`

```markdown
---
name: Bug report
about: Report a defect
labels: bug
---

**Summary**
<!-- One line. -->

**Steps to reproduce**
1. 
2. 
3. 

**Expected**
**Actual**

**Environment**
- App version + build:
- OS + device:
- Network condition:
- Account / employee id (if relevant):

**Logs / Sentry**
<!-- Sentry event link only; no raw PII. -->

**Severity** (SEV-1 / 2 / 3)
**Suspected area** (auth / sync / odoo / capture / ...)
```

### `nfr_debt.md`

```markdown
---
name: NFR debt
about: Tracks a deviation from a committed NFR or PB budget.
labels: nfr-debt
---

**Budget violated**
NFR-XXX or PB-X-XXX

**Observed value vs target**

**Impact / blast radius**

**Plan to recover**
<!-- Story split, or a path back to budget. -->

**Owner**
```

### `feature_request.md`

```markdown
---
name: Feature request
about: Propose a change to product or delivery
labels: feature-request
---

**Problem statement**
**User / role affected**
**Proposed change**
**Acceptance criteria (draft)**
**Affected FR / Epic / Sprint**
**Out of scope**
```

## Review SLAs

| Type | Max time to first review |
|---|---|
| Hotfix / pilot blocker | 1 h business hours |
| Standard PR | 1 business day |
| RFC / ADR draft | 3 business days |

## Conflict resolution

If owners disagree on a PR, the Engineering Lead decides. If an architectural conflict, escalate per `governance/escalation.md`.
