---
project: ct-planning-artifacts
author: Mary (BMad Analyst)
requested_by: Sehman.Hussain
date: 2026-04-28
source_input: docs/SearchAPI-AI-Input
mapped_epics: ["Epic 1", "Epic 2", "Epic 4", "Epic 5", "Epic 10", "Epic 13", "Epic 15", "Epic 17"]
status: draft
---

# Search API — Functional & Non-Functional Requirements

> Derived from `docs/SearchAPI-AI-Input`. Requirements are written as traceable statements mapped to existing Epics and Features in `epics.md`. New requirements not previously captured in the FR/NFR inventory are flagged explicitly.

---

## Search Input Methods — Functional Requirements

### FR-SA-01 — License Plate Search Input
The Search API **must** accept a License Plate Number and US State (or Canadian Province) as input parameters. The system must pass both values to the Mitchell1 VIN decode integration to resolve vehicle Year/Make/Model/Trim before returning tire results.
- **Validation:** License plate format must be validated for the selected state/province before the API call is dispatched. Invalid formats must return a structured error with a user-facing correction message.
- **Maps to:** Epic 1 — Tire Discovery & Vehicle Matching; Epic 4 — Search API
- **Extends:** FR1, FR2

### FR-SA-02 — Cascading Vehicle Search Input
The Search API **must** accept a four-part vehicle selector: Year → Make → Model → Trim, where each subsequent field is only enabled and populated **after** the prior field has a confirmed selection. The available values for each dependent field must be fetched dynamically based on the preceding selection.
- **Validation:** All four fields (Year, Make, Model, Trim) must be provided and resolved before a tire search query is dispatched.
- **Maps to:** Epic 1 — Tire Discovery & Vehicle Matching; Epic 4 — Search API
- **Extends:** FR1

