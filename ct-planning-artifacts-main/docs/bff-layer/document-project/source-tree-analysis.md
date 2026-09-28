# Source Tree Analysis — Costco Tires & Batteries

_Generated: 2026-04-13 | Scan Level: Deep_

---

## Repository Type

**Monolith** — single Visual Studio solution (`Costco.sln`) containing 15 projects as class libraries and one ASP.NET MVC web application. No microservices, no separate client/server split.

---

## Solution Structure (Annotated)

```
Tires-Batteries 1/                          ← Solution root
│
├── Costco.sln                              ← Visual Studio solution file
├── CostcoAssemblyInfo.cs                   ← Solution-level assembly metadata
├── Database-Table-Script.sql               ← Full SQL Server schema (tables) — 139 tables
├── Databse-StoredProcedure-script.sql      ← All stored procedure definitions
├── costco_tires_bff_api_summary.md         ← Migration plan: ASP.NET → Java Spring Boot BFF
│
├── Costco/                                 ← ★ ENTRY POINT — ASP.NET MVC 5 Web Application
│   ├── Global.asax.cs                      ← App startup: routes, bundles, thread pool, AI key
│   ├── Web.config                          ← All AppSettings, connection strings, httpModules
│   ├── Web.AzureDev.config                 ← XDT transform: Azure Dev environment
│   ├── Web.AzureQA2.config                 ← XDT transform: Azure QA2 environment
│   ├── Web.AzureStaging.config             ← XDT transform: Azure Staging
│   ├── Web.AzureUAT.config                 ← XDT transform: Azure UAT
│   ├── Web.AzurePreProd.config             ← XDT transform: Azure Pre-Production
│   ├── Web.AzureProduction.config          ← XDT transform: Azure Production
│   ├── ApplicationInsights.config          ← Application Insights SDK configuration
│   │
│   ├── App_Start/                          ← MVC bootstrap
│   │   ├── RouteConfig.cs                  ← MapMvcAttributeRoutes + area registration + default route
│   │   ├── FilterConfig.cs                 ← Global HandleErrorAttribute
│   │   ├── BundleConfig.cs                 ← JS/CSS bundles (jQuery, Bootstrap, app JS)
│   │   ├── BrowserDetection.cs             ← Browser/device detection
│   │   └── SameSiteCookieRewriter.cs       ← SameSite=None cookie fix for Safari 12/13, UC Browser
│   │
│   ├── Controllers/                        ← Tire domain controllers
│   │   ├── BaseController.cs               ← ★ All tire controllers extend this; culture, warehouse, cookies
│   │   ├── HomeController.cs               ← Homepage, language selector, banner loading
│   │   ├── ProductController.cs            ← Tire PDP (Product Detail Page)
│   │   ├── SearchResultsByVehicleController.cs   ← Search by Year/Make/Model/Option
│   │   ├── SearchResultsBySizeController.cs      ← Search by Width/Aspect/Rim
│   │   ├── SearchResultsByCustomTireSizeController.cs ← Custom/non-standard sizes
│   │   ├── SearchResultsByItemOrPartController.cs     ← Search by item no / part no
│   │   ├── SearchResultsByLicensePlateController.cs   ← US license plate → VIN → vehicle
│   │   ├── SearchWarehouseController.cs         ← Warehouse selector (sync)
│   │   ├── SearchWarehouseAsyncController.cs    ← Warehouse selector (async TAP)
│   │   ├── SearchWarehouseOfflineController.cs  ← Warehouse selector (in-store kiosk, DB-only)
│   │   ├── SearchFiltersController.cs           ← Search filter sidebar + dropdown AJAX
│   │   ├── ValidateVehicleController.cs         ← Fitment validation for a SKU
│   │   ├── CompareController.cs                 ← Tire comparison (up to 4 tires)
│   │   ├── SizeSelectionController.cs           ← Tire size selection page
│   │   ├── HeaderController.cs                  ← Site header (ChildAction)
│   │   ├── FooterController.cs                  ← CMS static pages + blog
│   │   ├── BlogController.cs                    ← Blog content (ChildAction)
│   │   ├── VendorConnectionController.cs        ← Vendor connection landing
│   │   ├── MichelinController.cs                ← Michelin brand landing
│   │   ├── BFGoodrich_TiresController.cs        ← BFGoodrich brand landing
│   │   ├── Bridgestone_TiresController.cs       ← Bridgestone brand landing
│   │   ├── Firestone_TiresController.cs         ← Firestone brand landing
│   │   ├── Pirelli_TiresController.cs           ← Pirelli brand landing
│   │   └── Admin/
│   │       ├── AdminHomeController.cs           ← Admin dashboard
│   │       ├── LoginController.cs               ← Admin login (hard-coded creds)
│   │       └── AdminPromotionsController.cs     ← Promotions management
│   │
│   ├── Areas/                              ← Battery domain (MVC Area)
│   │   └── Batteries/
│   │       ├── BatteriesAreaRegistration.cs     ← Area route registration
│   │       ├── Controllers/
│   │       │   ├── BaseBatteryController.cs     ← Battery base; culture, US redirect
│   │       │   ├── HomeController.cs            ← Battery homepage
│   │       │   ├── SearchResultsByVehicleController.cs ← Battery search
│   │       │   ├── ProductController.cs         ← Battery PDP
│   │       │   ├── SearchFiltersBatteryController.cs   ← Battery dropdown AJAX
│   │       │   ├── HeaderController.cs          ← Battery site header
│   │       │   ├── BatteryFooterController.cs   ← Battery CMS pages
│   │       │   └── BatteryBlogController.cs     ← Battery blog
│   │       └── Views/                           ← Battery Razor views (mirrors Costco/Views structure)
│   │
│   ├── Views/                              ← Tire Razor views
│   │   ├── Shared/
│   │   │   ├── _Layout.cshtml              ← Master layout (culture-agnostic)
│   │   │   ├── _Layout_en_us.cshtml        ← US English layout
│   │   │   ├── _Layout_en_fr_ca.cshtml     ← Canada (EN + FR) layout
│   │   │   ├── _AdminLayout.cshtml         ← Admin portal layout
│   │   │   ├── _JSConstants.cshtml         ← Server-injected JS config constants
│   │   │   ├── _PartialProductsGridViewResults.cshtml ← Grid result tiles
│   │   │   ├── _PartialProductsListViewResults.cshtml ← List result rows
│   │   │   ├── _TireDetailBlock.cshtml / _en_us / _en_fr_ca ← Tire detail block partials
│   │   │   ├── _AddToCart_en_us.cshtml / _en_fr_ca         ← Add-to-cart button partials
│   │   │   ├── _WarehouseHeaderSection.cshtml (+ culture variants)
│   │   │   └── Error.cshtml                ← Error page
│   │   ├── Home/                           ← Homepage views + banner partials
│   │   ├── SearchResultsByBase/            ← Shared search results view
│   │   ├── Product/                        ← PDP view
│   │   ├── Compare/                        ← Comparison view
│   │   ├── SearchWarehouse/ (+ Async/ + Offline/)  ← Warehouse selector partials
│   │   ├── SearchFilters/                  ← Filter sidebar views
│   │   ├── ValidateVehicle/                ← Fitment validation views
│   │   ├── SizeSelection/                  ← Size selection views
│   │   ├── Footer/                         ← CMS page views
│   │   ├── Michelin/ Michelin_Tires/ BFGoodrich_Tires/ etc. ← Brand landing views
│   │   └── Admin/                          ← Admin portal views
│   │
│   ├── Content/                            ← CSS, images, fonts
│   ├── Scripts/                            ← Custom JavaScript files
│   └── DomainCultureMapping.xml           ← Maps domain hostnames → culture codes
│
├── ApplicationInterfaces/                  ← All domain interfaces (contracts)
│   ├── ISearch.cs                          ← Search criteria contract
│   ├── ISearchResults.cs                   ← Search results contract
│   ├── ITireInfo.cs                        ← Tire product contract (60+ properties)
│   ├── ITireWarehousePrice.cs              ← Warehouse pricing contract
│   ├── IVehicleOperations.cs               ← Vehicle dropdown contract
│   ├── IURLOperations.cs                   ← Cart URL builder contract
│   ├── IWarehouses.cs / ICustomFitmentOperations.cs / etc.
│   ├── Battery/                            ← Battery-specific interfaces
│   └── Configurations/                     ← Config/settings interfaces
│
├── ApplicationModels/                      ← POCO model/DTO classes
│   ├── TireInfo.cs, SearchResults.cs, QuantityAndCart.cs, etc.
│   ├── Battery/                            ← Battery models
│   ├── Configurations/                     ← Config models
│   ├── Contentstack/                       ← ContentStack response models
│   ├── Warehouse/                          ← Warehouse models
│   └── Admin/                              ← Admin models (PromotionsInfo, etc.)
│
├── ApplicationFactory/                     ← Static object factory (replaces DI container)
│   ├── FactoryCreator.cs                   ← Tire domain factory: GetSearchObject(), GetVehicleOperations(), etc.
│   └── BatteryFactoryCreator.cs            ← Battery domain factory
│
├── BusinessObjects/                        ← Business logic layer
│   ├── Search.cs, SearchResults.cs         ← Core tire search logic
│   ├── TireInfo.cs, TireWarranty.cs, etc.  ← Domain entity implementations
│   ├── Warehouses.cs / WarehousesAsync.cs / WarehousesOffline.cs ← Warehouse strategies
│   ├── VehicleOperations.cs                ← Vehicle dropdown logic (HttpRuntime cached)
│   ├── URLOperations.cs                    ← Cart URL builder
│   ├── ContentstackDataOperations.cs       ← ContentStack CMS operations
│   ├── Contentstack.cs                     ← Blog data
│   ├── ConcreteConverter.cs                ← JSON interface deserializer
│   ├── Admin/                              ← Admin business objects
│   ├── Battery/                            ← Battery business logic
│   ├── Caching/                            ← Redis repository + key builders
│   ├── CommonHelper/                       ← Constants, utilities, Redis helpers
│   │   ├── CommonUtility.cs                ← 150+ route/cookie/culture constants
│   │   ├── DBConstants.cs                  ← DB column name constants
│   │   ├── WebConfigKeys.cs                ← ALL AppSettings access (static properties)
│   │   ├── RedisCacheHelper.cs             ← Redis `ConnectionMultiplexer` singleton
│   │   ├── RedisCacheExtensions.cs         ← `IDatabase` extension methods
│   │   └── DbKeyVaultManager.cs            ← Azure Key Vault secret retrieval singleton
│   ├── Configurations/                     ← Config/promotions/sorting business logic
│   └── Library/                            ← Shared logic
│       ├── Common/                         ← CommonSearchLogic, GenericFunctions, CookieHelper
│       ├── CommonTiresMethods/             ← Ship-to-home, comparison utilities
│       └── Extension/                      ← SearchResultsExtension (LINQ filtering/sorting)
│
├── DataAccess/                             ← Database access layer
│   ├── DataAccessHelper.cs                 ← Core ADO.NET wrapper (ExecuteDataset, etc.)
│   ├── SQLHelper.cs                        ← Generic SQL helper (ExecuteNonQuery, Scalar)
│   ├── SearchData.cs                       ← Tire search SPs
│   ├── TireData.cs                         ← Tire/warehouse SPs
│   ├── VehicleData.cs                      ← Vehicle dropdown SPs
│   ├── CustomFitmentData.cs                ← Custom fitment size SPs
│   ├── ConfigurableItemsAndAttributesData.cs ← UI filter SPs
│   ├── WebConfigKeys.cs                    ← AppSettings access (authoritative source)
│   ├── DbKeyVaultManager.cs                ← Key Vault singleton (authoritative source)
│   ├── Admin/                              ← Admin data access
│   ├── Battery/                            ← Battery data access (SearchData, SearchWidget)
│   └── Configurations/                     ← Config/promotions/sorting data access
│
├── Integration/                            ← External API client adapters
│   ├── Connector/
│   │   ├── HttpBaseClient.cs               ← General REST client (bearer auth, configurable timeout)
│   │   ├── HttpBaseClientAsync.cs          ← Async REST client
│   │   ├── HttpBaseClientSingle.cs         ← Singleton HttpClient variant
│   │   ├── HttpContentstackClient.cs       ← ContentStack-specific client
│   │   └── RestSharpBaseClient.cs          ← Mitchell1 Digest auth client
│   ├── Model/
│   │   ├── ServiceResponse.cs              ← Generic HTTP response wrapper
│   │   └── DigestAuthenticator.cs          ← MD5 Digest auth implementation
│   └── CommonHelper/
│       ├── Constants.cs                    ← ContentStack header constants
│       └── UtilityMethod.cs                ← XML parsing helpers
│
├── ErrorLogging/                           ← Logging utility
│   └── Logger.cs                           ← Static: LogError(ex, class, method), LogInfo(msg)
│                                              Delegates to System.Diagnostics.Trace
│
├── Resources/                              ← Tire localized string resources
│   ├── en-US/                              ← US English strings
│   ├── en-CA/                              ← Canada English strings
│   └── fr-CA/                              ← Canada French strings
│
├── ResourcesBattery/                       ← Battery localized string resources
│   ├── en-US/, en-CA/, fr-CA/
│
├── KenticoRestAPI/                         ← Kentico CMS connector (legacy, replaced by ContentStack)
│   ├── RestAPICall.cs                      ← Tire: Kentico blog, images, pages via Basic auth REST
│   └── BatteryRestAPICall.cs               ← Battery: Kentico data (separate endpoint/credentials)
│
├── KenticoRestApiAuthenticationGenerator/ ← Token generator for Kentico auth
│
├── KenticoAzureStorage/                   ← Azure Blob Storage via Kentico
│
├── CWC_Tires/                              ← SQL Server Database Project (.sqlproj)
│   └── (schema: 139 tables, stored procedures, views)
│
├── Costco.Tests/                           ← Unit test project (MSTest + Moq)
│   └── Utilities/
│       └── HomeControllerTest.cs           ← Commented out; near-zero coverage across solution
│
├── Lib/                                    ← Local DLL references (third-party without NuGet)
├── packages/                               ← NuGet package restore cache
│
├── docs/                                   ← ★ Project documentation (this folder)
│
└── _bmad-output/                          ← BMAD AI workflow outputs
    ├── planning-artifacts/
    └── implementation-artifacts/
```

