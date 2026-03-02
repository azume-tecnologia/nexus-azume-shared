# Leitor de Faturas de Energia — Cross-Project Implementation Spec

**Version:** 1.0.0
**Date:** 2026-03-02
**Feature Doc:** `docs/02_feature_leitor_de_faturas.md`
**Sample Invoices:** `samples/faturas-de-energia/`

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
19. [Backend]  Persist file URL in "proposal" collection (new invoiceFileUrl field)
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
      tusd?: number,                         // R$ (>= 0)
      icms?: number,                         // % (> 0, < 100)

      // Group A Verde fields (in addition to/instead of Group B)
      kwhPricePeak?: number,                 // R$/kWh (> 0, < 10)
      tusdPeak?: number,                     // R$/kWh (> 0, < 10)
      demand?: number,                       // kW (>= 30)
      demandTariff?: number,                 // R$ (> 0)
      averageConsumption?: number,           // kWh (> 0) — replicated to 12 months
      averageConsumptionPeak?: number,       // kWh (> 0) — replicated to 12 months

      // Group A Azul additional fields
      demandPeak?: number,                   // kW (> 0)
      demandTariffPeak?: number,             // R$ (> 0)
    },
    missingFields: string[],                 // Field names the agent could not extract
    invoiceFileUrl: string,                  // Permanent S3 URL of the uploaded invoice
  }
}
```

**Error Response:** `400 | 403 | 422 | 500`

```typescript
{
  success: false,
  message: string,                           // Portuguese user-facing message
  errorCode: "INVALID_FILE_FORMAT"           // Machine-readable error code
    | "FILE_TOO_LARGE"
    | "EXTRACTION_FAILED"                    // LLM could not extract mandatory fields
    | "NEXUS_UNAVAILABLE"                    // Nexus API unreachable/timeout
    | "INVALID_PROPOSAL"
    | "UNAUTHORIZED"
    | "INTERNAL_ERROR",
  missingFields?: string[],                  // Only for EXTRACTION_FAILED
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
  "file_url": "https://azume-files.cdn.digitaloceanspaces.com/uploads/archives/xxx/invoice.pdf?X-Amz-...",
  "model": "gpt-5-mini"
}
```

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
  "missing_fields": ["kwhPrice", "networkClass", "monthlyConsumption"],
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
- Azume Backend should set HTTP timeout to **60 seconds** for this endpoint
- Nexus should set LLM agent timeout to **45 seconds**

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
    monthly_consumption: Optional[list[Optional[float]]] = Field(
        default=None, min_length=12, max_length=12,
        description="Monthly consumption in kWh (Jan-Dec). None for missing months."
    )
    tusd: Optional[float] = Field(
        default=None, ge=0,
        description="TUSD tariff in R$"
    )
    icms: Optional[float] = Field(
        default=None, gt=0, lt=100,
        description="ICMS tax percentage"
    )


class GroupAVerdeExtraction(BaseModel):
    """Grupo A Verde (alta tensão, horo-sazonal verde) extraction result."""
    power_dist_company: Optional[str] = Field(
        default=None,
        description="Power distribution company name (concessionária)"
    )
    kwh_price: float = Field(
        ..., gt=0, lt=10,
        description="TE Fora Ponta (off-peak energy tariff) in R$"
    )
    kwh_price_peak: float = Field(
        ..., gt=0, lt=10,
        description="TE Ponta (peak energy tariff) in R$"
    )
    tusd: float = Field(
        ..., gt=0, lt=10,
        description="TUSD Fora Ponta (off-peak distribution tariff) in R$"
    )
    tusd_peak: float = Field(
        ..., gt=0, lt=10,
        description="TUSD Ponta (peak distribution tariff) in R$"
    )
    demand: float = Field(
        ..., ge=30,
        description="Contracted demand in kW"
    )
    demand_tariff: float = Field(
        ..., gt=0,
        description="Demand tariff in R$"
    )
    public_light_bill: Optional[float] = Field(
        default=None, gt=0,
        description="Public lighting tax (CIP/COSIP) in R$"
    )
    average_consumption: float = Field(
        ..., gt=0,
        description="Average monthly off-peak consumption in kWh"
    )
    average_consumption_peak: float = Field(
        ..., gt=0,
        description="Average monthly peak consumption in kWh"
    )
    icms: Optional[float] = Field(
        default=None, gt=0, lt=100,
        description="ICMS tax percentage"
    )


class GroupAAzulExtraction(BaseModel):
    """Grupo A Azul (alta tensão, horo-sazonal azul) extraction result.
    Extends Verde with peak demand fields.
    """
    power_dist_company: Optional[str] = Field(
        default=None,
        description="Power distribution company name (concessionária)"
    )
    kwh_price: float = Field(
        ..., gt=0, lt=10,
        description="TE Fora Ponta (off-peak energy tariff) in R$"
    )
    kwh_price_peak: float = Field(
        ..., gt=0, lt=10,
        description="TE Ponta (peak energy tariff) in R$"
    )
    tusd: float = Field(
        ..., gt=0, lt=10,
        description="TUSD Fora Ponta (off-peak distribution tariff) in R$"
    )
    tusd_peak: float = Field(
        ..., gt=0, lt=10,
        description="TUSD Ponta (peak distribution tariff) in R$"
    )
    demand: float = Field(
        ..., ge=30,
        description="Contracted off-peak demand in kW"
    )
    demand_tariff: float = Field(
        ..., gt=0,
        description="Off-peak demand tariff in R$"
    )
    demand_peak: float = Field(
        ..., gt=0,
        description="Contracted peak demand in kW"
    )
    demand_tariff_peak: float = Field(
        ..., gt=0,
        description="Peak demand tariff in R$"
    )
    public_light_bill: Optional[float] = Field(
        default=None, gt=0,
        description="Public lighting tax (CIP/COSIP) in R$"
    )
    average_consumption: float = Field(
        ..., gt=0,
        description="Average monthly off-peak consumption in kWh"
    )
    average_consumption_peak: float = Field(
        ..., gt=0,
        description="Average monthly peak consumption in kWh"
    )
    icms: Optional[float] = Field(
        default=None, gt=0, lt=100,
        description="ICMS tax percentage"
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


class AgentExtractionOutput(BaseModel):
    """Top-level structured output from the LLM agent."""
    tariff_modality: str = Field(
        ...,
        description="Modalidade tarifária: 'A' ou 'B'"
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
9. Post-process (fill missing months for Group B)
10. Map `AgentExtractionOutput` → `InvoiceExtractionResult`
11. Determine success/failure based on mandatory field presence

**Validation Rules:**

| Field | Rule | Action on Violation |
|-------|------|-------------------|
| `kwh_price` | `> 0 AND < 10` | Mark as missing |
| `kwh_price_peak` | `> 0 AND < 10` | Mark as missing |
| `tusd` (Group A) | `> 0 AND < 10` | Mark as missing |
| `tusd_peak` | `> 0 AND < 10` | Mark as missing |
| `demand` | `>= 30` | Mark as missing |
| `demand_tariff` | `> 0` | Mark as missing |
| `demand_peak` | `> 0` | Mark as missing |
| `demand_tariff_peak` | `> 0` | Mark as missing |
| `network_class` | Must be one of enum values | Mark as missing |
| `monthly_consumption[i]` | `>= 0` | Set to None (missing) |
| `public_light_bill` | `> 0` | Set to None (best-effort) |
| `tusd` (Group B) | `>= 0` | Set to None (best-effort) |
| `icms` | `> 0 AND < 100` | Set to None (best-effort) |
| `power_dist_company` | Must match concessionária enum | Set to None (best-effort) |
| `classification` | Must be "Azul" or "Verde" | Mark as missing |

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
```

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
def create_energy_invoice_service() -> EnergyInvoiceService:
    """Create EnergyInvoiceService with all dependencies."""
    from core_storage.client import StorageClient
    from core_http.client import HttpClient
    from nexus_ai.factories import create_energy_invoice_agent_factory

    return EnergyInvoiceServiceImpl(
        storage_client=StorageClient(),
        http_client=HttpClient(),
        agent_factory=create_energy_invoice_agent_factory(),
    )
