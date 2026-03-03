# Leitor de Faturas de Energia — Cross-Project Implementation Spec

**Version:** 1.4.0
**Date:** 2026-03-03
**Feature Doc:** `docs/02_feature_leitor_de_faturas.md`
**Sample Invoices:** `samples/faturas-de-energia/`

### Changelog (v1.3.0 → v1.4.0)

| # | Category | Change |
|---|----------|--------|
| 1 | **Critical** | Fixed Nexus auth — replaced non-existent `get_authenticated_client` (from `auth.oauth`) with actual `require_service_auth` (from `dependencies.service_auth`), returns `ServiceTokenClaims` |
| 2 | **Critical** | Resolved OpenAIAgent image passing — committed to Approach A (extend `run()` to accept `str \| list[dict]`) as a blocking prerequisite, removed Approach B fallback |
| 3 | **Important** | Fixed `kwh_price` derivation order — `total_energy_value / consumption` (more accurate) now checked before `te + tusd` (approximation), matching the stated preference |
| 4 | **Important** | Fixed `demand_tariff` and `demand_tariff_peak` unit descriptions — corrected from "R$" to "R$/kW" in API contract (Section 2.1) and Pydantic models (Section 2.4) to match agent prompt |
| 5 | **Important** | Fixed `provider` parameter in agent factory — replaced hardcoded `provider="openai"` with dynamic lookup from `AVAILABLE_MODELS[model]["provider"]` for multi-provider support |
| 6 | **Important** | Added `factories.py` → `factories/` package migration prerequisite — `nexus_ai/factories.py` must be renamed to `factories/__init__.py` before creating `factories/energy_invoice.py` |
| 7 | **Important** | Added `isSystemAdmin` to backend login HTTP response body — frontend reads response body (not JWT), so the field must be in both `jwt.sign()` and `res.json()` in `logUser.ts` and `logClientWithToken.ts` |
| 8 | **Important** | Added concessionaria alias resolution logic in service layer — `resolve_concessionaria()` function with `ALIAS_TO_CANONICAL` dict from Section 6, applied before enum validation |
| 9 | **Minor** | Deduplicated models list — `GET /models` endpoint now uses `AVAILABLE_MODELS` dict from Section 3.6 instead of hardcoded inline list |
| 10 | **Minor** | Typed `ExtractionResponse` group fields — replaced `Optional[dict]` with `Optional[GroupBExtraction]` etc. for OpenAPI schema preservation |
| 11 | **Minor** | Documented `extractInvoiceData` shouldLogout handling — returns `undefined` on logout, callers must check `if (!result) return` |
| 12 | **Minor** | Documented partial failure trade-off in archive → proposal persistence sequence |

### Changelog (v1.2.0 → v1.3.0)

| # | Category | Change |
|---|----------|--------|
| 1 | **Critical** | Made `GroupAVerdeExtraction` and `GroupAAzulExtraction` fields Optional — prevents Pydantic `ValidationError` on partial extraction results; success/failure now determined by success criteria logic, not model construction |
| 2 | **Critical** | Resolved `OpenAIAgent` image passing — specified Approach A (extend `run()` to accept `list[dict]` content parts) and Approach B (fallback to direct SDK), marked as `core_ai` prerequisite |
| 3 | **Critical** | Fixed Nexus auth — replaced `require_scope("energy-invoice:extract")` (user JWT auth) with `get_authenticated_client` (OAuth client credentials for service-to-service calls) |
| 4 | **Important** | Added `monthlyConsumption` and `monthlyConsumptionPeak` arrays to Group A fields in API contract (Section 2.1) — were sent by backend but undocumented |
| 5 | **Important** | Agent factory now uses `create_nexus_agent()` instead of direct `OpenAIAgent` instantiation — aligns with existing pattern, gets timeout/retry/token counting |
| 6 | **Important** | Added `RATE_LIMITED` error code for 429 responses in API contract, backend controller, and frontend error table |
| 7 | **Important** | Added `shouldLogout` check to frontend custom `fetch` in `extractInvoiceData` — prevents cryptic errors on expired tokens |
| 8 | **Important** | Clarified global error handler modification scope — backward-compatible, preserves S3 cleanup, system-wide `success: false` addition |
| 9 | **Important** | Added error handling for file download failures, GCS upload failures, and expired pre-signed URLs in file processing flow |
| 10 | **Minor** | Fixed `initialize_agent()` reference in Section 3.6 multi-provider routing note (was: `initialize_agent()`, now: agent constructor) |
| 11 | **Minor** | Fixed factory pattern — optional DI parameters with defaults, `HttpxHttpClient` instead of `HttpClient` protocol |
| 12 | **Minor** | Added `str(request.file_url)` conversion in router — Pydantic v2 `HttpUrl` is a `Url` object, not plain `str` |
| 13 | **Minor** | Fixed field mapping table formatting — removed broken 6th column, moved notes inline with field names |
| 14 | **Minor** | Added `test_energy_invoice_derivation.py` unit test for derivation rules (kwh_price fallbacks, icms calculation, tusd assignment) |
| 15 | **Minor** | Added `ucid` upper bound validation (`> 49` → 400 error) to prevent sparse array creation with hundreds of null entries |

### Changelog (v1.1.0 → v1.2.0)

