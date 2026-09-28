# Deep-Dive: User Personas & User Journeys
## Costco Tire & Battery E-Commerce Platform

**Version:** 1.0  
**Generated:** 2026-04-13  
**Source:** Functional specification, source code analysis (Controllers, BusinessObjects, DataAccess, Integration layers), and all existing docs  
**Scope:** All user-facing and admin-facing interactions across the US and Canada tire/battery platform

---

## Table of Contents

1. [Persona Overview](#1-persona-overview)
2. [Persona 1 — US Costco Member (Vehicle Owner)](#2-persona-1--us-costco-member-vehicle-owner)
3. [Persona 2 — Canadian Member (English)](#3-persona-2--canadian-member-english)
4. [Persona 3 — Quebec Member (French-Speaking)](#4-persona-3--quebec-member-french-speaking)
5. [Persona 4 — Costco Battery Shopper (Canada)](#5-persona-4--costco-battery-shopper-canada)
6. [Persona 5 — Mobile / QR Scan Technician](#6-persona-5--mobile--qr-scan-technician)
7. [Persona 6 — Promotions Admin](#7-persona-6--promotions-admin)
8. [User Journey 1 — Search by Vehicle (US)](#8-user-journey-1--search-by-vehicle-us)
9. [User Journey 2 — Search by Tire Size](#9-user-journey-2--search-by-tire-size)
10. [User Journey 3 — Search by License Plate](#10-user-journey-3--search-by-license-plate)
11. [User Journey 4 — Search by Item / Part Number](#11-user-journey-4--search-by-item--part-number)
12. [User Journey 5 — Battery Purchase (Canada)](#12-user-journey-5--battery-purchase-canada)
13. [User Journey 6 — Compare Tires and Purchase](#13-user-journey-6--compare-tires-and-purchase)
14. [User Journey 7 — Returning Customer (Saved Warehouse)](#14-user-journey-7--returning-customer-saved-warehouse)
15. [User Journey 8 — Admin Manages Promotions](#15-user-journey-8--admin-manages-promotions)
16. [User Journey 9 — Warehouse Selection Flow](#16-user-journey-9--warehouse-selection-flow)
17. [Cross-Cutting Experience Concerns](#17-cross-cutting-experience-concerns)
18. [Friction Points and Technical Impact on UX](#18-friction-points-and-technical-impact-on-ux)

---

## 1. Persona Overview

| Persona | Region | Entry Point | Primary Goal | Technical Touchpoints |
|---------|--------|------------|-------------|----------------------|
| US Vehicle Owner | en-US | Vehicle search / LP search | Buy tires for their car | Vehicle SP, Fitment validation, Mitchell1, Apigee |
| Canadian Member (English) | en-CA | Vehicle or size search | Buy tires, battery (CA only) | All search SPs, tire levy, promotions |
| Quebec Member (French) | fr-CA | French-language vehicle search | Buy in French | Localized SPs, fr-CA CMS content, FR promo copy |
| Battery Shopper (CA) | en-CA / fr-CA | Battery vehicle search | Buy car battery | Battery SPs, `is_sold_online` filter, battery promotions |
| QR Scan Technician | en-CA / fr-CA | QR code deep-link | View battery info for install | `source=qr` path, bypasses `is_sold_online` filter |
| Promotions Admin | en-US | `/Admin/Login` | Create / update tire promotions | `LoginController`, `AdminPromotionsController`, `SP_SavePromotion` |

---

## 2. Persona 1 — US Costco Member (Vehicle Owner)

### Profile

| Attribute | Detail |
|-----------|--------|
| **Name** | Alex (representative) |
| **Region** | United States |
| **Culture** | `en-US` |
| **Costco Status** | Active member |
| **Device** | Desktop (primary), mobile (secondary) |
| **Motivation** | Replace worn tires, seasonal swap, new vehicle tires |
| **Tech Savviness** | Moderate — comfortable with e-commerce but not automotive specs |
| **Pain Points** | Doesn't know tire sizes; concerned about fitment; wants clear pricing |

### Goals

1. Find tires guaranteed to fit their specific vehicle (Year/Make/Model/Options)
2. See transparent pricing including all fees and estimated delivery
3. Understand brand and warranty differences (Michelin vs. Bridgestone vs. BFGoodrich)
4. Know if a promotion applies before committing
5. Complete purchase via Costco cart with minimal friction

### Technical Profile

- Arrives with `tiresLocationCookie` set (previously visited) OR needs warehouse selection
- Browses in `en-US` culture — battery domain not accessible
- May have `mSign` cookie if signed in to Costco — enables member pricing display
- `MichelinPaidId` cookie may be set from Michelin ad campaigns

### Frustration Triggers

- Page redirects to Home if warehouse cookie is absent (no warehouse = no results)
- Winter tire results hidden at certain warehouses without explanation
- Out-of-stock tires shown (grayed out) without back-in-stock date clarity
- Fitment warnings shown but tires not clearly marked as incompatible

---

## 3. Persona 2 — Canadian Member (English)

### Profile

| Attribute | Detail |
|-----------|--------|
| **Name** | Sarah (representative) |
| **Region** | Canada (English) |
| **Culture** | `en-CA` |
| **Language Cookie** | `COSTCO_SELECTED_LANGUAGE = en-ca` |
| **Device** | Desktop (primary) |
| **Motivation** | End-of-season winter tire purchase; battery replacement |
| **Pain Points** | Wants to see both tires AND batteries; wants CA pricing; confused by tire levy |

### Goals

1. Buy winter tires before snowfall — searching by vehicle
2. Understand CA-specific promotions (Michelin, Bridgestone, BFGoodrich, Uniroyal, Pirelli)
3. Understand tire levy applied at Ontario/Quebec warehouses
4. Access battery section for secondary vehicle
5. Compare 2–3 tire options side-by-side

### Technical Profile

- `langId = -24` in cart redirect URLs
- Subject to `TireLevyPerTirePrice` for selected warehouse province
- Ontario: higher levy (`TireLevyPerTirePriceForOntarioLocation`)
- Winter tire warning shown (not hidden) at most CA warehouses
- `RimSizeConversiontoCM` applied — shows rim size in both inches and cm
- CMS content sourced from `en-CA` ContentStack bucket

### Frustration Triggers

- Tire levy added at checkout step (not always visible on search results page)
- Winter tire results vary by warehouse; moving warehouse resets results
- CA promotions may use different start dates than US — phrased differently

---

## 4. Persona 3 — Quebec Member (French-Speaking)

### Profile

| Attribute | Detail |
|-----------|--------|
| **Name** | Luc (representative) |
| **Region** | Canada (Quebec) |
| **Culture** | `fr-CA` |
| **Language Cookie** | `COSTCO_SELECTED_LANGUAGE = fr-ca` |
| **Device** | Mobile (primary) |
| **Motivation** | Legal requirement in Quebec — winter tires mandatory by December 1 |
| **Pain Points** | All content must be in French; English fallback is unacceptable |

### Goals

1. Browse and purchase in French throughout the entire flow
2. See French warranty text (e.g., "Garantie de la bande de roulement de 80 000 Km")
3. See French promotion copy ("Achetez un ensemble de 4 pneus...")
4. Complete checkout via French Costco cart (`CostcoCAURL_M`, `langId = -25`)
5. Understand snowflake / three-peak mountain tire requirements for Quebec

### Technical Profile

- `Thread.CurrentThread.CurrentCulture = fr-CA` set in `BaseController.BeginExecuteCore`
- Warranty formatted via `Warranty_fr_CA = "Garantie de la bande de roulement de"` + km value
- `langId = -25` appended to all cart redirect URLs
- `CostcoCAURL_M` (French mobile cart) used for Add-to-Cart redirect
- French legal notice fetched from ContentStack (`fr-CA` content type) via `SetLegalNotice_Ca`
- `ShowSpecificTiresForSnowflake_FrCa` config key controls Quebec snowflake filter
- `SEASON_HIVER = "Hiver"` used for winter tire labeling in French context
- Quebec warehouse in `PriceForSecificQuebecLocationsWarehosueId_CA` list — Quebec levy applied

### Frustration Triggers

- Any English content appearing in the French UI (CMS fallback to English)
- Snowflake filter not surfaced prominently (Quebec legal requirement creates urgency)
- French promotion dates presented in English format (MM/DD/YYYY vs DD/MM/YYYY)

---

## 5. Persona 4 — Costco Battery Shopper (Canada)

### Profile

| Attribute | Detail |
|-----------|--------|
| **Name** | David (representative) |
| **Region** | Canada (English or French) |
| **Domain** | Battery (`/Battery/`) |
| **Device** | Mobile (primary) |
| **Motivation** | Dead car battery; needs urgent replacement at Costco warehouse |
| **Pain Points** | Not a car expert; doesn't know battery specs; in a hurry |

### Goals

1. Find a battery compatible with their specific vehicle quickly
2. Confirm the battery is available online (sold online)
3. Understand price including any active promotions
4. Add to cart and pick up same day at warehouse

### Technical Profile

- `BaseBatteryController.BeginExecuteCore` enforces Canada-only: US requests redirected to `/`
- `is_sold_online` filter removes batteries not available for online purchase
- Battery category dropdown drives SP parameter `@batteryCategory` — filters to automotive
- Battery vehicle dropdowns: Year → Make → Model → Engine → Type (5-level hierarchy vs. 4 for tires)
- Member pricing: `SetSignInMembershipInfo` reads `mSign` cookie; signed-in members see member price
- Heavy reliance on `SP_GetBatteryByVehicle` returning limited resultset (batteries, not tires)

### Frustration Triggers

- Battery domain not accessible in US (no US expansion of battery program)
- If `is_sold_online = false` → battery disappears without explanation (online-only filter)
- Member pricing only visible when `mSign` cookie is set — non-members see only regular price

---

## 6. Persona 5 — Mobile / QR Scan Technician

### Profile

| Attribute | Detail |
|-----------|--------|
| **Name** | Marco (representative) |
| **Role** | Costco warehouse technician / automotive installer |
| **Region** | Canada |
| **Device** | Mobile (mandatory — QR code scan) |
| **Motivation** | Look up battery specs and details for a vehicle being serviced |
| **Entry Point** | Scans QR code on physical product or vehicle → direct deep-link URL |

### Goals

1. Quickly view battery specs for a vehicle being serviced
2. See ALL batteries regardless of `is_sold_online` status (internal use)
3. Access battery details without going through full shopping flow
4. No cart interaction needed — informational lookup only

### Technical Profile

- URL contains `source=qr` query param
- `Battery.SearchResultsByVehicleController.RenderPartialView` checks for `source=qr`
- When detected → **bypasses `is_sold_online` filter** — all batteries shown
- No `AddToCart` button rendered for QR path (informational mode)
- Mobile-first layout assumed

### Frustration Triggers

- QR link expiring (URL contains warehouse ID — if technician is at different warehouse, link fails)
- Deep-link construction requires correct `whs`, `Year`, `Make`, `Model`, `Engine` params in URL
- If warehouse cookie is different from URL param → potential warehouse mismatch

---

## 7. Persona 6 — Promotions Admin

### Profile

| Attribute | Detail |
|-----------|--------|
| **Name** | Internal Marketing Team |
| **Access** | `/Admin/Login` |
| **Credentials** | `admin` / `admin` (hardcoded — see Security Note below) |
| **Device** | Desktop |
| **Motivation** | Create time-limited supplier promotions (cash card rebates, discount off) |
| **Usage Frequency** | Seasonal — major promotions (spring, fall, holiday) |

### Goals

1. Create new promotion for a supplier (Michelin, Bridgestone, BFGoodrich, Pirelli) for US or CA
2. Set cash card vs. discount type, activation dates, per-tire discount
3. Deactivate expired promotions
4. View existing promotions for a region

### Technical Profile

- `LoginController.LoginAdmin` — hardcoded credential check; no session token issued; no lockout
- After login → `RedirectToAction("Index", "Admin/AdminHome")`
- `AdminPromotionsController.SavePromotions` → `SP_SavePromotion` DB write
- CA promotions: `lang` auto-set to `fr-CA` when `country_code = CA`
- US promotions: `lang = en-US`
- `GetPromotions` currently hardcodes `country_code = "USA"` and `promotion_id = 10` (fetch limitation)

### Security Note

> Admin credentials are hardcoded (`admin`/`admin`). No MFA, no lockout, no hashed passwords. Access to `/Admin/` must be restricted via network/firewall/IP allowlist outside of the application layer. This is a known technical debt item.

### Frustration Triggers

- No confirmation message on save (returns only `Json(result)`)
- `GetPromotions` hardcodes USA + ID=10 — cannot query by filter from UI
- CA country always saves `fr-CA` lang — no way to create separate `en-CA` promotion in admin
- No way to preview live promotion before activating

---

## 8. User Journey 1 — Search by Vehicle (US)

**Persona:** Persona 1 (Alex, US Member)  
**Entry:** Direct URL or Costco.com tire portal link  
**Goal:** Find and purchase a set of 4 tires for a 2020 Toyota Camry

### Journey Map

```
[1] ARRIVE — costco.com/tires
    │
    ├─ Has tiresLocationCookie?
    │   YES ─────────────────────┐
    │   NO → [2] Warehouse       │
    │         Selection Flow     │
    │         (see Journey 9)    │
    └────────────────────────────┘
                 │
    [3] HOME PAGE — HomeController.Index()
         ├─ CMS banner images loaded (ContentStack)
         ├─ Legal notice (CA only — not shown for en-US)
         └─ Vehicle search tab selected by default
                 │
    [4] SELECT YEAR
         ├─ SearchFiltersController.GetYearList()
         ├─ SP_GetYear → returns last N years (NoOfYears config)
         └─ HttpRuntime.Cache: YearList_{culture}
                 │
    [5] SELECT MAKE (Ajax call on Year change)
         ├─ SearchFiltersController.GetMakeList(year)
         └─ SP_GetMakeByYear → filtered to year selection
                 │
    [6] SELECT MODEL (Ajax call on Make change)
         ├─ SearchFiltersController.GetModelList(year, make)
         └─ SP_GetModelByYearMake
                 │
    [7] SELECT OPTIONS (Ajax call on Model change)
         ├─ SearchFiltersController.GetOptionsList(year, make, model)
         └─ SP_GetOptionsByYearMakeModel → trim levels
                 │
    [8] CLICK "SEARCH" → SearchResultsByVehicleController.Index()
         ├─ BeginExecuteCore: warehouse validated, config loaded
         ├─ Session: QueryString, Search objects written
         └─ View rendered with empty results state (AJAX load)
                 │
    [9] AJAX PARTIAL RENDER — RenderPartialView()
         ├─ Redis check: sbvr_{hash} ?
         │   HIT: deserialize cached results
         │   MISS: SP_GetSearchResultsByVehicle_Redesign_V8
         ├─ 5-pass processing pipeline:
         │   Pass 1: GetVehicleModel (display text)
         │   Pass 2: GetAllPromotions (SP_GetPromotionsList)
         │   Pass 3: SetConfigurablePromotions (apply promos)
         │   Pass 4: Fitment validation (SP_ValidateVehicle)
         │   Pass 5: EV filter (if selected)
         ├─ SetDeliveryInfo: delivery date calculation per tire
         ├─ SetWarranty: warranty text formatting
         ├─ SetAdditionalFeeInfo: state fee injection (US)
         ├─ HideWinterTiresForSpecificWarehouses (if applicable)
         ├─ RemoveItemsForBlackListVendor
         └─ Returns partial HTML with tire tiles + filter sidebar
                 │
    [10] BROWSE RESULTS
         ├─ Sees: tire name, brand, price, delivery estimate, promo badge
         ├─ Sees: fitment confirmation (✓ Fits your 2020 Camry)
         ├─ Sees: run-flat indicator, EV badge (if applicable)
         └─ Winter tires: hidden or warning, per warehouse config
                 │
    [11] APPLY FILTERS (optional)
         ├─ Brand filter: AJAX → BrandFilter action
         ├─ Season filter: AJAX → SeasonFilter action
         ├─ Price range: AJAX → PriceFilter action
         └─ Each filter: re-runs RenderPartialView with updated params
                 │
    [12] CLICK TIRE TILE → ProductController.Index()
         ├─ Redis: product_{tireId}_{culture}_{whs} ?
         │   HIT: cached TireInfo
         │   MISS: SP_GetTireDetails_Redesign_V8 (9-table DataSet)
         ├─ ValidateWarehouseBeforeAddtoCart: live Apigee call
         ├─ Features, Benefits, Warranty, Videos, Images loaded
         └─ PDP rendered with full tire detail
                 │
    [13] CLICK "ADD TO CART"
         ├─ POST /Product/AddToCart
         ├─ GetAddToCartURLAndRedirect:
         │   Build URL: CostcoUSURL + itemParams + langId=-1
         │   Member pricing: mSign check → member price token appended
         ├─ TPMS cookie set: CookieTireDetailsForTPMS
         └─ REDIRECT → Costco.com cart
                 │
    [14] CART — Outside platform scope
```

### System State After Journey

- `tiresLocationCookie` set (if created)
- `CookieForSearchCriteria` set with last search
- `Session["Search"]` contains last `ISearch` object
- Redis: search results cached (TTL per config)

---

## 9. User Journey 2 — Search by Tire Size

**Persona:** Persona 1 or 2 (knows their tire size from sidewall)  
**Entry:** "Shop by Tire Size" tab on home page or direct URL  
**Goal:** Replace tires using the size printed on their current tires (e.g., 235/55R18)

### Journey Map

```
[1] SELECT "SHOP BY SIZE" TAB
    └─ Populates Width dropdown via GetWidthList() → SP_GetWidth
                │
[2] SELECT WIDTH → Aspect Ratio dropdown populates
    └─ GetAspectRatioList(width) → SP_GetAspectRatioByWidth
                │
[3] SELECT ASPECT RATIO → Rim Size dropdown populates
    └─ GetRimSizeList(width, aspect) → SP_GetRimSizeByWidthAspect
                │
[4] CLICK "SEARCH" → SearchResultsBySizeController
    ├─ BeginExecuteCore
    ├─ Redis check: {width}_{aspect}_{rim}_{culture}_{whs}
    │   HIT: cached results
    │   MISS: SP_GetSearchResultsBySize_Redesign_V8
    ├─ SetFilterCounts:
    │   - Iterates all tires, computes per-filter counts
    │   - Stores in Redis with filter key suffixes
    ├─ GetSearchResultModel: builds full display model
    └─ Returns results with filter sidebar
                │
[5] OPTIONAL: SIZE SELECTION HUB
    └─ SizeSelectionController.RenderPartialViewFromTireSizeSelection
        ├─ Fetches CMS HTML block (ContentStack or Kentico)
        ├─ HtmlAgilityPack parses <a href> links
        ├─ Warehouse ID substituted into href placeholders
        └─ User clicks tile → routes to search by size with params
                │
[6] BROWSE, FILTER, COMPARE, ADD TO CART
    └─ Same as Journey 1 from step [10] onward
```

### Differentiator vs. Journey 1

- No vehicle fitment validation — size match only
- `SetEquipmentTiresDetails` run to flag standard equipment tires from the size
- User must self-verify that size matches vehicle fitment (UTQG, load, speed)
- Compare and PDP flows identical to Journey 1

---

## 10. User Journey 3 — Search by License Plate

**Persona:** Persona 1 (US Member — doesn't know their tire size or year/make/model)  
**Entry:** "Search by License Plate" tab  
**Goal:** Let the system identify their vehicle automatically

### Journey Map

```
[1] SELECT "LICENSE PLATE" TAB
    └─ Form: State dropdown + License Plate text field
                │
[2] ENTER STATE + PLATE → SearchResultsByLicensePlateController
                │
[3] VALIDATE: Is token cached in Redis?
    ├─ Redis key: LicAccessToken present?
    │   YES: use cached token
    │   NO: Mitchell1Integration.InitializeToken()
    │       HTTP POST (Digest auth) → Mitchell1 auth endpoint
    │       Store: LicAccessToken, LicRefreshToken, LicProxyId in Redis
                │
[4] LICENSE PLATE → VIN LOOKUP
    └─ Mitchell1Integration.GetVINFromLicensePlate(state, plate, token, proxyId)
        HTTP GET → Mitchell1 LP→VIN endpoint
        Returns: VIN string
                │
[5] VIN → VEHICLE RESOLUTION
    └─ Mitchell1Integration.GetVehicleFromVIN(vin, token, proxyId)
        HTTP GET → Mitchell1 VIN→Vehicle endpoint
        Returns: Year, Make, Model, Trim
                │
[6] TOKEN EXPIRY HANDLING (if any)
    ├─ 401 response → Mitchell1Integration.RefreshToken(refreshToken)
    ├─ Retry step [4] or [5] once
    └─ Repeated failure → log to App Insights → redirect with lpError=true
                │
[7] SUCCESS → REDIRECT
    └─ Constructs URL:
        /SearchResultsByVehicle?Year={y}&Make={m}&Model={mod}&Options={opt}&whs={whs}
        Continues as Journey 1 from step [8] (AJAX render)
                │
[8] FALLBACK (lpError=true)
    └─ Vehicle search page loads with error message
       User manually selects Year/Make/Model/Options
```

### Edge Cases

- **Rental / Fleet vehicles:** VIN resolves to commercial vehicle — no fitment data → shows all tires for size
- **Very new plates:** Not yet in Mitchell1 database — fallback to manual
- **Canadian plates:** Mitchell1 LP API US-only; Canadian users cannot use license plate search (no Canadian LP endpoint configured)

---

## 11. User Journey 4 — Search by Item / Part Number

**Persona:** Persona 1 or 2 (has a specific SKU from a previous receipt, recommendation, or research)  
**Entry:** `/SearchResultsByItemOrPart?ItemNo={sku}` or `?PartNo={partno}`  
**Goal:** Re-order a specific known tire or check a recommended part number

### Journey Map

```
[1] ARRIVE WITH ITEM/PART NUMBER (URL param)
    ├─ SearchBy=Item → lookup by SkuNumber
    └─ SearchBy=Part → lookup by SupplierNumber
                │
[2] VENDOR INTEGRATION CHECK
    ├─ VendorIntegrationName key setting set?
    │   YES: Integration.VendorIntegration.GetTireData(sku, warehouse)
    │        Returns: live supplier pricing + availability
    │        Merged with DB data
    │   NO: Standard DB path
                │
[3] STANDARD DB PATH
    ├─ SearchBy=Item → SP_GetTireDetailsByItemNumber
    └─ SearchBy=Part → SP_GetTireDetailsByPartNumber
                │
[4] RESULT COUNT CHECK
    ├─ 0 results → "No tires found" page
    ├─ 1 result  → Auto-redirect to PDP (ProductController.Index)
    └─ 2+ results → Show search results grid
                │
[5] RESULTS / PDP
    └─ Standard PDP or results flow (same as Journey 1 from step [12])
```

### Key Difference

- No vehicle fitment validation run — user intent is to buy a specific part, not to fit a vehicle
- `AddToCart` sets `Qty = 4` default (set of four) unless user modifies

---

## 12. User Journey 5 — Battery Purchase (Canada)

**Persona:** Persona 4 (David, Canadian battery shopper)  
**Entry:** `/Battery` or `/Battery/Index`  
**Goal:** Find compatible car battery, confirm online availability, add to cart

### Journey Map

```
[1] ARRIVE — /Battery/
    ├─ BaseBatteryController.BeginExecuteCore
    ├─ en-US culture? → REDIRECT to "/"  (battery Canada-only)
    ├─ Warehouse cookie: resolved same as tire flow
    ├─ SetConfigurablePromotions: battery promotions loaded
    └─ FetchImages: CMS battery banner (ContentStack or Kentico)
                │
[2] HOME PAGE — Battery banner CMS image displayed
    └─ "Shop by Vehicle" form shown
                │
[3] SELECT YEAR → MAKE → MODEL → ENGINE → TYPE
    ├─ 5-level dropdown hierarchy (vs. 4 for tires)
    ├─ SP_GetBatteryMakeByYear(@Year, @BatteryCategory)
    ├─ SP_GetBatteryModelByYearMake
    ├─ SP_GetBatteryEngineByYearMakeModel
    └─ SP_GetBatteryTypeByYearMakeModelEngine
                │
[4] CLICK "SEARCH" → Battery.SearchResultsByVehicleController
    └─ AJAX RenderPartialView
        ├─ SP_GetBatteryByVehicle(@Year, @Make, @Model, @Engine, @Type, @BatteryCategory)
        ├─ is_sold_online filter: removes batteries where is_sold_online = false
        │   (QR path: skipped, see Persona 5 journey)
        ├─ SetConfigurablePromotions: apply battery promos to results
        └─ Returns battery tile grid
                │
[5] CLICK BATTERY TILE → Battery.ProductController.Index
    ├─ SP_GetBatteryDetails(@BatteryId, @Culture, @WarehouseId)
    ├─ SetSignInMembershipInfo:
    │   mSign cookie == IsCookiValuehasTrue?
    │   YES: IsMemberSignedIn = true → member price displayed
    │   NO: regular price only
    └─ PDP rendered: specs, price, availability, promotions
                │
[6] CLICK "ADD TO CART"
    ├─ POST /Battery/Product/AddToCart
    ├─ GetAddToCartURLAndRedirect:
    │   Culture = en-CA → CostcoCAURL + langId=-24
    │   Culture = fr-CA → CostcoCAURL_M + langId=-25
    └─ REDIRECT → Costco Canada cart
```

### Key Differences from Tire Journey

| Aspect | Tire (Journey 1) | Battery (Journey 5) |
|--------|-----------------|---------------------|
| US access | Yes | **No — redirected to //** |
| Dropdown levels | 4 (Y/M/Mo/Opt) | **5 (Y/M/Mo/Eng/Type)** |
| `is_sold_online` filter | No such filter | **Yes — online-only default** |
| Fitment validation | Vehicle SP | N/A — category match |
| Promotions | Tier 1: web.config brand promos | Battery-specific promotions |
| Compare feature | Yes | **No Compare feature** |
| TPMS | Yes (tire-specific) | **No TPMS** |
| Staggered | Yes (F+R pairs) | **No staggered** |

---

## 13. User Journey 6 — Compare Tires and Purchase

**Persona:** Persona 1 or 2 (methodical shopper, wants to evaluate options)  
**Entry:** Multiple tire tiles with "Add to Compare" checkbox selected  
**Goal:** Compare 2–4 tires side by side on key spec, warranty, and price attributes

### Journey Map

```
[1] ON SEARCH RESULTS PAGE
    ├─ User checks "Compare" checkbox on tile 1
    ├─ User checks "Compare" checkbox on tile 2 (min)
    ├─ Optional: adds tile 3, tile 4 (max 4)
    └─ Selection order recorded client-side → CompareSearchID=id1,id2,id3
                │
[2] CLICK "COMPARE" → CompareController.Index()
    ├─ Parses CompareSearchID → List of TireIDs
    ├─ BeginExecuteCore
    └─ RenderPartialViewForCompare(tireIds, ...)
                │
[3] FETCH COMPARE DATA
    ├─ Redis: CompareKey_{ids}_{culture}_{whs} ?
    │   HIT: cached compare set
    │   MISS: SP_GetTireDetailsForCompare_Redesign_V8(@TireIDs)
    │        Returns full tire data for all IDs in one SP call
    └─ Result cached in Redis (CompareRedisCacheTimeOut TTL)
                │
[4] STAGGERED HANDLING
    ├─ ClubbedId present? → Front+Rear pair grouping
    ├─ Front tire: shows combined ClubbedTirePrice
    └─ Rear tire: displayed adjacent, price hidden (part of pair)
                │
[5] SEQUENCE RESTORATION
    └─ SP returns tires in DB order → re-mapped to user's selection order
       User always sees tires in the order they were selected
                │
[6] POST-PROCESSING
    ├─ SetDeliveryInfo (per-tire delivery estimates)
    ├─ SetWarranty (warranty text)
    ├─ SetConfigurablePromotions (promo badges)
    ├─ IsComparePage = true → compare-specific template rendering
    └─ Returns: side-by-side comparison table HTML
                │
[7] COMPARE PAGE VIEW
    ├─ Columns: Brand/Model | Price | Delivery | Warranty | Features | Benefits
    ├─ Promo badge shown if HasPromotion = 1
    ├─ Winter tire warning shown if IsShowWinterTireWarning = true
    └─ Staggered pair shown as grouped columns
                │
[8] SELECT PREFERRED TIRE → CLICK "ADD TO CART"
    └─ Same Add-to-Cart flow as Journey 1 step [13]
```

### Compare Popup (From Search Results)

- `GetTireDataForComparePopUp?TireID={id}` — lightweight summary popup before adding to compare
- User sees abbreviated specs: price, size, warranty, season type before deciding to compare

---

## 14. User Journey 7 — Returning Customer (Saved Warehouse)

**Persona:** Persona 1 or 2 (has visited before, warehouse cookie set)  
**Entry:** Direct URL or bookmarked search  
**Goal:** Quick repeat search or checking a different tire for same vehicle

### Journey Map

```
[1] ARRIVE — tiresLocationCookie present
    ├─ BaseController.BeginExecuteCore
    ├─ Reads tiresLocationCookie → warehouseId resolved immediately
    ├─ Skips warehouse selection step
    └─ Proceeds directly to search or last page
                │
[2] SESSION STATE CHECK
    ├─ Session["Search"] populated? (within same session)
    │   YES: results page pre-populated with previous search params
    │   NO: fresh search page
                │
[3] COOKIE-DRIVEN REPOPULATION
    ├─ CookieForSearchCriteria: last search criteria re-loaded
    ├─ Year/Make/Model/Options pre-selected in dropdowns
    └─ User clicks Search → fresh results for same vehicle at same warehouse
                │
[4] WAREHOUSE CHANGE (optional)
    ├─ "Change Warehouse" link → CostcoUS/CA ChangeWareHouse URL
    ├─ External Costco.com warehouse selector
    └─ New warehouseId returned → tiresLocationCookie updated
       ├─ Winter tire visibility may change (per warehouse config)
       ├─ Additional fees may change (per warehouse state)
       └─ Blacklisted vendors may differ (per warehouse config)
```

### Technical Note

When a returning customer changes their warehouse, several per-warehouse configurations reload:
- `SPECIFICWAREHOUSELIST` (winter hide)
- `WinterTireWarningWarehouses` (winter warn)
- `BlackListVendorForWarehouses` (supplier blacklist)
- `SpecificWarehouseAdditionalFees` (state fee)
- `OutOfStockForAllVendors` (full out-of-stock flag for warehouse)

---

## 15. User Journey 8 — Admin Manages Promotions

**Persona:** Persona 6 (Internal Marketing Team)  
**Entry:** `/Admin/Login`  
**Goal:** Create a new 6-week Michelin "Get $70 off a set of 4" US promotion

### Journey Map

```
[1] NAVIGATE TO /Admin/Login
    └─ Renders Login.cshtml form (username + password fields)
                │
[2] SUBMIT LOGIN — POST /Admin/LoginAdmin
    ├─ Credential check: username == "admin" && password == "admin"
    ├─ MATCH: RedirectToAction("Index", "Admin/AdminHome")
    └─ NO MATCH: Re-renders login (no error message shown)
                │
[3] NAVIGATE TO /Admin/Promotions
    └─ AdminPromotionsController.Index()
        ├─ Supplier dropdown: [Michelin=1, Bridgestone=2, BFGoodrich=8, Pirelli=12]
        ├─ Country: [USA, CA]
        ├─ Is Cash Card: [-1 Select, 0 No, 1 Yes]
        ├─ Is Active: [-1 Select, 0 No, 1 Yes]
        └─ Set Qty: [default = 4]
                │
[4] FILL FORM
    ├─ Supplier: Michelin (1)
    ├─ Country: USA
    ├─ Discount Price: 70.00
    ├─ Is Cash Card: No (0) — direct discount
    ├─ Set of Qty: 4
    ├─ Valid From: 2026-04-01
    ├─ Valid To: 2026-05-15
    └─ Is Active: Yes (1)
                │
[5] SUBMIT — POST /Admin/SavePromotions
    ├─ country_code = "USA" → lang = "en-US" set
    ├─ PromotionsBusinessObject.SavePromotions(promotionsInfo)
    │   └─ SP_SavePromotion(@supplier_id=1, @discount_price=70, ...)
    └─ Returns Json({ success: true/false })
                │
[6] PROMOTION ACTIVE
    ├─ Next search render: GetAllPromotions() loads new promotion
    ├─ CommonSearchLogic.SetConfigurablePromotions applies to all Michelin tires
    ├─ HasPromotion = 1 on relevant tires
    └─ Promo badge shown on search results + PDP
```

### Known Limitations

| Issue | Impact |
|-------|--------|
| No preview before save | Admin cannot see how promo appears on site |
| CA always saves fr-CA | Cannot create separate en-CA promotion |
| GetPromotions hardcoded | Cannot filter promotions by supplier/date from admin UI |
| No deactivation shortcut | Must edit records directly in DB to mass-deactivate |
| Hardcoded credentials | Security — must be restricted via network controls |

---

## 16. User Journey 9 — Warehouse Selection Flow

**Persona:** Any new visitor (no `tiresLocationCookie`)  
**Entry:** Any search page redirect or home page  
**Goal:** Set warehouse proximity to see accurate availability and pricing

### Journey Map

```
[1] ARRIVE — No tiresLocationCookie
    ├─ BaseController.BeginExecuteCore
    ├─ whs query param present?
    │   YES: use as warehouseId directly → skip to [6]
    └─ NO: redirect to Home page
                │
[2] HOME PAGE — Warehouse prompt visible
    ├─ User enters ZIP code or postal code
    │   OR user allows browser geolocation
    └─ OR user clicks "Find a Warehouse" → SearchWarehouseController
                │
[3a] ZIP / POSTAL CODE PATH
    ├─ SearchWarehouseController.IndexWithZipCode(zip)
    ├─ BingMapsIntegration.GetLocationFromZip(zip)
    │   HTTP GET → Bing Maps Locations API → lat/long
    ├─ SP_GetWarehousesByProximity(lat, long)
    └─ Returns sorted list of nearby warehouses
                │
[3b] GEOLOCATION PATH
    ├─ Browser GPS → lat/long
    ├─ SearchWarehouseController.GetWarehouseDataByLatLong(lat, lng)
    └─ SP_GetWarehousesByProximity(lat, long)
                │
[3c] DIRECT WAREHOUSE ID (if known)
    └─ SearchWarehouseController.GetWarehouseDataById(whs)
                │
[4] SELECT WAREHOUSE FROM LIST
    ├─ User clicks on warehouse
    ├─ tiresLocationCookie set: value = warehouseId (365-day expiry)
    └─ TiresZipCode_us / TiresZipCode_ca cookie set with searched ZIP
                │
[5] WAREHOUSE VALIDATED
    ├─ Is warehouseId in DEACTIVATED_WAREHOUSE_CODES?
    │   YES: redirect back to Home with "warehouse unavailable" message
    └─ NO: proceed to search
                │
[6] WAREHOUSE-SPECIFIC EXPERIENCE
    ├─ Winter tires: hidden or warned per SPECIFICWAREHOUSELIST
    ├─ Additional fees: state-specific fee per SecificWarehouseAdditionalFees
    ├─ Blacklisted suppliers: removed per BlackListVendorForWarehouses
    ├─ Out-of-stock: warehouse-level flag per OutOfStockForAllVendors
    └─ Live warehouse info: GetWarehouseDataFromAPI → Apigee call
```

### Async Warehouse Loading Modes

| Mode | Behavior | Config Key |
|------|---------|-----------|
| Standard | Synchronous Apigee call, page waits | Default |
| Async | JS polling — page renders before warehouse data arrives | `AsyncWarehouseAPI = true` |
| Offline | Last cached data used, no live call | `OfflineWarehouseLoading = true` |

---

## 17. Cross-Cutting Experience Concerns

### 17.1 Language Switching

Any page — user clicks language selector:
```
HomeController.LanguageSelector?lang={code}
  → Sets COSTCO_SELECTED_LANGUAGE cookie
  → Redirects to home page
  → BeginExecuteCore re-runs with new culture
  → All SPs re-called with new culture param
  → All CMS content re-fetched from new culture bucket
  → Cart redirect URLs switch (CostcoUSURL vs CostcoCAURL vs CostcoCAURL_M)
```

### 17.2 Filter Persistence Across Pages

When user navigates from search → PDP → Back to search:
- `CookieForSearchCriteria` restores last search params
- `Session["Search"]` restores ISearch object
- `Session["SearchResults"]` may restore cached results if session still active
- Filter state is NOT persisted in cookie — filters must be re-applied after PDP back-navigation

### 17.3 Back-in-Stock Display

When `IsOutOfStockEquipment = true` AND `Available_Qty = 0` AND `NextAvailableDate ≠ default`:
- `SetTempOutOfStockText` generates a "Back in stock: {date}" message
- Tire still appears in results (not removed) — shown with out-of-stock badge
- No waitlist / notification mechanism — purely informational

### 17.4 TPMS Awareness

TPMS (Tire Pressure Monitoring System) cookie flow:
1. On PDP view → `CookieTireDetailsForTPMS` set with `TireID`, `SkuNumber`, `TPMSIndex`
2. On Add-to-Cart → TPMS index `!= null` → appended to cart URL as param
3. Costco cart uses TPMS param to show TPMS sensor add-on option at checkout
4. Users purchasing tires with TPMS requirement see sensor upsell in Costco cart

### 17.5 Member Pricing Display

```
mSign cookie value == IsCookiValuehasTrue (web.config)?
  YES → IsMemberOnlyInfoFound = true on TireInfo
       → Member-only price shown in tile and PDP
       → Michelin/BFGoodrich SKU-specific member price check also runs
  NO  → Regular price displayed only
```

Note: The `mSign` cookie is set by Costco.com main site — the tire platform reads but never writes it.

---

## 18. Friction Points and Technical Impact on UX

### 18.1 Hard Dependencies That Create User-Visible Failures

| Dependency | Failure Mode | User Experience | Mitigation |
|------------|-------------|----------------|-----------|
| Warehouse cookie | Missing → redirect to Home on every search page | User cannot browse tires | `whs` query param bypasses cookie check |
| Redis | Connection failure → search falls through to SQL every req | Slow results (2–5s) | App Insights logs; Circuit breaker not implemented |
| Mitchell1 API | Timeout / failure | License plate search fails silently → fallback to manual | Retry once on 401; log to AI; redirect |
| Apigee Inventory | Failure on PDP | `ValidateWarehouseBeforeAddtoCart` fails → product page may show unavailable | No fallback — show last known DB availability |
| ContentStack | CMS fetch failure | Banner images missing; size selection page empty | Kentico fallback if `IsContentstackActive = false` |
| Bing Maps | Failure | Warehouse search by ZIP fails | No fallback — error shown to user |

### 18.2 Caching Latency vs. Real-Time Accuracy

| Data | Cache Type | TTL | Stale Risk |
|------|-----------|-----|-----------|
| Search results | Redis | Configurable (default 30–60 min) | Price/availability may be stale |
| PDP details | Redis | Configurable | Promotions applied at write time — won't reflect new promotions until TTL expires |
| Vehicle dropdowns | HttpRuntime.Cache | Until app recycle | New make/model data not visible until cache expires |
| Holiday list | Redis | `HolidayListRedisCacheTimeOut` | Delivery dates may be wrong if holiday added after cache write |
| Config (promos, fees) | Redis | `ConfigurableItemsAndAttributesTimeOut` | Config changes not live until TTL expires |

### 18.3 Filter UX Limitations

- Filters applied via AJAX re-render — each filter change triggers a new partial render with full SP re-execution
- No client-side filtering — every filter change is a server round-trip
- Filter counts pre-computed and stored in Redis during initial search render; not recomputed per filter change
- As a result: when one filter is applied, counts for other filters do not update (counts remain from initial full result set)

### 18.4 Staggered Tire UX Complexity

- Staggered pairs (`ClubbedId`) shown as separate tiles in search results
- `ClubbedTirePrice` shown on front tile only — rear tile tile may confuse users
- Compare page receives compound ID — users cannot compare just the front or just the rear
- No UX explanation of "why am I seeing two sizes for one tire?"

### 18.5 Admin Experience Gaps

| Gap | Impact |
|-----|--------|
| No WYSIWYG promotion preview | Marketing must verify live on site |
| GetPromotions hardcoded to USA/ID=10 | Admin UI cannot filter/search promotions |
| CA promotions always `fr-CA` lang | No separate en-CA promotion language |
| No audit trail | No log of who changed what and when |
| Session not established after login | Browser back after login may re-prompt |

---

*End of User Personas & User Journeys*
