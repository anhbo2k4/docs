# ADR-007: Idempotency by `client_id` (UUID v4)

**Status:** Accepted  
**Date:** 2026-05-13  
**Deciders:** Engineering Lead, Backend Lead, Mobile Lead  
**Consulted:** QA Lead, Odoo Specialist  
**Informed:** All

## Context

Mobile sync is unreliable by definition. Networks drop mid-request. The OS may kill a background task during retry. Without an idempotency model, a single user action can produce 0, 1, or many records in Odoo. Odoo writes are slow (1–3 s) and not all writes are naturally idempotent.

We need a way to guarantee **exactly-once effect** in Odoo regardless of how many times the mobile retries the same envelope.

## Decision

Every sync envelope carries a `client_id`: a UUID v4 generated on the device at the moment of the user action.

- Mobile stores `client_id` on the originating SQLite row.
- Mobile sends `client_id` as both the envelope payload field and the HTTP header `Idempotency-Key`.
- FastAPI persists each envelope keyed by `(employee_id, client_id)` with a `status` column. Duplicates are short-circuited.
- Celery task `sync_to_odoo` uses `client_id` to look up an existing Odoo external ID before creating, and stores the resulting Odoo ID in `sync_envelopes.odoo_ref`.
- Odoo custom module `field_mobile_sync` exposes a search by `external_id = client_id` to support dedup.

```
Mobile generates client_id → SQLite + envelope
FastAPI ingest: insert if new, update if same status, ignore if CONFIRMED
Celery: check Odoo for existing external_id, create if absent, update if pre-existing
```

## Rationale

- UUID v4 collision probability is negligible.
- Idempotency-Key header is a recognised pattern (Stripe, Square).
- Dedup happens at three layers (mobile DB, FastAPI, Odoo external_id), so a failure at any layer still yields exactly-once.
- Decouples mobile retry policy from server processing semantics.

## Alternatives considered

| Option | Pros | Cons | Why not chosen |
|---|---|---|---|
| Sequential server-issued IDs | Centralised | Requires roundtrip before user action; breaks offline | Fails offline-first |
| Hash of payload | Stateless | Same payload twice would dedup wrongly (e.g. two identical PPE checks at different shifts) | Semantics wrong |
| Mobile retry without idempotency | Simple | Duplicates in Odoo | Unacceptable |
| Odoo-side dedup only | Fewer moving parts | Network failures before reaching Odoo still create duplicates | Insufficient |

## Consequences

### Positive
- Safe retry from any failure point.
- Reasonable mental model for engineers.
- Enables exactly-once Odoo effects without distributed transactions.

### Negative
- Schema cost in three places (SQLite, Postgres, Odoo).
- Custom Odoo module must support `external_id = client_id` lookup.
- `client_id` must never be regenerated for the same user action.

### Neutral
- Adds 36 bytes per envelope.

## Compliance / verification

- Sync engine never inserts a sync_queue row without a `client_id`.
- Backend rejects envelopes missing `client_id` with `400 MISSING_CLIENT_ID`.
- Test cases TC-SYNC-024 and TC-ODOO-024 verify dedup under retry storms.

## Related

- FR-012, FR-013, FR-014, FR-015
- ADR-003, ADR-004, ADR-005
- `data/sync-state-machine.md`, `api-contracts/sync.md`
