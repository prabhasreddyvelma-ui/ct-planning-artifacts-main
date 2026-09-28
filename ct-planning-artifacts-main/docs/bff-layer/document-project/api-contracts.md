# API Contracts — Costco Tires & Batteries

_Generated: 2026-04-13 | Scan Level: Deep | Project Type: backend (ASP.NET MVC 5)_

---

## Overview

The application exposes HTML pages and partial-view endpoints via ASP.NET MVC 5 Attribute Routing. There is no REST API layer in this monolith — AJAX calls return rendered partial views (HTML) or lightweight JSON (dropdown data, tire arrays). The upcoming Java Spring Boot BFF will expose 133 proper REST endpoints; see `costco_tires_bff_api_summary.md`.

---

## Tire Domain — Controllers & Routes

### HomeController

| Route | HTTP | Returns | Notes |
|---|---|---|---|
| `/` | GET | `Home.cshtml` | Loads homepage banners from CMS; Michelin paid-id check |
| `/Home` | GET | `Home.cshtml` | Alias |
| `/Michelin` | GET | `Home.cshtml` | Michelin brand entry point |
| `/Bridgestone` | GET | `Home.cshtml` | Bridgestone brand entry point |

### SearchFiltersController (ChildAction + AJAX)

| Route | HTTP | Returns | Notes |
|---|---|---|---|
| `POST /SearchFilters/GetVehicleYearListAsJson` | POST | `JSON: List<SelectListItem>` | Vehicle year dropdown |
| `POST /SearchFilters/GetVehicleMakeListAsJson` | POST | `JSON: List<SelectListItem>` | Requires `year` param |
| `POST /SearchFilters/GetVehicleModelListAsJson` | POST | `JSON: List<SelectListItem>` | Requires `year`, `make` |
| `POST /SearchFilters/GetVehicleOptionListAsJson` | POST | `JSON: List<SelectListItem>` | Requires `year`, `make`, `model` |

### SearchResultsByVehicleController

| Route | HTTP | Returns | Notes |
|---|---|---|---|
| `GET /SearchResultsByVehicle` | GET | `SearchResultsByBase.cshtml` | FilterType=1, reads criteria from cookie |
| `POST /SearchResultsByVehicle/RenderPartialView` | POST | HTML partial | AJAX re-render of results grid |
| `POST /SearchResultsByVehicle/GetTireData` | POST | `JSON: tire array` | Returns tire data for vehicle |

### SearchResultsBySizeController

| Route | HTTP | Returns | Notes |
|---|---|---|---|
| `GET /SearchResultsBySize` | GET | `SearchResultsByBase.cshtml` | FilterType=3 (Width/Aspect/Rim) |
| `POST /SearchResultsBySize/RenderPartialView` | POST | HTML partial | AJAX partial with applied filters |

### SearchResultsByCustomTireSizeController

| Route | HTTP | Returns | Notes |
|---|---|---|---|
| `GET /SearchResultsByCustomTireSize` | GET | `SearchResultsByBase.cshtml` | FilterType=2 |
| `POST /SearchResultsByCustomTireSize/RenderPartialView` | POST | HTML partial | AJAX partial |

### SearchResultsByItemOrPartController

| Route | HTTP | Returns | Notes |
|---|---|---|---|
| `GET /SearchResultsByItemOrPart` | GET | `SearchResultsByItemOrPart.cshtml` | SearchType=4; handles vendor connection flow |
| `POST /SearchResultsByItemOrPart/RenderPartialView` | POST | HTML partial | AJAX partial |

### SearchResultsByLicensePlateController

| Route | HTTP | Returns | Notes |
|---|---|---|---|
| `GET /SearchResultsByLicensePlate` | GET | View | US only |
| `POST /SearchResultsByLicensePlate/GetVehicleData` | POST | `JSON: rendered partial` | Params: `LicensePlate`, `State`, `CultureCode`; calls Mitchell1 API; returns vehicle list as rendered HTML |

### SearchWarehouseController

| Route | HTTP | Returns | Notes |
|---|---|---|---|
| `GET /SearchWarehouse` | GET | `_WarehouseSelector` partial | |
| `GET /SearchWarehouse/IndexWithZipCode?zipcode={zip}` | GET | `_WarehouseSelector` partial | Sets tire zip cookie |
| `GET /SearchWarehouse/GetWarehouseDataByLatLong` | GET | `JSON: {html, warehouseId}` | Params: `lat`, `lng`, `count`, `IsSingleton` |
| `GET /SearchWarehouse/GetWarehouseDataById` | GET | `JSON: {html, warehouseId}` | Params: `culturecode`, `warehouseId`, `IsSingleton` |

### SearchWarehouseAsyncController

| Route | HTTP | Returns | Notes |
|---|---|---|---|
| `GET /SearchWarehouseAsync/GetWarehouseDataByLatLong` | GET (async) | `JSON: {html, warehouseId}` | TAP async variant |
| `GET /SearchWarehouseAsync/GetWarehouseDataById` | GET (async) | `JSON: {html, warehouseId}` | TAP async variant |

### SearchWarehouseOfflineController

| Route | HTTP | Returns | Notes |
|---|---|---|---|
| `GET /SearchWarehouseOffline/GetWarehouseDataByLatLong` | GET | `JSON: {html, warehouseId}` | Reads from DB (kiosk/offline mode) |
| `GET /SearchWarehouseOffline/GetWarehouseDataById` | GET | `JSON: {html, warehouseId}` | DB-only |

### ProductController

| Route | HTTP | Returns | Notes |
|---|---|---|---|
| `GET /Product` | GET | `TireItemDetails.cshtml` | PDP; item no / part no from query string or cookie |

