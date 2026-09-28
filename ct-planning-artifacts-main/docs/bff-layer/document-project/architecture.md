# Architecture — Costco Tires & Batteries

_Generated: 2026-04-13 | Scan Level: Deep_

---

## Executive Summary

**Costco Tires & Batteries** is an ASP.NET MVC 5 monolith running on .NET Framework 4.8, deployed to Azure App Service. It serves tire and battery shopping experiences for Costco.com visitors in the **USA** (en-US) and **Canada** (en-CA, fr-CA). The system queries Azure SQL via stored procedures, caches data in both in-process memory and Azure Redis Cache, integrates with multiple external APIs (warehouse availability, vehicle data, CMS), and renders server-side HTML with Razor views.

A strategic migration to a Java 17 / Spring Boot 3.2 BFF (Backend-for-Frontend) layer is underway using the Strangler Fig pattern (133 REST APIs across 15 domains). See `costco_tires_bff_api_summary.md`.

---

## Technology Stack

| Category | Technology | Version |
|---|---|---|
| Runtime | .NET Framework | **4.8** |
| Web Framework | ASP.NET MVC | **5** (Microsoft.AspNet.Mvc 5.2.4) |
| Language | C# | (VS 2022 tooling) |
| IDE | Visual Studio | 2022 v17.2 |
| Database | SQL Server / Azure SQL | — |
| ORM | None (domain data) / Entity Framework 6.2.0 (Identity only) | 6.2.0 |
| SQL Client | Microsoft.Data.SqlClient | 5.2.1 |
| Cache (in-process) | HttpRuntime.Cache | (BCL) |
| Cache (distributed) | StackExchange.Redis | 2.0.601 |
| Session | Microsoft.Web.RedisSessionStateProvider-Bundled | 2.2.0.5 |
| Serialization | Newtonsoft.Json | 11.0.1 |
| Secret Management | Azure.Security.KeyVault.Secrets | 4.6.0 |
| Authentication (Azure) | Azure.Identity | 1.11.3 |
| Token Handling | Microsoft.IdentityModel.Tokens | 6.35.0 |
| Monitoring | Microsoft.ApplicationInsights | 2.4.0 / .Web 2.4.1 |
| HTML Parsing | HtmlAgilityPack | 1.6.17 |
| External HTTP | System.Net.Http (HttpClient) + RestSharp (via RestSharpBaseClient) | — |
| CMS (active) | ContentStack (headless) via REST | — |
| CMS (legacy) | Kentico REST API | — |
| Mitchell1 (VIN/LP) | RestSharp + Digest Auth | — |
| Frontend | jQuery 3.3.1, Bootstrap 3.0.0, Moment.js 2.30.1 | — |
| Testing | MSTest + Moq 4.18.4 + Castle.Core 5.1.1 | — |
| Code Analysis | SonarLint (ruleset: `costcotires_redesigncsharp.ruleset`) | — |
| CI/CD | Azure DevOps (8 build configurations) | — |

---

## Architecture Pattern

**Layered Monolith** with strict tier separation:

```
┌─────────────────────────────────────────────────────────────┐
│  Presentation Layer — Costco (ASP.NET MVC 5)                │
│  Controllers → BaseController / BaseBatteryController        │
│  Views (Razor .cshtml) — culture-specific per locale        │
└─────────────────────┬───────────────────────────────────────┘
                      │ via ApplicationFactory (static factory)
┌─────────────────────▼───────────────────────────────────────┐
│  Business Logic Layer — BusinessObjects                      │
│  Search, Warehouses, VehicleOperations, URLOperations, etc.  │
│  Battery: BatterySearchResults, BatteryOperations, etc.      │
└──────────┬───────────────────────────┬───────────────────────┘
           │ (DB calls)                │ (external HTTP)
┌──────────▼──────────┐    ┌───────────▼────────────────────┐
│  DataAccess Layer    │    │  Integration Layer             │
│  SearchData          │    │  HttpBaseClient (warehouse)    │
│  TireData            │    │  HttpContentstackClient (CMS)  │
│  VehicleData         │    │  RestSharpBaseClient (Mitchell1)│
│  DataAccessHelper    │    │  HttpBaseClientAsync           │
└──────────┬──────────┘    └───────────┬────────────────────┘
           │                           │
┌──────────▼──────────┐    ┌───────────▼────────────────────┐
│  Azure SQL Server   │    │  External APIs                 │
│  (stored procs only) │    │  Costco Inventory / Apigee     │
│                     │    │  Mitchell1 (VIN/LP)            │
└─────────────────────┘    │  ContentStack CMS             │
                            │  Kentico CMS (legacy)         │
                            │  Bing Maps                    │
                            └────────────────────────────────┘
```

