# ADRs Index

This folder contains Architectural Decisions of Record. Format: [Michael Nygard ADR](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions).

| ID | Title | Status | Date |
|---|---|---|---|
| [ADR-001](./ADR-001-flutter-unified-mobile.md) | Use Flutter for the unified mobile app | Accepted | 2026-05-13 |
| [ADR-002](./ADR-002-supabase-edge-for-otp.md) | Supabase Edge Functions handle OTP issuance | Accepted | 2026-05-13 |
| [ADR-003](./ADR-003-sqlite-instead-of-sessionstorage.md) | SQLite as the durable local store | Accepted | 2026-05-13 |
| [ADR-004](./ADR-004-fastapi-as-integration-layer.md) | FastAPI as the integration layer between mobile and Odoo | Accepted | 2026-05-13 |
| [ADR-005](./ADR-005-no-direct-mobile-to-odoo.md) | Mobile never calls Odoo directly | Accepted | 2026-05-13 |
| [ADR-006](./ADR-006-riverpod-state-management.md) | Riverpod 2.x for state management | Accepted | 2026-05-13 |
| [ADR-007](./ADR-007-idempotency-by-client-id.md) | Idempotency by `client_id` (UUID v4) | Accepted | 2026-05-13 |
| [ADR-008](./ADR-008-on-device-whisper.md) | Whisper tiny.en runs on-device for voice notes | Accepted | 2026-05-13 |
| [ADR-009](./ADR-009-drift-over-sqflite.md) | Drift (over raw sqflite) for the local SQLite layer — closes DEC-011 | Accepted | 2026-05-13 |
| [ADR-010](./ADR-010-twilio-sms-only.md) | Twilio as the sole SMS / OTP provider for MVP — closes DEC-005 | Accepted | 2026-05-15 |
| [ADR-011](./ADR-011-odoo-19-enterprise.md) | Odoo 19 Enterprise as test target; RPC contract for production portability — closes DEC-001 | Accepted (amended 2026-05-15) | 2026-05-15 |
| [ADR-012](./ADR-012-aws-s3-object-storage.md) | DigitalOcean Spaces (sgp1) for evidence object storage — closes DEC-007 | Accepted (amended 2026-05-15) | 2026-05-15 |
| [ADR-013](./ADR-013-custom-gps-model.md) | GPS pings in custom Odoo model — replaced by ADR-016 | **Superseded** | 2026-05-15 |
| [ADR-014](./ADR-014-field-mobile-sync-module.md) | Custom Odoo module `field_mobile_sync` — replaced by ADR-015 | **Superseded** | 2026-05-15 |
| [ADR-015](./ADR-015-no-custom-module-rpc-adapter.md) | No custom Odoo module — RPC adapter pattern + `x_ngynapp_*` fields — supersedes ADR-014 | Accepted | 2026-05-15 |
| [ADR-016](./ADR-016-gps-in-fastapi-postgres.md) | GPS history in FastAPI Postgres; Odoo holds summary `x_ngynapp_gps_*` only — supersedes ADR-013 | Accepted | 2026-05-15 |

Use [_template.md](./_template.md) for new ADRs.
