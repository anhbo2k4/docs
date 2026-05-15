# Mobile Store Release Runbook (TestFlight + Play Console)

**Status:** Active
**Owner:** Mobile Lead + DevOps
**Last updated:** 2026-05-13
**Cadence:** Per release (mobile-vX.Y.Z tag)

This runbook is the human-driven complement to `engineering/ci-pipeline.md` §4.1. CI builds and uploads; this runbook owns the **decisions** around when to promote a build, who signs off, and what to do if a store rejects.

---

## 1. Cadence

- Internal: every successful merge to `main` → automatic upload to **internal** track.
- Pilot: weekly Wednesday 16:00 local → promote internal → **TestFlight + Play Closed Track (pilot group)**.
- Production: every other sprint review (post-S10) → promote pilot → **App Store + Play Open Track**.

Release cadence may be paused by the Engineering Lead if the pilot dashboard shows red NFRs (see `backlog/nfr-register.md`).

---

## 2. Pre-flight checklist (Mobile Lead)

Before tagging `mobile-vX.Y.Z`:

- [ ] Story status: every story planned for this release is `Done` per `governance/definition-of-done.md`.
- [ ] CHANGELOG: `CHANGELOG.md` has a `[X.Y.Z] — YYYY-MM-DD` entry with Added / Changed / Fixed / Security.
- [ ] Release notes: `backlog/releases/X.Y.Z.md` exists and matches `_template.md`.
- [ ] NFR check: every `backlog/nfr-register.md` row marked `pass` or has an open `nfr-debt` issue.
- [ ] Pilot gate (release ≥ 1.0.0): `qa/pilot-gate.md` checklist signed off.
- [ ] Crash-free rate ≥ 99.5 % on the previous build over the last 7 days (Sentry).
- [ ] No P0 / P1 bugs open against the build.
- [ ] App icon, screenshots, store description reviewed by Designer + PM.
- [ ] Privacy questionnaire (Apple) and Data Safety form (Play) reflect current data collection (`security/data-classification.md`).
- [ ] Force-update threshold (`DEC-012`) set if min-version bumped.

Sign-off: Engineering Lead + PO must each leave a 👍 on the release PR before tag is created.

---

## 3. Tag and trigger

```
git tag mobile-vX.Y.Z -s -m "Mobile release X.Y.Z"
git push origin mobile-vX.Y.Z
```

`release-mobile.yml` runs (see `engineering/ci-pipeline.md` §4.1). Watch the run. Expected duration: 25–40 min.

---

## 4. Apple App Store Connect — TestFlight upload

CI uploads automatically. Manual steps after upload:

1. Wait for processing (5–30 min).
2. App Store Connect → TestFlight → build appears with "Missing Compliance".
3. Answer **Export Compliance**: app uses HTTPS only; standard exemption applies.
4. Add to **Internal testing group** (Field Team Internal).
5. After 24 h soak: add to **External testing group** (Pilot — N participants).
6. External testing requires Apple Beta Review (24–48 h, occasional rejection — see §6).
7. On approval: testers receive invite email automatically.

### Production submission (release ≥ 1.0.0)

1. App Store Connect → App Store → `+ Version`.
2. Paste release notes from `backlog/releases/X.Y.Z.md`.
3. Attach the TestFlight build that completed external testing for ≥ 7 days.
4. Submit for review. Typical review time: 24–48 h.
5. On approval: choose "Manual release" so the team controls go-live time.

---

## 5. Google Play Console — Closed / Open track

CI uploads `.aab` to **Internal testing** by default.

### Promote internal → closed (pilot)

1. Play Console → Testing → Internal testing → select build → "Promote release" → Closed testing → "Pilot".
2. Set rollout to 100 % within the closed track.
3. Save and review.

### Closed → Production (open track)

1. Play Console → Testing → Closed testing → "Pilot" → select build → "Promote release" → Production.
2. Set staged rollout: **20 % → 50 % → 100 %** over 5 days.
3. Save and review.

Halt-on-crash threshold: if Play Console shows ANR rate > 0.47 % or crash rate > 1.09 % during staged rollout, halt and trigger `runbooks/rollback.md`.

---

## 6. Rejection handling

### Apple Beta / App Store rejection

| Code / theme | Likely cause | First fix |
|---|---|---|
| 5.1.1 (data and privacy) | Missing usage strings (NSCameraUsageDescription, NSLocationWhenInUseUsageDescription, NSMicrophoneUsageDescription) | Update Info.plist; bump build number; resubmit |
| 4.0 (UI) | Tab/screen reachable but empty in pilot config | Hide behind feature flag (`engineering/feature-flags.md`) and resubmit |
| 2.1 (info needed) | Reviewer can't log in | Ship a **demo account** in `screen/auth/preset_login.dart` for `flavor=appReview` |
| 5.4 (legal) | Missing privacy policy URL | Add to App Store Connect listing |
| Guideline 4.5.4 (push notifications) | Promotional content in pilot | Remove promo path; resubmit |

### Play Console rejection / warning

| Theme | Fix |
|---|---|
| Data Safety form mismatch | Update form to match `security/data-classification.md` |
| Permissions disclosure | Update store listing description; add in-app rationale dialog |
| Foreground service usage | Verify FOREGROUND_SERVICE_LOCATION manifest declaration matches use; update Data Safety |
| Target API level | Bump `targetSdkVersion` to current Play requirement |

If a rejection blocks pilot: open a P1 incident, log under `runbooks/incident-response.md`, page on-call.

---

## 7. Demo account for store review

A dedicated demo account is provisioned with:

- Phone: Apple/Google reviewer-safe number per `runbooks/otp-fallback.md` magic-OTP pattern.
- Pre-loaded shifts: 1 active, 1 future, 1 completed.
- Pre-captured PPE + 1 photo so reviewer sees a populated UI immediately.
- No real PII.

Credentials live in 1Password; rotate every 90 days. Never paste in store fields except via the secured "App Review" textbox.

---

## 8. Phased rollout dashboard

Track during a staged rollout:

| Metric | Source | Threshold |
|---|---|---|
| Crash-free sessions | Sentry | ≥ 99.5 % |
| Crash-free users | Sentry | ≥ 99.0 % |
| ANR rate | Play Vitals | < 0.47 % |
| Cold start P95 | Sentry perf | ≤ 2.5 s (PB-M-001) |
| Sync success | Backend dashboards | ≥ 99 % within 30 min |
| Pilot tickets | Support inbox | < 3 P1 in 24 h |

Cross any threshold → halt rollout → execute `runbooks/rollback.md` if user impact.

---

## 9. After release

- Update `WORKING.md` last-decisions block.
- Move `backlog/releases/X.Y.Z.md` from `Draft` to `Released YYYY-MM-DD`.
- Increase the release counter in the README badge (when added).
- Schedule retro slot: pilot retro within 5 working days of full rollout.

---

## 10. Cross-references

- CI pipeline: `engineering/ci-pipeline.md` §4.1
- Release process: `engineering/release-process.md`
- Rollback: `runbooks/rollback.md`
- Pilot gate: `qa/pilot-gate.md`
- Force-update enforcement: `traceability/decision-log.md` DEC-012
- Privacy / data classification: `security/data-classification.md`
- Release notes template: `backlog/releases/_template.md`
