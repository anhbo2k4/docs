# UAT plan (pilot acceptance)

**Status:** Draft (becomes Active at end of S9)
**Owner:** QA Lead + Product Owner
**Audience:** Pilot site supervisor + 5 field workers (cohort sized in US-QA-006)

This is the playbook for User Acceptance Testing during the pilot run. It bridges the engineering-side `qa/test-strategy.md` to a script that pilot users actually follow.

## 1. Goals

- Confirm the unified Flutter app supports a real shift end-to-end: login → My Shifts → PPE → pre-shift form → check-in → field capture → check-out → sync.
- Surface usability issues that automated tests cannot catch (gloves, sun glare, vehicle context, intermittent 4G).
- Validate offline-first promise under genuine field conditions.
- Provide a single signed UAT report for the Pilot Gate.

## 2. Scope

In:

- All P0 flows from FR-001..FR-015.
- iOS + Android, range of device tiers (see `qa/device-matrix.md`).
- Vietnamese + English locales.
- Offline (≥ 4 hours), spotty 4G, and stable Wi-Fi.

Out:

- Post-MVP features (EPIC-10).
- Stress / load (covered by `qa/load-plan.md`).
- Backend admin flows (Odoo native).

## 3. Cohort

| Role | Count | Source |
|---|---|---|
| Field worker | 10–15 | Pilot site |
| Field supervisor | 1–2 | Pilot site |
| Backup / spare | 2 | Pilot site |
| Observer (QA) | 1 | Internal |
| Observer (PM/PO) | 1 | Internal |

Recruitment + consent: `epics/EPIC-09-qa-pilot/stories/US-QA-006-pilot-launch.md`.

## 4. Pre-pilot setup

- Devices enrolled on TestFlight / Play Internal Testing.
- Pilot accounts seeded in Odoo with known phones.
- Pilot data isolated by `pilot=true` flag where applicable.
- Feedback channel live (form + Slack/Telegram bridge).
- `Pilot Health` dashboard shared with Sponsor.
- Rollback plan rehearsed within last 7 days (NFR-101).

## 5. Test scripts

Each script is a numbered, observable scenario. Pilot users execute; observers record.

### S-UAT-01 — First login (online)
1. Open app on a freshly installed device.
2. Enter phone, request OTP, verify, land on My Shifts.
3. Expected: ≤ 60 s end-to-end including SMS delivery.
4. Pass criteria: TC-AUTH-001..005 effectively pass on real device.

### S-UAT-02 — First login (poor signal)
- Repeat S-UAT-01 inside building / known weak-signal zone.
- Pass: app shows clear error states; no crash; OTP retry works once back in coverage.

### S-UAT-03 — Shift detail
- Tap a shift, view detail; observe sync badges.
- Pass: matches the assigned shift; states render in Vietnamese.

### S-UAT-04 — PPE check (online)
- Complete PPE checklist; submit.
- Pass: capture goes `PENDING → SYNCING → CONFIRMED` within 30 s.

### S-UAT-05 — PPE check (offline)
- Toggle airplane mode before submit; submit.
- Pass: capture goes `PENDING`; UI shows "queued"; rejoining network promotes to `CONFIRMED` within 5 min.

### S-UAT-06 — Pre-shift form
- Fill the form per real shift context.
- Pass: form validates, persists, syncs.

### S-UAT-07 — GPS check-in
- Tap check-in; allow GPS; verify accuracy gate ≤ 50 m.
- Pass: `gps_event` syncs; if accuracy fails, structured skip with reason works.

### S-UAT-08 — Photo capture
- Take 3 photos under varied lighting.
- Pass: each ≤ 500 KB; gallery shows all; deletion before submit works.

### S-UAT-09 — Voice note + transcription
- Record a 30 s field note in Vietnamese; review transcript; edit.
- Pass: transcript appears within 30 s; edit is preserved on sync.

### S-UAT-10 — Work result + sync
- Compose work result with photos, GPS, voice; submit.
- Pass: bundle reaches Odoo; supervisor sees record in Odoo within 2 min.

### S-UAT-11 — Sync Center
- Open Sync Center; observe states; manually retry a failed envelope.
- Pass: states match envelopes; retry returns row to PENDING and drains.

### S-UAT-12 — Force-kill resilience
- Submit a capture; immediately force-kill the app.
- Pass: on relaunch, in-flight rows reset to PENDING and drain; nothing duplicates in Odoo.

### S-UAT-13 — Logout
- Tap Logout; relaunch; observe DB and tokens cleared.
- Pass: app routes to login; previous data not visible; storage check shows DB removed.

### S-UAT-14 — A11Y smoke
- One worker completes S-UAT-04 + S-UAT-08 with TalkBack / VoiceOver enabled.
- Pass: all controls labelled; capture completes.

### S-UAT-15 — Locale switch
- Change device language EN ↔ VI.
- Pass: all visible strings translate; no overflow.

## 6. Pass / fail rules

- Every script has a pass criterion above.
- A failed script triggers a defect with severity: SEV-1 (blocker), SEV-2 (workaround exists), SEV-3 (cosmetic).
- The Pilot Gate (`qa/pilot-gate.md`) requires:
  - 0 SEV-1 defects.
  - ≤ 3 SEV-2 with documented workaround and committed fix in next sprint.
  - SEV-3 backlog visible.

## 7. Telemetry watched

- Crash-free sessions ≥ 99.5 %.
- Sync success ≥ 99 % within 1 h.
- OTP success ≥ 95 % first try.
- Mean time to feedback acknowledgment ≤ 24 h.

## 8. Daily cadence

- 0900 standup with pilot supervisor (15 min).
- Observer log → Slack/Telegram intake bridge → backlog by 1100.
- Bug triage at 1500 (PM + EL + QA).
- Hot-fix path documented in `runbooks/incident-response.md` and `engineering/release-process.md`.

## 9. Sign-off

UAT is signed off by:

- Sponsor (Accountable per RACI for pilot launch decision).
- PO (Responsible).
- QA Lead (Responsible).

Signatures live in the report bundled with the Pilot Gate evidence pack.

## 10. Open questions

- Bilingual transcript verification for Vietnamese (Whisper tiny.en is English-leaning) — track via DEC-014 if observed during S6.
- Network conditions matrix per pilot site (carrier, weak-signal map) — site-specific addendum at S9.