---

## Entry Points

| Entry Point | File | Purpose |
|---|---|---|
| **Application startup** | `Costco/Global.asax.cs :: Application_Start()` | Registers routes, filters, bundles, configures thread pool and App Insights |
| **HTTP request** | `Costco/Controllers/BaseController.cs` | All tire requests flow through BaseController for culture/warehouse resolution |
| **Battery HTTP request** | `Costco/Areas/Batteries/Controllers/BaseBatteryController.cs` | Battery requests; US users redirected to tire homepage |
| **Admin** | `Costco/Controllers/Admin/LoginController.cs` | Admin portal entry (hardcoded `admin`/`admin`) |
| **Database** | `DataAccess/DataAccessHelper.cs` | All SQL operations go through this class |
| **Object creation** | `ApplicationFactory/FactoryCreator.cs` | All business object instantiation |
| **Configuration** | `DataAccess/WebConfigKeys.cs` | All AppSettings access |
| **Secrets** | `DataAccess/DbKeyVaultManager.cs` | DB connection string from Key Vault |

---

## Critical Cross-Cutting Paths

| Concern | Location |
|---|---|
| Culture resolution | `BaseController.PreferredCulture` → cookie → Accept-Language header |
| Warehouse cookie | `BaseController.WareHouseId` read/set helpers |
| All Redis operations | `BusinessObjects/CommonHelper/RedisCacheHelper.cs` (singleton), `RedisCacheExtensions.cs` |
| CMS toggle | `WebConfigKeys.IsContentstackActive` → routes to either `RestAPICall` or `HttpContentstackClient` |
| DB security | `DbKeyVaultManager` → in-memory cache → Azure Key Vault |
| Error logging | `ErrorLogging/Logger.cs` → `System.Diagnostics.Trace` + auto App Insights |
| Assembly signing | All projects have `.snk` strong-name key files |
| SonarLint | `.sonarlint/costcotires_redesigncsharp.ruleset` applied to all build configs |
