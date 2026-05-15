# Pilot Gate

**Status:** Active  
**Owner:** QA Lead + Engineering Lead + Product Owner

The release gate that decides whether the build is ready for pilot launch. **Every box must be checked before pilot starts.**

## Decision

Pilot Go / No-Go decision is taken by Sponsor + PO + EL + QA after this checklist is fully evaluated. SEV-1 / SEV-2 issues are automatic No-Go.

## A. Functional readiness

- [ ] All P0 FRs (FR-001..015) implemented and passing on at least one iOS and one Android device.
- [ ] All P0 stories `Done` with linked PRs.
- [ ] All P0 test cases passing (see `qa/test-cases/`).
- [ ] All P0 acceptance criteria signed off by PO.
- [ ] No open Blocker bugs.
- [ ] No open Critical bugs.
- [ ] ≤ 3 open Major bugs, each with documented workaround.

## B. Non-functional readiness

- [ ] Cold start ≤ 2.5 s p95 (NFR-001).
- [ ] Photo compression target met (NFR-005).
- [ ] Sync POST p95 ≤ 800 ms (NFR-007).
- [ ] Backend `/sync/envelope` p95 ≤ 200 ms (NFR-008).
- [ ] Crash-free sessions ≥ 99.5 % over 7-day staging soak (NFR-023).
- [ ] Offline 72-hour endurance test passed (NFR-025).
- [ ] App size within targets (NFR-010, NFR-011).
- [ ] Idempotency under retry storm verified (NFR-024).

## C. Security readiness

- [ ] Threat model reviewed in last 30 days.
- [ ] All P0 / P1 security findings closed.
- [ ] Secret scanning clean on main.
- [ ] Dependency vulnerability scan: 0 P0, ≤ 5 P1 with mitigation.
- [ ] PII redaction verified on logs and Sentry.
- [ ] OTP rate limit + lockout verified.
- [ ] JWT key rotation procedure tested.
- [ ] Penetration test report received (or scheduled within 30 days post-pilot).

## D. Data readiness

- [ ] SQLite migrations forward + back tested for current version.
- [ ] Postgres migrations applied to staging without manual fixes.
- [ ] Idempotency dedup verified end-to-end.
- [ ] Odoo `field_mobile_sync` module deployed to pilot Odoo (DEC-003 closed).
- [ ] Phase 0 decisions DEC-001..009 closed (or formally waived).

## E. Operational readiness

- [ ] CI/CD green on main for 7 consecutive days.
- [ ] Deployment runbook tested (`runbooks/deploy.md`).
- [ ] Rollback runbook tested (`runbooks/rollback.md`).
- [ ] Sync recovery runbook tested (`runbooks/sync-recovery.md`).
- [ ] Incident response runbook reviewed (`runbooks/incident-response.md`).
- [ ] On-call rotation defined and acknowledged.
- [ ] PagerDuty / Slack alerts wired and tested.
- [ ] Backups configured (Postgres point-in-time, S3 versioning).
- [ ] Monitoring dashboards live: API RED, queue depth, sync success, Odoo error rate.
- [ ] Force-update mechanism tested.

## F. Compliance readiness

- [ ] Privacy policy in app + on web.
- [ ] Terms of service.
- [ ] Data residency confirmed (DEC-008).
- [ ] Data deletion procedure documented (`runbooks/data-deletion.md`).
- [ ] Audit log retention configured (7 years).

## G. Device matrix

- [ ] iPhone 12 (iOS 17), iPhone 15 (iOS 18), iPhone SE 2 (iOS 17): all pass smoke.
- [ ] Pixel 6a (Android 14), Galaxy A13 (Android 13), Pixel 8 (Android 14): all pass smoke.
- [ ] At least one low-bandwidth (3G) test session completed.
- [ ] At least one airplane-mode → reconnect cycle test.

## H. UAT

- [ ] UAT plan signed off by Stakeholder (`qa/uat-plan.md`).
- [ ] 5 pilot users completed full happy-path scenario.
- [ ] Stakeholder acceptance recorded in writing.

## I. Documentation

- [ ] Field worker user guide.
- [ ] Supervisor briefing deck.
- [ ] On-call runbook updated.
- [ ] CHANGELOG bumped.
- [ ] Release notes drafted.

## J. Communication

- [ ] Pilot kickoff scheduled.
- [ ] Feedback channel ready (in-app + Slack).
- [ ] Steering committee informed.

## Outcome record

Pilot Gate decision is recorded in `qa/pilot-gate-record-YYYYMMDD.md` (created at decision time) with:
- Date, attendees, signatures.
- Each section's pass/fail/waived.
- Open risks accepted.
- Rollback trigger criteria.

## Rollback triggers (post-pilot launch)

The following automatically trigger a rollback decision:

- Sync success rate < 95 % over 6 hours.
- Crash-free sessions < 99 % over 24 hours.
- Two SEV-1 incidents within 48 hours.
- PII leak detected.
- Odoo data corruption detected.

See `runbooks/rollback.md`.