| # | Category | Change |
|---|----------|--------|
| 1 | **Critical** | Fixed `OpenAIAgent` API calls — uses constructor params + `run()` method (was: `initialize_agent()`/`run_agent()`) |
| 2 | **Critical** | Added implementation note for image passing to `OpenAIAgent.run()` — needs verification against `core_ai` |
| 3 | **Critical** | Frontend `extractInvoiceData` now uses custom `fetch` + `AbortController` for 75s timeout (was: passing timeout to `sendRequest` which doesn't support it) |
| 4 | **Critical** | Added rate limiting implementation: in-memory `Map` with 60s cooldown per proposal+ucid, 429 response |
| 5 | **Important** | Standardized Group B `tusd` as R$/kWh unit rate (was: inconsistent — model said R$, agent prompt said R$/kWh) |
| 6 | **Important** | Documented `kwh_price` derivation limitation: `te + tusd` is an approximation excluding taxes |
| 7 | **Important** | Clarified `tusd` derivation from `tusd_unit_price` — both are R$/kWh, direct assignment is correct |
| 8 | **Important** | Documented relationship between new `isSystemAdmin` and existing `userIsAdmin` fields |
| 9 | **Important** | Fixed Nexus auth dependency — uses `require_scope()` pattern (was: non-existent `get_nexus_client`) |
| 10 | **Important** | Added Group A verde/azul mapping logic in service flow (agent returns single `group_a`, result splits into `group_a_verde`/`group_a_azul`) |
| 11 | **Minor** | Added `get_energy_invoice_service` dependency definition file |
| 12 | **Minor** | Added note to register energy_invoice router in `__init__.py` exports |
| 13 | **Minor** | Fixed `file_url` example to use bucket endpoint for pre-signed URLs (was: CDN URL) |
| 14 | **Minor** | Documented sparse array behavior and migration notes for `invoiceFileUrls` |
| 15 | **Minor** | Added backend telemetry logging spec (Section 4.6b) |
| 16 | **Minor** | Added migration notes for `isSystemAdmin` field (JWT re-login, manual user flagging) |

### Changelog (v1.0.0 → v1.1.0)

| # | Category | Change |
|---|----------|--------|
| 1 | **Critical** | Backend now maps Nexus HTTP errors properly: 400/422 → 422 `INVALID_FILE_FORMAT`, 401 → 500 `INTERNAL_ERROR`, 5xx/timeout → 502 `NEXUS_UNAVAILABLE` (was: all errors → 502) |
| 2 | **Critical** | Frontend loading text updated to "até 60 segundos" to match backend's 60s timeout (was: 30s) |
| 3 | **Critical** | Added multi-provider routing clarification — `OpenAIAgent` already supports non-OpenAI models via provider config |
| 4 | **Critical** | Added `customerId` ↔ proposal.customer validation; derived `resolvedCustomerId` from proposal to prevent cross-customer file archiving |
| 5 | **Significant** | API contract now explicitly documents two failure modes: HTTP errors (4xx/5xx) vs business failures (200 OK with `success: false`) |
| 6 | **Significant** | Added rate limiting: frontend button disable during extraction + backend 60s per-proposal cooldown |
| 7 | **Significant** | Added `FIELD_LABELS` Portuguese map for `missingFields` display in validation popup |
| 8 | **Significant** | Documented S3 upload-before-validation as accepted trade-off with lifecycle rule recommendation |
| 9 | **Significant** | Relaxed `demand` validation from `>= 30` to `> 0` (some Group A consumers have low demand) |
| 10 | **Significant** | `AgentExtractionOutput.tariff_modality` is now `Optional[str]` — agent returns null for non-invoice documents; service returns extraction failure with "not an energy invoice" message |
| 11 | **Minor** | `GroupAAzulExtraction` now inherits from `GroupAVerdeExtraction` instead of duplicating fields |
| 12 | **Minor** | Archive persistence logs a warning when no archive exists (was: silent skip) |
| 13 | **Minor** | `ExtractionRequest.file_url` uses `HttpUrl` instead of `str` for URL validation |
| 14 | **Minor** | Added CDN URL vs pre-signed URL clarification (public-read ACL on DO Spaces) |
| 15 | **Minor** | `ucid` now validated as non-negative integer (was: silently defaulted to 0) |
| 16 | **Minor** | Added extraction telemetry logging spec (model, success, timing, derived fields) |
| 17 | **Minor** | Frontend timeout set to 75s (> backend's 60s) to avoid premature timeout errors |
| 18 | **Minor** | `GroupBExtraction.monthly_consumption` uses `list[float]` (not `list[Optional[float]]`) to reflect post-processed state |

---

## Table of Contents

1. [Overview & Network Workflow](#1-overview--network-workflow)
2. [API Contracts](#2-api-contracts)
3. [Nexus Core Implementation Spec](#3-nexus-core-implementation-spec)
4. [Azume Backend Implementation Spec](#4-azume-backend-implementation-spec)
5. [Azume Frontend CRM Implementation Spec](#5-azume-frontend-crm-implementation-spec)
6. [Concessionárias Enum](#6-concessionárias-enum)
7. [Verification & Testing Strategy](#7-verification--testing-strategy)

---

## 1. Overview & Network Workflow

### System Diagram

```
┌──────────────────┐     ┌──────────────────┐     ┌──────────────────┐     ┌─────┐
│  Azume Frontend   │────▶│  Azume Backend    │────▶│  Nexus Core      │────▶│ LLM │
│  (React 17/MUI4) │     │  (Express.js)     │     │  (FastAPI)       │     │     │
│                  │◀────│                  │◀────│                  │◀────│     │
└──────────────────┘     └──────────────────┘     └──────────────────┘     └─────┘
                               │                         │
                               ▼                         ▼
                         ┌──────────┐              ┌──────────┐
                         │ S3/DO    │              │ GCS      │
                         │ Spaces   │              │ (temp)   │
                         └──────────┘              └──────────┘
```

### Sequence of Operations

```
1. [Frontend]  User clicks "Leitura Smart" → upload modal opens
2. [Frontend]  User selects file (+ optional model if isSystemAdmin)
3. [Frontend]  POST /api/proposals/:pid/invoice-reading (multipart/form-data)
4. [Backend]   checkAuth → validate file format/size
5. [Backend]   Upload file to S3 (DigitalOcean Spaces)
6. [Backend]   Generate pre-signed S3 URL (60 min expiry)
7. [Backend]   getNexusOAuthToken() → POST Nexus /api/v1/energy-invoice/extract
               Body: { file_url, model? }
8. [Nexus]     checkAuthNexusClient → validate request
9. [Nexus]     Download file from pre-signed URL
10. [Nexus]    Upload to GCS (temp storage for agent processing)
11. [Nexus]    If PDF: render pages as images (PyMuPDF)
12. [Nexus]    Create task agent (OpenAI Agents SDK, structured output)
13. [Nexus]    agent.run() with images → LLM extracts data
14. [Nexus]    Validate extraction against business rules
15. [Nexus]    Post-process (fill missing months with average for Group B)
16. [Nexus]    Return structured result to Azume Backend
17. [Backend]  Validate extraction result
18. [Backend]  Persist file URL in "archive" collection ("Faturas de Energia" folder)
19. [Backend]  Persist file URL in "proposal" collection (new invoiceFileUrls array, indexed by ucid)
20. [Backend]  Return extraction result to Frontend
21. [Frontend] Show validation popup with extracted fields (editable) + missing fields
22. [Frontend] User confirms → populate form fields via inputHandler
```

### System Responsibilities

| System | Responsibilities |
|--------|-----------------|
| **Frontend** | Upload UI, model selector (admin), validation popup, form population |
| **Azume Backend** | File validation, S3 upload, Nexus API orchestration, archive/proposal persistence |
| **Nexus Core** | File processing, PDF→image rendering, LLM agent execution, validation, post-processing |

---

## 2. API Contracts

### 2.1 Frontend → Azume Backend: Invoice Extraction

**Endpoint:** `POST /api/proposals/:pid/invoice-reading`

**Auth:** `checkAuth` (standard CRM JWT)

**Request:** `multipart/form-data`

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `file` | File | Yes | Invoice file (PDF or image) |
| `ucid` | number | Yes | Consumer unit index (0-based) |
| `customerId` | string | Yes | Customer MongoDB ObjectId |
| `model` | string | No | LLM model ID override (admin-only, ignored for non-admins) |

**File Validation Rules:**

| Rule | Value |
|------|-------|
| Accepted MIME types | `image/jpeg`, `image/png`, `image/gif`, `image/webp`, `application/pdf` |
| Accepted extensions | `.jpg`, `.jpeg`, `.png`, `.gif`, `.webp`, `.pdf` |
| Max file size | 20 MB |

**Success Response:** `200 OK`

```typescript
{
  success: true,
  extraction: {
    tariffModality: "A" | "B",
    classification?: "Azul" | "Verde",      // Group A only
    fields: {
      // Group B fields
      powerDistCompany?: string,             // Concessionária enum value
      kwhPrice?: number,                     // R$/kWh (> 0, < 10)
      publicLightBill?: number,              // R$ (> 0)
      networkClass?: "Trifásica" | "Bifásica" | "Monofásica",
      monthlyConsumption?: number[],         // 12 entries (kWh, >= 0)
      tusd?: number,                         // R$/kWh (>= 0)
      icms?: number,                         // % (> 0, < 100)

      // Group A Verde fields (in addition to/instead of Group B)
      kwhPricePeak?: number,                 // R$/kWh (> 0, < 10)
      tusdPeak?: number,                     // R$/kWh (> 0, < 10)
      demand?: number,                       // kW (> 0)
      demandTariff?: number,                 // R$/kW (> 0)
      averageConsumption?: number,           // kWh (> 0) — replicated to 12 months
      averageConsumptionPeak?: number,       // kWh (> 0) — replicated to 12 months
      monthlyConsumption?: number[],         // 12 entries (kWh) — replicated from averageConsumption
      monthlyConsumptionPeak?: number[],     // 12 entries (kWh) — replicated from averageConsumptionPeak

      // Group A Azul additional fields
      demandPeak?: number,                   // kW (> 0)
      demandTariffPeak?: number,             // R$/kW (> 0)
    },
    missingFields: string[],                 // Field names the agent could not extract
    invoiceFileUrl: string,                  // Permanent S3 URL of the uploaded invoice
  }
}
```

**Business Failure Response:** `200 OK` (extraction ran but mandatory fields could not be extracted)

> **Note:** Extraction failure is a business outcome, not an HTTP error. The LLM processed the
> file but couldn't extract required fields. The frontend must handle **both** HTTP errors
> (status >= 400) **and** 200 OK responses with `success: false`.

```typescript
{
  success: false,
  message: string,                           // Portuguese user-facing message
  errorCode: "EXTRACTION_FAILED",            // Always EXTRACTION_FAILED for business failures
  missingFields: string[],                   // Field names the agent could not extract
}
```

**HTTP Error Response:** `400 | 403 | 404 | 422 | 429 | 500 | 502`

```typescript
{
  success: false,
  message: string,                           // Portuguese user-facing message
  errorCode: "INVALID_FILE_FORMAT"           // Machine-readable error code
    | "FILE_TOO_LARGE"
    | "EXTRACTION_FAILED"                    // File could not be processed by Nexus (422)
    | "NEXUS_UNAVAILABLE"                    // Nexus API unreachable/timeout (502)
    | "INVALID_PROPOSAL"                     // Proposal not found or customerId mismatch (404/400)
    | "UNAUTHORIZED"                         // Auth failure (403)
    | "RATE_LIMITED"                         // Too many requests for same proposal+ucid (429)
    | "INTERNAL_ERROR",                      // Server error (500)
  missingFields?: string[],                  // Only present for EXTRACTION_FAILED
}
```

---

### 2.2 Frontend → Azume Backend → Nexus: Available Models

**Purpose:** System admins can select which LLM model to use for extraction. This endpoint returns the list of available mid-tier vision models.

#### Nexus Endpoint

**Endpoint:** `GET /api/v1/energy-invoice/models`

**Auth:** `checkAuthNexusClient` (Nexus OAuth client credentials)

**Response:** `200 OK`

```json
{
  "models": [
    {
      "id": "gpt-5-mini",
      "display_name": "GPT-5 Mini",
      "provider": "openai",
      "is_default": true
    },
    {
      "id": "gemini-3-flash-preview",
      "display_name": "Gemini 3 Flash",
      "provider": "google",
      "is_default": false
    },
    {
      "id": "claude-sonnet-4-5",
      "display_name": "Claude Sonnet 4.5",
      "provider": "anthropic",
      "is_default": false
    },
    {
      "id": "grok-4-1-fast",
      "display_name": "Grok 4.1 Fast",
      "provider": "xai",
      "is_default": false
    }
  ]
}
```

#### Azume Backend Proxy Endpoint

**Endpoint:** `GET /api/proposals/invoice-reading/models`

**Auth:** `checkAuth` (standard CRM JWT)

**Behavior:**
- Backend calls Nexus `GET /api/v1/energy-invoice/models` with cached OAuth token
- Cache the response for 1 hour (models list changes infrequently)
- Only accessible when user has `isSystemAdmin === true` (return 403 otherwise)

**Response:** Same structure as Nexus response above.

---

### 2.3 Azume Backend → Nexus Core: Extraction

**Endpoint:** `POST /api/v1/energy-invoice/extract`

**Auth:** Nexus OAuth client credentials (`checkAuthNexusClient` — existing pattern)

**Request:** `application/json`

```json
{
  "file_url": "https://azume-files.nyc3.digitaloceanspaces.com/uploads/archives/xxx/invoice.pdf?X-Amz-...",
  "model": "gpt-5-mini"
}
```

> **Note on URL format:** The `file_url` is a **pre-signed S3 URL** generated by the backend (60 min expiry). Pre-signed URLs use the **bucket endpoint** (`azume-files.nyc3.digitaloceanspaces.com`), not the CDN endpoint (`azume-files.cdn.digitaloceanspaces.com`). The CDN URL is used for permanent public access (e.g., `invoiceFileUrl` returned to the frontend), while the pre-signed URL is used for temporary Nexus access.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `file_url` | string (URL) | Yes | Pre-signed S3 URL to the invoice file |
| `model` | string | No | LLM model ID. Default: `gpt-5-mini` |

**Success Response:** `200 OK`

```json
{
  "success": true,
  "tariff_modality": "B",
  "classification": null,
  "group_b": {
    "power_dist_company": "CPFL PAULISTA",
    "kwh_price": 0.85,
    "public_light_bill": 42.50,
    "network_class": "Trifásica",
    "monthly_consumption": [320, 280, 350, 310, 290, 270, 300, 340, 330, 360, 310, 290],
    "tusd": 0.35,
    "icms": 18.0
  },
  "group_a_verde": null,
  "group_a_azul": null,
  "missing_fields": ["tusd"],
  "model_used": "gpt-5-mini"
}
```

**Group A Verde example:**

```json
{
  "success": true,
  "tariff_modality": "A",
  "classification": "Verde",
  "group_b": null,
  "group_a_verde": {
    "power_dist_company": "COPEL-DIS",
    "kwh_price": 0.42,
    "kwh_price_peak": 1.15,
    "tusd": 0.28,
    "tusd_peak": 0.75,
    "demand": 120.0,
    "demand_tariff": 32.50,
    "public_light_bill": 85.00,
    "average_consumption": 4500,
    "average_consumption_peak": 1200,
    "icms": 18.0
  },
  "group_a_azul": null,
  "missing_fields": [],
  "model_used": "gpt-5-mini"
}
```

**Group A Azul example:**

```json
{
  "success": true,
  "tariff_modality": "A",
  "classification": "Azul",
  "group_b": null,
  "group_a_verde": null,
  "group_a_azul": {
    "power_dist_company": "ENEL SP",
    "kwh_price": 0.38,
    "kwh_price_peak": 1.05,
    "tusd": 0.25,
    "tusd_peak": 0.68,
    "demand": 150.0,
    "demand_tariff": 28.00,
    "demand_peak": 80.0,
    "demand_tariff_peak": 45.00,
    "public_light_bill": 120.00,
    "average_consumption": 5800,
    "average_consumption_peak": 1500,
    "icms": 25.0
  },
  "missing_fields": ["icms"],
  "model_used": "gpt-5-mini"
}
```

**Failure Response:** `200 OK` (extraction failure is a business outcome, not an HTTP error)

```json
{
  "success": false,
  "tariff_modality": null,
  "classification": null,
  "group_b": null,
  "group_a_verde": null,
  "group_a_azul": null,
  "missing_fields": ["kwh_price", "network_class", "monthly_consumption"],
  "model_used": "gpt-5-mini",
  "error_message": "Não foi possível extrair os campos obrigatórios da fatura de energia."
}
```

**HTTP Error Responses:**

| Status | Condition |
|--------|-----------|
| 400 | Invalid request body (missing file_url, invalid model ID) |
| 401 | Invalid/missing OAuth token |
| 422 | File could not be downloaded or processed (corrupt, unsupported) |
| 500 | Internal server error |

**Timeout Considerations:**
- LLM processing: 10-30 seconds typical
- Nexus should set LLM agent timeout to **45 seconds**
- Azume Backend should set HTTP timeout to **60 seconds** for Nexus calls
- Azume Frontend should set HTTP timeout to **75 seconds** (> backend's 60s, to avoid confusing errors where the frontend times out before the backend)

---

### 2.4 Response Data Schemas

#### Pydantic Models (Nexus)

```python
# packages/nexus/nexus-models/src/nexus_models/energy_invoice.py

from pydantic import BaseModel, Field
from typing import Optional
from enum import Enum


class TariffModality(str, Enum):
    B = "B"
    A = "A"


class HoroSeasonalClassification(str, Enum):
    AZUL = "Azul"
    VERDE = "Verde"


class NetworkClass(str, Enum):
    TRIFASICA = "Trifásica"
    BIFASICA = "Bifásica"
    MONOFASICA = "Monofásica"


class GroupBExtraction(BaseModel):
    """Grupo B (baixa tensão) extraction result."""
    power_dist_company: Optional[str] = Field(
        default=None,
        description="Power distribution company name (concessionária)"
    )
    kwh_price: Optional[float] = Field(
        default=None, gt=0, lt=10,
        description="kWh price in R$ (including all taxes)"
    )
    public_light_bill: Optional[float] = Field(
        default=None, gt=0,
        description="Public lighting tax (CIP/COSIP) in R$"
    )
    network_class: Optional[NetworkClass] = Field(
        default=None,
        description="Network class: Trifásica, Bifásica, or Monofásica"
    )
    monthly_consumption: Optional[list[float]] = Field(
        default=None, min_length=12, max_length=12,
        description="Monthly consumption in kWh (Jan-Dec). After post-processing, all 12 entries are guaranteed to be floats (missing months filled with average)."
    )
    tusd: Optional[float] = Field(
        default=None, ge=0,
        description="TUSD tariff in R$/kWh (unit rate, not total charge)"
    )
    icms: Optional[float] = Field(
        default=None, gt=0, lt=100,
        description="ICMS tax percentage"
    )


class GroupAVerdeExtraction(BaseModel):
    """Grupo A Verde (alta tensão, horo-sazonal verde) extraction result.

    All fields are Optional to allow partial extraction results. Success/failure
    is determined by the success criteria logic (Section 3.1), not by model
    construction — this prevents Pydantic ValidationError when the LLM fails
    to extract a mandatory Group A field.
    """
    power_dist_company: Optional[str] = Field(
        default=None,
        description="Power distribution company name (concessionária)"
    )
    kwh_price: Optional[float] = Field(
        default=None, gt=0, lt=10,
        description="TE Fora Ponta (off-peak energy tariff) in R$"
    )
    kwh_price_peak: Optional[float] = Field(
        default=None, gt=0, lt=10,
        description="TE Ponta (peak energy tariff) in R$"
    )
    tusd: Optional[float] = Field(
        default=None, gt=0, lt=10,
        description="TUSD Fora Ponta (off-peak distribution tariff) in R$"
    )
    tusd_peak: Optional[float] = Field(
        default=None, gt=0, lt=10,
        description="TUSD Ponta (peak distribution tariff) in R$"
    )
    demand: Optional[float] = Field(
        default=None, gt=0,
        description="Contracted demand in kW"
    )
    demand_tariff: Optional[float] = Field(
        default=None, gt=0,
        description="Demand tariff in R$/kW"
    )
    public_light_bill: Optional[float] = Field(
        default=None, gt=0,
        description="Public lighting tax (CIP/COSIP) in R$"
    )
    average_consumption: Optional[float] = Field(
        default=None, gt=0,
        description="Average monthly off-peak consumption in kWh"
    )
    average_consumption_peak: Optional[float] = Field(
        default=None, gt=0,
        description="Average monthly peak consumption in kWh"
    )
    icms: Optional[float] = Field(
        default=None, gt=0, lt=100,
        description="ICMS tax percentage"
    )
    monthly_consumption: Optional[list[float]] = Field(
        default=None, min_length=12, max_length=12,
        description="Monthly off-peak consumption in kWh (replicated from average_consumption)"
    )
    monthly_consumption_peak: Optional[list[float]] = Field(
        default=None, min_length=12, max_length=12,
        description="Monthly peak consumption in kWh (replicated from average_consumption_peak)"
    )


class GroupAAzulExtraction(GroupAVerdeExtraction):
    """Grupo A Azul (alta tensão, horo-sazonal azul) extraction result.
    Inherits all Verde fields and adds peak demand fields.
    """
    demand_peak: Optional[float] = Field(
        default=None, gt=0,
        description="Contracted peak demand in kW"
    )
    demand_tariff_peak: Optional[float] = Field(
        default=None, gt=0,
        description="Peak demand tariff in R$/kW"
    )


class InvoiceExtractionResult(BaseModel):
    """Top-level extraction result returned by Nexus."""
    success: bool
    tariff_modality: Optional[TariffModality] = None
    classification: Optional[HoroSeasonalClassification] = None
    group_b: Optional[GroupBExtraction] = None
    group_a_verde: Optional[GroupAVerdeExtraction] = None
    group_a_azul: Optional[GroupAAzulExtraction] = None
    missing_fields: list[str] = Field(default_factory=list)
    model_used: str
    error_message: Optional[str] = None
```

#### Agent Output Models (Structured Output for LLM)

These are the models passed as `output_type` to the OpenAI Agents SDK agent. They differ from the API response models — the agent returns raw extracted values, and the service layer validates and transforms them.

```python
# packages/nexus/nexus-models/src/nexus_models/energy_invoice_agent.py

from pydantic import BaseModel, Field
from typing import Optional


class AgentGroupBOutput(BaseModel):
    """Structured output for Grupo B extraction by the LLM agent."""
    power_dist_company: Optional[str] = Field(
        default=None,
        description="Nome da concessionária de energia elétrica"
    )
    kwh_price: Optional[float] = Field(
        default=None,
        description="Valor do kWh em R$ (incluindo todos os tributos)"
    )
    public_light_bill: Optional[float] = Field(
        default=None,
        description="Taxa de iluminação pública (CIP/COSIP) em R$"
    )
    network_class: Optional[str] = Field(
        default=None,
        description="Classe de rede: 'Trifásica', 'Bifásica' ou 'Monofásica'"
    )
    jan: Optional[float] = Field(default=None, description="Consumo janeiro em kWh")
    feb: Optional[float] = Field(default=None, description="Consumo fevereiro em kWh")
    mar: Optional[float] = Field(default=None, description="Consumo março em kWh")
    apr: Optional[float] = Field(default=None, description="Consumo abril em kWh")
    may: Optional[float] = Field(default=None, description="Consumo maio em kWh")
    jun: Optional[float] = Field(default=None, description="Consumo junho em kWh")
    jul: Optional[float] = Field(default=None, description="Consumo julho em kWh")
    aug: Optional[float] = Field(default=None, description="Consumo agosto em kWh")
    sep: Optional[float] = Field(default=None, description="Consumo setembro em kWh")
    oct: Optional[float] = Field(default=None, description="Consumo outubro em kWh")
    nov: Optional[float] = Field(default=None, description="Consumo novembro em kWh")
    dec: Optional[float] = Field(default=None, description="Consumo dezembro em kWh")
    tusd: Optional[float] = Field(default=None, description="Tarifa TUSD em R$")
    icms: Optional[float] = Field(
        default=None,
        description="Percentual de ICMS (%)"
    )

    # Raw component fields for deterministic derivation
    total_energy_value: Optional[float] = Field(
        default=None,
        description="Valor total de energia elétrica na fatura em R$ (inclui TE, TUSD, encargos)"
    )
    current_month_consumption: Optional[float] = Field(
        default=None,
        description="Consumo do mês atual em kWh (usado para derivar kWh price se ausente)"
    )
    te_unit_price: Optional[float] = Field(
        default=None,
        description="Tarifa de Energia (TE) unitária em R$/kWh, se listada separadamente"
    )
    tusd_unit_price: Optional[float] = Field(
        default=None,
        description="TUSD unitária em R$/kWh, se listada separadamente"
    )
    icms_value: Optional[float] = Field(
        default=None,
        description="Valor absoluto do ICMS em R$ (para derivar percentual)"
    )
    icms_base: Optional[float] = Field(
        default=None,
        description="Base de cálculo do ICMS em R$ (para derivar percentual)"
    )


class AgentGroupAOutput(BaseModel):
    """Structured output for Grupo A extraction by the LLM agent.
    Covers both Verde and Azul — Azul-only fields are optional.
    """
    power_dist_company: Optional[str] = Field(
        default=None,
        description="Nome da concessionária de energia elétrica"
    )
    classification: Optional[str] = Field(
        default=None,
        description="Classificação horo-sazonal: 'Azul' ou 'Verde'"
    )
    kwh_price: Optional[float] = Field(
        default=None,
        description="TE Fora Ponta (tarifa de energia fora ponta) em R$/kWh"
    )
    kwh_price_peak: Optional[float] = Field(
        default=None,
        description="TE Ponta (tarifa de energia ponta) em R$/kWh"
    )
    tusd: Optional[float] = Field(
        default=None,
        description="TUSD Fora Ponta em R$/kWh"
    )
    tusd_peak: Optional[float] = Field(
        default=None,
        description="TUSD Ponta em R$/kWh"
    )
    demand: Optional[float] = Field(
        default=None,
        description="Demanda contratada (fora ponta para Azul, única para Verde) em kW"
    )
    demand_tariff: Optional[float] = Field(
        default=None,
        description="Tarifa da demanda (fora ponta para Azul, única para Verde) em R$"
    )
    demand_peak: Optional[float] = Field(
        default=None,
        description="Demanda contratada ponta em kW (somente Azul)"
    )
    demand_tariff_peak: Optional[float] = Field(
        default=None,
        description="Tarifa da demanda ponta em R$ (somente Azul)"
    )
    public_light_bill: Optional[float] = Field(
        default=None,
        description="Taxa de iluminação pública (CIP/COSIP) em R$"
    )
    average_consumption: Optional[float] = Field(
        default=None,
        description="Consumo médio mensal fora ponta em kWh"
    )
    average_consumption_peak: Optional[float] = Field(
        default=None,
        description="Consumo médio mensal ponta em kWh"
    )
    icms: Optional[float] = Field(
        default=None,
        description="Percentual de ICMS (%)"
    )

    # Raw component fields for deterministic derivation
    icms_value: Optional[float] = Field(
        default=None,
        description="Valor absoluto do ICMS em R$ (para derivar percentual)"
    )
    icms_base: Optional[float] = Field(
        default=None,
        description="Base de cálculo do ICMS em R$ (para derivar percentual)"
    )


class AgentExtractionOutput(BaseModel):
    """Top-level structured output from the LLM agent."""
    tariff_modality: Optional[str] = Field(
        default=None,
        description="Modalidade tarifária: 'A', 'B', ou null se o documento não for uma fatura de energia"
    )
    group_b: Optional[AgentGroupBOutput] = Field(
        default=None,
        description="Dados extraídos do Grupo B (preenchido somente se tariff_modality='B')"
    )
    group_a: Optional[AgentGroupAOutput] = Field(
        default=None,
        description="Dados extraídos do Grupo A (preenchido somente se tariff_modality='A')"
    )
```

#### Field Mapping Table

| Feature Doc Name | Nexus Agent Field | Nexus API Field | Azume Backend Field | Frontend Form ID |
|------------------|-------------------|-----------------|--------------------|--------------------|
| **Common** | | | | |
| Modalidade Tarifária | `tariff_modality` | `tariff_modality` | `tariffModality` | `tariffModality` |
| **Group B** | | | | |
| Concessionária | `group_b.power_dist_company` | `group_b.power_dist_company` | `powerDistCompany` | `powerDistCompany` |
| Valor kWh | `group_b.kwh_price` | `group_b.kwh_price` | `kwhPrice` | `kwhPrice` |
| Taxa Ilum. Pública | `group_b.public_light_bill` | `group_b.public_light_bill` | `publicLightBill` | `publicLightBill` |
| Rede | `group_b.network_class` | `group_b.network_class` | `networkClass` | `networkClass` |
| Consumo Jan | `group_b.jan` | `group_b.monthly_consumption[0]` | `monthlyConsumption[0]` | `jan` |
| Consumo Fev | `group_b.feb` | `group_b.monthly_consumption[1]` | `monthlyConsumption[1]` | `feb` |
| Consumo Mar | `group_b.mar` | `group_b.monthly_consumption[2]` | `monthlyConsumption[2]` | `mar` |
| Consumo Abr | `group_b.apr` | `group_b.monthly_consumption[3]` | `monthlyConsumption[3]` | `apr` |
| Consumo Mai | `group_b.may` | `group_b.monthly_consumption[4]` | `monthlyConsumption[4]` | `may` |
| Consumo Jun | `group_b.jun` | `group_b.monthly_consumption[5]` | `monthlyConsumption[5]` | `jun` |
| Consumo Jul | `group_b.jul` | `group_b.monthly_consumption[6]` | `monthlyConsumption[6]` | `jul` |
| Consumo Ago | `group_b.aug` | `group_b.monthly_consumption[7]` | `monthlyConsumption[7]` | `aug` |
| Consumo Set | `group_b.sep` | `group_b.monthly_consumption[8]` | `monthlyConsumption[8]` | `sep` |
| Consumo Out | `group_b.oct` | `group_b.monthly_consumption[9]` | `monthlyConsumption[9]` | `oct` |
| Consumo Nov | `group_b.nov` | `group_b.monthly_consumption[10]` | `monthlyConsumption[10]` | `nov` |
| Consumo Dez | `group_b.dec` | `group_b.monthly_consumption[11]` | `monthlyConsumption[11]` | `dec` |
| TUSD | `group_b.tusd` | `group_b.tusd` | `tusd` | `tusd` |
| ICMS | `group_b.icms` | `group_b.icms` | `icms` | `icms` |
| Valor Total Energia *(derivation only)* | `group_b.total_energy_value` | — | — | — |
| Consumo Mês Atual *(derivation only)* | `group_b.current_month_consumption` | — | — | — |
| TE Unitária *(derivation only)* | `group_b.te_unit_price` | — | — | — |
| TUSD Unitária *(derivation only)* | `group_b.tusd_unit_price` | — | — | — |
| ICMS Valor (B) *(derivation only)* | `group_b.icms_value` | — | — | — |
| ICMS Base (B) *(derivation only)* | `group_b.icms_base` | — | — | — |
| **Group A** | | | | |
| Classificação | `group_a.classification` | `classification` | `classification` | `classification` |
| TE Fora Ponta | `group_a.kwh_price` | `group_a_verde.kwh_price` or `group_a_azul.kwh_price` | `kwhPrice` | `kwhPrice` |
| TE Ponta | `group_a.kwh_price_peak` | `*.kwh_price_peak` | `kwhPricePeak` | `kwhPricePeak` |
| TUSD Fora Ponta | `group_a.tusd` | `*.tusd` | `tusd` | `tusd` |
| TUSD Ponta | `group_a.tusd_peak` | `*.tusd_peak` | `tusdPeak` | `tusdPeak` |
| Demanda Contratada | `group_a.demand` | `*.demand` | `demand` | `demand` |
| Tarifa da Demanda | `group_a.demand_tariff` | `*.demand_tariff` | `demandTariff` | `demandTariff` |
| Demanda Contrat. Ponta | `group_a.demand_peak` | `group_a_azul.demand_peak` | `demandPeak` | `demandPeak` |
| Tarifa Demanda Ponta | `group_a.demand_tariff_peak` | `group_a_azul.demand_tariff_peak` | `demandTariffPeak` | `demandTariffPeak` |
| Taxa Ilum. Pública | `group_a.public_light_bill` | `*.public_light_bill` | `publicLightBill` | `publicLightBill` |
| Consumo Médio FP | `group_a.average_consumption` | `*.average_consumption` | `monthlyConsumption[0-11]` | `averageValue` |
| Consumo Médio Ponta | `group_a.average_consumption_peak` | `*.average_consumption_peak` | `monthlyConsumptionPeak[0-11]` | `averageValuePeak` |
| ICMS | `group_a.icms` | `*.icms` | `icms` | `icms` |
| Consumo Mensal FP *(post-processed)* | — | `*.monthly_consumption` | `monthlyConsumption` | `jan`-`dec` |
| Consumo Mensal Ponta *(post-processed)* | — | `*.monthly_consumption_peak` | `monthlyConsumptionPeak` | — |
| ICMS Valor (A) *(derivation only)* | `group_a.icms_value` | — | — | — |
| ICMS Base (A) *(derivation only)* | `group_a.icms_base` | — | — | — |

> **Note on `kwh_price`:** For Group B this is the all-inclusive price (TE + TUSD + taxes). For Group A this is the TE component only (TUSD is a separate field). Same field name, different semantics — this matches domain conventions where Group A tariffs are always disaggregated.

---

## 3. Nexus Core Implementation Spec

### 3.1 Package Responsibilities

#### `nexus-models` — Pydantic Extraction Models

**New file:** `packages/nexus/nexus-models/src/nexus_models/energy_invoice.py`

Contains the Pydantic models from Section 2.4:
- `TariffModality`, `HoroSeasonalClassification`, `NetworkClass` enums
- `GroupBExtraction`, `GroupAVerdeExtraction`, `GroupAAzulExtraction` data models
- `InvoiceExtractionResult` top-level result model

**New file:** `packages/nexus/nexus-models/src/nexus_models/energy_invoice_agent.py`

Contains the agent structured output models from Section 2.4:
- `AgentGroupBOutput`, `AgentGroupAOutput` — per-group extraction
- `AgentExtractionOutput` — top-level agent output (used as `output_type`)

#### `nexus-services` — Extraction Service

**New file:** `packages/nexus/nexus-services/src/nexus_services/protocols/energy_invoice_service.py`

```python
from typing import Protocol
from dataclasses import dataclass


@dataclass(frozen=True)
class EnergyInvoiceExtractionInput:
    """Input for energy invoice extraction."""
    file_url: str                   # Pre-signed S3 URL
    model: str = "gpt-5-mini"       # LLM model ID


class EnergyInvoiceService(Protocol):
    """Protocol for energy invoice extraction operations."""

    async def extract_invoice_data(
        self,
        extraction_input: EnergyInvoiceExtractionInput,
    ) -> "InvoiceExtractionResult":
        """
        Download file → process → run agent → validate → post-process → return.
        """
        ...
```

**New file:** `packages/nexus/nexus-services/src/nexus_services/impl/energy_invoice_service_impl.py`

Implementation responsibilities:
1. Download file from pre-signed URL (httpx)
2. Detect file type from Content-Type / extension
3. Upload to GCS temp bucket
4. If PDF: render pages as images using PyMuPDF (`fitz`)
5. If image: use directly
6. Call agent factory to create extraction agent
7. Run agent with images → get `AgentExtractionOutput`
8. Validate extracted values against business rules
9. Apply deterministic derivation rules (derive missing fields from raw components)
10. Post-process (fill missing months for Group B, replicate averages for Group A)
11. Map `AgentExtractionOutput` → `InvoiceExtractionResult`
12. Determine success/failure based on mandatory field presence

**Validation Rules:**

| Field | Rule | Action on Violation |
|-------|------|-------------------|
| `kwh_price` | `> 0 AND < 10` | Mark as missing |
| `kwh_price_peak` | `> 0 AND < 10` | Mark as missing |
| `tusd` (Group A) | `> 0 AND < 10` | Mark as missing |
| `tusd_peak` | `> 0 AND < 10` | Mark as missing |
| `demand` | `> 0` | Mark as missing |
| `demand_tariff` | `> 0` | Mark as missing |
| `demand_peak` | `> 0` | Mark as missing |
| `demand_tariff_peak` | `> 0` | Mark as missing |
| `network_class` | Must be one of enum values | Mark as missing |
| `monthly_consumption[i]` | `>= 0` | Set to None (missing) |
| `public_light_bill` | `> 0` | Set to None (best-effort) |
| `tusd` (Group B) | `>= 0` | Set to None (best-effort) |
| `icms` | `> 0 AND < 100` | Set to None (best-effort) |
| `power_dist_company` | Must match concessionária enum after alias resolution (see below) | Set to None (best-effort) |
| PDF page count | <= 5 pages | Return 422 with "PDF com muitas páginas. Máximo: 5 páginas." |
| `classification` | Must be "Azul" or "Verde" | Mark as missing |

**Concessionária Alias Resolution (applied BEFORE validation):**

The LLM agent may return alias names (e.g., "ELETROPAULO") instead of canonical names (e.g., "ENEL SP"). The service layer must resolve aliases before validating against the enum:

```python
def resolve_concessionaria(name: Optional[str]) -> Optional[str]:
    """Resolve concessionária alias to canonical name.

    Uses the alias mapping from Section 6. Returns the canonical name if found,
    or the original name (uppercased) if it's already canonical. Returns None
    if the name doesn't match any canonical name or alias.
    """
    if name is None:
        return None
    normalized = name.strip().upper()
    # Check canonical names first
    if normalized in CONCESSIONARIAS:
        return normalized
    # Check aliases
    return ALIAS_TO_CANONICAL.get(normalized)
```

The `ALIAS_TO_CANONICAL` dict should be built from Section 6's alias table (e.g., `{"ELETROPAULO": "ENEL SP", "AES ELETROPAULO": "ENEL SP", "CPFL": "CPFL PAULISTA", ...}`). This dict lives in the service implementation alongside the validation logic.

**Deterministic Derivation Rules (applied AFTER validation, BEFORE post-processing):**

```python
def derive_missing_fields_group_b(extraction: AgentGroupBOutput) -> AgentGroupBOutput:
    """Derive missing Group B fields from raw components.
    Applied after validation, before post-processing.
    """
    # 1. kwh_price: derive from total / consumption (preferred), or te + tusd (fallback)
    #    total_energy_value / consumption is more accurate because it implicitly
    #    includes taxes (ICMS, PIS/COFINS). te + tusd is an APPROXIMATION that
    #    excludes taxes and underestimates the all-inclusive price.
    if extraction.kwh_price is None:
        if (extraction.total_energy_value is not None
              and extraction.current_month_consumption is not None
              and extraction.current_month_consumption > 0):
            extraction.kwh_price = round(
                extraction.total_energy_value / extraction.current_month_consumption, 6
            )
        elif extraction.te_unit_price is not None and extraction.tusd_unit_price is not None:
            extraction.kwh_price = round(extraction.te_unit_price + extraction.tusd_unit_price, 6)

    # 2. icms: derive percentage from absolute values
    if extraction.icms is None:
        if (extraction.icms_value is not None
            and extraction.icms_base is not None
            and extraction.icms_base > 0):
            extraction.icms = round(
                (extraction.icms_value / extraction.icms_base) * 100, 2
            )

    # 3. tusd: derive from tusd_unit_price if present
    #    Both tusd and tusd_unit_price are in R$/kWh (unit rate), so direct
    #    assignment is correct. tusd_unit_price is extracted from the tariff
    #    breakdown while tusd is the primary field — they represent the same value.
    if extraction.tusd is None and extraction.tusd_unit_price is not None:
        extraction.tusd = extraction.tusd_unit_price

    return extraction


def derive_missing_fields_group_a(extraction: AgentGroupAOutput) -> AgentGroupAOutput:
    """Derive missing Group A fields from raw components."""
    # 1. icms: derive percentage from absolute values
    if extraction.icms is None:
        if (extraction.icms_value is not None
            and extraction.icms_base is not None
            and extraction.icms_base > 0):
            extraction.icms = round(
                (extraction.icms_value / extraction.icms_base) * 100, 2
            )

    return extraction
```

**Post-Processing Rules (Group B only):**

```python
def fill_missing_months_with_average(monthly: list[Optional[float]]) -> list[float]:
    """Fill None months with the average of present months.

    If no months have values, returns all zeros (extraction considered failed).
    """
    present = [v for v in monthly if v is not None]
    if not present:
        return [0.0] * 12
    avg = sum(present) / len(present)
    return [v if v is not None else round(avg, 2) for v in monthly]
```

**Post-Processing Rules (Group A):**

```python
def replicate_average_to_months(average: float) -> list[float]:
    """Replicate the average value to all 12 months."""
    return [round(average, 2)] * 12


def post_process_group_a(extraction) -> None:
    """Post-process Group A extraction: replicate averages to monthly arrays."""
    if extraction.average_consumption is not None:
        extraction.monthly_consumption = replicate_average_to_months(
            extraction.average_consumption
        )
    if extraction.average_consumption_peak is not None:
        extraction.monthly_consumption_peak = replicate_average_to_months(
            extraction.average_consumption_peak
        )
```

> **Missing months reporting:** Individual missing months are NOT reported individually in `missing_fields`. The entire `monthly_consumption` field is reported as missing only if ALL 12 months are null/zero after the fill-with-average post-processing. Partial months are silently filled.

**Not-an-Invoice Handling:**

If `tariff_modality` is null (agent determined the document is not an energy invoice):
- Return `InvoiceExtractionResult(success=False, ...)` with `error_message = "O documento enviado não é uma fatura de energia elétrica."`
- `missing_fields` should list all mandatory fields for Group B as a reasonable default

**Success Criteria:**

Group B success requires:
- `kwh_price` is present and valid
- `network_class` is present and valid
- All 12 months have consumption values (after fill)

Group A success requires:
- `classification` is present and valid ("Azul" or "Verde")
- `kwh_price` and `kwh_price_peak` are present and valid
- `tusd` and `tusd_peak` are present and valid
- `demand` and `demand_tariff` are present and valid
- `average_consumption` and `average_consumption_peak` are present and valid
- If Azul: `demand_peak` and `demand_tariff_peak` also required

**Factory:**

**Modified file:** `packages/nexus/nexus-services/src/nexus_services/factories.py`

```python
def create_energy_invoice_service(
    storage_client=None,
    http_client=None,
    agent_factory=None,
) -> EnergyInvoiceService:
    """Create EnergyInvoiceService with all dependencies.

    Follows the existing factory pattern: optional DI parameters with defaults.
    Pass explicit dependencies for testing; production uses auto-created defaults.
    """
    from core_storage.client import get_storage_client
    from core_http.adapters import HttpxHttpClient
    from nexus_ai.factories.energy_invoice import EnergyInvoiceAgentFactory

    return EnergyInvoiceServiceImpl(
        storage_client=storage_client or get_storage_client(),
        http_client=http_client or HttpxHttpClient(),
        agent_factory=agent_factory or EnergyInvoiceAgentFactory(),
    )
```

#### `nexus-ai` — Agent Factory

**New file:** `packages/nexus/nexus-ai/src/nexus_ai/factories/energy_invoice.py`

> **Migration prerequisite:** The existing `nexus_ai/factories.py` is a single file, not a package directory. Before creating this file, rename `factories.py` → `factories/__init__.py`. All existing imports (`from nexus_ai.factories import create_nexus_agent`) will continue to work unchanged.

```python
import base64
import mimetypes
from nexus_ai.factories import create_nexus_agent
from nexus_contexts.energy_invoice import ENERGY_INVOICE_SYSTEM_PROMPT
from nexus_models.energy_invoice_agent import AgentExtractionOutput
from nexus_services.impl.energy_invoice_config import AVAILABLE_MODELS


class EnergyInvoiceAgentFactory:
    """Factory for creating energy invoice extraction agents."""

    async def create_and_run(
        self,
        images: list[bytes],
        model: str = "gpt-5-mini",
    ) -> AgentExtractionOutput:
        """Create a task agent and run extraction.

        Uses structured output (output_type) — no tools, no streaming.
        Single-step: agent classifies (Group A/B, Verde/Azul) and extracts in one call.

        Image handling: OpenAIAgent.run() accepts `user_message: str` only.
        To pass images to vision models, we need to build structured content
        parts (list of dicts) following the OpenAI vision API format.

        **Prerequisite (BLOCKING):** Extend `OpenAIAgent.run()` in `core_ai` to
        accept `user_message: str | list[dict]`, where the list form supports
        OpenAI-style content parts. This change must be implemented in
        `packages/core/core-ai/src/core_ai/agent.py` BEFORE the energy invoice
        feature. The extension should:
        1. Change `run()` signature: `user_message: str | list[dict]`
        2. When `list[dict]`, pass content parts directly to the underlying
           SDK's message construction (no str conversion)
        3. Maintain backward compatibility — existing `str` callers unchanged
        """
        # Resolve provider from model config (must match AVAILABLE_MODELS in Section 3.6)
        provider = AVAILABLE_MODELS.get(model, {}).get("provider", "openai")

        agent = create_nexus_agent(
            name="Energy Invoice Extractor",
            system_prompt=ENERGY_INVOICE_SYSTEM_PROMPT,
            provider=provider,
            model=model,
            output_type=AgentExtractionOutput,
            tools=[],
            timeout_seconds=45.0,  # Match Nexus LLM agent timeout
        )

        # Build content parts with base64-encoded images.
        # See implementation requirement above for the chosen approach.
        content_parts = [
            {"type": "text", "text": "Extraia os dados desta fatura de energia."},
        ]
        for img in images:
            b64 = base64.b64encode(img).decode()
            content_parts.append({
                "type": "image_url",
                "image_url": {"url": f"data:image/png;base64,{b64}"},
            })

        result = await agent.run(
            user_message=content_parts,  # Requires core_ai extension (Approach A)
            session=None,
            context=None,
        )

        return result.output
```

> **Implementation note:** `OpenAIAgent` is a Pydantic `BaseModel` — configured via constructor parameters. The extraction result is accessed via `result.output` (not `result.final_output`). The `run()` method signature is `run(user_message, session, context)`. **PREREQUISITE:** Extending `OpenAIAgent.run()` to accept `user_message: str | list[dict]` is a blocking dependency — implement this in `core_ai` first (see prerequisite note in agent factory above). Use `create_nexus_agent()` (from `nexus_ai.factories`) instead of directly instantiating `OpenAIAgent` to get proper timeout, retry, and token counting.

#### `nexus-contexts` — System Prompt

**New file:** `packages/nexus/nexus-contexts/src/nexus_contexts/energy_invoice.py`

### 3.2 Extraction Agent System Prompt

```python
ENERGY_INVOICE_SYSTEM_PROMPT = """Você é um agente especializado em extrair dados de faturas de energia elétrica brasileiras. Sua tarefa é analisar imagens de faturas de energia e extrair valores específicos de forma precisa e estruturada.

## Sua Função

Você receberá uma ou mais imagens de uma fatura de energia elétrica. Sua única tarefa é extrair os valores solicitados e retorná-los no formato estruturado especificado. Você NÃO deve realizar cálculos, estimativas ou inferências — apenas extraia os valores que estão explicitamente presentes na fatura.

## Passo 1: Identificar a Modalidade Tarifária

Primeiro, identifique se a fatura é do **Grupo B (Baixa Tensão)** ou **Grupo A (Alta Tensão)**.

Indicadores de Grupo A:
- Presença de campos de "Demanda Contratada" ou "Demanda" em kW
- Presença de tarifas diferenciadas Ponta/Fora Ponta
- Referência a classificação horo-sazonal (Azul ou Verde)
- Presença de TE (Tarifa de Energia) e TUSD separados para Ponta e Fora Ponta
- Tensão de fornecimento acima de 2,3 kV

Indicadores de Grupo B:
- Ausência de campos de demanda contratada
- Tarifa única de kWh (sem diferenciação ponta/fora ponta)
- Classificação como residencial, comercial BT, ou rural BT
- Tensão de fornecimento em 127V, 220V ou 380V

Defina `tariff_modality` como "A" ou "B". Se o documento **não** for uma fatura de energia elétrica (ex: conta de água, boleto, contrato), retorne `tariff_modality` como null e deixe `group_b` e `group_a` como null.

## Passo 2: Extrair Valores

### Se Grupo B:

Extraia os seguintes campos:

1. **power_dist_company**: Nome da concessionária de energia. Procure no cabeçalho, logo, ou rodapé da fatura. Use o nome oficial da concessionária. Exemplos de nomes e aliases conhecidos:
   - CPFL PAULISTA (CPFL, Companhia Paulista de Força e Luz)
   - ENEL SP (ELETROPAULO, AES ELETROPAULO)
   - CEMIG-D (CEMIG, Companhia Energética de Minas Gerais)
   - COPEL-DIS (COPEL, Companhia Paranaense de Energia)
   - LIGHT (LIGHT S.A.)
   - NEOENERGIA COELBA (COELBA)
   - NEOENERGIA PERNAMBUCO (CELPE)
   - ENERGISA (várias regionais: ENERGISA PB, ENERGISA MT, etc.)
   - EQUATORIAL (várias regionais: EQUATORIAL PA, EQUATORIAL MA, etc.)
   Se encontrar um alias, retorne o nome oficial correspondente.

2. **kwh_price**: Valor do kWh em R$. Este é o valor final cobrado por kWh **incluindo todos os tributos e encargos**. Procure por:
   - "Tarifa de Energia" ou "TE" na seção de composição tarifária
   - "Valor unitário" na tabela de itens faturados
   - Faixa esperada: R$ 0,30 a R$ 2,50 por kWh (valores fora desta faixa podem indicar erro de leitura)
   - **IMPORTANTE**: Procure pelo valor final do kWh (Valor Unitário) já incluindo todos os componentes. Se não encontrar um valor único final, retorne null (os componentes separados serão capturados em campos auxiliares abaixo).

3. **public_light_bill**: Taxa de iluminação pública (CIP ou COSIP) em R$. Procure por:
   - "Contrib Ilum Publica Municipal" ou "CIP" ou "COSIP"
   - Geralmente aparece como item separado na fatura

4. **network_class**: Classe de rede elétrica. Procure por:
   - "Classificação" ou "Classe" ou "Tipo de Fornecimento"
   - "Trifásico" / "Trifásica" → retorne "Trifásica"
   - "Bifásico" / "Bifásica" → retorne "Bifásica"
   - "Monofásico" / "Monofásica" → retorne "Monofásica"

5. **Consumo mensal (jan a dec)**: Histórico de consumo dos últimos 12 meses em kWh. Procure por:
   - Tabela ou gráfico de "Histórico de Consumo" ou "Consumo nos Últimos 12 Meses"
   - Colunas com meses e valores de kWh
   - Se a fatura apresentar menos de 12 meses, extraia apenas os meses disponíveis e deixe os demais como null
   - **NÃO calcule médias ou estime valores** — extraia apenas o que está explicitamente na fatura
   - Mapeie os meses para os campos: jan, feb, mar, apr, may, jun, jul, aug, sep, oct, nov, dec

6. **tusd**: Tarifa de Uso do Sistema de Distribuição em R$/kWh. Procure por:
   - "TUSD" na composição tarifária
   - Pode estar listado como item separado

7. **icms**: Percentual de ICMS. Procure por:
   - "ICMS" com valor percentual (%)
   - Geralmente entre 12% e 33%
   - Se encontrar apenas valores absolutos (R$), retorne null aqui — preencha icms_value e icms_base abaixo.

### Campos Auxiliares (Grupo B)

Estes campos auxiliares são usados para derivar valores quando os campos principais não estão disponíveis. Extraia-os SEMPRE que visíveis na fatura, mesmo se o campo principal correspondente já foi preenchido.

8. **total_energy_value**: Valor total da conta de energia em R$. Procure por:
   - "Valor Total" ou "Total a Pagar" na fatura
   - Apenas o valor de energia elétrica (excluir iluminação pública se listada separadamente)

9. **current_month_consumption**: Consumo do mês atual em kWh. Procure por:
   - "Consumo" ou "kWh" no período de faturamento atual
   - Diferente do histórico — este é o consumo do mês da fatura

10. **te_unit_price**: Tarifa de Energia (TE) unitária em R$/kWh. Procure por:
    - "TE" ou "Tarifa de Energia" na composição tarifária
    - Apenas o componente TE, sem TUSD ou tributos

11. **tusd_unit_price**: TUSD unitária em R$/kWh. Procure por:
    - "TUSD" na composição tarifária com valor unitário (R$/kWh)
    - Diferente do tusd total em R$

12. **icms_value**: Valor absoluto do ICMS em R$. Procure por:
    - "Valor ICMS" ou valor em R$ associado ao ICMS

13. **icms_base**: Base de cálculo do ICMS em R$. Procure por:
    - "Base de Cálculo ICMS" ou "Base Cálc. ICMS"

### Se Grupo A:

Primeiro, identifique a **classificação horo-sazonal**:
- **Azul**: Demanda contratada diferenciada para Ponta e Fora Ponta
- **Verde**: Demanda contratada única (sem diferenciação ponta/fora ponta)

Defina `classification` como "Azul" ou "Verde".

Extraia os seguintes campos:

1. **power_dist_company**: Mesmo procedimento do Grupo B.

2. **kwh_price**: TE Fora Ponta (Tarifa de Energia Fora Ponta) em R$/kWh.
   - Procure por "TE Fora Ponta" ou "Energia Fora Ponta" ou "Consumo Fora Ponta"

3. **kwh_price_peak**: TE Ponta (Tarifa de Energia Ponta) em R$/kWh.
   - Procure por "TE Ponta" ou "Energia Ponta" ou "Consumo Ponta"

4. **tusd**: TUSD Fora Ponta em R$/kWh.
   - Procure por "TUSD Fora Ponta" ou "Distribuição Fora Ponta"

5. **tusd_peak**: TUSD Ponta em R$/kWh.
   - Procure por "TUSD Ponta" ou "Distribuição Ponta"

6. **demand**: Demanda Contratada em kW.
   - Para Verde: valor único de demanda contratada
   - Para Azul: demanda contratada **fora ponta**
   - Procure por "Demanda Contratada" ou "Demanda" na seção de demanda

7. **demand_tariff**: Tarifa da Demanda em R$/kW.
   - Para Verde: tarifa única de demanda
   - Para Azul: tarifa de demanda **fora ponta**
   - Procure por "Tarifa de Demanda" ou valor unitário junto à demanda

8. **demand_peak** (somente Azul): Demanda Contratada Ponta em kW.
   - Procure por "Demanda Contratada Ponta" ou "Demanda Ponta"

9. **demand_tariff_peak** (somente Azul): Tarifa da Demanda Ponta em R$/kW.
   - Procure por "Tarifa de Demanda Ponta"

10. **public_light_bill**: Mesmo procedimento do Grupo B.

11. **average_consumption**: Consumo médio mensal fora ponta em kWh.
    - Procure por "Consumo Médio" ou "Consumo Médio Fora Ponta"
    - Se não encontrar explicitamente, retorne null

12. **average_consumption_peak**: Consumo médio mensal ponta em kWh.
    - Procure por "Consumo Médio Ponta" ou consumo ponta no histórico

13. **icms**: Mesmo procedimento do Grupo B.

### Campos Auxiliares (Grupo A)

14. **icms_value**: Valor absoluto do ICMS em R$. Mesmo procedimento do Grupo B.

15. **icms_base**: Base de cálculo do ICMS em R$. Mesmo procedimento do Grupo B.

## Regras Importantes

1. **Extraia apenas valores explícitos**: Não invente, estime ou calcule valores que não estão na fatura. Se um valor não está presente, retorne null para o campo.

2. **Valores monetários**: Extraia em R$ (Reais). Atenção ao formato brasileiro de números: vírgula para decimais (ex: "1,50" = 1.50), ponto para milhares (ex: "1.500,00" = 1500.00). Retorne sempre como número decimal (float).

3. **Consumo em kWh**: Valores inteiros ou decimais. Mesmo atenção ao formato brasileiro de números.

4. **Percentuais**: Retorne como número (ex: 18% → 18.0, não 0.18).

5. **Concessionária**: Retorne EXATAMENTE um dos nomes oficiais da lista de concessionárias. Se encontrar um alias ou nome parcial, mapeie para o nome oficial. Se não conseguir identificar com certeza, retorne null.

6. **Múltiplas páginas**: Se receber múltiplas imagens, elas são páginas da mesma fatura. Considere todas as páginas para extrair os valores.

7. **Grupo A vs Grupo B**: Preencha APENAS os campos do grupo identificado. Se for Grupo B, preencha `group_b` e deixe `group_a` como null. Se for Grupo A, preencha `group_a` e deixe `group_b` como null.

8. **Meses do consumo (Grupo B)**: Mapeie os meses corretamente. Se a fatura mostra "JAN/2025: 320 kWh", coloque 320 no campo `jan`. Se um mês não está na fatura, retorne null para esse mês.

9. **Priorize precisão**: É preferível retornar null para um campo do que retornar um valor incorreto."""
```

### 3.3 New Endpoints

#### `POST /api/v1/energy-invoice/extract`

**New file:** `services/api/src/nexus_core_api/routers/energy_invoice.py`

```python
from fastapi import APIRouter, Depends, status
from nexus_core_api.schemas.energy_invoice import (
    ExtractionRequest,
    ExtractionResponse,
)
from nexus_services.protocols.energy_invoice_service import (
    EnergyInvoiceService,
    EnergyInvoiceExtractionInput,
)
from nexus_core_api.dependencies.energy_invoice import get_energy_invoice_service

# Auth dependency for service-to-service calls (Azume Backend → Nexus).
# This uses OAuth client credentials auth (NOT user JWT auth).
# The existing `require_service_auth` dependency validates Bearer tokens issued
# by the OAuth token exchange endpoint and returns ServiceTokenClaims with
# decoded claims (client_id, scopes, etc.). Do NOT use require_scope() or
# get_current_user() — those check UserScope enums for user-scoped JWT auth.
from nexus_core_api.dependencies.service_auth import require_service_auth
from nexus_services.protocols.service_client_service import ServiceTokenClaims

router = APIRouter(tags=["Energy Invoice"], prefix="/energy-invoice")


@router.post(
    "/extract",
    response_model=ExtractionResponse,
    status_code=status.HTTP_200_OK,
    summary="Extract data from an energy invoice",
)
async def extract_invoice(
    request: ExtractionRequest,
    _: ServiceTokenClaims = Depends(require_service_auth),  # OAuth client credentials auth
    service: EnergyInvoiceService = Depends(get_energy_invoice_service),
) -> ExtractionResponse:
    result = await service.extract_invoice_data(
        EnergyInvoiceExtractionInput(
            file_url=str(request.file_url),  # HttpUrl → str conversion (Pydantic v2)
            model=request.model or "gpt-5-mini",
        )
    )
    return ExtractionResponse.from_domain(result)
```

#### `GET /api/v1/energy-invoice/models`

```python
@router.get(
    "/models",
    status_code=status.HTTP_200_OK,
    summary="List available vision models for invoice extraction",
)
async def list_models(
    _: ServiceTokenClaims = Depends(require_service_auth),  # OAuth client credentials auth
) -> dict:
    # Uses the single source of truth from Section 3.6 — do NOT duplicate the list here.
    from nexus_services.impl.energy_invoice_config import AVAILABLE_MODELS, DEFAULT_MODEL
    return {
        "models": [
            {"id": model_id, **config, "is_default": model_id == DEFAULT_MODEL}
            for model_id, config in AVAILABLE_MODELS.items()
        ]
    }
```

**New file:** `services/api/src/nexus_core_api/dependencies/energy_invoice.py`

```python
from nexus_services.protocols.energy_invoice_service import EnergyInvoiceService
from nexus_services.factories import create_energy_invoice_service


def get_energy_invoice_service() -> EnergyInvoiceService:
    """FastAPI dependency that provides the EnergyInvoiceService instance."""
    return create_energy_invoice_service()
```

**Router registration in `main.py`:**

```python
# In services/api/src/nexus_core_api/routers/__init__.py — add to selective exports:
from nexus_core_api.routers import energy_invoice

# In services/api/src/nexus_core_api/main.py — add router registration:
app.include_router(energy_invoice.router, prefix=API_PREFIX)
```

### 3.4 Request/Response Schemas

**New file:** `services/api/src/nexus_core_api/schemas/energy_invoice.py`

```python
from pydantic import BaseModel, Field, HttpUrl
from typing import Optional
from nexus_models.energy_invoice import (
    GroupBExtraction, GroupAVerdeExtraction, GroupAAzulExtraction,
)


class ExtractionRequest(BaseModel):
    file_url: HttpUrl = Field(..., description="Pre-signed URL to the invoice file")
    model: Optional[str] = Field(
        default=None,
        description="LLM model ID. Defaults to gpt-5-mini."
    )


class ExtractionResponse(BaseModel):
    """Maps InvoiceExtractionResult to HTTP response.

    Uses typed Pydantic models for group fields instead of dict to preserve
    schema information for OpenAPI/Swagger documentation.
    """
    success: bool
    tariff_modality: Optional[str] = None
    classification: Optional[str] = None
    group_b: Optional[GroupBExtraction] = None
    group_a_verde: Optional[GroupAVerdeExtraction] = None
    group_a_azul: Optional[GroupAAzulExtraction] = None
    missing_fields: list[str] = []
    model_used: str
    error_message: Optional[str] = None

    @classmethod
    def from_domain(cls, result) -> "ExtractionResponse":
        return cls(
            success=result.success,
            tariff_modality=result.tariff_modality.value if result.tariff_modality else None,
            classification=result.classification.value if result.classification else None,
            group_b=result.group_b,
            group_a_verde=result.group_a_verde,
            group_a_azul=result.group_a_azul,
            missing_fields=result.missing_fields,
            model_used=result.model_used,
            error_message=result.error_message,
        )
```

### 3.5 File Processing Flow

```
1. Receive pre-signed S3 URL
   ↓
2. Download file via httpx (timeout: 30s, max size: 20MB)
   │  Error: URL expired / 403 Forbidden → raise HTTPException(422, "URL do arquivo expirada ou inacessível.")
   │  Error: Download timeout / connection error → raise HTTPException(422, "Não foi possível baixar o arquivo.")
   │  Error: File too large (> 20MB) → raise HTTPException(422, "Arquivo muito grande.")
   │  Error: Empty response / 0 bytes → raise HTTPException(422, "Arquivo vazio ou corrompido.")
   ↓
3. Detect content type from HTTP headers + file magic bytes
   │  Error: Unsupported content type → raise HTTPException(422, "Formato de arquivo não suportado.")
   ↓
4. Upload to GCS temp path: energy-invoices/temp/{uuid}/{filename}
   │  Error: GCS upload failure → raise HTTPException(500, "Erro interno ao processar arquivo.")
   ↓
5. If PDF:
   │  a. Open with PyMuPDF (fitz)
   │  b. Check page count: if > 5 pages, return 422 error
   │  c. For each page: render as PNG at 300 DPI
   │  d. Collect page images as bytes[]
   ↓
   If Image (.jpg/.png/.gif/.webp):
   │  a. Use image bytes directly
   │  b. images = [file_bytes]
   ↓
6. Create agent via EnergyInvoiceAgentFactory.create_and_run(images, model)
   ↓
7. Agent returns AgentExtractionOutput (structured output)
   ↓
8. Validate against business rules (ranges from feature doc tables)
   ↓
8b. Apply deterministic derivation rules (derive missing kwh_price, icms, tusd from raw components)
   ↓
9. Post-process:
   │  - Group B: fill missing months with average
   │  - Group A: replicate average to 12-month arrays
   ↓
10. Map AgentExtractionOutput → InvoiceExtractionResult:
   │  - If group_b: map AgentGroupBOutput → GroupBExtraction (direct field mapping)
   │  - If group_a with classification="Verde":
   │    map AgentGroupAOutput → GroupAVerdeExtraction (exclude peak demand)
   │    set group_a_verde, leave group_a_azul=null
   │  - If group_a with classification="Azul":
   │    map AgentGroupAOutput → GroupAAzulExtraction (include peak demand)
   │    set group_a_azul, leave group_a_verde=null
   ↓
11. Determine success/failure based on mandatory fields
   ↓
12. GCS temp file cleanup: Use GCS object lifecycle policy configured on the
    `energy-invoices/temp/` prefix to auto-delete objects older than 1 hour.
    No application-level cleanup needed — configure lifecycle rule at infra level.
   ↓
13. Return result
```

### 3.6 Model Configuration

```python
AVAILABLE_MODELS = {
    "gpt-5-mini": {
        "display_name": "GPT-5 Mini",
        "provider": "openai",
        "is_default": True,
    },
    "gemini-3-flash-preview": {
        "display_name": "Gemini 3 Flash",
        "provider": "google",
        "is_default": False,
    },
    "claude-sonnet-4-5": {
        "display_name": "Claude Sonnet 4.5",
        "provider": "anthropic",
        "is_default": False,
    },
    "grok-4-1-fast": {
        "display_name": "Grok 4.1 Fast",
        "provider": "xai",
        "is_default": False,
    },
}

DEFAULT_MODEL = "gpt-5-mini"
```

> **Multi-provider routing:** `core_ai.agent.OpenAIAgent` already supports multi-provider routing via the OpenAI-compatible API pattern. Non-OpenAI models (Gemini, Claude, Grok) are routed through their respective provider endpoints configured in `core_ai`. The `model` string passed to the agent constructor (via `create_nexus_agent()`) is used to resolve the provider and endpoint automatically — no adapter work is needed. If a new provider is added in the future, it must be registered in `core_ai`'s provider configuration.

### 3.7 Extraction Telemetry Logging

The service implementation should emit a structured log entry after each extraction attempt for monitoring LLM accuracy, cost, and performance:

```python
# In EnergyInvoiceServiceImpl, after extraction completes (success or failure):
logger.info(
    "energy_invoice_extraction",
    extra={
        "model_used": model,
        "success": result.success,
        "tariff_modality": result.tariff_modality.value if result.tariff_modality else None,
        "classification": result.classification.value if result.classification else None,
        "missing_fields": result.missing_fields,
        "missing_fields_count": len(result.missing_fields),
        "processing_time_ms": elapsed_ms,           # Total wall-clock time
        "agent_time_ms": agent_elapsed_ms,           # LLM inference time only
        "file_type": content_type,                   # "application/pdf" | "image/jpeg" etc.
        "pdf_page_count": page_count if is_pdf else None,
        "derivation_applied": list_of_derived_fields, # e.g. ["kwh_price", "icms"]
    }
)
```

This data enables:
- Accuracy dashboards (success rate by model, by concessionária, by file type)
- Cost tracking (requests per model per day)
- Performance monitoring (P50/P95 latency)
- Identifying problematic invoice formats (repeated failures for a specific concessionária)

---

## 4. Azume Backend Implementation Spec

### 4.1 New Route File

**New file:** `src/routes/invoiceReadingRoutes.ts`

```typescript
import express from "express";
import { checkAuth } from "../middlewares/checkAuth";
import { singleFilesUpload } from "../middlewares/fileUpload";
import { extractInvoice } from "../controllers/invoiceReading/extractInvoice";
import { getAvailableModels } from "../controllers/invoiceReading/getAvailableModels";

const invoiceReadingRoutes = express.Router();

invoiceReadingRoutes
  .use(checkAuth)
  .post("/:pid/invoice-reading", singleFilesUpload, extractInvoice)
  .get("/invoice-reading/models", getAvailableModels);

export default invoiceReadingRoutes;
```

> **Note:** `singleFilesUpload` middleware expects the file field name `"file"` in the multipart form data, matching the frontend's `formData.append("file", file)`.

> **S3 upload before validation (accepted trade-off):** The `singleFilesUpload` middleware uploads the file to S3 _before_ the controller validates the proposal exists or the file meets business rules. If controller validation fails, the file remains as an orphan on S3. This is an accepted trade-off — DigitalOcean Spaces lifecycle rules on the `uploads/archives/` prefix should be configured to auto-delete objects older than 30 days. This also matches the existing pattern used by other controllers (voucher upload, contract upload, etc.).

**Mount in `app.ts`:**

```typescript
import invoiceReadingRoutes from "./routes/invoiceReadingRoutes";
// ...
app.use("/api/proposals", invoiceReadingRoutes);
```

### 4.2 Auth & User Model Changes

> **Note on `isSystemAdmin` vs existing `userIsAdmin`:** The frontend auth context already has a `userIsAdmin: boolean` field (set via `uIsAdmin` parameter in the login flow). This is a **different concept** — `userIsAdmin` is the Azume CRM admin flag (currently always `false` for regular users, with admin access via `ADMIN_USER_ACCESS_KEY` backdoor). The new `isSystemAdmin` field represents a **platform-level privilege** for system administrators who can access advanced features like model selection. Both fields coexist independently.

**Modified file:** `src/models/user.ts`

Add new field to `UserDocument` interface and schema:

```typescript
// In UserDocument interface:
isSystemAdmin: boolean;

// In UserSchema:
isSystemAdmin: { type: Boolean, default: false },
```

**Modified file:** `src/middlewares/checkAuth.ts`

Add `isSystemAdmin` to the JWT payload read (it should be included in the login token generation):

```typescript
// In checkAuth, add to req.userData:
req.userData = {
  // ... existing fields ...
  isSystemAdmin: decodedToken.isSystemAdmin || false,
};
```

**Modified file:** `src/custom.d.ts`

Add to the Request augmentation:

```typescript
isSystemAdmin?: boolean;
```

**Modified file:** `src/controllers/user/logUser.ts`

Add `isSystemAdmin` to the JWT payload in `jwt.sign()` AND to the HTTP response body:

```typescript
// In the jwt.sign() payload object, add:
isSystemAdmin: existingUser.isSystemAdmin || false,

// In the res.json() response body (alongside token, userId, email, etc.), add:
isSystemAdmin: existingUser.isSystemAdmin || false,
```

> **Why both?** The JWT payload carries `isSystemAdmin` for `checkAuth` middleware (server-side). The response body carries it for the frontend `login()` function (client-side) — the frontend does not decode the JWT, it reads the response body.

**Migration note:** Existing users in MongoDB do not have `isSystemAdmin`. Mongoose's `default: false` applies on read, so existing users will correctly evaluate as non-admin. To grant system admin access, manually set `isSystemAdmin: true` for specific users via MongoDB shell or admin tool. Adding `isSystemAdmin` to the JWT payload requires all users to **re-login** to get a new token containing the field — until then, `decodedToken.isSystemAdmin` will be `undefined`, which defaults to `false` via `|| false` in checkAuth.

**Modified file:** `src/controllers/user/logClientWithToken.ts`

Add `isSystemAdmin` to the token-refresh JWT payload AND response body as well:

```typescript
// In the jwt.sign() payload object, add:
isSystemAdmin: existingUser.isSystemAdmin || false,

// In the res.json() response body, add:
isSystemAdmin: existingUser.isSystemAdmin || false,
```

### 4.2b HttpError Extension

The existing `HttpError` class (`src/classes/HttpError.ts`) has no `errorCode` field. Since the API contract returns machine-readable `errorCode` values, extend the class:

**Modified file:** `src/classes/HttpError.ts`

```typescript
// Add new optional field:
errorCode?: string;

// Extend constructor (backward-compatible — new param is optional and last):
constructor(
  message: string,
  errCode: number,
  originalError: Error | null = Error(""),
  shouldLogout: boolean = false,
  errorTitle: string = "Erro",
  errorCode?: string,             // NEW — machine-readable error code for API responses
)
```

**Modified file:** Global error handler in `src/app.ts`

Include `errorCode` and `success: false` in the error response when present:

> **Scope note:** This is a backward-compatible, system-wide enhancement — `success: false` and `errorCode` are added to ALL error responses from all 50+ route files. This is safe because existing frontend code checks `!response.ok` for HTTP errors, not `success`. The existing S3 file cleanup logic (`if (req.file && req.file.key) { s3.deleteObject(...) }`) in the error handler **must be preserved** — do not remove it. For the invoice extraction controller specifically, this cleanup is beneficial: if the controller throws before archiving, the orphan file is cleaned up immediately.

```typescript
// In the error handler, modify the response JSON to include success and errorCode.
// PRESERVE the existing S3 file cleanup code above this line.
res.status(error.code || 500).json({
  success: false,                             // NEW — consistent with API contract
  message: error.message,
  shouldLogout: error.shouldLogout,
  title: error.errorTitle,
  errorCode: error.errorCode || undefined,    // NEW — machine-readable error code
});
```

### 4.3 Main Extraction Controller

> **Rate limiting / abuse protection:** Each extraction triggers an LLM API call ($$). The following protections apply:
> - **Frontend:** The "Extrair Dados" button is disabled while `isExtracting` is true, preventing double-clicks.
> - **Backend (per-proposal):** The controller checks for a recent extraction on the same proposal+ucid. If an extraction was completed within the last **60 seconds**, return 429 with "Aguarde antes de tentar novamente." This prevents rapid-fire resubmissions.
> - **Future consideration:** Per-user daily rate limit (e.g., 50 extractions/day) can be added if abuse is observed in production.

**New file:** `src/controllers/invoiceReading/extractInvoice.ts`

```typescript
import { Request, Response, NextFunction } from "express";
import { HttpError } from "../../classes/HttpError";
import { getNexusOAuthToken } from "../../util/nexus/nexusOAuthClient";
import { Proposal } from "../../models/proposal";
import { Archive } from "../../models/archive";
import { Customer } from "../../models/customer";
import { s3 } from "../../middlewares/fileUpload";
import axios from "axios";
import { v1 as uuidv1 } from "uuid";

// --- Rate limiting: per-proposal+ucid cooldown (60s) ---
// In-memory map: key = `${proposalId}:${ucIndex}` → last extraction timestamp.
// Prevents rapid-fire resubmissions that would incur unnecessary LLM costs.
// Acceptable to lose on process restart (not persisted — protection, not critical state).
const recentExtractions = new Map<string, number>();
const EXTRACTION_COOLDOWN_MS = 60_000; // 60 seconds

// Periodic cleanup to prevent unbounded growth (every 10 minutes)
setInterval(() => {
  const now = Date.now();
  for (const [key, timestamp] of recentExtractions) {
    if (now - timestamp > EXTRACTION_COOLDOWN_MS) {
      recentExtractions.delete(key);
    }
  }
}, 10 * 60_000);

const ACCEPTED_MIME_TYPES = [
  "image/jpeg",
  "image/png",
  "image/gif",
  "image/webp",
  "application/pdf",
];
const MAX_FILE_SIZE = 20 * 1024 * 1024; // 20 MB

export const extractInvoice = async (
  req: Request,
  res: Response,
  next: NextFunction
) => {
  try {
    const { pid } = req.params;
    const { ucid, customerId, model } = req.body;
    const ucIndex = parseInt(ucid, 10); // Consumer unit index for multi-UC proposals (validated below)
    const file = req.file;

    // 1. Validate file
    if (!file) {
      return next(new HttpError("Nenhum arquivo enviado.", 400));
    }
    if (!ACCEPTED_MIME_TYPES.includes(file.mimetype)) {
      return next(new HttpError(
        "Formato de arquivo não suportado. Envie PDF, JPG, PNG, GIF ou WebP.",
        400
      ));
    }
    if (file.size > MAX_FILE_SIZE) {
      return next(new HttpError("Arquivo muito grande. Tamanho máximo: 20 MB.", 400));
    }

    // 2. Validate proposal exists
    const proposal = await Proposal.findById(pid);
    if (!proposal) {
      return next(new HttpError("Proposta não encontrada.", 404, null, false, "Erro", "INVALID_PROPOSAL"));
    }

    // 2b. Validate customerId matches the proposal's customer
    //     Prevents archiving files under a different customer.
    if (customerId && proposal.customer?.toString() !== customerId) {
      return next(new HttpError(
        "O cliente informado não corresponde à proposta.", 400, null, false, "Erro", "INVALID_PROPOSAL"
      ));
    }
    // Derive customerId from proposal if not provided, to ensure consistency
    const resolvedCustomerId = proposal.customer?.toString();

    // 2c. Validate ucid is a valid non-negative integer with upper bound
    if (isNaN(ucIndex) || ucIndex < 0 || ucIndex > 49) {
      return next(new HttpError(
        "Índice da unidade consumidora (ucid) inválido.", 400
      ));
    }

    // 2d. Rate limiting: check per-proposal+ucid cooldown (60s)
    const cooldownKey = `${pid}:${ucIndex}`;
    const lastExtraction = recentExtractions.get(cooldownKey);
    if (lastExtraction && Date.now() - lastExtraction < EXTRACTION_COOLDOWN_MS) {
      return next(new HttpError(
        "Aguarde antes de tentar novamente.", 429, null, false, "Erro", "RATE_LIMITED"
      ));
    }

    // 3. File is already uploaded to S3 by singleFilesUpload middleware
    //    The singleFilesUpload middleware uses ACL: "public-read", so the CDN URL
    //    is permanently accessible without authentication. This is the existing pattern
    //    used by all archive files in the system.
    const fileKey = (file as any).key;
    const fileUrl = `https://azume-files.cdn.digitaloceanspaces.com/${fileKey}`;

    //    Generate pre-signed URL for Nexus (temporary, 60 min) — needed because
    //    Nexus should not rely on public-read ACL; pre-signed URLs are more explicit.
    const presignedUrl = s3.getSignedUrl("getObject", {
      Bucket: "azume-files",
      Key: fileKey,
      Expires: 3600, // 60 minutes
    });

    // 4. Call Nexus extraction API
    const nexusToken = await getNexusOAuthToken();

    // Only pass model if user is system admin
    const nexusPayload: any = { file_url: presignedUrl };
    if (model && req.userData?.isSystemAdmin) {
      nexusPayload.model = model;
    }

    let nexusResult;
    try {
      const nexusResponse = await axios.post(
        `${process.env.NEXUS_API_BASE_URL}/api/v1/energy-invoice/extract`,
        nexusPayload,
        {
          headers: {
            Authorization: `Bearer ${nexusToken}`,
            "Content-Type": "application/json",
          },
          timeout: 60000, // 60s timeout for LLM processing
        }
      );
      nexusResult = nexusResponse.data;
    } catch (nexusErr: any) {
      console.error(
        `[Invoice Reading] Nexus extraction failed: ${nexusErr.message}`,
        { status: nexusErr.response?.status, data: nexusErr.response?.data }
      );

      // Map Nexus HTTP errors to appropriate backend responses
      const nexusStatus = nexusErr.response?.status;
      if (nexusStatus === 400 || nexusStatus === 422) {
        // Nexus 400/422 = bad file or invalid request — user's file is the problem
        const nexusMessage = nexusErr.response?.data?.detail
          || "Arquivo inválido ou não foi possível processar. Verifique o arquivo e tente novamente.";
        return next(new HttpError(nexusMessage, 422, nexusErr, false, "Erro", "INVALID_FILE_FORMAT"));
      }
      if (nexusStatus === 401) {
        // Nexus 401 = internal config issue (bad OAuth token)
        console.error("[Invoice Reading] Nexus OAuth token rejected — check NEXUS_CLIENT_ID/SECRET config.");
        return next(new HttpError(
          "Erro interno de configuração. Contate o suporte.",
          500, nexusErr, false, "Erro", "INTERNAL_ERROR"
        ));
      }
      // Nexus 500, timeout, ECONNREFUSED, or any other error = Nexus unavailable
      return next(new HttpError(
        "Não foi possível conectar ao serviço de leitura. Tente novamente.",
        502, nexusErr, false, "Erro", "NEXUS_UNAVAILABLE"
      ));
    }

    // 4b. Record extraction timestamp for rate limiting
    recentExtractions.set(cooldownKey, Date.now());

    // 5. Handle extraction failure
    if (!nexusResult.success) {
      return res.status(200).json({
        success: false,
        message: nexusResult.error_message ||
          "Não foi possível extrair os dados obrigatórios da fatura de energia.",
        errorCode: "EXTRACTION_FAILED",
        missingFields: nexusResult.missing_fields || [],
      });
    }

    // 6. Persist file in archive collection
    await persistInvoiceInArchive(resolvedCustomerId, fileUrl, file.originalname);

    // 7. Persist file URL in proposal
    // Note: If this fails after step 6 succeeds, the archive has the file but the
    // proposal does not. This is an accepted trade-off — the archive file is still
    // valid and accessible. The user can retry the extraction to update the proposal.
    await Proposal.findByIdAndUpdate(pid, {
      $set: { [`invoiceFileUrls.${ucIndex}`]: fileUrl },
    });

    // 8. Transform and return result
    const extraction = transformNexusResult(nexusResult);

    return res.status(200).json({
      success: true,
      extraction: {
        ...extraction,
        invoiceFileUrl: fileUrl,
      },
    });
  } catch (err: any) {
    return next(new HttpError(
      "Erro ao processar a fatura de energia.",
      500,
      err
    ));
  }
};
```

### 4.4 Archive Folder Logic

```typescript
/**
 * Find or create "Faturas de Energia" folder in the customer's archive,
 * then add the invoice file to it.
 */
async function persistInvoiceInArchive(
  customerId: string,
  fileUrl: string,
  originalFilename: string
): Promise<void> {
  const archive = await Archive.findOne({ customer: customerId });
  if (!archive) {
    console.warn(`[Invoice Reading] No archive found for customer ${customerId} — skipping file persistence.`);
    return;
  }

  const FOLDER_NAME = "Faturas de Energia";

  // Find existing folder
  let folder = archive.folders.find(
    (f: any) => f.name === FOLDER_NAME
  );

  // Create folder if not found
  if (!folder) {
    const newFolder = {
      name: FOLDER_NAME,
      order: archive.folders.length,
      semantic: "Faturas de Energia", // New semantic type
    };
    archive.folders.push(newFolder as any);
    await archive.save();
    folder = archive.folders[archive.folders.length - 1];
  }

  // Add file to the folder
  const newFile = {
    name: originalFilename,
    url: fileUrl,
    folder: folder._id.toString(),
    order: archive.files.filter(
      (f: any) => f.folder === folder!._id.toString()
    ).length,
    semantic: "Conta de Energia" as const,
    status: "APPROVED" as const,
  };

  archive.files.push(newFile as any);
  await archive.save();
}
```

**Type changes needed:**

In `src/data/types.ts`, extend the semantic types:

```typescript
export type FolderSemantics = "Financiamento" | "Faturas de Energia";

// FileSemantics already includes "Conta de Energia" — no change needed
```

### 4.5 Proposal Model Changes

**Modified file:** `src/models/proposal.ts`

Add new field to store the invoice file URL:

```typescript
// In ProposalDocument interface:
invoiceFileUrls?: string[];

// In ProposalSchema:
invoiceFileUrls: { type: [String], default: [] },
```

> **Sparse array note:** The controller uses `$set: { [\`invoiceFileUrls.${ucIndex}\`]: fileUrl }` to set the URL at the UC index position. If `ucIndex` is 2 but the array is empty, MongoDB creates a sparse array `[null, null, fileUrl]`. This is acceptable — UC indices are sequential and gaps are unlikely in practice. If needed, initialize the array with empty strings on proposal creation.
>
> **Migration note:** Existing proposals in MongoDB do not have the `invoiceFileUrls` field. Mongoose's `default: []` only applies to new documents. Existing proposals will have `invoiceFileUrls: undefined`, which is safe — the `$set` operator creates the field on first use.

### 4.6 Nexus Result Transformation

```typescript
/**
 * Transform Nexus extraction result to the format expected by the frontend.
 */
function transformNexusResult(nexusResult: any) {
  const { tariff_modality, classification, group_b, group_a_verde, group_a_azul, missing_fields } = nexusResult;

  const fields: any = {};

  if (tariff_modality === "B" && group_b) {
    fields.powerDistCompany = group_b.power_dist_company;
    fields.kwhPrice = group_b.kwh_price;
    fields.publicLightBill = group_b.public_light_bill;
    fields.networkClass = group_b.network_class;
    fields.monthlyConsumption = group_b.monthly_consumption;
    fields.tusd = group_b.tusd;
    fields.icms = group_b.icms;
  }

  if (tariff_modality === "A") {
    const groupA = group_a_azul || group_a_verde;
    if (groupA) {
      fields.powerDistCompany = groupA.power_dist_company;
      fields.kwhPrice = groupA.kwh_price;
      fields.kwhPricePeak = groupA.kwh_price_peak;
      fields.tusd = groupA.tusd;
      fields.tusdPeak = groupA.tusd_peak;
      fields.demand = groupA.demand;
      fields.demandTariff = groupA.demand_tariff;
      fields.publicLightBill = groupA.public_light_bill;
      fields.averageConsumption = groupA.average_consumption;
      fields.averageConsumptionPeak = groupA.average_consumption_peak;
      fields.monthlyConsumption = groupA.monthly_consumption;       // Replicated 12-month array
      fields.monthlyConsumptionPeak = groupA.monthly_consumption_peak; // Replicated peak array
      fields.icms = groupA.icms;

      // Azul-specific fields
      if (group_a_azul) {
        fields.demandPeak = group_a_azul.demand_peak;
        fields.demandTariffPeak = group_a_azul.demand_tariff_peak;
      }
    }
  }

  return {
    tariffModality: tariff_modality,
    classification: classification,
    fields,
    missingFields: (missing_fields || []).map((f: string) => snakeToCamel(f)),
  };
}

function snakeToCamel(s: string): string {
  return s.replace(/_([a-z])/g, (_, c) => c.toUpperCase());
}
```

### 4.6b Backend Telemetry Logging

The extraction controller should log a structured entry after each extraction attempt for operational monitoring:

```typescript
console.log(JSON.stringify({
  event: "invoice_reading_extraction",
  proposalId: pid,
  ucIndex,
  userId: req.userData?.userId,
  success: nexusResult.success,
  tariffModality: nexusResult.tariff_modality,
  modelUsed: nexusResult.model_used,
  missingFieldsCount: nexusResult.missing_fields?.length || 0,
  processingTimeMs: Date.now() - startTime,
  fileType: file.mimetype,
  fileSizeBytes: file.size,
}));
```

### 4.7 Available Models Endpoint

**New file:** `src/controllers/invoiceReading/getAvailableModels.ts`

```typescript
import { Request, Response, NextFunction } from "express";
import { HttpError } from "../../classes/HttpError";
import { getNexusOAuthToken } from "../../util/nexus/nexusOAuthClient";
import axios from "axios";

// Simple in-memory cache (1 hour TTL)
let modelsCache: { data: any; expiresAt: number } | null = null;

export const getAvailableModels = async (
  req: Request,
  res: Response,
  next: NextFunction
) => {
  try {
    // Only system admins can access this endpoint
    if (!req.userData?.isSystemAdmin) {
      return next(new HttpError("Acesso restrito a administradores do sistema.", 403));
    }

    // Check cache
    if (modelsCache && modelsCache.expiresAt > Date.now()) {
      return res.status(200).json(modelsCache.data);
    }

    // Fetch from Nexus
    const token = await getNexusOAuthToken();
    const response = await axios.get(
      `${process.env.NEXUS_API_BASE_URL}/api/v1/energy-invoice/models`,
      {
        headers: { Authorization: `Bearer ${token}` },
        timeout: 10000,
      }
    );

    // Cache for 1 hour
    modelsCache = {
      data: response.data,
      expiresAt: Date.now() + 3600000,
    };

    return res.status(200).json(response.data);
  } catch (err: any) {
    return next(new HttpError(
      "Erro ao buscar modelos disponíveis.",
      500,
      err
    ));
  }
};
```

---

## 5. Azume Frontend CRM Implementation Spec

### 5.1 UI Components Overview

```
ProposalStepOne.tsx
│
├── [NEW] SmartReadingButton              ← Before InputsStep1TariffModalities
│         (click → opens InvoiceUploadModal)
│
├── [NEW] InvoiceUploadModal              ← Upload modal
│         ├── File picker + drop zone
│         ├── [CONDITIONAL] Model selector dropdown (admin only)
│         └── Upload/cancel buttons
│
├── [NEW] InvoiceValidationPopup          ← After extraction succeeds
│         ├── Extracted fields (editable inputs)
│         ├── Missing fields feedback
│         └── Confirm/cancel buttons
│
├── InputsStep1TariffModalities           ← (existing)
├── InputsStep1UCs                        ← (existing)
├── InputsStep1EnergyBillData             ← (existing, populated by extraction)
├── InputsStep1EnergyBillDataAGroup       ← (existing, populated by extraction)
├── InputsStep1AverageConsumption         ← (existing, populated by extraction)
└── ...
```

### 5.2 SmartReadingButton Component

**New file:** `src/proposal/components/SmartReadingButton.tsx`

**Location in Step 1:** Placed before the "Modalidade Tarifária" section heading in `ProposalStepOne.tsx`.

```typescript
interface SmartReadingButtonProps {
  onClick: () => void;
  disabled?: boolean;
}
```

**UI specification:**
- Button styled with Nexus brand color (`#6C63FF` or similar from Nexus palette)
- Contains Nexus logo icon (small, 20px) + text "Leitura Smart"
- MUI v4 `Button` component with `variant="contained"`
- Full-width within the form grid
- Disabled state during extraction (isLoading)

### 5.3 InvoiceUploadModal Component

**New file:** `src/proposal/components/InvoiceUploadModal.tsx`

```typescript
interface InvoiceUploadModalProps {
  open: boolean;
  onClose: () => void;
  onSubmit: (file: File, model?: string) => void;
  isLoading: boolean;
  isSystemAdmin: boolean;
  availableModels?: Array<{
    id: string;
    display_name: string;
    provider: string;
    is_default: boolean;
  }>;
}
```

**UI specification:**
- MUI v4 `Dialog` component (same pattern as `ModalConfirm.tsx`)
- `DialogTitle`: "Leitura Smart de Fatura de Energia"
- `DialogContent`:
  - File input (hidden) + styled drop zone area
  - Drop zone: dashed border, "Arraste o arquivo aqui ou clique para selecionar"
  - Accepted formats text: "Formatos aceitos: PDF, JPG, PNG, GIF, WebP (máx. 20 MB)"
  - Selected file name display
  - **Admin model selector** (conditional):
    - MUI v4 `Select` / `InputSelectRequired` dropdown
    - Label: "Modelo de IA"
    - Only visible when `isSystemAdmin === true`
    - Options populated from `availableModels` prop
    - Default selection: the model with `is_default: true`
- `DialogActions`:
  - "Cancelar" button (secondary)
  - "Extrair Dados" button (primary, disabled until file selected)
- Loading state: replace content with `LoadingSpinnerFullScreenGraph` + text "Extraindo dados da fatura... Isso pode levar até 60 segundos."

### 5.4 InvoiceValidationPopup Component

**New file:** `src/proposal/components/InvoiceValidationPopup.tsx`

```typescript
interface InvoiceValidationPopupProps {
  open: boolean;
  onClose: () => void;
  onConfirm: (editedFields: ExtractionFields) => void;
  extraction: {
    tariffModality: "A" | "B";
    classification?: "Azul" | "Verde";
    fields: ExtractionFields;
    missingFields: string[];
  };
}
```

**UI specification:**
- MUI v4 `Dialog` with `maxWidth="md"`, `fullWidth`
- `DialogTitle`: "Dados Extraídos da Fatura"
- `DialogContent`:
  - Success banner: "Leitura realizada com sucesso!" (green/teal)
  - Extracted fields section:
    - Each field displayed as an **editable input** (using existing Input components)
    - Group B: kwhPrice, publicLightBill, networkClass, powerDistCompany, 12 months, tusd, icms
    - Group A Verde: classification, kwhPrice, kwhPricePeak, tusd, tusdPeak, demand, demandTariff, publicLightBill, averageConsumption, averageConsumptionPeak, icms
    - Group A Azul: Verde fields + demandPeak, demandTariffPeak
    - Fields organized in a 2-column grid (using `form-inputs-grid-1fr-1fr` class)
  - Missing fields section (if any):
    - Yellow/amber warning banner
    - "Os seguintes campos não foram encontrados na fatura:"
    - List of missing field labels (in Portuguese, using `FIELD_LABELS` map below)
- `DialogActions`:
  - "Cancelar" — close without applying
  - "Confirmar e Preencher" — apply values to the main form

**Portuguese Field Labels Map:**

Used by `InvoiceValidationPopup` to display `missingFields` in user-friendly Portuguese:

```typescript
const FIELD_LABELS: Record<string, string> = {
  // Common
  tariffModality: "Modalidade Tarifária",
  classification: "Classificação Horo-Sazonal",
  powerDistCompany: "Concessionária",
  // Group B
  kwhPrice: "Valor do kWh",
  publicLightBill: "Taxa de Iluminação Pública",
  networkClass: "Classe de Rede",
  monthlyConsumption: "Consumo Mensal (12 meses)",
  tusd: "TUSD",
  icms: "ICMS",
  // Group A
  kwhPricePeak: "TE Ponta (kWh)",
  tusdPeak: "TUSD Ponta",
  demand: "Demanda Contratada",
  demandTariff: "Tarifa da Demanda",
  demandPeak: "Demanda Contratada Ponta",
  demandTariffPeak: "Tarifa da Demanda Ponta",
  averageConsumption: "Consumo Médio Fora Ponta",
  averageConsumptionPeak: "Consumo Médio Ponta",
};

// Usage: missingFields.map(f => FIELD_LABELS[f] || f)
```

### 5.5 Integration with useForm and inputHandler

When the user confirms the validation popup, populate form fields:

```typescript
const applyExtractionToForm = (
  extraction: ExtractionResult,
  inputHandler: InputHandlerFn,
  setFormData: SetFormDataFn,
  formState: FormState,
) => {
  const { tariffModality, classification, fields } = extraction;

  // Set tariff modality
  inputHandler(
    "tariffModality",
    tariffModality === "B" ? "Grupo B (Baixa Tensão)" : "Grupo A (Alta Tensão)",
    true,
    "Modalidade Tarifária"
  );

  if (tariffModality === "B") {
    // Group B fields
    if (fields.kwhPrice != null)
      inputHandler("kwhPrice", String(fields.kwhPrice), true, "Valor kWh");
    if (fields.publicLightBill != null)
      inputHandler("publicLightBill", String(fields.publicLightBill), true, "Taxa Ilum. Pública");
    if (fields.networkClass != null)
      inputHandler("networkClass", fields.networkClass, true, "Rede");
    if (fields.powerDistCompany != null)
      inputHandler("powerDistCompany", fields.powerDistCompany, true, "Concessionária");
    if (fields.tusd != null)
      inputHandler("tusd", String(fields.tusd), true, "TUSD");
    if (fields.icms != null)
      inputHandler("icms", String(fields.icms), true, "ICMS");

    // Monthly consumption
    const months = ["jan","feb","mar","apr","may","jun","jul","aug","sep","oct","nov","dec"];
    if (fields.monthlyConsumption) {
      fields.monthlyConsumption.forEach((val: number, i: number) => {
        if (val != null) {
          inputHandler(months[i], String(val), true, `Consumo ${months[i]}`);
        }
      });
    }
  }

  if (tariffModality === "A") {
    // Classification
    if (classification)
      inputHandler("classification", classification, true, "Classificação");
    // Group A fields
    if (fields.kwhPrice != null)
      inputHandler("kwhPrice", String(fields.kwhPrice), true, "TE Fora Ponta");
    if (fields.kwhPricePeak != null)
      inputHandler("kwhPricePeak", String(fields.kwhPricePeak), true, "TE Ponta");
    if (fields.tusd != null)
      inputHandler("tusd", String(fields.tusd), true, "TUSD Fora Ponta");
    if (fields.tusdPeak != null)
      inputHandler("tusdPeak", String(fields.tusdPeak), true, "TUSD Ponta");
    if (fields.demand != null)
      inputHandler("demand", String(fields.demand), true, "Demanda Contratada");
    if (fields.demandTariff != null)
      inputHandler("demandTariff", String(fields.demandTariff), true, "Tarifa da Demanda");
    if (fields.publicLightBill != null)
      inputHandler("publicLightBill", String(fields.publicLightBill), true, "Taxa Ilum. Pública");
    if (fields.averageConsumption != null)
      inputHandler("averageValue", String(fields.averageConsumption), true, "Consumo Médio FP");
    if (fields.averageConsumptionPeak != null)
      inputHandler("averageValuePeak", String(fields.averageConsumptionPeak), true, "Consumo Médio Ponta");
    if (fields.icms != null)
      inputHandler("icms", String(fields.icms), true, "ICMS");

    // Replicated monthly consumption (from average)
    const months = ["jan","feb","mar","apr","may","jun","jul","aug","sep","oct","nov","dec"];
    if (fields.monthlyConsumption) {
      fields.monthlyConsumption.forEach((val: number, i: number) => {
        if (val != null) {
          inputHandler(months[i], String(val), true, `Consumo ${months[i]}`);
        }
      });
    }

    // Azul-specific
    if (fields.demandPeak != null)
      inputHandler("demandPeak", String(fields.demandPeak), true, "Demanda Contratada Ponta");
    if (fields.demandTariffPeak != null)
      inputHandler("demandTariffPeak", String(fields.demandTariffPeak), true, "Tarifa Demanda Ponta");
  }
};
```

### 5.6 API Functions

**Modified file:** `src/proposal/api/proposalsAPI.ts`

Add new functions:

```typescript
/**
 * Extract data from an energy invoice via Leitura Smart.
 *
 * Note: the shared `sendRequest` hook (from useHttpClient) does NOT support
 * a custom timeout parameter. Its signature is:
 *   sendRequest(url, method, body, headers, successMessage, zeroResultsKey, pollOnTimeout)
 *
 * Since LLM extraction can take 60+ seconds (backend timeout is 60s),
 * we use a custom fetch with AbortController to enforce a 75s timeout.
 * This avoids the default sendRequest behavior while still using auth headers.
 */
export const extractInvoiceData = async (props: {
  auth: AuthContext;
  pid: string;
  ucid: number;
  customerId: string;
  file: File;
  model?: string;
  abortController?: AbortController;
}) => {
  const { auth, pid, ucid, customerId, file, model, abortController } = props;

  const formData = new FormData();
  formData.append("file", file);
  formData.append("ucid", String(ucid));
  formData.append("customerId", customerId);
  if (model) formData.append("model", model);

  const apiUrl = `${process.env.REACT_APP_BACKEND_URL}/proposals/${pid}/invoice-reading`;

  // Timeout must be > backend's 60s Nexus timeout to avoid the frontend
  // timing out while the backend is still waiting for Nexus.
  const EXTRACTION_TIMEOUT_MS = 75000; // 75 seconds

  // Use AbortController for timeout (sendRequest doesn't support custom timeouts)
  const controller = abortController || new AbortController();
  const timeoutId = setTimeout(() => controller.abort(), EXTRACTION_TIMEOUT_MS);

  try {
    const response = await fetch(apiUrl, {
      method: "POST",
      body: formData,
      headers: { Authorization: "Bearer " + auth.token },
      // Note: do NOT set Content-Type — browser sets it with boundary for FormData
      signal: controller.signal,
    });

    const responseData = await response.json();

    // Replicate sendRequest's auto-logout behavior for expired tokens.
    // Returns undefined — callers MUST check: `if (!result) return;`
    if (responseData.shouldLogout) {
      auth.logout();
      return;
    }

    if (!response.ok && !responseData.success) {
      throw new Error(responseData.message || "Erro ao processar a fatura.");
    }

    return responseData;
  } finally {
    clearTimeout(timeoutId);
  }
};

/**
 * Fetch available LLM models for invoice extraction (admin only).
 */
export const getInvoiceReadingModels = async (props: {
  sendRequest: SendRequestFn;
  auth: AuthContext;
}) => {
  const { sendRequest, auth } = props;

  const apiUrl = `${process.env.REACT_APP_BACKEND_URL}/proposals/invoice-reading/models`;

  const responseData = await sendRequest(
    apiUrl,
    "GET",
    null,
    {
      Authorization: "Bearer " + auth.token,
      "Content-Type": "application/json",
    },
  );

  return responseData;
};
```

### 5.7 Admin Model Selector

The model selector dropdown is conditionally rendered inside `InvoiceUploadModal`:

```typescript
// Inside InvoiceUploadModal component
const [selectedModel, setSelectedModel] = useState<string>("");
const [models, setModels] = useState<Model[]>([]);

useEffect(() => {
  if (isSystemAdmin && open) {
    // Fetch available models when modal opens (for admin users)
    getInvoiceReadingModels({ sendRequest, auth })
      .then((data) => {
        setModels(data.models);
        const defaultModel = data.models.find((m: any) => m.is_default);
        if (defaultModel) setSelectedModel(defaultModel.id);
      })
      .catch(() => { /* silently fall back to no model override */ });
  }
}, [isSystemAdmin, open]);

// In render:
{isSystemAdmin && models.length > 0 && (
  <InputSelectRequired
    id="invoiceModel"
    label="Modelo de IA"
    options={models.map((m) => m.display_name)}
    initialValue={models.find((m) => m.is_default)?.display_name || ""}
    onInput={(id, value) => {
      const model = models.find((m) => m.display_name === value);
      if (model) setSelectedModel(model.id);
    }}
  />
)}
```

### 5.8 Multi-UC Flow

Each invoice corresponds to 1 consumer unit. The flow for multiple UCs:

1. User uploads first invoice → extraction populates UC 0 fields
2. User confirms and form is populated with UC 0 data
3. User clicks "Adicionar UC" (existing button) → ucid increments
4. User clicks "Leitura Smart" again → uploads second invoice
5. Extraction populates UC 1 fields
6. Previous UC data is preserved (already saved in proposal via createProposal/editProposalStepOne)

The `SmartReadingButton` is always visible regardless of current ucid. The upload modal always submits with the current `ucid` state value.

### 5.9 Component Hierarchy and State Management

```
ProposalStepOne (state owner)
│
├── State: showUploadModal, showValidationPopup, extractionResult, isExtracting
│   State: availableModels (fetched lazily when modal opens, if isSystemAdmin)
│
├── SmartReadingButton
│   └── onClick → setShowUploadModal(true)
│
├── InvoiceUploadModal
│   ├── Props: open, onClose, onSubmit, isLoading, isSystemAdmin, availableModels
│   └── onSubmit → extractInvoiceData() → if (!result) return → setExtractionResult() → setShowValidationPopup(true)
│
├── InvoiceValidationPopup
│   ├── Props: open, onClose, onConfirm, extraction
│   └── onConfirm → applyExtractionToForm(editedFields, inputHandler, ...)
│
├── [existing components continue below...]
└── InputsStep1TariffModalities, InputsStep1UCs, etc.
```

### 5.10 Loading States

```
Upload modal opened → file selected → "Extrair Dados" clicked
│
├── InvoiceUploadModal: isLoading=true
│   └── Shows: "Extraindo dados da fatura... Isso pode levar até 60 segundos."
│   └── Progress indicator (indeterminate spinner)
│   └── "Cancelar" button remains active (AbortController)
│
├── On success: close modal → open validation popup
├── On failure: show error in modal (ModalError pattern)
└── On cancel: abort request, close modal
```

### 5.11 Error Handling

| Error | User Message | Action |
|-------|-------------|--------|
| Invalid file format | "Formato de arquivo não suportado. Envie PDF, JPG, PNG, GIF ou WebP." | Show in modal, don't submit |
| File too large | "Arquivo muito grande. Tamanho máximo: 20 MB." | Show in modal, don't submit |
| Extraction failed | "Não foi possível extrair os dados obrigatórios da fatura." | Show in modal with missingFields list |
| Rate limited (429) | "Aguarde antes de tentar novamente." | Show in modal, disable button briefly |
| Network error | "Erro de conexão. Verifique sua internet e tente novamente." | Show in modal (ModalError) |
| Timeout | "A leitura demorou mais que o esperado. Tente novamente." | Show in modal (ModalError) |

### 5.12 AuthContext Changes

> **Note:** The existing auth context has a `userIsAdmin: boolean` field (set by the `uIsAdmin` parameter in the login flow). The new `isSystemAdmin` is a **separate field** with different semantics (see Section 4.2 note). Both fields coexist — `userIsAdmin` gates CRM admin features, `isSystemAdmin` gates platform features like model selection.

**Modified file:** `src/shared/context/authContext.ts`

Add `isSystemAdmin` to the auth context (alongside existing `userIsAdmin`):

```typescript
// In AuthContextProps interface — ADD (do not remove userIsAdmin):
isSystemAdmin: boolean;

// In createContext default:
isSystemAdmin: false,
```

**Modified file:** `src/shared/hooks/authHook.ts`

Add `isSystemAdmin` to the login flow. The backend login response now includes `isSystemAdmin` in the JWT payload (added in Section 4.2). Extract it alongside the existing `uIsAdmin` parameter:

```typescript
// In login function — add isSystemAdmin as a new parameter:
// The existing login function takes 17 parameters. Add isSystemAdmin as the 18th.
const login = useCallback(
  (userId, email, ..., isSystemAdmin, ...) => {
    setIsSystemAdmin(isSystemAdmin);
    // Also persist in localStorage userData for session restoration
  },
  []
);

// In session restoration (useEffect) — restore isSystemAdmin from stored data:
setIsSystemAdmin(storedData.isSystemAdmin || false);
```

---

## 6. Concessionárias Enum

The full list of accepted concessionária values. This list is shared across all 3 systems:
- **Nexus**: Used in the agent system prompt for alias resolution and in validation
- **Azume Backend**: Used for validation of extraction results
- **Azume Frontend**: Used in the powerDistCompany dropdown (already exists in fee data)

| Company | Aliases |
|---------|---------|
| AMAZONAS ENERGIA | AmE; CEAM; Amazonas Distribuidora de Energia |
| CASTRO-DIS | Castro Distribuidora de Energia |
| CEA EQUATORIAL | CEA; Companhia de Eletricidade do Amapa; Equatorial Amapa; EQUATORIAL ENERGIA AMAPÁ |
| CEDRAP | Cooperativa de Eletrificacao e Desenvolvimento Rural do Alto Paraiba |
| CEDRI | Cooperativa de Energizacao e Desenvolvimento Rural do Vale do Itariri |
| CEEE EQUATORIAL | CEEE; CEEE-D; CEEE Distribuicao; Equatorial RS; EQUATORIAL ENERGIA RIO GRANDE DO SUL |
| CEGERO | Cooperativa de Eletricidade de Sao Ludgero |
| CEJAMA | Cooperativa de Eletricidade Jacinto Machado |
| CELESC-DIS | CELESC; CELESC-D; CELESC Distribuicao; Centrais Eletricas de Santa Catarina |
| CELETRO | Cooperativa de Eletrificacao Centro Jacui |
| CEMIG-D | CEMIG; CEMIG Distribuicao; Companhia Energetica de Minas Gerais |
| CEMIRIM | Cooperativa de Eletrificacao e Desenvolvimento da Regiao de Mogi Mirim |
| CEPRAG | Cooperativa de Eletricidade Praia Grande |
| CERACA | Cooperativa Distribuidora de Energia Vale do Araca |
| CERAL ANITAPOLIS | Cooperativa de Distribuicao de Energia Eletrica de Anitapolis |
| CERAL ARARUAMA | Cooperativa de Eletrificacao Rural de Araruama |
| CERAL DIS | Cooperativa de Distribuicao de Energia Eletrica de Arapoti |
| CERBRANORTE | Cooperativa de Eletrificacao de Braco do Norte |
| CERCI | Cooperativa de Eletrificacao Rural Cachoeiras de Macacu - Itaborai |
| CERCOS | Cooperativa de Eletrificacao e Desenvolvimento Rural Centro Sul de Sergipe |
| CEREJ | Cooperativa Senador Esteves Junior |
| CERES | Cooperativa de Eletrificacao Rural de Resende |
| CERFOX | Cooperativa de Distribuicao de Energia Fontoura Xavier |
| CERGAL | Cooperativa de Eletrificacao Anita Garibaldi |
| CERGAPA | Cooperativa de Eletricidade Grao Para |
| CERGRAL | Cooperativa de Eletricidade Gravatal |
| CERILUZ | Cooperativa Regional de Energia e Desenvolvimento Ijui |
| CERIM | Cooperativa de Eletrificacao Rural de Itu-Mairinque |
| CERIPA | Cooperativa de Eletrificacao Rural de Itai-Paranapanema-Avare |
| CERIS | Cooperativa de Eletrificacao da Regiao de Itapecerica da Serra |
| CERMC | Cooperativa de Eletrificacao e Desenvolvimento da Regiao de Mogi das Cruzes |
| CERMISSOES | CERMISSÕES; Cooperativa de Distribuicao e Geracao de Energia das Missoes |
| CERMOFUL | Cooperativa Fumacense de Eletricidade; CERMOFUL Energia |
| CERNHE | Cooperativa de Eletrificacao e Desenvolvimento Rural da Regiao de Novo Horizonte |
| CERPALO | Cooperativa de Eletrificacao Rural de Paulo Lopes |
| CERPRO | Cooperativa de Eletrificacao Rural da Regiao de Promissao |
| CERRP | Cooperativa de Eletrificacao e Desenvolvimento da Regiao de Sao Jose do Rio Preto |
| CERSAD | Cooperativa de Distribuicao de Energia Eletrica Salto Donner; Cersad Distribuidora |
| CERSUL | Cooperativa de Distribuicao de Energia Sul Catarinense |
| CERTAJA | CERTAJA Energia; Cooperativa Regional de Energia Taquari Jacui |
| CERTEL | CERTEL Energia; Cooperativa de Distribuicao de Energia Teutonia |
| CERTHIL | Cooperativa de Energia e Desenvolvimento Rural Entre Rios |
| CERTREL | Cooperativa de Energia Treviso |
| CERVAM | Cooperativa de Energizacao e de Desenvolvimento do Vale do Mogi |
| CETRIL | Cooperativa de Eletrificacao de Ibiuna e Regiao; CERI |
| CHESP | Companhia Hidroeletrica Sao Patricio |
| COCEL | Companhia Campolarguense de Energia |
| CODESAM | Cooperativa de Distribuicao de Energia Eletrica Santa Maria |
| COOPERA | Cooperativa Pioneira de Eletrificacao |
| COOPERALIANCA | COOPERALIANÇA; Cooperativa Alianca |
| COOPERCOCAL | Cooperativa Energetica Cocal |
| COOPERLUZ | Cooperativa Distribuidora de Energia Fronteira Noroeste |
| COOPERMILA | Cooperativa de Eletrificacao Lauro Muller |
| COOPERNORTE | Cooperativa Regional de Distribuicao de Energia do Litoral Norte |
| COOPERSUL | Cooperativa Regional de Eletrificacao Rural Fronteira Sul |
| COOPERZEM | Cooperativa de Distribuicao de Energia Eletrica Armazem |
| COORSEL | Cooperativa Regional Sul de Eletrificacao Rural |
| COPEL-DIS | COPEL; COPEL-D; COPEL Distribuicao; Companhia Paranaense de Energia |
| COPREL | COPREL Energia; Coprel Cooperativa de Energia |
| CPFL PAULISTA | CPFL; Companhia Paulista de Forca e Luz |
| CPFL PIRATININGA | Companhia Piratininga de Forca e Luz; PIRATININGA |
| CPFL SANTA CRUZ | Companhia Luz e Forca Santa Cruz; CLFSC |
| CRELUZ-D | CRELUZ; CRELUZ Distribuicao; CRELUZ - COOPERATIVA DE DISTRIBUIÇÃO DE ENERGIA |
| CRERAL | CRERAR Cooperativa Regional de Eletrificação Rural do Alto Uruguai |
| DCELT | DCELT Energia; Distribuidora Catarinense de Energia Elétrica S/A |
| DEMEI | Departamento Municipal de Energia de Ijui |
| DMED | DME Distribuicao; DME; DME-D |
| EDP ES | ESCELSA; EDP Escelsa; EDP Espirito Santo |
| EDP SP | BANDEIRANTE; EDP Bandeirante; Bandeirante Energia; EDP Sao Paulo |
| EFLJC | Empresa Forca e Luz Joao Cesa |
| EFLUL | Empresa Forca e Luz Urussanga |
| ELETROCAR | Centrais Eletricas de Carazinho |
| ELFSM | Empresa Luz e Forca Santa Maria |
| ENEL CE | COELCE; Companhia Energetica do Ceara; Enel Distribuicao Ceara |
| ENEL GO | Enel Distribuicao Goias |
| ENEL RJ | AMPLA; CERJ; Ampla Energia e Servicos; Enel Distribuicao Rio |
| ENEL SP | ELETROPAULO; AES ELETROPAULO; Eletropaulo Metropolitana; Enel Distribuicao Sao Paulo |
| ENERGISA AC | EAC; ELETROACRE; Empresa de Eletricidade do Acre |
| ENERGISA BORBOREMA | EBO; CELB; Companhia Energetica da Borborema |
| ENERGISA MG | EMG; CATAGUASES; CFLCL; Companhia Forca e Luz Cataguazes-Leopoldina |
| ENERGISA MS | EMS; ENERSUL; Empresa Energetica de Mato Grosso do Sul |
| ENERGISA MT | EMT; CEMAT; Centrais Eletricas Matogrossenses |
| ENERGISA NOVA FRIBURGO | ENF; CENF; Companhia de Eletricidade de Nova Friburgo |
| ENERGISA PB | EPB; SAELPA; Sociedade Anonima de Eletrificacao da Paraiba |
| ENERGISA RO | ERO; CERON; Centrais Eletricas de Rondonia |
| ENERGISA SE | ESE; ENERGIPE; Empresa Energetica de Sergipe |
| ENERGISA SUL SUDESTE | ESS; ENERGISA SUL-SUDESTE |
| ENERGISA TO | ETO; CELTINS; Companhia de Energia Eletrica do Estado do Tocantins |
| EQUATORIAL AL | EAL; CEAL; Companhia Energetica de Alagoas |
| EQUATORIAL GO | EGO; CELG-D; CELG Distribuicao; Companhia Energetica de Goias |
| EQUATORIAL MA | EMA; CEMAR; Companhia Energetica do Maranhao |
| EQUATORIAL PA | EPA; CELPA; Centrais Eletricas do Para |
| EQUATORIAL PI | EPI; CEPISA; Companhia Energetica do Piaui |
| FORCEL | Forca e Luz Coronel Vivida |
| HIDROPAN | Hidroeletrica Panambi |
| LIGHT | LIGHT S.A.; Light Servicos de Eletricidade |
| MUXENERGIA | MUX Energia; MUX |
| NEOENERGIA BRASILIA | CEB-DIS; CEB-D; CEB; CEB Distribuicao; Companhia Energetica de Brasilia |
| NEOENERGIA COELBA | COELBA; Companhia de Eletricidade do Estado da Bahia |
| NEOENERGIA COSERN | COSERN; Companhia Energetica do Rio Grande do Norte |
| NEOENERGIA ELEKTRO | ELEKTRO; Elektro Redes; Elektro Eletricidade e Servicos |
| NEOENERGIA PERNAMBUCO | CELPE; Companhia Energetica de Pernambuco; NEOENERGIA PE |
| NOVA PALMA | Nova Palma Energia |
| RGE | RGE SUL; Rio Grande Energia; CPFL RGE; AES SUL |
| RORAIMA ENERGIA | CERR; Boa Vista Energia; Companhia Energetica de Roraima |
| SULGIPE | Companhia Sul Sergipana de Eletricidade |

---

## 7. Verification & Testing Strategy

### 7.1 Nexus Core Tests

**Unit Tests:**

| Test File | Tests |
|-----------|-------|
| `tests/unit/models/test_energy_invoice.py` | Pydantic model validation — GroupB, GroupAVerde, GroupAAzul field ranges, required vs optional |
| `tests/unit/models/test_energy_invoice_agent.py` | Agent output model validation — all fields optional, correct types |
| `tests/unit/services/test_energy_invoice_service.py` | Service validation logic — valid ranges pass, invalid ranges → missing |
| `tests/unit/services/test_energy_invoice_derivation.py` | `derive_missing_fields_group_b` — kwh_price from te+tusd, from total/consumption; icms from value/base; tusd from tusd_unit_price. `derive_missing_fields_group_a` — icms derivation. Verify derivation skipped when primary field already present. |
| `tests/unit/services/test_energy_invoice_postprocessing.py` | `fill_missing_months_with_average` — partial months, all present, all missing |
| `tests/unit/services/test_energy_invoice_postprocessing.py` | `replicate_average_to_months` — correct replication |
| `tests/unit/services/test_energy_invoice_success_criteria.py` | Success criteria — Group B mandatory fields, Group A Verde/Azul mandatory fields |
| `tests/unit/services/test_concessionaria_validation.py` | Concessionária enum matching — exact match, alias resolution, unknown → None |

**Integration Tests:**

| Test File | Tests |
|-----------|-------|
| `tests/integration/test_energy_invoice_extraction.py` | Full extraction flow with sample invoices from `shared/samples/faturas-de-energia/` |
| `tests/integration/test_energy_invoice_endpoint.py` | API endpoint — auth, request validation, response format |
| `tests/integration/test_energy_invoice_models_endpoint.py` | Models list endpoint — auth, response format |

**Sample Invoice Test Matrix:**

| Sample File | Group | Expected Fields | Notes |
|-------------|-------|----------------|-------|
| `fatura-bulbe-grupo-b.pdf` | B | kwhPrice, networkClass, monthlyConsumption (partial) | Test average fill |
| `fatura-CERRP-grupo-b.pdf` | B | kwhPrice, networkClass, powerDistCompany=CERRP | Test cooperative alias |
| `fatura-CPFL-grupo-b.pdf` | B | kwhPrice, networkClass, powerDistCompany=CPFL PAULISTA | Test major utility |
| `fatura-enel-grupo-b.pdf` | B | kwhPrice, networkClass, powerDistCompany=ENEL SP | Test Enel |
| `fatura-enel-grupo-b-2.pdf` | B | kwhPrice, networkClass | Second Enel variant |
| `fatura-copel-grupo-a.pdf` | A | classification, kwhPrice, kwhPricePeak, demand | Test Group A |
| `fatura-enel-grupo-a.pdf` | A | classification, kwhPrice, kwhPricePeak, demand | Test Enel Group A |
| `fatura-equatorial-grupo-a.pdf` | A | classification, kwhPrice, kwhPricePeak, demand | Test Equatorial Group A |

### 7.2 Azume Backend Tests

**Unit Tests:**

| Test | Description |
|------|-------------|
| `extractInvoice` controller | File validation (format, size), proposal lookup, Nexus API call mock, archive persistence, result transformation |
| `getAvailableModels` controller | Admin check, Nexus API call mock, cache behavior |
| `persistInvoiceInArchive` | Find/create "Faturas de Energia" folder, add file to folder |
| `transformNexusResult` | Group B transformation, Group A Verde transformation, Group A Azul transformation, snake_to_camel field mapping |

### 7.3 Frontend Tests

**Component Tests:**

| Component | Tests |
|-----------|-------|
| `SmartReadingButton` | Renders with Nexus logo and text, click handler fires, disabled state |
| `InvoiceUploadModal` | File selection, drag-and-drop, format validation, model selector (admin vs non-admin), loading state, cancel/submit |
| `InvoiceValidationPopup` | Displays extracted fields, editable inputs, missing fields warning, confirm applies changes |
| `applyExtractionToForm` | Group B form population (all fields), Group A Verde population, Group A Azul population, partial fields (some missing) |

### 7.4 E2E Test

Full flow with sample invoices:

```
1. Login as CRM user
2. Navigate to proposal step 1
3. Click "Leitura Smart"
4. Upload sample invoice (e.g., fatura-CPFL-grupo-b.pdf)
5. Wait for extraction (mock Nexus response for speed)
6. Verify validation popup shows correct fields
7. Confirm → verify form fields are populated
8. Verify tariffModality, kwhPrice, monthlyConsumption, networkClass are set
9. Proceed to step 2 (verify data persists)
```

### 7.5 Key File Paths (Reference)

**Nexus Core:**
- Router: `services/api/src/nexus_core_api/routers/energy_invoice.py`
- Schemas: `services/api/src/nexus_core_api/schemas/energy_invoice.py`
- Models: `packages/nexus/nexus-models/src/nexus_models/energy_invoice.py`
- Agent models: `packages/nexus/nexus-models/src/nexus_models/energy_invoice_agent.py`
- Service protocol: `packages/nexus/nexus-services/src/nexus_services/protocols/energy_invoice_service.py`
- Service impl: `packages/nexus/nexus-services/src/nexus_services/impl/energy_invoice_service_impl.py`
- Agent factory: `packages/nexus/nexus-ai/src/nexus_ai/factories/energy_invoice.py`
- System prompt: `packages/nexus/nexus-contexts/src/nexus_contexts/energy_invoice.py`
- Existing auth: `services/api/src/nexus_core_api/auth/dependencies.py`
- Existing storage: `packages/core/core-storage/src/core_storage/client.py`
- Existing agent: `packages/core/core-ai/src/core_ai/agent.py`

**Azume Backend:**
- Route: `src/routes/invoiceReadingRoutes.ts`
- Controller: `src/controllers/invoiceReading/extractInvoice.ts`
- Models controller: `src/controllers/invoiceReading/getAvailableModels.ts`
- Proposal model: `src/models/proposal.ts` (add `invoiceFileUrls`)
- User model: `src/models/user.ts` (add `isSystemAdmin`)
- Archive model: `src/models/archive.ts` (no schema changes, types only)
- Types: `src/data/types.ts` (extend `FolderSemantics`)
- Nexus OAuth: `src/util/nexus/nexusOAuthClient.ts` (existing, reuse)
- File upload: `src/middlewares/fileUpload.ts` (existing `singleFilesUpload`)
- Auth: `src/middlewares/checkAuth.ts` (add `isSystemAdmin`)
- App mount: `src/app.ts`

**Azume Frontend:**
- SmartReadingButton: `src/proposal/components/SmartReadingButton.tsx`
- InvoiceUploadModal: `src/proposal/components/InvoiceUploadModal.tsx`
- InvoiceValidationPopup: `src/proposal/components/InvoiceValidationPopup.tsx`
- Step 1 page: `src/proposal/pages/ProposalStepOne.tsx`
- API functions: `src/proposal/api/proposalsAPI.ts`
- Auth context: `src/shared/context/authContext.ts`
- Auth hook: `src/shared/hooks/authHook.ts`
- Form hook: `src/shared/hooks/formHook.ts`
- Types: `src/shared/data/types.ts`