**Supporting Infrastructure:**
- `ApplicationInterfaces/` — Contracts between layers
- `ApplicationModels/` — POCOs/DTOs
- `ApplicationFactory/` — Static object factory (`FactoryCreator`, `BatteryFactoryCreator`)
- `ErrorLogging/` — Static `Logger` → `System.Diagnostics.Trace` + App Insights
- `Resources/` / `ResourcesBattery/` — Localized strings (en-US, en-CA, fr-CA)
- `KenticoRestAPI/` — Legacy Kentico connector

---

## Domain Separation

The solution has two parallel domains sharing infrastructure:

| Concern | Tire Domain | Battery Domain |
|---|---|---|
| Area | Main `Costco/` project | `Costco/Areas/Batteries/` |
| Base Controller | `BaseController` | `BaseBatteryController` |
| Factory | `FactoryCreator` | `BatteryFactoryCreator` |
| Search | `Search`, `SearchResults`, `SearchResultsExtension` | `BatterySearchResults` |
| Vehicle/Fitment | `VehicleOperations` | `BatteryOperations` |
| Data Access | `SearchData`, `TireData`, `VehicleData` | `SearchBatteryResultByVehicle`, `BatteryMakeList`, etc. |
| DB Tables | `MST_TireDetails`, `Tire Fitment_*`, etc. | `Battery_Details`, `BatteryFitment_*`, etc. |
| Resources | `Resources/` (en-US, en-CA, fr-CA) | `ResourcesBattery/` (en-US, en-CA, fr-CA) |
| CMS Channel | Kentico (tires) / ContentStack (migrating) | Kentico battery endpoint / ContentStack |

Battery area redirects US users (en-US) to the tire homepage — batteries are Canada-only for some features.

---

## Dependency Injection Strategy

**No IoC container.** The static Abstract Factory pattern is used throughout:

```csharp
// Tire domain — in controllers:
ISearch search = FactoryCreator.GetSearchObject();
IVehicleOperations vehicleOps = FactoryCreator.GetVehicleOperations();
IWarehouses warehouses = FactoryCreator.GetWarehouses();
IURLOperations urlOps = FactoryCreator.GetURLOperations();

// Battery domain:
IBatterySearchResults batterySearch = BatteryFactoryCreator.GetBatterySearchResults();
```

Each factory method returns the interface type. Concrete implementations are defined in `BusinessObjects/`.

---

## Caching Architecture

**Two-tier caching:** in-process + distributed.

```
Request
  │
  ├─► Check HttpRuntime.Cache  (short-lived, in-process)
  │      ↓ miss
  ├─► Check Redis IDatabase    (longer-lived, cross-instance)
  │      ↓ miss
  └─► Query DB via Stored Procedure
             ↓
        Write to Redis  ──►  Write to HttpRuntime.Cache (some paths)
```

| Data Type | Cache Layer | TTL Source |
|---|---|---|
| Vehicle dropdowns (year/make/model/option) | `HttpRuntime.Cache` | `WebConfigKeys.CacheExpiration` |
| DB-driven config (`Mst_GeneralConfiguration`) | `HttpRuntime.Cache` | Hard-coded (24h) |
| Custom fitment tire sizes | `HttpRuntime.Cache` | `WebConfigKeys.CacheExpiration` |
| Warehouse data | Redis | `WebConfigKeys.RedisTimeOut` |
| Search results | Redis | `WebConfigKeys.CacheExpirationFrontEnd` |
| CMS content (blogs, pages, images) | Redis | Configured per content type |
| Configurable items/attributes | Redis | `WebConfigKeys.CacheExpirationFrontEnd` |
| Azure Key Vault secrets | `System.Runtime.Caching.MemoryCache` | `WebConfigKeys.MemoryCacheExpiration` (days) |

Redis key pattern: `{domain}_{culture}_{warehouseId}` (see `BusinessObjects/Caching/RedisKeys.cs`).

Redis singleton: `RedisCacheHelper.Connection` (lazy-initialized `ConnectionMultiplexer`).

---

## Configuration Management

Three-tier configuration:

1. **`Web.config` / `app.config`** — static defaults; `<appSettings>` and `<connectionStrings>`
2. **`WebConfigKeys` static class** — all access to `ConfigurationManager.AppSettings` is centralized here; never read AppSettings inline
3. **`Mst_GeneralConfiguration` DB table** — runtime feature flags and overrides fetched at startup and cached in `HttpRuntime.Cache` via `Configurations.SetDatabaseCostcoWebConfigKey()`
4. **Azure Key Vault** (production only) — DB connection string fetched by `DbKeyVaultManager` (Lazy singleton with in-memory cache)

Environment overrides applied via XDT transforms (`Web.{env}.config`) at Azure publish time.

---

## Security Architecture

