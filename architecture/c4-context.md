# C4 Level 1 — System Context

**Status:** Active  
**Owner:** Engineering Lead

## Purpose

Show the unified Flutter app in its environment: who uses it, what external systems it depends on.

## Diagram

```mermaid
flowchart TB
    subgraph Users
        FW[Field Worker / Contractor]
        SUP[Field Supervisor / Manager]
        ADM[ERP Administrator]
    end

    subgraph "Unified Mobile App (in scope)"
        APP[Flutter Mobile App<br/>iOS + Android<br/>offline-first]
    end

    subgraph "Backend Services (in scope)"
        API[FastAPI Integration Backend<br/>Celery + Redis]
    end

    subgraph "External Systems"
        SUPA[Supabase Edge Functions<br/>OTP issuance]
        TW[Twilio<br/>SMS delivery]
        OD[Odoo ERP<br/>JSON-RPC]
        PG[PostgreSQL<br/>FastAPI metadata]
        S3[Object Storage<br/>S3-compatible<br/>photos + audio]
    end

    FW -->|uses| APP
    SUP -->|uses| APP
    ADM -->|reviews data in| OD

    APP -->|HTTPS / OTP request| SUPA
    SUPA -->|sends SMS| TW
    APP -->|HTTPS REST + JWT| API
    API -->|JSON-RPC| OD
    API -->|reads / writes| PG
    API -->|presigned uploads| S3
    APP -.->|direct upload via presigned URL| S3
```

## Actors

| Actor | Description | Authentication |
|---|---|---|
| **Field Worker** | Performs shifts, captures evidence, completes forms | Phone OTP |
| **Field Supervisor** | Oversees shifts, reviews submissions in Odoo | Phone OTP (mobile) and Odoo SSO (web) |
| **ERP Administrator** | Maintains employees, projects, and Odoo config | Odoo native auth |

## External systems

| System | Why it exists | Owner | SLA expectation |
|---|---|---|---|
| **Supabase Edge Functions** | OTP request handling | Vendor (Supabase) | 99.9 % |
| **Twilio** | SMS delivery for OTP | Vendor (Twilio) | 99.95 % |
| **Odoo ERP** | System of record for HR, projects, tasks | Internal IT | TBD (DEC-008) |
| **PostgreSQL** | FastAPI's metadata (sessions, sync_queue mirror, audit) | DevOps | 99.9 % |
| **Object Storage** | Durable storage for photos and audio | DevOps | 99.95 % |

## Trust boundaries

```
[ Internet ] --- [ Supabase Edge ] --- [ Twilio ]
[ Internet ] --- [ FastAPI ] --- [ Internal: Odoo, Postgres ]
[ Internet ] --- [ S3 (presigned) ]
```

- The mobile app is untrusted. Validate everything server-side.
- Odoo is internal-only. Never expose to internet.
- Object storage uses presigned URLs scoped per upload.

See [non-functional-requirements.md](./non-functional-requirements.md) for SLOs we commit to.
