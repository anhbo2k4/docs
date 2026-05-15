# Developer environment

**Status:** Active
**Owner:** Engineering Lead

## Pinned tool versions

| Tool | Version | Notes |
|---|---|---|
| Flutter SDK | 3.22.x stable | Pinned in `mobile/pubspec.yaml` and `.fvm/fvm_config.json`. |
| Dart | bundled | Use Flutter-bundled Dart only. |
| Xcode | 15.x | Required for iOS builds. |
| Android Studio | 2024.1.x | With Android SDK 34, NDK r26d. |
| Node.js | 20.x LTS | For Supabase Edge functions and tooling only. |
| Python | 3.12.x | Backend (FastAPI, Celery). Pinned via `pyproject.toml`. |
| Poetry | 1.8.x | Backend dependency management. |
| Docker | 25.x | Local Odoo + Postgres + Redis stack. |
| docker-compose | v2 plugin | `docker compose` (no hyphen). |
| Odoo | per DEC-001 | Run via container; never installed locally. |
| Make | any | Driving common targets. |
| gh | latest | GitHub CLI for PRs and issues. |

Versions live in `.tool-versions` (asdf-compatible) and CI matches them exactly. Do not upgrade without an ADR.

## Repo layout (target)

```
.
├── mobile/                     Flutter app
│   ├── lib/
│   │   ├── @core/              infra: db, network, sync, storage, auth
│   │   ├── @share/             helpers
│   │   ├── application/        bootstrap, router, lifecycle
│   │   ├── plugins/            camera, GPS, audio
│   │   ├── resource/           theme tokens
│   │   ├── screen/             features (auth, shifts, field_work, sync_center)
│   │   ├── widgets/            reusable
│   │   └── main.dart
│   ├── test/
│   ├── integration_test/
│   ├── ios/
│   ├── android/
│   ├── pubspec.yaml
│   └── analysis_options.yaml
│
├── backend/                    FastAPI + Celery
│   ├── app/
│   │   ├── auth/
│   │   ├── shifts/
│   │   ├── sync/
│   │   ├── media/
│   │   ├── odoo/
│   │   ├── observability/
│   │   └── main.py
│   ├── tests/
│   ├── alembic/
│   └── pyproject.toml
│
├── edge/                       Supabase Edge Functions (Deno + TS)
│   ├── request_otp/
│   └── verify_otp/
│
├── infra/
│   ├── odoo/field_mobile_sync/  custom Odoo module (Python)
│   ├── docker/                  local stack
│   ├── ci/                      shared CI fragments
│   └── terraform/               (post-MVP target)
│
├── field_work_delivery/         this blueprint
└── Makefile
```

## First-day setup

1. `git clone` the monorepo.
2. `asdf install` (or install pinned versions manually).
3. `make bootstrap` — installs Flutter packages, Poetry env, pre-commit hooks, Docker images.
4. `make up` — starts Postgres, Redis, Odoo container (per DEC-001).
5. `make db-migrate` — applies Alembic migrations and Odoo module install.
6. `make seed` — seeds an `hr.employee` with a known phone for local OTP testing.
7. `make mobile-run` (Android) or `make mobile-run-ios` — boots the app pointing at local backend.
8. `make backend-run` — starts FastAPI on `:8000`, Celery worker, Celery beat.

## Make targets (canonical)

| Target | Purpose |
|---|---|
| `make bootstrap` | Install all dependencies and hooks. |
| `make up` / `make down` | Local infra stack. |
| `make logs` | Tail local stack. |
| `make backend-test` | Run pytest with coverage. |
| `make mobile-test` | Run flutter test + integration_test. |
| `make lint` | Run all linters (Dart, Python, TS, YAML). |
| `make format` | Auto-format. |
| `make build-mobile-apk` | Release APK. |
| `make build-mobile-ipa` | Release IPA (Mac only). |
| `make e2e` | End-to-end suite (device matrix subset). |
| `make load` | Local k6 load test against staging. |

If any target is missing, treat it as a S1 backlog item and add a story.

## Environment variables

Loaded from `.env.local` (gitignored). Template at `.env.example`. Required keys:

```
SUPABASE_URL=
SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=     # backend only
TWILIO_ACCOUNT_SID=
TWILIO_AUTH_TOKEN=
TWILIO_FROM_NUMBER=
ODOO_URL=
ODOO_DB=
ODOO_API_KEY=
POSTGRES_DSN=
REDIS_URL=
JWT_PRIVATE_KEY_PATH=
JWT_PUBLIC_KEY_PATH=
S3_ENDPOINT=
S3_BUCKET=
S3_ACCESS_KEY=
S3_SECRET_KEY=
SENTRY_DSN_BACKEND=
SENTRY_DSN_MOBILE=             # mobile-only build env
```

Secrets are loaded from the secret manager in higher environments. See `security/secrets.md`.

## Common pitfalls

- **iOS Keychain in simulator.** Some Keychain entries persist across reinstalls of the same simulator; use `xcrun simctl erase all` to wipe.
- **Android emulator GPS.** Use the emulator's "Send GPS coordinates" UI; raw lat/lng injection via ADB is unreliable.
- **Whisper model.** First-launch download adds ~75 MB; bundling vs download is DEC-010.
- **Odoo container memory.** Allocate at least 4 GB to Docker Desktop or Odoo will OOM during install.
- **Time skew.** Idempotency tests assume monotonic clocks. Don't run while NTP is correcting.

## Definition of "ready to code"

You are ready when:

- `make backend-test` and `make mobile-test` both pass on a clean checkout.
- You can complete OTP login locally end-to-end.
- You can submit a PPE envelope and see it reach Odoo locally.