| Concern | Implementation |
|---|---|
| **DB Secrets** | Azure Key Vault (`DbKeyVaultManager`); `DefaultAzureCredential` (Managed Identity in Azure) |
| **Admin Authentication** | Hard-coded username/password in `LoginController` (`admin`/`admin`) — **must be changed for production** |
| **Cookie Security** | `SameSite=None; Secure` enforced via `SameSiteCookieRewriter` for cross-site compatibility; some cookies deliberately `HttpOnly=false` for JS access |
| **SSL / TLS** | Kentico REST calls bypass SSL validation (`ServicePointManager.ServerCertificateValidationCallback = delegate { return true; }`) — legacy issue in `KenticoRestAPI` |
| **ASP.NET Identity** | Entity Framework 6 + `AspNetIdentity` for admin portal users |
| **External API Auth** | Mitchell1: Digest auth (MD5); ContentStack: `api_key`/`access_token` headers; Warehouse: Bearer token via `HttpBaseClient` |
| **Error disclosure** | `HandleErrorAttribute` globally redirects to `Error.cshtml`; custom error logging controlled by `LogCustomErrors` flag |

---

## Multi-Region / Localization

| Locale | Market | Notes |
|---|---|---|
| `en-US` | USA | Default; full feature set |
| `en-CA` | Canada (English) | Battery area accessible; some US-only promos hidden |
| `fr-CA` | Canada (French) | French views (`_en_fr_ca.cshtml`); French fitment tables |

**Culture resolution chain (both controllers):**
1. Mobile App header (`CultureCode` header)
2. `COSTCO_SELECTED_LANGUAGE` cookie
3. `Accept-Language` HTTP header
4. Default (`en-US`)

**Domain-to-culture mapping:** `Costco/DomainCultureMapping.xml` maps hostnames to culture codes. Add new environment domains here.

**Resource strings:** `Resources/` for tires, `ResourcesBattery/` for batteries — never hardcode locale strings in views or controllers.

---

## CMS Integration

Feature flag `WebConfigKeys.IsContentstackActive` controls the CMS source:

```
IsContentstackActive = false → KenticoRestAPI (RestAPICall / BatteryRestApiCall)
IsContentstackActive = true  → Integration/HttpContentstackClient
```

| CMS | Auth | Used For |
|---|---|---|
| ContentStack | `api_key` + `access_token` headers | Pages, blogs, images, banners |
| Kentico (legacy) | Basic Auth HTTP header | Fallback; battery endpoint separate |

Both implementations return the same model types (`ContentstackPage`, `ContentstackBlog`, `ContentstackImageSetting`).

---

## Warehouse API Resilience

Three parallel warehouse implementations serve different operational modes:

| Class | Mode | DB | External API |
|---|---|---|---|
| `Warehouses` + `Warehouse` | Normal (sync) | No | Yes (Costco / Apigee) |
| `WarehousesAsync` | Normal (async TAP) | No | Yes (async) |
| `WarehousesOffline` | Kiosk / in-store | Yes | No |

Warehouse API supports two gateway modes (`ApigeeWarehouseFlag` in DB config): direct Costco API or via Apigee Gateway.

---

## Deployment Architecture

```
Azure DevOps Pipeline
        │
        ▼
MSBuild /p:Configuration=Azure{Env}
        │  (applies XDT transforms)
        ▼
Azure App Service (Windows)
  ├── Costco Web App (.NET 4.8 IIS)
  ├── Redis Cache (Azure Redis)
  ├── Azure SQL Database
  ├── Azure Key Vault
  └── Azure Application Insights
```

**8 deployment environments:** Dev → QA2 → Staging → UAT → PreProd → Production (+ Debug, Release for local).

---

## Testing Strategy

| Layer | Approach | Framework |
|---|---|---|
| Unit | Near-zero coverage; `[ExcludeFromCodeCoverage]` on all production classes | MSTest + Moq 4.18.4 |
| Integration | Not automated | Manual |
| E2E | Not automated | Manual |

New tests should target `BusinessObjects` classes (interfaces enable Moq mocking). DataAccess and controller classes are excluded from coverage by project convention.

---

## Migration Context (BFF)

The BFF migration (`costco_tires_bff_api_summary.md`) will extract all business logic and data access into a **Java 17 / Spring Boot 3.2** service layer using the Strangler Fig pattern:

- 133 REST API endpoints across 15 domains
- Spring Data JPA/Hibernate 6 (replacing raw ADO.NET)
- Spring Cache + Redis 7 (same Redis, new client: Lettuce)
- Spring Security (JWT/RBAC replacing hard-coded admin auth)
- Resilience4j circuit breaker (replacing ad-hoc retry logic)
- 20-week implementation plan, 3 developers, AI-assisted (64% time savings)
