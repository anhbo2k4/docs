# ADR-012: DigitalOcean Spaces (SGP1) for evidence object storage

**Status:** Accepted (amended 2026-05-15, blueprint v2.1.0)
**Date:** 2026-05-15
**Last amended:** 2026-05-15
**Deciders:** Sponsor, DevOps, Backend Lead
**Consulted:** Engineering Lead, Security Champion, Mobile Lead
**Informed:** PO, QA Lead

> **Amendment notice (2026-05-15, v2.1.0).** Storage backend changed from AWS S3 (`ap-southeast-1`) to **DigitalOcean Spaces (`sgp1`)**. Primary driver: pricing and operational simplicity for testing — DO is the chosen compute host (see `architecture/deployment.md`), single vendor billing, predictable flat-rate Spaces pricing. The S3-compatible API means migration to AWS S3 or Cloudflare R2 later is an endpoint swap, not a re-architecture.

## Context

DEC-007 asks which object storage backend hosts evidence (photos, voice notes) uploaded from mobile.

Mobile captures evidence offline, queues it locally with a checksum, and uploads via a presigned URL when online. The storage backend must:

- Issue short-lived presigned URLs (mobile never holds long-term credentials).
- Tolerate multi-part uploads with retry.
- Encrypt at rest and in transit.
- Stay close to pilot users (Vietnam / SEA) for latency.
- Integrate cleanly with FastAPI + Celery.
- Fit the program's vendor and cost posture.

Sponsor has reconfirmed that the **compute host is DigitalOcean** (testing first; production likely the same — see `architecture/deployment.md`). The storage decision is therefore re-evaluated in that context.

## Decision

Use **DigitalOcean Spaces in `sgp1` (Singapore)** as the object storage backend.

- Bucket per environment: `field-mobile-{env}-evidence-sgp1` (env = pilot, prod, staging).
- Server-side encryption at rest (AES-256, managed by DO).
- TLS 1.2+ in transit; TLS 1.3 preferred.
- Public-access blocked at bucket level; presigned URLs only for read/write from mobile.
- Lifecycle rule: objects older than 365 days transitioned to cold-archive prefix `archive/` (DO Spaces does not have S3-IA tiers; we use a storage-class proxy via prefix + cron compaction if needed).
- Daily access-log inventory enabled.

S3-compatible SDKs are used everywhere: `boto3` (Python), `minio-dart` or `aws_s3_upload` (Dart). The endpoint is `https://sgp1.digitaloceanspaces.com`.

## Rationale

- **Same vendor as compute.** FastAPI runs on DO Droplets / App Platform; Postgres and Redis are DO managed services. One vendor, one invoice, one IAM model.
- **Predictable cost.** Spaces is flat $5/month for 250 GB storage + 1 TB outbound transfer. At pilot scale (~360 MB voice + ~10 GB photos / month for 50 workers), we are deep inside the included quota.
- **Latency.** `sgp1` is the closest DO region to Vietnam pilot users; round-trip typically under 60 ms.
- **S3-compatible escape hatch.** If we ever outgrow Spaces or want zero-egress, swap endpoint + credentials to Cloudflare R2 or AWS S3 with no code change beyond config.
- **Operationally simple.** Spaces does not require a separate IAM model; access keys are scoped to the team account, with read/write per bucket.

## Alternatives considered

| Option | Pros | Cons | Why not chosen |
|---|---|---|---|
| AWS S3 `ap-southeast-1` (previous decision) | Mature, broad tooling | New vendor relationship vs DO compute; egress fees on cross-vendor reads; more IAM overhead | Vendor sprawl for pilot scale |
| Cloudflare R2 | Zero egress, S3-compatible, cheap storage | Multi-part / lifecycle features less mature; second vendor account; no SGP region (uses global) | Worth revisiting at scale; not for testing |
| MinIO self-hosted | Lowest per-GB; data sovereignty | Ops burden: backups, replication, monitoring | Not justified at pilot scale |
| DigitalOcean Spaces `sgp1` (chosen) | Same vendor as compute; flat pricing; SEA region; S3-compatible | Smaller ecosystem than AWS; lifecycle features lighter | Best fit for testing + pilot |
| Google Cloud Storage `asia-southeast1` | Comparable tech | New cloud account, IAM, observability stack | Operational complexity |

## Consequences

### Positive

- Presigned-URL flow is unchanged; `api-contracts/media.md` reads cleanly against any S3-compatible store.
- DevOps manages one vendor; secrets, billing, support contracts simpler.
- Flat-rate cost model gives finance predictable numbers for the pilot.
- Future migration to R2 or AWS S3 is a config swap, not a refactor.

### Negative

- DO Spaces lifecycle / tiering is less granular than AWS S3 (no IA / Glacier tiers). Cold archive uses prefix-based convention.
- DO ecosystem is smaller — fewer pre-built tooling integrations than AWS.
- Egress beyond the included 1 TB / month is billed at $0.01/GB (still cheap; budget alert at 80%).
- Vendor lock-in to DO at the operational level, though the storage API is portable.

### Neutral

- Bucket naming convention: `field-mobile-{env}-evidence-sgp1` (env = staging, pilot, prod).
- Region code in bucket name aids future migration audits.

## Compliance / verification

- Bucket policy denies anonymous public access; CORS restricted to the FastAPI domain.
- Access keys scoped per environment; rotated every 90 days per `security/secrets.md`.
- Daily check: assert no public-read ACL on any object via `s3 ls --recursive --summarize` cron.
- `qa/test-cases.md` TC-MEDIA-005 verifies presigned URL expiry < 15 minutes.
- Smoke test on first deploy: round-trip a 1 MB test object under 5 seconds from a `sgp1` Droplet.

## Cost projection (pilot, 50 workers)

| Asset | Volume / month | Notes |
|---|---|---|
| Photos (avg 1.5 MB × ~6/day × 22 days × 50) | ~10 GB | Within base 250 GB |
| Voice notes (Opus, ~1 MB / minute × ~5 × 30s × 22 × 50) | ~360 MB | Within base 250 GB |
| API requests (PUT + GET) | ~30k | Within Spaces quotas |
| **Estimated bill** | **$5/month** | Flat rate |

Production year-1 (1000 workers, ~200 GB/month) is projected at $10–15/month — still inside one Spaces unit if storage is recycled at 365-day retention.

## Migration path (if needed)

If a future tenant requires a different storage backend:

1. Toggle FastAPI config: `STORAGE_ENDPOINT`, `STORAGE_REGION`, `STORAGE_BUCKET`, `STORAGE_ACCESS_KEY`, `STORAGE_SECRET_KEY`.
2. Re-issue presigned URLs against the new endpoint.
3. Run a one-off `rclone copy` from old bucket to new (background, throttled).
4. Flip read endpoint after copy verification.

No mobile change required — the mobile app receives presigned URLs from the backend.

## Related

- FR / NFR: FR-010, FR-011 (photos, voice), NFR-MEDIA-01 (upload p95 < 8s for 2 MB photo)
- Stories: US-DISC-005, US-MEDIA-001..006, US-CAP-009..011, US-VOICE-001..004, US-API-005
- Other ADRs: ADR-004 (FastAPI), ADR-007 (idempotency), ADR-008 (Whisper on-device)
- Decisions closed: DEC-007 (amended — DO Spaces, not AWS S3)