```

#### `nexus-ai` — Agent Factory

**New file:** `packages/nexus/nexus-ai/src/nexus_ai/factories/energy_invoice.py`

```python
from core_ai.agent import OpenAIAgent
from nexus_contexts.energy_invoice import ENERGY_INVOICE_SYSTEM_PROMPT
from nexus_models.energy_invoice_agent import AgentExtractionOutput


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
        """
        agent = OpenAIAgent()
        await agent.initialize_agent(
            name="Energy Invoice Extractor",
            instructions=ENERGY_INVOICE_SYSTEM_PROMPT,
            tools=[],
            mcp_servers=[],
            output_type=AgentExtractionOutput,
            model=model,
            model_settings=None,
        )

        # Build user message with images
        # Images are passed as base64-encoded content parts
        result = await agent.run_agent(
            user_message="Extraia os dados desta fatura de energia.",
            session=None,
            context=None,
            images=images,
        )

        return result.final_output
```

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

Defina `tariff_modality` como "A" ou "B".

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
   - Divida o valor total de energia pelo consumo se necessário para verificar
   - **IMPORTANTE**: O valor deve incluir TODOS os componentes (TE + TUSD + tributos). Se a fatura apresentar componentes separados, some-os.
   - Faixa esperada: R$ 0,30 a R$ 2,50 por kWh (valores fora desta faixa podem indicar erro de leitura)

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
   - "Base de Cálculo ICMS" e "Valor ICMS" (calcule o percentual se necessário)
   - Geralmente entre 12% e 33%

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
    - Procure por "Consumo Médio" ou calcule a partir do histórico se disponível
    - Se houver histórico de consumo, use o consumo fora ponta

12. **average_consumption_peak**: Consumo médio mensal ponta em kWh.
    - Procure por "Consumo Médio Ponta" ou consumo ponta no histórico

13. **icms**: Mesmo procedimento do Grupo B.

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
from nexus_core_api.auth.dependencies import get_nexus_client
from nexus_core_api.schemas.energy_invoice import (
    ExtractionRequest,
    ExtractionResponse,
)
from nexus_services.protocols.energy_invoice_service import (
    EnergyInvoiceService,
    EnergyInvoiceExtractionInput,
)
from nexus_core_api.dependencies.energy_invoice import get_energy_invoice_service

router = APIRouter(tags=["Energy Invoice"], prefix="/energy-invoice")


@router.post(
    "/extract",
    response_model=ExtractionResponse,
    status_code=status.HTTP_200_OK,
    summary="Extract data from an energy invoice",
)
async def extract_invoice(
    request: ExtractionRequest,
    client=Depends(get_nexus_client),
    service: EnergyInvoiceService = Depends(get_energy_invoice_service),
) -> ExtractionResponse:
    result = await service.extract_invoice_data(
        EnergyInvoiceExtractionInput(
            file_url=request.file_url,
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
    client=Depends(get_nexus_client),
) -> dict:
    return {
        "models": [
            {
                "id": "gpt-5-mini",
                "display_name": "GPT-5 Mini",
                "provider": "openai",
                "is_default": True,
            },
            {
                "id": "gemini-3-flash-preview",
                "display_name": "Gemini 3 Flash",
                "provider": "google",
                "is_default": False,
            },
            {
                "id": "claude-sonnet-4-5",
                "display_name": "Claude Sonnet 4.5",
                "provider": "anthropic",
                "is_default": False,
            },
            {
                "id": "grok-4-1-fast",
                "display_name": "Grok 4.1 Fast",
                "provider": "xai",
                "is_default": False,
            },
        ]
    }
```

**Router registration in `main.py`:**

```python
from nexus_core_api.routers import energy_invoice
app.include_router(energy_invoice.router, prefix=API_PREFIX)
```

### 3.4 Request/Response Schemas

**New file:** `services/api/src/nexus_core_api/schemas/energy_invoice.py`

```python
from pydantic import BaseModel, Field, HttpUrl
from typing import Optional


class ExtractionRequest(BaseModel):
    file_url: str = Field(..., description="Pre-signed URL to the invoice file")
    model: Optional[str] = Field(
        default=None,
        description="LLM model ID. Defaults to gpt-5-mini."
    )


class ExtractionResponse(BaseModel):
    """Maps InvoiceExtractionResult to HTTP response."""
    success: bool
    tariff_modality: Optional[str] = None
    classification: Optional[str] = None
    group_b: Optional[dict] = None
    group_a_verde: Optional[dict] = None
    group_a_azul: Optional[dict] = None
    missing_fields: list[str] = []
    model_used: str
    error_message: Optional[str] = None

    @classmethod
    def from_domain(cls, result) -> "ExtractionResponse":
        return cls(
            success=result.success,
            tariff_modality=result.tariff_modality.value if result.tariff_modality else None,
            classification=result.classification.value if result.classification else None,
            group_b=result.group_b.model_dump(exclude_none=False) if result.group_b else None,
            group_a_verde=result.group_a_verde.model_dump(exclude_none=False) if result.group_a_verde else None,
            group_a_azul=result.group_a_azul.model_dump(exclude_none=False) if result.group_a_azul else None,
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
   ↓
3. Detect content type from HTTP headers + file magic bytes
   ↓
4. Upload to GCS temp path: energy-invoices/temp/{uuid}/{filename}
   ↓
5. If PDF:
   │  a. Open with PyMuPDF (fitz)
   │  b. For each page: render as PNG at 300 DPI
   │  c. Collect page images as bytes[]
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
9. Post-process:
   │  - Group B: fill missing months with average
   │  - Group A: replicate average to 12-month arrays
   ↓
10. Map to InvoiceExtractionResult
   ↓
11. Determine success/failure based on mandatory fields
   ↓
12. Schedule GCS temp file cleanup (async, non-blocking)
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

**Mount in `app.ts`:**

```typescript
import invoiceReadingRoutes from "./routes/invoiceReadingRoutes";
// ...
app.use("/api/proposals", invoiceReadingRoutes);
```

### 4.2 Auth & User Model Changes

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

### 4.3 Main Extraction Controller

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
      return next(new HttpError("Proposta não encontrada.", 404));
    }

    // 3. File is already uploaded to S3 by singleFilesUpload middleware
    //    Generate pre-signed URL for Nexus
    const fileKey = (file as any).key;
    const fileUrl = `https://azume-files.cdn.digitaloceanspaces.com/${fileKey}`;

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
        `[Invoice Reading] Nexus extraction failed: ${nexusErr.message}`
      );
      return next(new HttpError(
        "Não foi possível conectar ao serviço de leitura. Tente novamente.",
        502
      ));
    }

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
    await persistInvoiceInArchive(customerId, fileUrl, file.originalname);

    // 7. Persist file URL in proposal
    await Proposal.findByIdAndUpdate(pid, {
      $set: { invoiceFileUrl: fileUrl },
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
  if (!archive) return; // No archive for this customer — skip silently

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
invoiceFileUrl?: string;

// In ProposalSchema:
invoiceFileUrl: { type: String, default: null },
```

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
- Loading state: replace content with `LoadingSpinnerFullScreenGraph` + text "Extraindo dados da fatura... Isso pode levar até 30 segundos."

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
    - List of missing field labels (in Portuguese)
- `DialogActions`:
  - "Cancelar" — close without applying
  - "Confirmar e Preencher" — apply values to the main form

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
 */
export const extractInvoiceData = async (props: {
  sendRequest: SendRequestFn;
  auth: AuthContext;
  pid: string;
  ucid: number;
  customerId: string;
  file: File;
  model?: string;
}) => {
  const { sendRequest, auth, pid, ucid, customerId, file, model } = props;

  const formData = new FormData();
  formData.append("file", file);
  formData.append("ucid", String(ucid));
  formData.append("customerId", customerId);
  if (model) formData.append("model", model);

  const apiUrl = `${process.env.REACT_APP_BACKEND_URL}/proposals/${pid}/invoice-reading`;

  const responseData = await sendRequest(
    apiUrl,
    "POST",
    formData,
    { Authorization: "Bearer " + auth.token },
    // Note: do NOT set Content-Type — browser sets it with boundary for FormData
  );

  return responseData;
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
│   State: availableModels (fetched on mount if isSystemAdmin)
│
├── SmartReadingButton
│   └── onClick → setShowUploadModal(true)
│
├── InvoiceUploadModal
│   ├── Props: open, onClose, onSubmit, isLoading, isSystemAdmin, availableModels
│   └── onSubmit → extractInvoiceData() → setExtractionResult() → setShowValidationPopup(true)
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
│   └── Shows: "Extraindo dados da fatura... Isso pode levar até 30 segundos."
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
| Network error | "Erro de conexão. Verifique sua internet e tente novamente." | Show in modal (ModalError) |
| Timeout | "A leitura demorou mais que o esperado. Tente novamente." | Show in modal (ModalError) |

### 5.12 AuthContext Changes

**Modified file:** `src/shared/context/authContext.ts`

Add `isSystemAdmin` to the auth context:

```typescript
// In AuthContextProps interface:
isSystemAdmin: boolean;

// In createContext default:
isSystemAdmin: false,
```

**Modified file:** `src/shared/hooks/authHook.ts`

Add `isSystemAdmin` to the login flow:

```typescript
// In login function — extract from JWT or from backend response:
const login = useCallback(
  (userId, email, ..., isSystemAdmin, ...) => {
    // Store isSystemAdmin in state
    setIsSystemAdmin(isSystemAdmin);
  },
  []
);
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
- Proposal model: `src/models/proposal.ts` (add `invoiceFileUrl`)
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
