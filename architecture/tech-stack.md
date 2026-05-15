# Tech Stack

**Status:** Active  
**Owner:** Engineering Lead  
**Last updated:** 2026-05-13

This is the authoritative dependency list. Pin every version. New deps require an ADR or `governance/change-management.md` flow if they change architecture.

## Mobile (Flutter)

| Concern | Package | Min version | Notes |
|---|---|---|---|
| SDK | Flutter | 3.22.0 | Stable channel |
| Language | Dart | 3.4.0 | |
| State management | flutter_riverpod | 2.5.x | ADR-006 |
| Navigation | go_router | 14.x | Type-safe routes |
| Local DB | drift | 2.18.x | Compile-time SQL |
| Local DB (alt) | sqflite | 2.3.x | If team prefers raw SQL |
| Secure storage | flutter_secure_storage | 9.x | iOS Keychain / Android Keystore |
| HTTP | dio | 5.x | + interceptors for auth and retry |
| Background sync (Android) | workmanager | 0.5.x | |
| Background sync (iOS) | flutter_background_fetch / BGTaskScheduler binding | latest | |
| Camera | camera + image_picker | latest | |
| Image compression | flutter_image_compress | 2.x | Target ≤ 500 KB |
| GPS | geolocator | 11.x | Permissions handled per-call |
| Permissions | permission_handler | 11.x | |
| Audio recording | record | 5.x | |
| Audio playback | just_audio | 0.9.x | |
| On-device STT | whisper_flutter or whisper_cpp_dart | TBD | tiny.en model only |
| Logging | logger | 2.x | Redact PII |
| Crash reporting | sentry_flutter | 8.x | Opt-in for pilot |
| Localization | flutter_localizations + intl | latest | EN + VI |
| Test runner | flutter_test | bundled | |
| Mocking | mocktail | 1.x | |
| Integration tests | integration_test + patrol | latest | |

## Backend (FastAPI)

| Concern | Package | Min version | Notes |
|---|---|---|---|
| Runtime | Python | 3.11.x | |
| Web framework | fastapi | 0.110.x | |
| ASGI server | uvicorn[standard] + gunicorn | latest | |
| Data validation | pydantic | 2.7.x | |
| Settings | pydantic-settings | 2.x | |
| ORM | sqlalchemy | 2.x | Async session |
| DB driver | asyncpg | 0.29.x | |
| Migrations | alembic | 1.13.x | |
| Task queue | celery | 5.4.x | |
| Broker | redis | 7.2 | server side |
| Redis client | redis-py | 5.x | |
| HTTP client | httpx | 0.27.x | For Odoo + Supabase |
| Odoo RPC | custom thin wrapper around `httpx` | — | Don't use `odoolib` (unmaintained for our use) |
| JWT | python-jose | 3.x | Or PyJWT 2.x — pick one |
| JWKS | jwcrypto / cryptography | latest | |
| Object storage | boto3 | 1.34.x | S3-compatible |
| Logging | structlog | 24.x | JSON output |
| Metrics | prometheus-fastapi-instrumentator | 7.x | |
| Tracing | opentelemetry-sdk + instrumentation | latest | |
| Test runner | pytest | 8.x | |
| HTTP mocks | respx | 0.21.x | |
| Test containers | testcontainers | 4.x | Postgres for integration |

## Edge Functions (Supabase)

| Concern | Package | Min version | Notes |
|---|---|---|---|
| Runtime | Deno | 1.40+ | Provided by Supabase |
| Language | TypeScript | 5.x | |
| Twilio SDK | twilio | latest | |
| Validation | zod | 3.x | |

## Infrastructure

| Concern | Tech | Min version | Notes |
|---|---|---|---|
| Database | PostgreSQL | 15.x | RDS / managed |
| Broker | Redis | 7.x | Managed |
| Object storage | S3 / S3-compatible | — | Presigned URLs only |
| ERP | Odoo | TBD (DEC-001) | |
| Container | Docker | latest | Multi-stage builds |
| Orchestration | Kubernetes or ECS | TBD | DEC-009 |
| CI/CD | GitHub Actions | — | |
| Secrets | AWS Secrets Manager / Vault | TBD | |
| Monitoring | Prometheus + Grafana | latest | |
| Logging | CloudWatch / Loki | TBD | |
| Tracing | OpenTelemetry → Tempo / X-Ray | TBD | |
| Crash reporting | Sentry | latest | |

## Pinning policy

- Mobile: pin to minor (`^x.y.0`) so patches flow in.
- Backend: pin to exact version in `pyproject.toml` lockfile.
- Renovate / Dependabot configured to open weekly upgrade PRs.
- Security advisories handled within 72 hours (P0) or sprint (P1).

## Forbidden / discouraged

- Direct `package:flutter_bloc` use unless team-wide ADR (we standardise on Riverpod).
- `package:hive` for sync data (use SQLite).
- `package:shared_preferences` for tokens (use `flutter_secure_storage`).
- Any plugin without Android + iOS + null-safety support.
- Backend: any sync HTTP call to Odoo from a request handler. Route through Celery.