### FR-SA-03 — Tire Size Search Input
The Search API **must** accept three discrete tire dimension parameters: Width (numeric, e.g., 225), Aspect Ratio (numeric, e.g., 65), and Rim Diameter (numeric, e.g., 17). The combined format must be validated as Width/AspectRatio/Diameter (###/##R##) before dispatch.
- **Validation:** Non-numeric input and out-of-range values must be rejected with a structured error and a human-readable correction hint.
- **Maps to:** Epic 1 — Tire Discovery & Vehicle Matching; Epic 4 — Search API
- **Extends:** FR1, FR2

### FR-SA-04 — Staggered Fitment — Rear Tire Input ⚠️ NEW
When a vehicle fitment requires staggered sizing (different front and rear tire sizes), the Search API **must** accept an optional second tire size set (Width, Aspect, Rim) for the rear axle. The system must surface a clear UI control enabling the member to specify a different rear tire size, and must return separate product result sets for front and rear where applicable.
- **Maps to:** Epic 1 — Tire Discovery & Vehicle Matching; Epic 4 — Search API
- **Note:** Not explicitly covered in existing FR inventory — net-new requirement.

### FR-SA-05 — Item Number / Part Number Search
The Search API **must** accept a Costco Item Number **or** a Tire Manufacturer Part Number as a single search input. The system must resolve the input against the product catalog and return an exact match or a ranked near-match list.
- **Validation:** Item/Part numbers containing invalid characters must be rejected with a structured error message.
- **Maps to:** Epic 1 — Tire Discovery & Vehicle Matching; Epic 4 — Search API
- **Extends:** FR1

---

## Product Listing Page (PLP) — Filter Parameters

### FR-SA-06 — Price Range Filter
The Search API **must** support a Price Range filter parameter (min price, max price) to restrict returned tire results to products within the specified price band. Results outside the band must be excluded from the response payload.
- **Maps to:** Epic 5 — Tire Browsing, Filters & PDP
- **Extends:** FR18

### FR-SA-07 — Promotion Filter
The Search API **must** support a boolean Promotion filter. When set to `true`, results must be restricted to tires currently tagged with an active promotion or discount. The filter must respect promotion start/end dates in real time.
- **Maps to:** Epic 5 — Tire Browsing, Filters & PDP
- **Extends:** FR18

### FR-SA-08 — Brand Filter
The Search API **must** support a multi-select Brand filter. The available brand values must be dynamically derived from the current result set (faceted filtering), and the API must return an updated result count per brand value.
- **Maps to:** Epic 5 — Tire Browsing, Filters & PDP
- **Extends:** FR18

### FR-SA-09 — Season Filter
The Search API **must** support a Season filter with the following enumerated values: All-Season, Summer, Winter, All-Weather. Only seasons represented in the current result set must be returned as available filter options.
- **Maps to:** Epic 5 — Tire Browsing, Filters & PDP
- **Extends:** FR18

### FR-SA-10 — Vehicle Compatibility Filter
The Search API **must** support a Compatibility filter that, when a vehicle context is present in the session, restricts results to tires confirmed compatible with the member's selected vehicle fitment. When no vehicle context is present, this filter must be disabled.
- **Maps to:** Epic 5 — Tire Browsing, Filters & PDP; Epic 15 — Cross Application Context Preservation
- **Extends:** FR18, FR5

### FR-SA-11 — Run Flat Filter
The Search API **must** support a boolean Run Flat filter. When enabled, only tires carrying a Run Flat / Extended Mobility designation must be returned.
- **Maps to:** Epic 5 — Tire Browsing, Filters & PDP
- **Extends:** FR18

### FR-SA-12 — Load Index Filter
The Search API **must** support a Load Index filter accepting a numeric range (min/max load index value). Results must be restricted to products whose load index falls within the specified range.
- **Maps to:** Epic 5 — Tire Browsing, Filters & PDP
- **Extends:** FR18

### FR-SA-13 — Speed Rating Filter
The Search API **must** support a Speed Rating filter accepting one or more enumerated values from the standard speed rating scale (e.g., H, V, W, Y, ZR). Results must match any one of the selected values (OR logic).
- **Maps to:** Epic 5 — Tire Browsing, Filters & PDP
- **Extends:** FR18

---

## Product Listing Page (PLP) — Sort Parameters

### FR-SA-14 — Best Match Sort
The Search API **must** support a "Best Match" sort mode as the default sort order. Best Match ranking must factor in fitment compatibility, promotional status, stock availability, and historical sales relevance. The ranking algorithm must be configurable without a code deployment.
- **Maps to:** Epic 4 — Search API; Epic 5 — Tire Browsing, Filters & PDP
- **Extends:** FR9

### FR-SA-15 — Top Selling Sort
The Search API **must** support a "Top Selling" sort parameter, ordering results by historical sales volume (descending) scoped to the member's warehouse region where data is available.
- **Maps to:** Epic 5 — Tire Browsing, Filters & PDP

### FR-SA-16 — Price Sort (Ascending / Descending)
The Search API **must** support two price-based sort parameters: "Price: Low to High" (ascending) and "Price: High to Low" (descending). Sort must apply to the member-visible price (after promotions are applied, where applicable).
- **Maps to:** Epic 5 — Tire Browsing, Filters & PDP

### FR-SA-17 — UTQG Sort ⚠️ NEW
The Search API **must** support a "UTQG: High to Low" sort parameter, ordering results by the Treadwear component of the UTQG (Uniform Tire Quality Grading) rating in descending order. Products without a UTQG rating must be ranked last.
- **Maps to:** Epic 5 — Tire Browsing, Filters & PDP
- **Note:** Not called out in existing FR inventory — net-new requirement.

### FR-SA-18 — Promotion Sort
The Search API **must** support a "Promotion" sort parameter, surfacing currently promoted tires at the top of the result set. Within the promoted group, secondary sort must default to Best Match.
- **Maps to:** Epic 5 — Tire Browsing, Filters & PDP; Epic 2 — Homepage & Brand Content

---

## Non-Functional Requirements

### NFR-SA-01 — Search API Response Time
All Search API queries (across all four input methods) must return complete results in **≤500ms at p95** under normal load. Autocomplete/typeahead variants of the API must return in **≤150ms at p95**.
- **Maps to:** Epic 4 — Search API; Epic 13 — Performance, Reliability & Observability
- **Extends:** NFR2

### NFR-SA-02 — Filter & Sort Response Time ⚠️ NEW SPECIFICITY
Applying any PLP filter or changing the sort parameter must produce an updated result set within **≤300ms at p95** without a full page reload (incremental/API-driven result refresh).
- **Maps to:** Epic 5 — Tire Browsing, Filters & PDP; Epic 13 — Performance, Reliability & Observability
- **Note:** Adds precision not captured in existing NFR2.

### NFR-SA-03 — Search Input Validation & Injection Prevention
All Search API input parameters (license plate, vehicle fields, tire dimensions, item numbers, filter values) must be validated and sanitised server-side before processing. SQL injection, XSS, and parameter tampering vectors must be blocked at the API gateway layer.
- **Maps to:** Epic 17 — Security and Privacy
- **Extends:** NFR7, NFR10

### NFR-SA-04 — Search API Rate Limiting
Search API endpoints must be rate-limited to **100 requests/minute per authenticated member** and **30 requests/minute per unauthenticated session**. Requests exceeding the limit must receive HTTP 429 with a `Retry-After` header.
- **Maps to:** Epic 17 — Security and Privacy
- **Extends:** NFR10

### NFR-SA-05 — Search API Availability & Graceful Degradation
The Search API must maintain **99.99% uptime**. In the event of a downstream dependency failure (e.g., Mitchell1 VIN decode unavailable), the system must degrade gracefully — disabling the affected input method (License Plate Search) while keeping Size and Vehicle search operational, with a clear user-facing status message.
- **Maps to:** Epic 13 — Performance, Reliability & Observability
- **Extends:** NFR4, NFR5

### NFR-SA-06 — Cascading Vehicle Dropdown Performance ⚠️ NEW SPECIFICITY
Each dependent dropdown call (Make after Year, Model after Make, Trim after Model) must resolve in **≤200ms at p95**. Data must be sourced from a cached lookup layer (Redis); the cache must be invalidated and refreshed on catalog updates.
- **Maps to:** Epic 4 — Search API; Epic 13 — Performance, Reliability & Observability
- **Note:** Cascade dependency not previously quantified in NFR inventory.

### NFR-SA-07 — Filter Facet Count Accuracy ⚠️ NEW
Filter facet result counts returned by the Search API must exactly match the number of products that would be returned if that filter value were applied. Stale or approximate counts are not acceptable.
- **Maps to:** Epic 5 — Tire Browsing, Filters & PDP
- **Note:** Data quality requirement for faceted counts — not previously captured.

### NFR-SA-08 — Search Observability
All Search API calls must emit structured telemetry: input method used, filter/sort parameters applied, result count returned, response latency, and session/member context (non-PII). A search success rate dashboard (target: **≥95% of queries returning ≥1 result**) must be maintained in real time.
- **Maps to:** Epic 4 — Search API; Epic 10 — Customer Analytics & Personalization; Epic 13 — Performance, Reliability & Observability
- **Extends:** NFR19

---

## Epic & Feature Mapping Summary

| Requirement | Epic | Feature Area |
|---|---|---|
| FR-SA-01 License Plate Search | Epic 1, Epic 4 | License Plate Search, Search API |
| FR-SA-02 Cascading Vehicle Search | Epic 1, Epic 4 | Vehicle Search, Search API |
| FR-SA-03 Tire Size Search | Epic 1, Epic 4 | Tire Size Search, Search API |
| FR-SA-04 Staggered Fitment Rear Input ⚠️ | Epic 1, Epic 4 | Fitment Resolution, Search API |
| FR-SA-05 Item # / Part # Search | Epic 1, Epic 4 | Item Number Search, Search API |
| FR-SA-06 Price Range Filter | Epic 5 | PLP Filters |
| FR-SA-07 Promotion Filter | Epic 5 | PLP Filters, Pricing & Promotions |
| FR-SA-08 Brand Filter | Epic 5 | PLP Filters |
| FR-SA-09 Season Filter | Epic 5 | PLP Filters |
| FR-SA-10 Compatibility Filter | Epic 5, Epic 15 | PLP Filters, Context Preservation |
| FR-SA-11 Run Flat Filter | Epic 5 | PLP Filters |
| FR-SA-12 Load Index Filter | Epic 5 | PLP Filters |
| FR-SA-13 Speed Rating Filter | Epic 5 | PLP Filters |
| FR-SA-14 Best Match Sort | Epic 4, Epic 5 | Search API, PLP Navigation Filters |
| FR-SA-15 Top Selling Sort | Epic 5 | PLP Navigation Filters |
| FR-SA-16 Price Sort Asc/Desc | Epic 5 | PLP Navigation Filters |
| FR-SA-17 UTQG Sort ⚠️ | Epic 5 | PLP Navigation Filters |
| FR-SA-18 Promotion Sort | Epic 5, Epic 2 | PLP Navigation Filters |
| NFR-SA-01 Search Response Time | Epic 4, Epic 13 | Performance |
| NFR-SA-02 Filter/Sort Response Time ⚠️ | Epic 5, Epic 13 | Performance |
| NFR-SA-03 Input Validation & Security | Epic 17 | Security |
| NFR-SA-04 Rate Limiting | Epic 17 | Security |
| NFR-SA-05 API Availability & Degradation | Epic 13 | Reliability |
| NFR-SA-06 Cascade Dropdown Performance ⚠️ | Epic 4, Epic 13 | Performance |
| NFR-SA-07 Facet Count Accuracy ⚠️ | Epic 5 | Data Quality |
| NFR-SA-08 Search Observability | Epic 4, Epic 10, Epic 13 | Observability |

> ⚠️ = Net-new requirement not previously captured in the epics.md FR/NFR inventory.

---

## Gap Summary — New Requirements Surfaced

The following requirements were identified from the SearchAPI-AI-Input that are **not** covered by the existing FR/NFR inventory and should be reviewed for backlog inclusion:

1. **FR-SA-04** — Staggered fitment rear tire input (separate rear size parameter in search)
2. **FR-SA-17** — UTQG sort option on PLP
3. **NFR-SA-02** — Filter/sort incremental response time (≤300ms p95, no full page reload)
4. **NFR-SA-06** — Cascading vehicle dropdown individual call performance (≤200ms p95, Redis-backed)
5. **NFR-SA-07** — Facet count accuracy guarantee (exact match, not approximate)
