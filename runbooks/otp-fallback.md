# Runbook: OTP Fallback

**Status:** Active  
**Owner:** Backend Lead + DevOps

## Symptom

Users report "I never got the SMS code" at scale. Or:

- Twilio status page reports incident.
- Supabase Edge function error rate > 5 %.
- OTP request → verify success rate < 90 %.

## Severity

SEV-1 if pilot site cannot log in at all. SEV-2 if intermittent.

## Diagnose

- Check Twilio status: https://status.twilio.com
- Check Supabase status: https://status.supabase.com
- Check Edge function logs in Supabase dashboard.
- Spot-check by sending OTP to a test phone.

## Mitigations (in order)

### 1. Resend with backoff
The first-line response — many SMS issues are transient. Mobile should auto-prompt user to resend after 60 s.

### 2. Switch SMS provider (if a fallback is configured per DEC-005)
If a regional fallback (e.g. Vonage, MessageBird) was configured during S0:

- Toggle env var `SMS_PROVIDER=fallback` on Edge functions.
- Redeploy Edge functions (zero-downtime).
- Verify by sending a test OTP.

### 3. Manual override for critical pilot users

For pilot supervisors who must access urgently:

- Admin tool / Odoo backend action: "Issue manual OTP".
- Generates a one-time code (10 min TTL).
- Code communicated out-of-band (phone call from PM).
- User enters into normal verify screen; backend accepts manual override.
- All overrides audited.

This requires `field_mobile_sync` module to expose the manual-override action (see DEC-003).

### 4. Service-account fallback (last resort, prod only)

If the OTP path is fully down and the pilot must continue:

- Backend exposes `/v1/auth/admin-issue` (admin-only, IP-restricted, MFA-protected).
- PM, with explicit Sponsor approval, issues a JWT directly bound to a specific employee + device.
- TTL forced to 1 hour.
- All issuances audited and reviewed within 24 h.

This is intentionally awkward to prevent normalising the bypass.

## Mobile UX during OTP outage

Mobile recognises 503 from Supabase Edge:

```dart
if (e.code == 'OTP_PROVIDER_DOWN') {
  showBlockingDialog(
    'SMS service temporarily unavailable. Please try again in a few minutes.'
    'If urgent, contact your supervisor.'
  );
}
```

## Communicate

- `#field-app-incidents`: status updates every 15 min.
- `#field-app-stakeholders`: single message per incident with current status, ETA, workaround.
- Pilot supervisor: direct call from PM if SEV-1.

## Verify

- Test OTP send to 3 phones across different networks succeeds.
- Pilot supervisor can log in.
- Status page green.

## After-action

- Postmortem if SEV-1 lasted > 30 min or affected pilot site.
- If we relied on manual override, audit all issuances.
- If we lacked a fallback provider and needed one, escalate DEC-005 to "decided" with the missing fallback configured.
