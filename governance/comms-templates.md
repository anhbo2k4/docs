# Communication Templates

**Status:** Active
**Owner:** Project Manager
**Audience:** PM, Engineering Lead, on-call engineers, stakeholders.

Reusable templates for the recurring communications described in `governance/communication-plan.md` and `engineering/oncall.md`. Copy a block, replace placeholders in `<...>`, send.

---

## 1. Sprint review email (Friday end of sprint)

```text
Subject: Sprint <NN> review — Field Work Unified App — <date range>

Hi all,

Sprint <NN> wrapped today. Highlights:

✅ Shipped
- <story 1 summary> (US-XXX-NNN)
- <story 2 summary>
- <doc / governance change>

🔁 Carried to S<NN+1>
- <story> — reason: <one-liner>

📊 Health
- Velocity: <pts done>/<pts planned> ( <%> )
- CI green: <%> on main
- Crash-free sessions (pilot build): <%>
- NFR debt: <count> open, <count> closed

🚧 Risks / blockers
- <risk RAID-NNN>: <state> — owner <name>

🔮 Next sprint focus
- <theme + 2-3 bullet points>

Demo recording: <link>
Sprint board: <link>
Sprint plan: <link to sprints/sprint-NN/README.md>

Thanks team.
— <PM name>
```

---

## 2. Stakeholder weekly update (every Friday)

```text
Subject: Field Work Program — weekly update — <ISO week>

This week:
- Phase: <S0 Discovery / S1 Foundation / ...>
- Decisions closed: DEC-XXX, DEC-YYY (now Decided in decision-log)
- Decisions still open: DEC-AAA (owner X, target Y)
- Build status: <green | yellow | red> — reason if not green
- Pilot gate readiness: <Not started | In progress | Ready | Blocked>

Top 3 risks:
1. RISK-NNN: <one-line> — owner / mitigation
2. RISK-NNN: ...
3. RISK-NNN: ...

Asks of stakeholders this week:
- <ask 1>
- <ask 2>

Reading for the curious:
- <link to README>
- <link to specific doc>

— <PM>
```

---

## 3. Release notes email (post mobile-vX.Y.Z)

```text
Subject: Field Work App vX.Y.Z released — <date>

Pilot users,

We rolled out vX.Y.Z today.

✨ New
- <feature 1> (US-XXX-NNN)
- <feature 2>

🛠 Improved
- <improvement>

🐛 Fixed
- <bug fix>

🔒 Security
- <only if user-visible>

Known issues:
- <issue + workaround> (tracked as <link>)

How to update:
- iOS: TestFlight will prompt; if not, force-quit and reopen.
- Android: Play Store update arrives within 24 h.

Need to report something? Reply to this email or use the feedback button under Settings → Help.

— <PM>
```

---

## 4. Incident — initial notification (within 15 min of detection)

```text
Subject: [INCIDENT P<X>] <one-line summary>

Status: Investigating
Started: <UTC time>
Detected by: <Sentry / dashboard / user report>
Impact: <user-visible behaviour, % users, severity>
Workaround: <short one-liner if known else "investigating">
Owner: <on-call engineer>
Channel: #field-incidents
Status page: <link>

Updates every 30 min until mitigated.
```

---

## 5. Incident — resolution notification

```text
Subject: [RESOLVED] <one-line summary>

Status: Resolved
Started: <UTC>
Mitigated: <UTC>
Resolved: <UTC>
Duration: <H:MM>
Impact (final): <users / capture envelopes / shifts affected>
Root cause (1 line): <…>
Mitigation: <…>

Action items: <link to issue board>
Postmortem: scheduled <date> — public report by <date> (per runbooks/_postmortem-template.md)

Thanks for your patience.
— <on-call>
```

---

## 6. Postmortem invite (within 5 working days)

```text
Subject: Postmortem — <incident title> — <date> at <time>

Attendees: <on-call, engineers involved, PM, EL, Security Champion if security-related>
Duration: 60 minutes
Format: blameless; we discuss the system, not the people.
Pre-read: <link to incident channel transcript + draft postmortem>
Template: runbooks/_postmortem-template.md
Output: filled postmortem, action items in the issue tracker.

Please block the slot.
— <PM>
```

---

## 7. Decision request (escalation)

Use when escalating a decision per `governance/escalation.md`.

```text
Subject: Decision needed: <topic> by <date>

Context (5 lines max):
<...>

Options:
A) <option> — pro/con
B) <option> — pro/con
C) <option> — pro/con

Recommendation: <option + 1-line reason>

Impact if no decision by <date>: <which stories / sprints / pilot dates blocked>

Decision will be logged as DEC-NNN in traceability/decision-log.md.

Please reply with A / B / C or "discuss".
— <requester>
```

---

## 8. ADR proposal note

```text
Subject: ADR-<N>: <topic> — request for review

Status: Proposed
Context: <2-3 lines>
Decision: <1 line>
Rationale: <bullets>
Alternatives considered: <table>
Consequences: <bullets>

Full ADR: <link to adr/ADR-N-...md>
Reviewers: <names per RACI>
Decision deadline: <date>
```

---

## 9. New joiner welcome (Day 1)

```text
Hi <name>, welcome.

Your first day:
1. Read field_work_delivery/README.md and WORKING.md.
2. Run through governance/onboarding.md "Day 1" checklist.
3. Pair with <buddy>; first stand-up at <time>.
4. Slack channels: #field-program, #field-builds, #field-incidents.
5. Pick a "Good first PR" from the pinned issue.

Buddy: <name>
PM: <name>
Engineering Lead: <name>
On-call (this week): <name>

Questions: ask in #field-program — no question is too small.

— <PM>
```

---

## 10. End-of-pilot summary (post S10)

```text
Subject: Field Work Pilot — week <X> wrap-up

Pilot cohort: <N users>, <N shifts captured>, <N envelopes synced>
Crash-free sessions: <%>
Sync success: <%> within 30 min
Median capture-to-Odoo time: <minutes>
Top 3 issues raised: <list>
Top 3 successes: <list>

Recommendation: <continue / extend pilot / cutover / pause>
Decision needed by: <date>

Detailed report: <link>
Postmortems / debriefs scheduled: <list>

— <PM>
```

---

## 11. Cross-references

- Communication plan & cadence: `governance/communication-plan.md`
- Stakeholder list: `governance/stakeholders.md`
- Escalation: `governance/escalation.md`
- Incident response: `runbooks/incident-response.md`
- Postmortem template: `runbooks/_postmortem-template.md`
- Mobile release runbook: `runbooks/mobile-release.md`
