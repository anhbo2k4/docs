# QA

**Status:** Active  
**Owner:** QA Lead

| File | Purpose |
|---|---|
| [test-strategy.md](./test-strategy.md) | Pyramid, tooling, ownership, reporting |
| [test-cases.md](./test-cases.md) | Catalog of TC-AREA-NNN test cases |
| [pilot-gate.md](./pilot-gate.md) | Go / No-Go checklist for pilot launch |
| [device-matrix.md](./device-matrix.md) | Devices, tiers, coverage |
| [load-plan.md](./load-plan.md) | Backend load test scenarios (LP-001..008) |
| [uat-plan.md](./uat-plan.md) | Pilot UAT scripts (S-UAT-01..15) and sign-off |

## Test case ID convention

`TC-<AREA>-<NNN>` where `<AREA>` is one of: `AUTH`, `SHIFT`, `PPE`, `FORM`, `GPS`, `MEDIA`, `VOICE`, `OFF`, `SYNC`, `ODOO`, `SEC`, `PERF`.

## Reading guide

- Engineers: read `test-strategy.md` and the TCs for the area you own.
- QA: own the test catalog, drive pilot gate.
- PO: read `pilot-gate.md` and use it as the release gate.
