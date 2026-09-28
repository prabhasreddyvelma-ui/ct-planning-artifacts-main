# Deep-Dive Functional Specification
## Costco Tire & Battery E-Commerce Platform

**Version:** 1.0  
**Generated:** 2026-04-13  
**Coverage:** All domains — Tire Search, Tire Product, Tire Compare, Battery, Warehouse, Admin, Configuration, CMS, External Integrations  
**Source:** Source code analysis across 39 files (Controllers, BusinessObjects, DataAccess, Integration, Models, Interfaces)

---

## Table of Contents

1. [System Overview](#1-system-overview)
2. [Architecture and Infrastructure](#2-architecture-and-infrastructure)
3. [Cross-Cutting Concerns (BaseController)](#3-cross-cutting-concerns-basecontroller)
4. [Tire Search — By Vehicle](#4-tire-search--by-vehicle)
5. [Tire Search — By Size](#5-tire-search--by-size)
6. [Tire Search — By Custom Tire Size](#6-tire-search--by-custom-tire-size)
7. [Tire Search — By Item or Part Number](#7-tire-search--by-item-or-part-number)
8. [Tire Search — By License Plate](#8-tire-search--by-license-plate)
9. [Tire Product Detail Page (PDP)](#9-tire-product-detail-page-pdp)
10. [Tire Compare Page](#10-tire-compare-page)
11. [Tire Size Selection Page](#11-tire-size-selection-page)
12. [Search Filters Sidebar](#12-search-filters-sidebar)
13. [Vehicle Validation](#13-vehicle-validation)
14. [Warehouse Search](#14-warehouse-search)
15. [Battery Domain](#15-battery-domain)
16. [Admin Panel](#16-admin-panel)
17. [Configuration Engine](#17-configuration-engine)
18. [Promotions Engine](#18-promotions-engine)
19. [CMS Integration (ContentStack & Kentico)](#19-cms-integration-contentstack--kentico)
20. [External Integrations](#20-external-integrations)
21. [Data Model Reference](#21-data-model-reference)
22. [Session, Cookie, and Cache Architecture](#22-session-cookie-and-cache-architecture)
23. [Localization and Multi-Region Support](#23-localization-and-multi-region-support)
24. [Business Rules Reference](#24-business-rules-reference)
25. [Constants and Configuration Reference](#25-constants-and-configuration-reference)

---

## 1. System Overview

The Costco Tire & Battery E-Commerce Platform is a monolithic ASP.NET MVC 5 web application (.NET Framework 4.8) serving the US and Canadian markets. It enables Costco members to search for tires by vehicle, size, custom size, item/part number, or license plate, and to search for batteries by vehicle. Products are sold exclusively through Costco warehouse locations.

### Regions

| Region | Culture | Country Code | URL Path | Language Options |
|--------|---------|-------------|----------|-----------------|
| United States | `en-US` | `USA` | Default | English only |
| Canada (English) | `en-CA` | `CA` | `/en-ca/` | English |
| Canada (French) | `fr-CA` | `CA` | `/fr-ca/` | French |

### Domains

| Domain | Controller Namespace | Description |
|--------|---------------------|-------------|
| Tire Home | `HomeController` | Landing page with CMS imagery |
| Tire Search By Vehicle | `SearchResultsByVehicleController` | Guided vehicle selection |
| Tire Search By Size | `SearchResultsBySizeController` | Direct tire size input |
| Tire Search By Custom Size | `SearchResultsByCustomTireSizeController` | Staggered / custom sizes |
| Tire Search By Item / Part | `SearchResultsByItemOrPartController` | SKU / part number lookup |
| Tire Search By License Plate | `SearchResultsByLicensePlateController` | VIN/LP via Mitchell1 |
| Tire Product Detail | `ProductController` | Individual tire PDP |
| Tire Compare | `CompareController` | Side-by-side comparison |
| Tire Size Selection | `SizeSelectionController` | CMS-driven hub page |
| Search Filters | `SearchFiltersController` | Dropdown population & sidebar |
| Vehicle Validation | `ValidateVehicleController` | Fitment compatibility check |
| Warehouse Search | `SearchWarehouseController` | Find a warehouse by location |
| Battery Home | `Battery.HomeController` | Battery landing page |
| Battery By Vehicle | `Battery.SearchResultsByVehicleController` | Battery search |
| Battery Product | `Battery.ProductController` | Battery PDP |
| Battery Filters | `Battery.SearchFiltersBatteryController` | Battery filter dropdowns |
| Admin | `Admin.LoginController`, `AdminPromotionsController` | Back-office promotions |

---

## 2. Architecture and Infrastructure

### Layer Structure

```
┌─────────────────────────────────────────────┐
│  Presentation Layer (ASP.NET MVC 5 Views)   │
│  jQuery / Bootstrap / Razor .cshtml         │
├─────────────────────────────────────────────┤
│  Controller Layer (CWC_Tires / Costco)      │
│  BaseController → domain controllers        │
├─────────────────────────────────────────────┤
│  BusinessObjects Layer                      │
│  Search.cs, Configurations.cs, etc.         │
├─────────────────────────────────────────────┤
│  DataAccess Layer                           │
│  ADO.NET stored procedures only             │
├─────────────────────────────────────────────┤
│  Infrastructure                             │
│  Azure SQL · Redis · Azure Key Vault        │
│  ContentStack / Kentico CMS                 │
│  Mitchell1 · Bing Maps · Apigee             │
└─────────────────────────────────────────────┘
```

### Key Technologies

| Component | Technology |
|-----------|-----------|
| Runtime | .NET Framework 4.8 |
| Web Framework | ASP.NET MVC 5 (v5.2.4) |
| Database | Azure SQL / SQL Server |
| DB Access | ADO.NET — raw stored procedures, no ORM |
| Caching (L1) | `HttpRuntime.Cache` (in-process, per-instance) |
| Caching (L2) | Redis via `StackExchange.Redis 2.0.601` |
| CMS | ContentStack (active) / Kentico REST API (legacy) |
| Dependency Injection | None — static Abstract Factory (`FactoryCreator`, `BatteryFactoryCreator`) |
| Serialization | `Newtonsoft.Json 11.0.1` |
| HTML Parsing | `HtmlAgilityPack` (SizeSelectionController) |
| Monitoring | Azure Application Insights 2.4.x |
| Frontend | jQuery 3.3.1, Bootstrap 3.0.0, Moment.js 2.30.1 |
| Testing | MSTest + Moq 4.18.4 + Castle.Core 5.1.1 |

### Build Configurations

| Config | Target |
|--------|--------|
| Debug | Local development |
| Release | Generic release |
| AzureDev | Azure Dev environment |
| AzureQA2 | Azure QA2 environment |
| AzureStaging | Azure Staging |
| AzureUAT | Azure UAT |
| AzurePreProd | Azure Pre-Production |
| AzureProduction | Azure Production |

### Deployment

- Azure App Service
- Azure Key Vault for all secrets at app startup
- Thread pool configured from `WORKER_THREADS` / `IOCP_THREADS` config keys
- Redis connection string from `RedisCacheConnectionString` / `RedisTimeOut`

---

## 3. Cross-Cutting Concerns (BaseController)

All tire domain controllers inherit `BaseController`. All battery domain controllers inherit `BaseBatteryController`. Both perform the same core initialization pattern.

### 3.1 Request Initialization (`BeginExecuteCore`)

Every incoming request triggers the following ordered sequence:

**Step 1 — Language / Culture Resolution**

1. Check `lang` query parameter (mobile app language override: `IsMobileAppLangParamater` flag)
2. Check `COSTCO_SELECTED_LANGUAGE` cookie
3. Default to `en-US`
4. Set `Thread.CurrentThread.CurrentCulture` and `CurrentUICulture`
5. Set `ViewBag.LanguageCode` and `ViewBag.CountryCode`

**Step 2 — Warehouse Resolution**

Priority order for warehouse determination:

| Priority | Source | Details |
|----------|--------|---------|
| 1 | `whs` query parameter | Direct warehouse override |
| 2 | `tiresLocationCookie` cookie | Previously selected warehouse |
| 3 | `TiresZipCode_us` / `TiresZipCode_ca` cookie | Zip-code-to-warehouse resolution |
| 4 | `memberPrimaryPostal` / `client-zip-short` | Akamai / member postal code |
| 5 | Missing | Redirect to Home |

**Step 3 — Warehouse Validation**

- Checks if resolved warehouse ID is in `DEACTIVATED_WAREHOUSE_CODES` configurable list
- If deactivated → redirect to home page
- If not found at all → redirect to home page

**Step 4 — Warehouse API Fetch**

`GetWarehouseDataFromAPI(warehouseId)`:
- Calls Apigee/Costco Inventory API via `HttpClient`
- Response is deserialized into warehouse data object
- Warehouse details (name, address, hours) stored in `ViewBag`

**Step 5 — Configuration Load**

`GetSetNewCostcoConfigurations()`:
- Reads `TriggerDatabaseKeyCheck` and `TriggerDatabaseStaticKeys` from `web.config`
- Loads `ConfigurableItemsAndAttributes` from Redis → DB via `Configurations.cs`
- `SetDatabaseKey()` — sets DB connection key from configuration
- Loads `KeySettings` list (general settings) from DB

**Battery Addition**: `BaseBatteryController.BeginExecuteCore` also:
- Redirects any US request (`en-US` culture) to `/` (battery is Canada-only)
- Calls `SetConfigurablePromotions` for battery promotions

### 3.2 Cookie Helpers

| Cookie | Key Constant | Expiry | Content |
|--------|-------------|--------|---------|
| Warehouse location | `tiresLocationCookie` | 365 days | Warehouse ID |
| Tire site ZIP (US) | `TiresZipCode_us` | 365 days | US ZIP code |
| Tire site ZIP (CA) | `TiresZipCode_ca` | 365 days | CA postal code |
| Member primary postal | `memberPrimaryPostal` | 365 days | From Costco main site |
| Akamai client ZIP | `client-zip-short` | 365 days | Short ZIP from Akamai |
| Michelin Paid ID | `MichelinPaidId` | 365 days | Michelin campaign tracking |
| Michelin Session Paid ID | `MichelinSessionPaidId` | Session | Michelin session tracking |
| Search criteria | `CookieForSearchCriteria` | 365 days | Serialized search parameters |
| Tire details | `CookieTireDetails` | 365 days | Serialized tire detail for TPMS |
| Tire details TPMS | `CookieTireDetailsForTPMS` | Session | TPMS tire cookie |
| Language | `COSTCO_SELECTED_LANGUAGE` | 365 days | Language code |
| State code | `StateCode` | 365 days | US state code |
| Membership info | `mSign` | 365 days | Signed-in member indicator |

Cookie operations use `SameSite` and `Secure` attributes gated by `ActivateSameSite` / `CookieSecure` config flags.

### 3.3 Delivery Date Calculation (`SetDeliveryInfo`)

Computes estimated delivery window per tire. Two delivery windows are calculated — promo and non-promo:

**`CalculateBusinessDaysFromInputDate(days, nonBusinessDays)`**:
1. Start from `DateTime.Now`
2. Skip weekends (Saturday, Sunday)
3. Fetch holiday list from `GetHolidayListFromCache(culture)`:
   - Redis key: `HolidayListKey`
   - If not cached → calls `SP_GetHolidayList` via DataAccess
   - Parsed into `List<IHoliday>` with `HolidayDate` and `IsObserved`
4. Skip observed holidays
5. Continue counting until target business days reached
6. Format: "Mon, Jan 1" style using locale-aware format string
7. Returns formatted date string for display

Delivery day ranges come from:
- `PromoBusinessDays` / `PromoNonBusinessDays` — promo window (from KeySettings DB)
- `OrderDeliveryDays_All_Location_Min/Max` — standard window
- `OrderDeliveryDays_SPC_Location_Min/Max` — specific warehouse override

The computed dates are formatted into `TireInfo`:
- `PromoOrderDeliverybyDate` / `NonPromoOrderDeliverybyDate` — date string
- `PromoOrderDeliverybyDateId` / `NonPromoOrderDeliverybyDateId` — DOM element IDs

### 3.4 Warranty Formatting (`SetWarranty`)

Formats `TireInfo.TireWaranty` list into `TireWarrantyText`:
- US: appended as-is from DB warranty records
- CA English: format `"{value} Km Treadlife Warranty"`
- CA French: format `"Garantie de la bande de roulement de {value} Km"`
- `WarrantyDisclaimner` is set per locale

### 3.5 Additional Fee Handling (`SetAdditionalFeeInfo`)

For US tires:
- Reads `SpecificWarehouseAdditionalFees` from general configurable settings
- If current warehouse in the fee list → sets `TireInfo.TireStateFee` and `AdditionalFeePopupContent`
- Format: fee amount string + popup HTML

### 3.6 Vendor / Blacklist Filtering (`RemoveItemsForBlackListVendor`)

- Reads `BlackListVendorForWarehouses` from configurable settings
- Format: `"warehouseId:supplierId1,supplierId2|warehouseId2:..."`
- For current warehouse → removes any tires whose `Supplier_ID` is in blacklist
- Applied after DataAccess returns — before rendering

### 3.7 Winter Tire Handling

Two separate mechanisms:

**Hide winter tires** (`HideWinterTiresForSpecificWarehouses`):
- Reads `SPECIFICWAREHOUSELIST` from configurable items
- If current warehouse in list → removes all tires where `TireTypeInfo.TireType == "Winter"`

**Show winter tire warning** (`SetWinterTireWarning`):
- Reads `WinterTireWarningWarehouses` from config
- If current warehouse in list → sets `TireInfo.IsShowWinterTireWarning = true`
- Also checks `WinterTiresONForSupplierId` — if a winter tire's supplier is NOT in this list, it is removed

### 3.8 Temp Out-of-Stock Text (`SetTempOutOfStockText`)

- If `TireInfo.IsOutOfStockEquipment && TireInfo.Available_Qty == 0` and `NextAvailableDate` is non-default
- Sets out-of-stock message string for display in tile
- Used for "back in stock" messaging

### 3.9 Image Accessibility (`FetchImagesAccessibility`)

- Calls `ContentstackIntegration.GetImages()` or `KenticoRestApiIntegration.GetImages()` based on `IsContentstackActive` flag
- Used by Home controller and PDP for banner/promotional image display

### 3.10 Render Razor to String (`RenderRazorViewToString`)

Utility method for AJAX partial renders — renders a view to a string for JSON responses:
- Uses `ViewEngineResult` to find the view
- Creates a `StringWriter` and `ViewContext` manually
- Returns HTML string for partial view embedding in AJAX responses

### 3.11 Search Parameter Persistence (`SaveSearchParameters`)

Serializes search criteria to:
1. `Session["QueryString"]` — URL query string for pagination
2. `Session["Search"]` — full `ISearch` object
3. `COOKIE_SEARCHCRITERIA` cookie — for page reloads and cross-page navigation

---

## 4. Tire Search — By Vehicle

**Controller:** `SearchResultsByVehicleController`  
**Route:** `GET /SearchResultsByVehicle?whs={id}&...`  
**Session Keys:** `QueryString`, `Search`, `SearchResults`, `YearList`, `MakeList`, `ModelList`, `OptionsList`

### 4.1 Initial Page Load (`Index()`)

1. Calls `BeginExecuteCore` (warehouse + config init)
2. Reads `SESSION_SEARCH_PARAM` from session for repopulating filter UI
3. Loads Year dropdown: `GetYearList()` → `SearchFiltersController` calls `SP_GetYear` via `SearchFiltersData.GetYearList()`
4. Sets `ViewBag.WarehouseInfo`, `ViewBag.Language`
5. Returns `View()`

### 4.2 Full Search Render (`RenderPartialView`)

AJAX called after vehicle selection is complete. Executes in this ordered sequence:

**Step 1 — Build `IConfigurableItemsAndAttributes`**  
Assembled from:
- Warehouse ID (from warehouse cookie)
- Culture / country code
- Year, Make, Model, Options (from query string / form params)
- Page index, sort order
- Staggered flag (`isStaggered=true/false`)
- `originalFitment` flag
- `vehiclefitmentdata` (fitment override from query param)

**Step 2 — Retrieve Search Results**  
Calls `ISearch.GetSearchResults(configurableItems, keySettings)`:
1. Checks Redis for cached results (key: `sbvr` prefix + hash of Year/Make/Model/Options/Culture)
2. If cache miss → calls `GetSearchResultsByVehicleFromDB()`:
   - SP: `SP_GetSearchResultsByVehicle_Redesign_V8`
   - Returns multi-table DataSet (tables: tires, fitment data, promotions, features, benefits, images, warranty, footnotes, videos, PDF, supplier, price)
3. If cache hit → deserializes JSON from Redis

**Step 3 — Five-Pass Processing Pipeline**  

| Pass | Operation |
|------|-----------|
| 1 | `GetVehicleModel()` — sets Make/Model/Options display text |
| 2 | `GetAllPromotions()` — loads promotions from DB for culture |
| 3 | `SetConfigurablePromotions()` — applies promotions to each tire |
| 4 | Fitment validation — checks load/speed rating deviations, removes non-fitting tires |
| 5 | EV filter — if `EvCompatible` filter set to `Y`, adds EV-compatible tires to EV-dedicated set |

**Step 4 — Fitment Validation Algorithm**

`ValidateVehicleInDB()` is called when `VehicleFitmentData` is present:
1. SP: `SP_ValidateVehicle` — returns list of `IVehicleFitmentData`
2. Each fitment record has: `FitmentLoadIndex`, `FitmentSPEED_RATIO`, `FitmentTireSize`, `STD_OR_OPT`, `FRB`
3. For each tire in results:
   - `FitmentTireSize` must match or be within `LoadRatingDeviation` (from `ConfigurableItems`)
   - Speed rating check: `SpeedMilesPerHour >= FitmentSpeedMilesPerHour` (from computed allowed speed list)
   - `STD_OR_OPT` = "S" → standard equipment; "1/2/3" → optional
   - `FRB` = "F" (Front), "R" (Rear), "B" (Both) — staggered per-axle sizing
4. Tires not meeting fitment → `IsAvailable = false` but still shown (with fitment warning)

**Step 5 — Staggered Tire Grouping**

When `isStaggered=true`:
- Front tires (`FRB=F`) and Rear tires (`FRB=R`) have different sizes
- Each tire set is matched by `ClubbedId`
- `ClubbedTirePrice` = front price + rear price (set of 4: 2 front × 2 rear)
- Compare page receives both the front and rear `TireID` in a compound ID

**Step 6 — Display Model Assembly**

- `SetDeliveryInfo()` — per-tire delivery date calculation
- `SetWarranty()` — warranty text formatting
- `SetAdditionalFeeInfo()` — state fee injection
- `SetWinterTireWarning()` / `HideWinterTiresForSpecificWarehouses()` — winter filtering
- `RemoveItemsForBlackListVendor()` — blacklist filtering
- `SetTempOutOfStockText()` — out-of-stock messaging

**Step 7 — Filter State Setup**

Reads per-filter Redis keys to build `CheckBox` model:
- `brand_names`, `season_names`, `price_filter`, `runflat_filter`, `loadindex`, `speedrating`, `promotion_filter`, `ev_compatibility_filter`
- Each filter is a list of `ICheckBoxBindingModel` with `Name`, `Value`, `IsSelected`, `Count`

**Step 8 — Sort Setup**

`BindSortValues()` builds sort dropdown model from `IDropDownBindingModel` list:
- Options: Best Match, Price Low-High, Price High-Low, Brand A-Z

**Step 9 — AJAX Response**

Returns `Json(new { html = RenderRazorViewToString("_PartialByBaseSearch", model) })`.

### 4.3 Filter Actions (AJAX)

All filter actions follow the same pattern — they receive updated filter state and re-run `RenderPartialView` with the filtered `IConfigurableItemsAndAttributes`.

| Action | Endpoint | Filter Applied |
|--------|---------|---------------|
| `BrandFilter` | `POST /SearchResultsByVehicle/BrandFilter` | `BrandName` multi-select |
| `SeasonFilter` | `POST /SearchResultsByVehicle/SeasonFilter` | `Season` single-select |
| `PriceFilter` | `POST /SearchResultsByVehicle/PriceFilter` | `PriceRange` (min-max) |
| `RunFlatFilter` | `POST /SearchResultsByVehicle/RunFlatFilter` | `RunFlat` boolean |
| `PromotionFilter` | `POST /SearchResultsByVehicle/PromotionFilter` | `HasPromotion` flag |
| `EVCompatibilityFilter` | `POST /SearchResultsByVehicle/EVCompatibilityFilter` | `EvCompatible` / `EvDedicated` |
| `LoadIndexFilter` | `POST /SearchResultsByVehicle/LoadIndexFilter` | `LoadIndex` multi-select |
| `SpeedRatingFilter` | `POST /SearchResultsByVehicle/SpeedRatingFilter` | `SpeedRating` multi-select |

### 4.4 Add-to-Cart (`AddToCart`)

`POST /SearchResultsByVehicle/AddToCart`

1. Reads `TireID`, `Qty`, `SkuNumber`, `WarehouseId` from form
2. Validates `WarehouseId` is not in deactivated list
3. Calls `GetAddToCartURLAndRedirect()` in `CommonSearchLogic`:
   - Builds Costco cart URL based on culture:
     - US: `CostcoUSURL` + item params
     - CA EN: `CostcoCAURL` + item params
     - CA FR: `CostcoCAURL_M` + French params
   - Appends `langId` query param (`-1` for US, `-24` for CA EN, `-25` for CA FR)
   - Member pricing: if `mSign` cookie == `IsCookiValuehasTrue` → appends member price token
4. Returns redirect JSON with constructed URL

### 4.5 Vehicle Fitment Popup (`GetVehicleFitmentData`)

`GET /SearchResultsByVehicle/GetVehicleFitmentData?TireID={id}&...`

1. Calls `ISearch.ValidateVehicleInDB()` → `SP_ValidateVehicle`
2. Returns JSON with fitment data including load index and speed rating requirements
3. Used in the tire tile "Vehicle Fitment" popup

### 4.6 Popup Content (`GetPopupContent`)

`GET /SearchResultsByVehicle/GetPopupContent?TireID={id}`

Returns partial HTML for the tire detail popup in search results.

---

## 5. Tire Search — By Size

**Controller:** `SearchResultsBySizeController`  
**Route:** `GET /SearchResultsBySize?whs={id}&Width={w}&AspectRatio={a}&RimSize={r}&...`

### 5.1 Initial Page Load (`Index()`)

1. Calls `BeginExecuteCore`
2. Reads Width, AspectRatio, RimSize from query string
3. Populates size dropdowns via `SearchFiltersController`
4. Returns `View()`

### 5.2 Search Flow (`RenderPartialView` / `GetSearchResults`)

**`GetSearchResults()` — dual-path execution:**

**Path A — Redis Cache Hit:**
- Key constructed from: `Width + "_" + AspectRatio + "_" + RimSize + "_" + Culture + "_" + WarehouseId`
- Deserializes cached `ISearchResults` JSON
- Redis TTL from `BySizeRedisCacheTimeOut` key setting

**Path B — Cache Miss → Database:**
- SP: `SP_GetSearchResultsBySize_Redesign_V8`
- Parameters: `@Width`, `@AspectRatio`, `@RimSize`, `@Culture`, `@WarehouseId`
- Multi-table DataSet processed same as vehicle search
- Result cached in Redis with configured timeout

### 5.3 Filter Counts (`SetFilterCounts`)

After size search returns results, builds filter count objects:
- Iterates all tires to count matching tires per:
  - Brand name
  - Season type
  - Price range (configurable buckets)
  - Run-flat availability
  - Load index values
  - Speed rating values
  - Promotion availability
  - EV compatibility
- Filter counts stored in Redis with filter-specific key suffixes

### 5.4 Equipment Type Flag (`SetEquipmentTiresDetails`)

After search results returned:
- Checks if any `STD_OR_OPT` == "S" (standard equipment) — sets `ISearchResults.HasStandardEquipment`
- Checks if staggered fitment results present — sets staggered UI mode
- Called once per full search render, not filtered

### 5.5 Search Result Model Assembly (`GetSearchResultModel`)

Assembles the full `SearchResults` model for the view:
- Applies all post-processing passes (same as vehicle search Steps 5–7)
- Annotates `TireInfo.FitmentTireSize` from the by-size search query params
- Sets `TireInfo.ShowTireSizeImagePopup` if `TireSizeImageUrl` is configured
- Returns `ISearchResults` with populated `TireInfo` and `FitmentTireInfo` lists

---

## 6. Tire Search — By Custom Tire Size

**Controller:** `SearchResultsByCustomTireSizeController`  
**Route:** `GET /SearchResultsByCustomTireSize?...`

### 6.1 Purpose

Handles non-standard / staggered tire size searches where the user enters a full tire size string directly (e.g., `235/55R18` or separate front and rear sizes). Also handles OE (original equipment) custom fitment searches.

### 6.2 Core SP

`SP_GetSearchResultsByCustomSize_Redesign_V8`

Parameters include:
- `@TireSize` (full formatted tire size string)
- `@WarehouseId`, `@Culture`
- `@IsStaggered` — boolean for staggered searches
- Optional front/rear size params for staggered

### 6.3 Filter Actions

Identical set to By Vehicle filter actions — all 8 filter types supported. Filters applied in-memory after initial SP result is cached.

### 6.4 Custom Fitment Search Results

`GetCustomFitmentSearchResults()` on `ISearch`:
- Used when the user is on PDP and has clicked "Check Other Sizes"
- SP: `SP_GetCustomFitmentSearchResults`
- Returns tires matching the same size category as the selected tire
- Enables "similar tires" suggestions on PDP

---

## 7. Tire Search — By Item or Part Number

**Controller:** `SearchResultsByItemOrPartController`  
**Route:** `GET /SearchResultsByItemOrPart?ItemNo={sku}` or `?PartNo={part}`  
**Query Params:** `SearchBy=Item|Part`, `SearchByBrand={brand}`

### 7.1 Search Types

| `SearchBy` Value | Lookup Field | SP Used |
|-----------------|-------------|---------|
| `Item` | `SkuNumber` (Costco SKU) | `SP_GetTireDetailsByItemNumber` |
| `Part` | `SupplierNumber` (supplier part) | `SP_GetTireDetailsByPartNumber` |

### 7.2 Vendor Integration Path

When `VendorIntegrationName` key setting is set:
1. Checks for Apigee/vendor integration flag
2. If vendor integration active → calls `Integration.VendorIntegration.GetTireData(sku, warehouse)`
3. Response merged with DB data
4. Used for real-time inventory/pricing from supplier feeds

**Standard Path (no vendor integration):**
1. Reads `ItemNo` or `PartNo` from query string
2. Calls `GetSearchResults(configurableItems, keySettings)`:
   - SP path determined by `SearchBy` type
   - Returns single or multiple matching tires
3. If only one result → auto-redirect to PDP
4. If multiple → show search results grid

### 7.3 Add-to-Cart

Same flow as By Vehicle `AddToCart`. `QuantityAndCart.Qty` set to 4 by default (set of four tires).

---

## 8. Tire Search — By License Plate

**Controller:** `SearchResultsByLicensePlateController`  
**Route:** `GET /SearchResultsByLicensePlate?state={state}&plate={plate}`  
**External Service:** Mitchell1 License Plate API

### 8.1 Full Flow

1. Receives `state` and `plate` (or `vin`) from query string
2. Calls `GetVehicleData(state, plate)`:

**Sub-Step A — Token Initialization:**
- Checks Redis for `LicAccessToken` and `LicRefreshToken`
- If not cached → calls `Mitchell1Integration.InitializeToken()`:
  - HTTP POST to Mitchell1 auth endpoint with Digest authentication
  - Stores access token and refresh token in Redis
  - Stores proxy ID in Redis as `LicProxyId`
  
**Sub-Step B — VIN Lookup (if LP provided):**
- Calls `Mitchell1Integration.GetVINFromLicensePlate(state, plate, accessToken, proxyId)`:
  - HTTP GET to Mitchell1 LP→VIN endpoint
  - Returns VIN string
  
**Sub-Step C — Vehicle Resolution:**
- Calls `Mitchell1Integration.GetVehicleFromVIN(vin, accessToken, proxyId)`:
  - HTTP GET to Mitchell1 VIN→Vehicle endpoint
  - Returns: Year, Make, Model, Trim/Options
  
**Sub-Step D — Redirect to By Vehicle:**
- On success → constructs search URL: `/SearchResultsByVehicle?Year={y}&Make={m}&Model={mod}&Options={opt}&whs={whs}`
- On token expiry (401 response) → refresh token, retry once
- On failure → redirect to By Vehicle search home with error flag

### 8.2 Config Keys

| Key | Description |
|-----|-------------|
| `LicensePlateAPITimeout` | HTTP timeout for Mitchell1 calls (ms) |
| `IsNewLcensePlateConfigOn` | Feature flag for new LP config |
| `NoOfYears` | Year range for vehicle dropdowns |

### 8.3 Error Handling

- Mitchell1 API errors → logged to Application Insights
- Redirect to vehicle search with `lpError=true` query param
- Frontend shows graceful fallback to manual vehicle selection

---

## 9. Tire Product Detail Page (PDP)

**Controller:** `ProductController`  
**Route:** `GET /TireItemDetails?whs={id}&TireID={id}&SearchBy={type}&SearchID={q}`

### 9.1 Page Load Flow (`Index()`)

1. Calls `BeginExecuteCore`
2. Reads `TireID` from query string
3. Calls `GetSetCacheTireItemDetailModel(tireId, configurableItems, keySettings)`:
   - Redis key: `PRODUCT + "_" + tireId + "_" + culture + "_" + warehouseId`
   - Cache TTL: `ProductRedisTimeOut` key setting
   - If cache miss → calls `ISearch.GetTireDetails()` → `SP_GetTireDetails_Redesign_V8`
4. Calls `SetStaggeredDetailsForSearchByVehicle()` if coming from vehicle search
5. Calls `ValidateWarehouseBeforeAddtoCart()`
6. Assembles `TireItemDetailsModel` with full `TireInfo`

### 9.2 DataSet Structure from `SP_GetTireDetails_Redesign_V8`

Returns 9-table DataSet:

| Table | Contents |
|-------|---------|
| 0 | Core tire data (all `TireInfo` scalar fields) |
| 1 | Warranty records → `TireWaranty` list |
| 2 | Features → `TireFeatures` list |
| 3 | Benefits → `TireBenifits` list |
| 4 | Images → `TireImages` list (small/medium/large) |
| 5 | Footnotes → `TireFootNotes` list |
| 6 | Videos → `TireVideo` list |
| 7 | PDF brochure → `TirePdfBrochure` list |
| 8 | Supplier info → `SupplierInfo` |

### 9.3 Staggered PDP (`SetStaggeredDetailsForSearchByVehicle`)

When `isStaggered=true` and user came from vehicle search:
- Reads staggered front/rear tire IDs from `CookieTireDetails` cookie
- Calls `GetSearchResults()` for both tires
- Combines into `StaggeredTireModel` with front and rear `TireInfo`
- `ClubbedTirePrice = FrontPrice + RearPrice`
- PDP shows both tires with a combined "Add Both to Cart" button

### 9.4 Warehouse Validation (`ValidateWarehouseBeforeAddtoCart`)

Before PDP render:
- Calls Apigee/Costco warehouse inventory API with current warehouse + TireID
- Checks `IsAvailable` flag from real-time inventory
- If not available → sets `IsAvailable = false` on `TireInfo` before render
- Note: this is a real-time check unlike search results (which may be cached)

### 9.5 Add-to-Cart from PDP

- `POST /Product/AddToCart` (same as search controllers)
- TPMS cookie set: `CookieTireDetailsForTPMS` = JSON with TireID, SkuNumber, TPMS index
- Redirect to Costco cart URL with TPMS parameters appended if `TPMSIndex` is non-null

### 9.6 Search Results from PDP (`GetSearchResults`)

Used for "Similar Tires" / "Other Sizes" panel on PDP:
- Calls `ISearch.GetCustomFitmentSearchResults()` → `SP_GetCustomFitmentSearchResults`
- Returns tires of same size class, filtered by warehouse
- Displayed as horizontal tile strip below main tire info

---

## 10. Tire Compare Page

**Controller:** `CompareController`  
**Route:** `GET /Compare?CompareSearchID={ids}&whs={id}`  
**Session Key:** `SESSION_COMPARESEARCH_CRITERIA`

### 10.1 Request Flow (`Index()`)

1. Parses `CompareSearchID` query param — comma-delimited `TireID` values (2–4 tires)
2. Calls `BeginExecuteCore`
3. Calls `RenderPartialViewForCompare(tireIds, configurableItems, keySettings)`

### 10.2 Compare Processing (`RenderPartialViewForCompare`)

**Step 1 — Sequence Maintenance:**
- User selects tires in a specific order from search results
- `CompareSearchID` preserves that order
- Compare page must render in the same order the user selected

**Step 2 — Data Fetch:**
- Calls `ISearch.GetTireDetailsforCompare(tireIds, configurableItems, keySettings)`:
  - SP: `SP_GetTireDetailsForCompare_Redesign_V8`
  - `@TireIDs` — comma-delimited IDs
  - Returns tires with full properties + features/benefits/warranty

**Step 3 — Staggered Tire Clubbing:**
- If staggered compare: front+rear pairs identified by `ClubbedId`
- Front and rear displayed in adjacent columns as a "coupled" pair
- `ClubbedTirePrice` set on front tire; rear tire column hidden from pricing
- `IsStaggeredTire = true` on both tires in the pair

**Step 4 — Re-order by selection sequence:**
- Returned SP results may be in any DB order
- Applied sort maps `TireID` back to the user's selection order
- Ensures compare columns are displayed in user-intended order

**Step 5 — Full post-processing:**
- Full pipeline: promotions, delivery dates, warranty, winter filtering
- `IsComparePage = true` set on all tires (affects rendering)

### 10.3 Compare Popup (`GetTireDataForComparePopUp`)

`GET /Compare/GetTireDataForComparePopUp?TireID={id}`

Returns lightweight tire summary HTML for the compare selection popup on search results pages.

### 10.4 Tire Data Fetch (`GetTiresDetails`)

`POST /Compare/GetTiresDetails`

AJAX call to refresh compare data after user changes compare selection. Returns updated `_Compare.cshtml` partial HTML via `RenderRazorViewToString`.

---

## 11. Tire Size Selection Page

**Controller:** `SizeSelectionController`  
**Route:** `GET /SizeSelection`  
**View:** `~/Views/SizeSelection/_SizeSelection.cshtml`

### 11.1 Purpose

A CMS-driven hub page showing tire size categories. Contains embedded HTML from ContentStack/Kentico with size category tiles. Acts as a visual entry point for by-size searches.

### 11.2 Content Rendering (`RenderPartialViewFromTireSizeSelection`)

1. Fetches HTML content block from CMS:
   - ContentStack: `ContentstackIntegration.GetSizeSelectionContent(culture)`
   - Kentico: `KenticoRestApiIntegration.GetSizeSelectionContent(culture)`
2. Parses HTML using `HtmlAgilityPack`:
   - Finds all `<a href="...">` elements with size category links
   - Extracts `href` values and rewrites relative URLs to absolute
   - Replaces warehouse-placeholder tokens with actual warehouse ID
3. Returns assembled HTML with corrected links
4. View renders the CMS HTML block directly (bypasses MVC template for this section)

---

## 12. Search Filters Sidebar

**Controller:** `SearchFiltersController`  
**Routes:** Various AJAX endpoints for dropdown population

### 12.1 Vehicle Dropdown Chain

All dropdowns follow the same caching pattern:
- Check `HttpRuntime.Cache` → if hit, return cached list
- If miss → call DataAccess SP → cache result → return

| Dropdown | Action | SP | Cache Key |
|----------|--------|----|-----------|
| Year | `GetYearList()` | `SP_GetYear` | `YearList_{culture}` |
| Make | `GetMakeList(year)` | `SP_GetMakeByYear` | `MakeList_{year}_{culture}` |
| Model | `GetModelList(year, make)` | `SP_GetModelByYearMake` | `ModelList_{year}_{make}_{culture}` |
| Options | `GetOptionsList(year, make, model)` | `SP_GetOptionsByYearMakeModel` | `OptionsList_{year}_{make}_{model}_{culture}` |

### 12.2 Size Filter Dropdowns

| Dropdown | Action | SP |
|----------|--------|----|
| Width | `GetWidthList()` | `SP_GetWidth` |
| Aspect Ratio | `GetAspectRatioList(width)` | `SP_GetAspectRatioByWidth` |
| Rim Size | `GetRimSizeList(width, aspect)` | `SP_GetRimSizeByWidthAspect` |

### 12.3 Sidebar Filter (`SideBarFilter`)

`POST /SearchFilters/SideBarFilter`

Builds the left-rail filter panel with checkbox + count data. Reads pre-computed filter counts from Redis (set during the search render). Returns partial HTML.

### 12.4 Search Results Filter (`SearchResultsFilter`)

`POST /SearchFilters/SearchResultsFilter`

Rebuilds the inline filter bar at top of search results (used on mobile / compact view). Same data source as sidebar — Redis filter count keys.

---

## 13. Vehicle Validation

**Controller:** `ValidateVehicleController`  
**Routes:** `GET /ValidateVehicle/GetTireData`, `GET /ValidateVehicle/Validate`, `GET /ValidateVehicle/FetchKey`

### 13.1 Tire Fitment Check (`GetTireData()`)

`GET /ValidateVehicle/GetTireData?Year={y}&Make={m}&Model={mod}&Options={opt}&TireID={id}`

1. Calls `ISearch.ValidateVehicleInDB(configurableItems)` → `SP_ValidateVehicle`
2. Returns JSON with fitment compatibility:
   - `IsCompatible`: bool
   - `FitmentTireSize`, `FitmentLoadIndex`, `FitmentSpeedRating`
   - `STD_OR_OPT` — S or 1/2/3

### 13.2 Validate Form (`Validate()`)

`GET /ValidateVehicle/Validate?tireSize={size}&Year={y}&Make={m}&Model={mod}&Options={opt}`

1. Parses `tireSize` using `TIRESIZEREGEXPATTERNS` from configurable settings
   - Regex: matches formats like `235/55R18`, `235/55/18`, `235-55-18`
   - Extracts: Width (section), Aspect, RimSize
2. Builds `IConfigurableItemsAndAttributes` with parsed size + vehicle params
3. Calls `ISearch.ValidateVehicleInDB()` for cross-validation
4. Returns JSON `{ isValid: bool, fitmentDetails: {...} }`

### 13.3 Key Fetch (`FetchKey()`)

`GET /ValidateVehicle/FetchKey`

Returns current anti-forgery token and session key for subsequent AJAX calls. Used for session validation on dynamic search scenarios.

---

## 14. Warehouse Search

**Controller:** `SearchWarehouseController`  
**Routes:** Multiple warehouse lookup endpoints

### 14.1 Endpoints

| Method | Route | Input | Operation |
|--------|-------|-------|-----------|
| GET | `/SearchWarehouse` | None | Returns empty warehouse search view |
| GET | `/SearchWarehouse/IndexWithZipCode?zip={z}` | ZIP/postal code | Calls Bing Maps geocode → warehouse DB lookup |
| GET | `/SearchWarehouse/GetWarehouseDataByLatLong?lat={lat}&lng={lng}` | GPS coordinates | Direct warehouse DB lookup by proximity |
| GET | `/SearchWarehouse/GetWarehouseDataById?whs={id}` | Warehouse ID | Direct warehouse lookup by ID |
| GET | `/SearchWarehouse/GetWarehouseDataByPostalCode?postal={p}` | Postal code | Calls Bing Maps geocode → warehouse lookup |

### 14.2 ZIP/Postal Code to Location Flow

1. Calls `BingMapsIntegration.GetLocationFromZip(zip)`:
   - HTTP GET to Bing Maps Locations API
   - Returns: `Latitude`, `Longitude`, `FormattedAddress`
2. Calls `SP_GetWarehousesByProximity` with lat/long
   - Returns list of warehouses sorted by distance
   - Includes: `WH_ID`, `Name`, `Address`, `City`, `State`, `Zip`, `Phone`, `Distance`
3. Returns `Json(warehouseList)`

### 14.3 Warehouse Data API Path

For three async loading modes configured via `AsyncWarehouseAPI` / `OfflineWarehouseLoading` key settings:

| Mode | Behavior |
|------|---------|
| Standard | Synchronous Apigee API call in `BeginExecuteCore` |
| Async | Fires API call; JS polls for result; page renders before result |
| Offline | Uses last cached warehouse data; skips live API call |

Warehouse list can also be sourced from `HttpSingleton` (singleton-shared warehouse cache, refreshed on schedule).

---

## 15. Battery Domain

### 15.1 Overview

Battery search is Canada-only. `BaseBatteryController.BeginExecuteCore` redirects US requests (detected by `en-US` culture) to `/`.

**All battery controllers parallel the tire domain** but with battery-specific SPs and models.

### 15.2 Battery Home Controller

**Route:** `GET /Battery` or `GET /Battery/Index`

1. Calls `BeginExecuteCore` (US redirect, warehouse init)
2. Fetches CMS images via `FetchImages()`:
   - ContentStack path: `ContentstackIntegration.GetBatteryImages(culture)`
   - Kentico path: `KenticoRestApiIntegration.GetBatteryImages(culture)`
3. Sets home page banner data in view model
4. Returns `View()`

### 15.3 Battery Search By Vehicle

**Route:** `GET /Battery/SearchResultsByVehicle`

**`RenderPartialView` — AJAX render sequence:**

1. Builds `IConfigurableItemsAndAttributes` with battery-specific category type
2. Adds `batteryCategory` parameter to distinguish battery sub-types (e.g., automotive, marine, motorcycle)
3. Calls `BatterySearch.GetSearchResults()` → SP: `SP_GetBatteryByVehicle`
4. Special filter: `is_sold_online` flag — removes batteries where `is_sold_online = false`
5. QR Scan Path: if `source=qr` query param → bypasses `is_sold_online` filter (technician-facing flow)
6. Applies battery-specific promotions via `SetConfigurablePromotions`
7. Returns partial HTML

### 15.4 Battery Vehicle Dropdowns

**`SearchFiltersBatteryController`** — dropdown endpoints:

| Endpoint | SP | Parameters |
|----------|----|----|
| `GetMakeList(year)` | `SP_GetBatteryMakeByYear` | `@Year`, `@BatteryCategory` |
| `GetModelList(year, make)` | `SP_GetBatteryModelByYearMake` | `@Year`, `@Make`, `@BatteryCategory` |
| `GetEngineList(year, make, model)` | `SP_GetBatteryEngineByYearMakeModel` | `@Year`, `@Make`, `@Model`, `@BatteryCategory` |
| `GetTypeList(year, make, model, engine)` | `SP_GetBatteryTypeByYearMakeModelEngine` | Full params + `@BatteryCategory` |

All battery dropdowns accept a `batteryCategory` parameter which scopes the results to a specific battery category (automotive, commercial, etc.).

### 15.5 Battery Product Detail

**Route:** `GET /Battery/Product?TireID={batteryId}&whs={id}`

1. Reads battery ID from query string
2. Calls `BatterySearch.GetBatteryDetails()` → SP: `SP_GetBatteryDetails`
3. Assembles battery product model
4. Calls `SetSignInMembershipInfo()`:
   - Reads `mSign` cookie
   - If value matches `IsCookiValuehasTrue` config → sets `IsMemberSignedIn = true`
   - Enables member-only pricing display on PDP
5. `AddToCart`:
   - Same URL construction as tire AddToCart
   - Battery uses Canada-only cart URLs (`CostcoCAURL`)
   - Appends `langId=-25` for French or `-24` for English CA

### 15.6 Battery Promotions

Battery domain uses `BatteryConfigurations` (parallel to tire's `Configurations`) and `BatteryWebConfigKeys`.

`SetConfigurablePromotions()` in `BaseBatteryController`:
- Reads battery promotions from Redis / DB using battery-specific cache keys
- Applies promotions to batteries during search/PDP render
- Promotions are brand-specific (stored in `BatteryPromotionList` on battery model)

---

## 16. Admin Panel

**Routes:** `GET /Admin/Login`, `POST /Admin/LoginAdmin`, `GET /Admin/Promotions`, `POST /Admin/SavePromotions`, `GET /Admin/GetPromotions`

### 16.1 Authentication (`LoginController`)

| Step | Details |
|------|---------|
| GET `/Admin/Login` | Returns login form view |
| POST `/Admin/LoginAdmin` | Validates credentials |
| Credential check | Hardcoded: `username == "admin" && password == "admin"` |
| Success | Redirect to `Admin/AdminHome/Index` |
| Failure | Re-renders login view (no error message) |

> **Security Note:** Credentials are hardcoded — no hashing, no lockout policy. This is a known limitation. See Business Rules §10.

### 16.2 Promotions Management (`AdminPromotionsController`)

**`GET /Admin/Promotions` — Index:**

Renders promotions admin form with dropdowns:

| Dropdown | Values |
|----------|--------|
| Suppliers | Michelin (1), Bridgestone (2), BFGoodrich (8), Pirelli (12) |
| Countries | USA, CA |
| Is Cash Card | -1 (Select), 0 (No), 1 (Yes) |
| Is Active | -1 (Select), 0 (No), 1 (Yes) |
| Set Qty | 4 (default) |

**`POST /Admin/SavePromotions` — Save:**

1. Receives `PromotionsInfo` form model
2. If `country_code == "CA"` → forces `lang = "fr-CA"`
3. Else → `lang = "en-US"`
4. Calls `PromotionsBusinessObject.SavePromotions(promotionsInfo)` → `SP_SavePromotion`
5. Returns `Json(result)` — `{ success: true/false, message: "..." }`

**`GET /Admin/GetPromotions` — Get:**

> Note: Current implementation has hardcoded `country_code = "USA"` and `promotion_id = 10`, ignoring query params.

1. Calls `PromotionsBusinessObject.GetPromotions(objViewRequest)` → `SP_GetPromotions`
2. Returns `Json(objPromotionsInfo)` — list of `PromotionsInfo`

**`PromotionsInfo` Model:**

| Field | Type | Description |
|-------|------|-------------|
| `promotion_id` | `int` | DB primary key |
| `promotion_name` | `string` | Display name |
| `discount_price` | `decimal` | Discount amount |
| `is_cash_card` | `bool` | Cash card vs. discount |
| `supplier_id` | `int` | Linked supplier |
| `country_code` | `string` | USA / CA |
| `lang` | `string` | Language code |
| `set_of_qty` | `int` | Required quantity |
| `valid_from`, `valid_to` | `DateTime` | Promotion date range |
| `is_active` | `bool` | Active flag |

---

## 17. Configuration Engine

### 17.1 Two-Tier Configuration System

The system uses two types of configuration:

**Tier 1 — Static Web.Config Keys (`WebConfigKeys`)**

`WebConfigKeys` is a static class with `readonly` fields pointing to `ConfigurationManager.AppSettings` and `ConnectionStrings`. Loaded once at app start. Cannot change without deployment.

**Tier 2 — Dynamic Database Config Keys (`Configurations.cs`)**

`Configurations.cs` / `SetDatabaseCostcoWebConfigKey()`:
1. Reads `TriggerDatabaseKeyCheck` from web.config (a boolean toggle)
2. Checks Redis for cached config object (`ConfigurableItemsAndAttributesTimeOut` TTL)
3. If cache miss → calls `SP_GetConfigurableItemsAndAttributes`
4. SP returns `Tuple<string, string>` pairs (key, value), delimited by `|||`
5. Splits and maps to `ConfigurableItemsAndAttributes` POCO
6. Stores in Redis with configured expiry
7. Returns fully populated `ConfigurableItemsAndAttributes`

**`ConfigurableItemsAndAttributes` key categories:**

| Category | Keys |
|----------|------|
| Warehouse filtering | `SPECIFICWAREHOUSELIST`, `DEACTIVATED_WAREHOUSE_CODES`, `BlackListVendorForWarehouses` |
| Delivery | `PromoBusinessDays`, `PromoNonBusinessDays`, `OrderDeliveryDays_*` |
| Pricing | `TirePriceMultipleOf`, `TirePriceMultipleOf_Staggered`, `MinAvailableQuantity` |
| Feature flags | `EnableMaximumTraction`, `AsyncWarehouseAPI`, `OfflineWarehouseLoading` |
| Regex | `TireSizeRegexPatterns` |
| Fitment | `LoadRatingDeviation`, `SpeedRatingDeviation` |
| Vendor | `VendorIntegrationName` |
| Promotions | (via `GetAllPromotions()` → separate SP) |

### 17.2 Key Settings (`KeySettings`)

`List<IKeySettings>` — a finer-grained key-value store loaded from DB table `KeySettings`:

| Type | Example Keys |
|------|-------------|
| Redis timeouts | `ByVehicleRedisCacheTimeOut`, `BySizeRedisCacheTimeOut`, `ProductRedisTimeOut` |
| Delivery | `PromoBusinessDays`, `PromoNonBusinessDays`, `SupplierSpecificDeliveryInfo` |
| Supplier promo | `USMichelinDiscountPrice`, `CABFGoodRichDiscountPrice`, etc. |
| Feature flags | (various) |

---

## 18. Promotions Engine

### 18.1 Promotion Types

| Type | Mechanism | Display |
|------|-----------|---------|
| Cash Card Rebate | `is_cash_card = 1` | Rebate card imagery + text |
| Discount Price | `is_cash_card = 0` | Strikethrough price + discounted price |
| Set-of-Four | `set_of_qty = 4` | "When you buy a set of 4" messaging |
| Member-Only | `mSign` cookie flag | Additional member price display |
| SKU-Specific | `PromoSKUs` allowlist in web.config | Per-SKU override pricing |

### 18.2 Promotion Loading (`GetAllPromotions`)

`CommonSearchLogic.GetAllPromotions(culture)`:
1. Calls `SP_GetPromotionsList` with `@Culture`
2. Returns `List<IPromotions>` — all active promotions for the culture
3. Cached per request (not Redis-cached — loaded once per search render)

### 18.3 Promotion Application (`SetConfigurablePromotions`)

Iterates every tire in results:
1. Matches `Supplier_ID` against active promotion records
2. Checks promotion `valid_from`/`valid_to` date range (server `DateTime.Now`)
3. If match found and active → sets on `TireInfo`:
   - `HasPromotion = 1`
   - Discount price fields
   - Promotion text strings (from `WebConfigKeys` or DB)
   - `ImgUSPromoOffPrice_*` image URLs
4. **SKU-specific override:** `ApplySpecificMemberOnlyPricePromoCheck` flag + `BFGoodrichMemberOnlyPricePromoSKUsCheck` allowlist → different promotion for specific SKUs

### 18.4 US Brand Promotions (WebConfig-driven)

For US tire promotions (Michelin, Bridgestone, BFGoodrich), configuration is stored in web.config (not DB):

| Key Pattern | Example |
|-------------|---------|
| `USPromoOffPrice_{Brand}` | Dollar off per tire |
| `USAnySetOfFourTireText_{Brand}` | Marketing copy: "Get $X off any set of 4" |
| `USPromotion_{Brand}_ValidDates` | Date range string for display |
| `ImgUSPromoOffPrice_{Brand}` | Promo image URL |
| `{Brand}DiscountPrice` | Final discounted price |
| `{Brand}TirDiscountPriceperTire` | Per-tire discount |

### 18.5 Canada Promotions (Webb Web.Config-driven)

Mirrors US but with locale variants:
- `CA{Brand}DiscountPrice` — CA dollar discount
- `CAAnySetOfFourTireTextEn_{Brand}` — English copy
- `CAAnySetOfFourTireTextFr_{Brand}` — French copy (Quebec)

### 18.6 Tire Levy (CA Ontario / Quebec)

Canada-specific fee applied on top of tire price:
- `TireLevyPerTirePrice` — standard CA levy
- `TireLevyPerTirePriceForOntarioLocation` — Ontario-specific levy
- Applied when warehouse is in the Ontario/Quebec warehouse lists:
  - `PriceForSecificQuebecLocationsWarehosueId_CA`
  - `PriceForSecificOntarioLocationsWarehosueId_CA`

---

## 19. CMS Integration (ContentStack & Kentico)

### 19.1 Toggle Mechanism

`WebConfigKeys.IsContentstackActive` — boolean flag:
- `true` → route to `ContentstackIntegration`
- `false` → route to `KenticoRestApiIntegration`

All CMS-consuming controllers check this at the call site:
```csharp
if (WebConfigKeys.IsContentstackActive)
    images = ContentstackIntegration.GetImages(culture);
else
    images = KenticoRestApiIntegration.GetImages(culture);
```

### 19.2 ContentStack Integration

**Interface:** `IHttpContentstackClient`  
**Params:** `ContentstackReqParams` (environment, API key, content type, language)

**Calls Made:**

| Operation | Content Type | Controller Using It |
|-----------|-------------|---------------------|
| Home images | `home_banner` | `HomeController.FetchImages` |
| Battery home images | `battery_banner` | `Battery.HomeController.FetchImages` |
| Legal notices (CA) | `legal_notice` | `HomeController.SetLegalNotice_Ca` |
| Size selection content | `size_selection_hub` | `SizeSelectionController` |

All ContentStack responses are JSON, deserialized to typed models via `Newtonsoft.Json`.

### 19.3 Kentico REST API Integration

**Base URL:** `WebConfigKeys.RequestRestUrl`  
**Auth:** `WebConfigKeys.BasicAuth` (HTTP Basic Auth header)

**Endpoints:**

| Operation | REST Path | Auth |
|-----------|----------|------|
| Home images | `/documents/{ImageTable}/{ImageSubTable}` | Basic |
| Home blog content | `/documents/{BlogTable}/{BlogSubTable}` | Basic |
| CMS documents | `/documents/{DocumentTable}/{DocumentSubTable}` | Basic |

Response is XML — parsed into typed models.

### 19.4 CMS Caching

- ContentStack responses cached in Redis:
  - Key: `HomeSettingRedisTimeOut` TTL setting
  - Key prefix per content type + culture
- Kentico responses cached in `HttpRuntime.Cache`
- CMS cache is invalidated by deploying new content (no active invalidation)

---

## 20. External Integrations

### 20.1 Mitchell1 (License Plate / VIN)

**Purpose:** Convert license plate → VIN → Year/Make/Model  
**Auth:** Digest authentication (username:password in web.config under Key Vault)  
**Timeout:** `LicensePlateAPITimeout` ms  

**Session/Token Flow:**
1. OAuth-style token lifecycle using Redis:
   - `LicAccessToken`, `LicRefreshToken`, `LicProxyId` keys in Redis
2. On 401 → refresh token via `/auth/refresh` endpoint, retry once
3. On repeated 401 → log error, fallback to manual vehicle entry

**Mitchell1 Error Handling:**
- API errors logged to Application Insights
- Exception does not surface to user — graceful redirect to vehicle search

### 20.2 Bing Maps

**Purpose:** Geocode ZIP/postal codes to lat/long for warehouse proximity search  
**Endpoint:** `https://dev.virtualearth.net/REST/v1/Locations/{zip}`  
**Auth:** Bing Maps API key from Azure Key Vault  

**Response mapping:**
- `resourceSets[0].resources[0].point.coordinates` → `[lat, lng]`
- Passed to `SP_GetWarehousesByProximity`

### 20.3 Apigee / Costco Inventory API

**Purpose:**
1. Warehouse data fetch (name, address, hours, services)
2. Real-time product availability check on PDP

**Endpoints:** Managed via Apigee proxy — actual paths in Azure Key Vault / web.config  
**Auth:** API key in `Authorization` header  

**Response:**
- Warehouse: JSON object with `warehouseId`, `name`, `address`, `operatingHours`
- Availability: JSON array with `itemNumber`, `isAvailable`, `quantity`

### 20.4 Azure Key Vault

At application startup (`Application_Start`):
1. Authenticates to Azure Key Vault using Managed Identity or ClientId/Secret
2. Loads all secret values into `ConfigurationManager.AppSettings` overrides
3. Secrets include: DB connection strings, API keys, basic auth credentials

### 20.5 Application Insights

- Telemetry client initialized in `BaseController`
- Custom events logged for:
  - Zero-result searches (`SaveZeroResultsParamater`)
  - Mitchell1 API failures
  - Redis connection failures
  - Configuration load failures
- `LogCustomErrors` flag gates custom error logging
- `IsLogInfoActive` flag gates informational event logging

---

## 21. Data Model Reference

### 21.1 Core `TireInfo` Properties (Selected Key Fields)

| Field | Type | Source | Description |
|-------|------|--------|-------------|
| `TireID` | `int` | DB | Primary key |
| `SkuNumber` | `string` | DB | Costco SKU |
| `CostcoProductId` | `string` | DB | Costco internal ID |
| `TireName` | `string` | DB | Brand + model name |
| `TireSize` | `string` | DB | Full size string (e.g., `235/55R18`) |
| `Price` | `double` | DB | Base per-tire price |
| `Supplier_ID` | `int` | DB | Foreign key to supplier |
| `Tire_Type_ID` | `int` | DB | Foreign key to tire type |
| `STD_OR_OPT` | `string` | DB | `S`=standard, `1-3`=optional fitment |
| `FRB` | `string` | DB | `F`=front, `R`=rear, `B`=both |
| `IsAvailable` | `bool` | Computed | Availability after all checks |
| `IsOutOfStockEquipment` | `bool` | DB | Out-of-stock flag |
| `Available_Qty` | `int` | DB | Available quantity |
| `NextAvailableDate` | `DateTime` | DB | Back-in-stock date |
| `HasPromotion` | `int` | Computed | 1 if active promotion applies |
| `EvCompatible` | `bool` | DB | Works with EVs |
| `EvDedicated` | `bool` | DB | Designed for EVs |
| `IsStaggeredTire` | `bool` | Computed | Part of staggered pair |
| `ClubbedId` | `string` | Computed | Groups F+R staggered tires |
| `ClubbedTirePrice` | `double` | Computed | Combined F+R price |
| `RunFlat` | `string` | DB | Run-flat specification |

### 21.2 `SearchResults` Model

```
SearchResults
├── FitmentTireInfo: List<IFitmentTireInfo>  — vehicle fitment data
├── TireInfo: List<ITireInfo>                — all matching tires
└── Search: ISearch                          — the search criteria used
```

### 21.3 `ConfigurableItemsAndAttributes` (Request Context)

Built per request in `BeginExecuteCore` and passed to all BusinessObject calls:

| Property | Source | Description |
|----------|--------|-------------|
| `WarehouseId` | Cookie/param | Current warehouse ID |
| `CultureInfo` | Thread culture | Current locale |
| `CountryCode` | Derived from culture | USA / CA |
| `Year`, `Make`, `Model`, `Options` | Query params | Vehicle selection |
| `Width`, `AspectRatio`, `RimSize` | Query params | Size selection |
| `TireID` | Query param | Item ID for PDP |
| `IsStaggered` | Query param | Staggered search flag |
| `PageIndex` | Query param | Pagination |
| `SortOrder` | Query/session | Sort selection |
| `LoadRatingDeviation` | DB config | Allowed load deviation |
| `SpeedRatingDeviation` | DB config | Allowed speed deviation |

### 21.4 Stored Procedure Catalog (Tire Domain)

| SP Name | Operation | Called By |
|---------|-----------|-----------|
| `SP_GetSearchResultsByVehicle_Redesign_V8` | Vehicle search | `SearchData.GetSearchResultsByVehicle` |
| `SP_GetSearchResultsBySize_Redesign_V8` | Size search | `SearchData.GetSearchResultsBySize` |
| `SP_GetSearchResultsByCustomSize_Redesign_V8` | Custom/staggered size | `SearchData.GetCustomTireSizeSearchResults` |
| `SP_GetTireDetailsByItemNumber` | Item number search | `SearchData.GetSearchResultsByItem` |
| `SP_GetTireDetailsByPartNumber` | Part number search | `SearchData.GetSearchResultsByPart` |
| `SP_GetTireDetails_Redesign_V8` | PDP / tire details | `SearchData.GetTireDetails` |
| `SP_GetTireDetailsForCompare_Redesign_V8` | Compare page | `SearchData.GetTireDetailsForCompare` |
| `SP_GetCustomFitmentSearchResults` | Custom fitment / PDP similar | `SearchData.GetCustomFitmentSearchResults` |
| `SP_ValidateVehicle` | Fitment validation | `SearchData.ValidateVehicleInDB` |
| `SP_GetYear` | Year dropdown | `SearchFiltersData.GetYearList` |
| `SP_GetMakeByYear` | Make dropdown | `SearchFiltersData.GetMakeList` |
| `SP_GetModelByYearMake` | Model dropdown | `SearchFiltersData.GetModelList` |
| `SP_GetOptionsByYearMakeModel` | Options dropdown | `SearchFiltersData.GetOptionsList` |
| `SP_GetWidth` | Width dropdown | `SearchFiltersData.GetWidthList` |
| `SP_GetAspectRatioByWidth` | Aspect ratio dropdown | `SearchFiltersData.GetAspectRatioList` |
| `SP_GetRimSizeByWidthAspect` | Rim size dropdown | `SearchFiltersData.GetRimSizeList` |
| `SP_GetHolidayList` | Holiday calendar | `SearchData.GetHolidayList` |
| `SP_GetPromotionsList` | All active promotions | `SearchData.GetAllPromotions` |
| `SP_SavePromotion` | Save promotion | `PromotionsData.SavePromotion` |
| `SP_GetPromotions` | Get promotions | `PromotionsData.GetPromotions` |
| `SP_GetConfigurableItemsAndAttributes` | App configuration | `ConfigurationsData.GetConfigurableItems` |
| `SP_GetWarehousesByProximity` | Nearby warehouses | `WarehouseData.GetWarehousesByProximity` |

### 21.5 Stored Procedure Catalog (Battery Domain)

| SP Name | Operation |
|---------|-----------|
| `SP_GetBatteryByVehicle` | Battery vehicle search |
| `SP_GetBatteryDetails` | Battery PDP data |
| `SP_GetBatteryMakeByYear` | Battery make dropdown |
| `SP_GetBatteryModelByYearMake` | Battery model dropdown |
| `SP_GetBatteryEngineByYearMakeModel` | Battery engine dropdown |
| `SP_GetBatteryTypeByYearMakeModelEngine` | Battery type dropdown |

---

## 22. Session, Cookie, and Cache Architecture

### 22.1 Session Keys

| Key Constant | Value | Content Type | Lifetime |
|-------------|-------|-------------|---------|
| `SESSION_SEARCH_PARAM` | `"QueryString"` | URL query string | Per search |
| `SESSION_SEARCH_CRITERIA` | `"Search"` | `ISearch` object | Per search |
| `SESSION_SEARCH_RESULTS` | `"SearchResults"` | `ISearchResults` | Per search |
| `SESSION_YEAR_LIST` | `"YearList"` | Year dropdown items | Till session expire |
| `SESSION_MAKE_LIST` | `"MakeList"` | Make dropdown items | Till session expire |
| `SESSION_MODEL_LIST` | `"ModelList"` | Model dropdown items | Till session expire |
| `SESSION_OPTIONS_LIST` | `"OptionsList"` | Options dropdown items | Till session expire |
| `SESSION_COMPARESEARCH_CRITERIA` | `"Compare"` | Compare search params | Per compare |
| `SESSION_TIRE_ITEM_DETAILS` | `"TIRE_ITEM_DETAILS"` | tire detail model | Per PDP visit |
| `VehicleFitmentData` | `"VehicleFitmentData"` | fitment data list | Per vehicle search |

### 22.2 Redis Cache Keys

| Key/Pattern | Content | TTL Setting |
|-------------|---------|------------|
| `sbvr_{hash}` | Vehicle search results JSON | `ByVehicleRedisCacheTimeOut` |
| `{width}_{aspect}_{rim}_{culture}_{whs}` | Size search results JSON | `BySizeRedisCacheTimeOut` |
| `product_{tireId}_{culture}_{whs}` | PDP tire details JSON | `ProductRedisTimeOut` |
| `CompareKey_{ids}_{culture}_{whs}` | Compare results JSON | `CompareRedisCacheTimeOut` |
| `HolidayListKey_{culture}` | Holiday list | `HolidayListRedisCacheTimeOut` |
| `HomeSettingRedisTimeOut` | CMS home images | `HomeSettingRedisTimeOut` |
| `BlogRedisTimeOut` | CMS blog content | `BlogRedisTimeOut` |
| `ConfigurableItemsAndAttributesTimeOut` | App config object | `ConfigurableItemsAndAttributesRedisTimeOut` |
| `LicAccessToken` | Mitchell1 access token | Managed by Mitchell1 token TTL |
| `LicRefreshToken` | Mitchell1 refresh token | Managed by Mitchell1 token TTL |
| `LicProxyId` | Mitchell1 proxy ID | Managed by Mitchell1 token TTL |
| `brand_names_{searchKey}` | Brand filter counts | Same as search result TTL |
| `season_names_{searchKey}` | Season filter counts | Same as search result TTL |
| `price_filter_{searchKey}` | Price filter counts | Same as search result TTL |
| `runflat_filter_{searchKey}` | Run-flat filter counts | Same as search result TTL |

### 22.3 In-Process Cache (`HttpRuntime.Cache`)

| Key Pattern | Content | Eviction |
|-------------|---------|---------|
| `YearList_{culture}` | Year dropdown | On expiry / recycle |
| `MakeList_{year}_{culture}` | Make dropdown | On expiry / recycle |
| `ModelList_{year}_{make}_{culture}` | Model dropdown | On expiry / recycle |
| `OptionsList_{year}_{make}_{model}_{culture}` | Options dropdown | On expiry / recycle |
| CMS content keys | Kentico XML responses | `CacheExpiration` / `CacheExpirationFrontEnd` |

### 22.4 Cache Layer Decision Logic

```
Request arrives
    ↓
Check HttpRuntime.Cache (in-process, instant) → HIT: return
    ↓ MISS
Check Redis (distributed, sub-ms) → HIT: deserialize + return
    ↓ MISS
Call SQL Server SP → process + serialize → store in Redis + HttpRuntime.Cache → return
```

---

## 23. Localization and Multi-Region Support

### 23.1 Culture Resolution Order

1. `lang` URL query parameter (mobile app override only when `IsMobileAppLangParamater = true`)
2. `COSTCO_SELECTED_LANGUAGE` cookie
3. URL path prefix: `/en-ca/`, `/fr-ca/`
4. Default → `en-US`

### 23.2 Culture-Specific Behavior

| Behavior | en-US | en-CA | fr-CA |
|----------|-------|-------|-------|
| Battery domain accessible | No (redirect to /) | Yes | Yes |
| Currency symbol | `$` | `$` (CAD) | `$` (CAD) |
| Tire levy applied | No | Yes (Ontario: higher) | Yes (Quebec: variant) |
| Winter tire info visible | No | Yes | Yes (French labels) |
| Warranty units | Miles | Km | Km (French: "garantie...") |
| Cart URL | `CostcoUSURL` | `CostcoCAURL` | `CostcoCAURL_M` |
| langId param | `-1` | `-24` | `-25` |
| Size display (rim) | Inches | Inches + CM | Inches + CM |
| Promo text | English | English | French |
| CMS content | en-US bucket | en-CA bucket | fr-CA bucket |

### 23.3 Language Selector (`HomeController.LanguageSelector`)

`GET /Home/LanguageSelector?lang={code}`

1. Sets `COSTCO_SELECTED_LANGUAGE` cookie to specified language code
2. Redirects to home page with new culture context

### 23.4 French-Specific Content Keys

`HomeController.SetLegalNotice_Ca`:
- Only executes for `fr-CA` culture
- Fetches legal notice text from CMS
- Sets in `ViewBag.LegalNotice` for footer display

---

## 24. Business Rules Reference

### BR-01: Warehouse Required
All search result pages require a resolved warehouse. If no warehouse is determined from cookie/param/postal code, user is redirected to Home page.

### BR-02: Deactivated Warehouses
Any warehouse ID in the `DEACTIVATED_WAREHOUSE_CODES` configurable setting returns a redirect to Home. This is a killswitch for temporarily disabling specific warehouse tire programs.

### BR-03: Staggered Tire Pairs
Tires with different front and rear sizes (staggered fitment) are grouped by `ClubbedId`. Compare page and PDP handle staggered pairs as a unit — combined price, minimum qty 2+2. `ClubbedTirePrice = FrontPrice + RearPrice`.

### BR-04: Winter Tire Regional Control
Two mechanisms controlled independently:
- **Hide** (`SPECIFICWAREHOUSELIST`): Winter tires not shown at listed warehouses  
- **Warning** (`WinterTireWarningWarehouses`): Warning shown instead of hiding at listed warehouses
- `WinterTiresONForSupplierId`: Only approved suppliers' winter tires shown; others removed

### BR-05: EV Tire Filtering
`EvDedicated` tires are always shown when EV filter is active. `EvCompatible` tires shown only when EV Compatibility filter is explicitly ON. When filter = `evc`, both compatible and dedicated shown.

### BR-06: Member-Only Pricing
`mSign` cookie matching `IsCookiValuehasTrue` config value indicates signed-in Costco member. Member-only pricing display enabled on search results and PDP when flag is set.

### BR-07: Load/Speed Rating Tolerance
Vehicle fitment validation applies configurable deviation:
- `LoadRatingDeviation` — allows tires within N load index points of required
- `SpeedRatingDeviation` — allows tires within one speed rating class of required
- Winter tires have additional `SpeedMilesPerHour` check against `WinterLoadRatingDeviation`

### BR-08: Promotions Precedence
1. SKU-specific promotions override brand-level promotions
2. DB promotions (admin-managed) override web.config promotions
3. Promotions outside `valid_from`/`valid_to` date range are not applied
4. `is_active = false` promotions ignored

### BR-09: Battery Canada-Only
Battery domain is exclusively for Canada. US requests are redirected to `/` at `BaseBatteryController.BeginExecuteCore`. This is enforced at the controller level, not routing.

### BR-10: Admin Security Limitation
Admin panel uses hardcoded credentials (`admin`/`admin`). No rate limiting, no account lockout, no password hashing. Access is typically restricted via network/firewall rules.

### BR-11: Price Multiples
Tire prices must be multiples of `TirePriceMultipleOf` (standard) or `TirePriceMultipleOf_Staggered` (staggered pair). Any price from DB that is not a multiple of the configured value is rounded appropriately.

### BR-12: Minimum Available Quantity
Per-supplier minimum: `USBridgestoneMinAvailableQuantity`, etc. If `Available_Qty < MinAvailableQuantity` for that supplier → `IsAvailable = false`. Configurable per brand, per region.

### BR-13: Blacklisted Vendor/Warehouse Combinations
Format: `"warehouseId:supplierId1,supplierId2|warehouseId2:..."`. Tires from blacklisted suppliers at specific warehouses are removed from results post-SP-call.

### BR-14: `is_sold_online` Battery Filter
Battery search removes items where `is_sold_online = false` when the request is a standard web request. QR scan path (`source=qr`) bypasses this filter for technician-facing use.

### BR-15: Real-Time Warehouse Validation (PDP Only)
PDP performs a live Apigee API call to verify current availability before page render. Search results use cached availability (Redis). Only PDP does real-time validation.

### BR-16: Tire Levy (CA Ontario / Quebec)
`TireLevyPerTirePrice` added to base price for all CA tires. `TireLevyPerTirePriceForOntarioLocation` used for warehouses in Ontario list. Quebec warehouses use variant.

---

## 25. Constants and Configuration Reference

### 25.1 Culture and Country Codes

| Constant | Value |
|----------|-------|
| `UNITED_STATES_CULTURE_CODE` | `"en-us"` |
| `CANADA_ENG_CULTURE_CODE` | `"en-ca"` |
| `FRENCH_CULTURE_CODE` | `"fr-ca"` |
| `UNITED_STATES_COUNTRY_CODE` | `"USA"` |
| `CANADA_COUNTRY_CODE` | `"CA"` |

### 25.2 Key Separators

| Constant | Value | Use |
|----------|-------|-----|
| `STRING_SEPERATOR` | `"|||"` | Search criteria segment delimiter |
| `DoublePipeSeperator` | `"||"` | Fitment data segment delimiter |
| `StaggeredVehicleTabSeperator` | `"^"` | Staggered tab delimiter |

### 25.3 Season Constants

| Constant | Value |
|----------|-------|
| `SEASON_ALLSEASON` | `"All-Season"` |
| `SEASON_SUMMER` | `"Summer"` |
| `SEASON_WINTER` | `"Winter"` |
| `MAXIMUM_TRACTION` | `"Maximum Traction"` |
| `MAXIMUM_TRACTION_HY` | `"Maximum-Traction"` |
| `SEASON_HIVER` | `"Hiver"` (French) |

### 25.4 EV Type Constants

| Constant | Value |
|----------|-------|
| `EVTYPE_EVC` | `"evc"` (EV Compatible filter value) |
| `EVTYPE_EVD` | `"evd"` (EV Dedicated filter value) |

### 25.5 `langId` for Cart Redirects

| Culture | `langId` Value |
|---------|---------------|
| `en-US` | `-1` |
| `en-CA` | `-24` |
| `fr-CA` | `-25` |

### 25.6 View Name Constants

| Constant | Path |
|----------|------|
| `SEARCHRESULTSBY_BASE` | `~/Views/SearchResultsByBase/SearchResultsByBase.cshtml` |
| `PARTIALBYBASESEARCH_VIEW` | `_PartialByBaseSearch` |
| `TIREITEMDETAILS_VIEW` | `TireItemDetails` |
| `SIZESELECTION_VIEW` | `~/Views/SizeSelection/_SizeSelection.cshtml` |
| `COMPARE_PARTIAL_VIEW` | `~/Views/Compare/_Compare.cshtml` |
| `SEARCHRESULTSBYITEMORPART_VIEW` | `SearchResultsByItemOrPart` |

### 25.7 Image Path Constants

| Constant | Path Pattern |
|----------|-------------|
| `IMG_REL_PATH` | `/app_themes/cwc/images` |
| `RESPONSIVE_IMG_REL_PATH` | `/app_themes/cwc/images_responsive` |
| `IMG_TIRE_PATH` | `/app_themes/cwc/tireimages` |
| Winter tire logo | `/tiretypes/winter.png` |
| EV Compatible logo | `{ImageHostName}/app_themes/cwc/evcompatibility/evcompatible-icon.png` |
| EV Dedicated logo | `{ImageHostName}/app_themes/cwc/evcompatibility/dedicated-icon.png` |
| Run-flat icon | `{ImageHostName}/app_themes/cwc/tiretypes/runflat-icon.png` |

---

*End of Functional Specification*
