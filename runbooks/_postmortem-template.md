# Postmortem — INC-YYYYMMDD-NN

**Status:** Draft / Final
**Author:** <name>
**Date of incident:** YYYY-MM-DD
**Severity:** SEV-1 / SEV-2 / SEV-3

## TL;DR

One paragraph: what failed, who was impacted, how we fixed it, what we changed.

## Impact

| Dimension | Detail |
|---|---|
| Users affected | <count or %> |
| Duration | <start> → <mitigated> → <resolved> |
| Data integrity | <none/recoverable/lost> |
| Compliance | <any regulatory implications> |

## Timeline (UTC)

- `HH:MM` first symptom detected (alert / user / monitor).
- `HH:MM` page acknowledged by <name>.
- `HH:MM` incident commander declared.
- `HH:MM` mitigation applied: <what>.
- `HH:MM` impact ceased.
- `HH:MM` root cause identified.
- `HH:MM` long-term fix landed.

## What happened

A factual narrative. No blame. Start before the incident, end after.

## Root cause

The technical and process root cause(s). Multiple causes are normal.

## Detection

- How was it detected (monitor, customer report, internal canary)?
- Could it have been detected sooner?

## Response

- What worked?
- What didn't?
- Where did communication break?

## Action items

| ID | Action | Owner | Severity | Due |
|---|---|---|---|---|
| AI-1 | <fix the cause> | <name> | High | YYYY-MM-DD |
| AI-2 | <add monitor> | <name> | Med | YYYY-MM-DD |
| AI-3 | <update runbook> | <name> | Med | YYYY-MM-DD |

Action items are tracked in the engineering backlog with the prefix `INC-YYYYMMDD-NN/AI-X` and shipped within two sprints.

## Lessons

- What did we learn?
- What assumption was wrong?
- What process needs to change?

## Appendix

- Logs, dashboards, traces, screenshots.
- Links to PRs and code changes.
- Related incidents.