### CompareController

| Route | HTTP | Returns | Notes |
|---|---|---|---|
| `GET /Tirecomparison?ItemNumber={item}` | GET | `Compare.cshtml` | Up to 4 tires |
| `POST /Tirecomparison/RenderPartialViewForCompare` | POST | HTML partial | Re-orders by user sequence |

### SizeSelectionController

| Route | HTTP | Returns | Notes |
|---|---|---|---|
| `GET /SizeSelection` | GET | `TireSizeSelection.cshtml` | |
| `POST /SizeSelection/RenderPartialViewFromTireSizeSelection` | POST | `JSON + HTML` | Returns size diagram images from CMS |

### ValidateVehicleController

| Route | HTTP | Returns | Notes |
|---|---|---|---|
| `GET /ValidateVehicle/GetTireData` | GET | `_ValidateVehicle` partial | Params: `skuNumber`, `criteria` |
| `GET /ValidateVehicle/Validate` | GET | `JSON: fitment result` | Checks load/speed deviation |

### Brand Landing Pages

| Route | Controller | CMS Source |
|---|---|---|
| `GET /Michelin_Tires` | `MichelinController` | ContentStack or Kentico |
| `GET /BFGoodrich_Tires` | `BFGoodrich_TiresController` | ContentStack or Kentico |
| `GET /Bridgestone_Tires` | `Bridgestone_TiresController` | ContentStack or Kentico |
| `GET /Firestone_Tires` | `Firestone_TiresController` | ContentStack or Kentico |
| `GET /Pirelli_Tires` | `Pirelli_TiresController` | ContentStack or Kentico |

### Footer (Static CMS Pages)

Routes: `/Footer/FAQ`, `/Footer/TireGlossary`, `/Footer/TireBasics`, `/Footer/TireTips`, `/Footer/RoadHazardWarranty`, `/Footer/CostcoAdvantage`, `/Footer/Warranty`, `/Footer/Disclaimers`, `/Footer/PrivacyPolicy`, `/Footer/TermsAndCondition`, `/Footer/CostcoRoadHazardWarranty`, `/Footer/RedesignNeedHelp`, `/Footer/ConsumerHealthData`
All: GET → CMS page content (ContentStack or Kentico); Redis-cached.

### Admin Panel

| Route | HTTP | Notes |
|---|---|---|
| `GET /Admin/Home` | GET | Admin dashboard |
| `GET /Admin/Login` | GET | Login form |
| `POST /Admin/LoginAdmin` | POST | Hard-coded credentials: `admin`/`admin` |
| `GET /Admin/Promotions` | GET | Promotions management |
| `POST /Admin/SavePromotions` | POST | Body: `PromotionsInfo` |

---

## Battery Domain — Area Routes (`/Batteries/`)

### Batteries/HomeController

| Route | HTTP | Returns |
|---|---|---|
| `GET /Batteries/Home` | GET | Battery homepage |

### Batteries/SearchResultsByVehicleController

| Route | HTTP | Returns | Notes |
|---|---|---|---|
| `GET /Batteries/SearchResultsByVehicle` | GET | Battery search results view | |
| `POST /Batteries/SearchResultsByVehicle/RenderPartialView` | POST | HTML partial | Body: `BatterySearchPayload`; filters `is_sold_online==0` unless QR scan |

### Batteries/ProductController

| Route | HTTP | Returns | Notes |
|---|---|---|---|
| `GET /Batteries/Product` | GET | Battery PDP | Category-aware (Automotive vs Powersport) |

### Batteries/SearchFiltersBatteryController

| Route | HTTP | Returns | Notes |
|---|---|---|---|
| `POST /Batteries/SearchFilters/GetVehicleBatteryMakeListAsJson` | POST | JSON | Params: `year`, `batteryCategory`, `make`; HttpRuntime cached |
| `POST /Batteries/SearchFilters/GetVehicleBatteryModelListAsJson` | POST | JSON | |
| `POST /Batteries/SearchFilters/GetVehicleBatteryEngineListAsJson` | POST | JSON | |
| `POST /Batteries/SearchFilters/GetBatteryTypeListAsJson` | POST | JSON | Param: `batteryCategory` |

### Battery Footer (CMS Pages)

Routes under `[RouteArea("Batteries")]`: FAQ, Warranty, PrivacyPolicy, TermsAndCondition, Specifications, BatteryTechnology, BatteryRecycling, HowToInstallBattery
All: GET → ContentStack or Kentico; Redis-cached.

---

## External Integrations Consumed

| Integration | Purpose | Client Class |
|---|---|---|
| **Mitchell1** | VIN / License Plate lookup | `RestSharpBaseClient` (Digest Auth) |
| **Bing Maps** | Warehouse lat/long geocoding | `HttpBaseClient` |
| **Costco Inventory / Apigee** | Warehouse data by lat/long and ID | `HttpBaseClient` / `HttpBaseClientAsync` |
| **ContentStack** | Headless CMS (pages, blogs, images) | `HttpContentstackClient` |
| **Kentico REST API** | Legacy CMS | `RestAPICall` / `BatteryRestApiCall` |
| **Azure Key Vault** | DB connection string secret | `DbKeyVaultManager` (SDK) |

---

## Response Patterns

- **Full page renders:** Standard MVC `View()` — Razor cshtml with `_Layout.cshtml`
- **AJAX partial views:** `PartialView(...)` returned as HTML string — jQuery replaces DOM elements
- **JSON endpoints:** `Json(data, JsonRequestBehavior.AllowGet)` — plain C# objects (not REST standards)
- **Culture-specific views:** Same action, different view name resolved at runtime (e.g., `_WarehouseDetails_en_fr_ca.cshtml`)
