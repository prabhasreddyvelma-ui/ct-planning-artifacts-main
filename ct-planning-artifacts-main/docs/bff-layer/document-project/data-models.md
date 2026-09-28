# Data Models — Costco Tires & Batteries

_Generated: 2026-04-13 | Scan Level: Deep_

---

## Overview

All domain data is stored in **Azure SQL (SQL Server)**. The schema is maintained in the `CWC_Tires` SQL Server Database Project (`.sqlproj`). Data access is exclusively via **stored procedures** using raw ADO.NET. Entity Framework 6 is used only for the `AspNetUsers` admin identity tables.

---

## Domain Groups

### 1. Tire Domain

#### Core Tire Tables

| Table | Purpose | Key Columns |
|---|---|---|
| `MST_TireDetails` | Core tire catalog | `TireID`, `SkuNumber`, `CostcoProductId`, `TireName`, `TireSize`, `Sidewall`, `FRB` (Front/Rear/Both), `SupplierNumber`, `SupplierID`, `IsItemNew`, `DisplayOnWebsite`, `ZeroPressure`, `IsEVCompatible`, `IsEVDedicated`, `STD_OR_OPT`, `LoadIndex`, `SpeedRating`, `IsStaggeredTire`, `ClubbedId`, `StaggeredTirePosition` |
| `MST_Tire_Specs` | Tire specifications | Technical specs per SKU |
| `MST_TireFeatures` | Feature bullets | TextEN, TextFR, SortOrder, SkuNumber |
| `MST_TireFootNotes` | Footnote/disclaimer text | TextEN, TextFR, SkuNumber |
| `MST_TireImages` | Tire images | ImageURL, ImageType, SkuNumber |
| `MST_TireVideos` | Product videos | VideoURL, SkuNumber |
| `MST_Tire_Pdf_Brochure` | PDF brochure links | BrochureURL, SkuNumber |
| `MST_Tire_Warranty` | Warranty data | WarrantyTextEN, WarrantyTextFR, MileageWarranty, MileageWarrantyKM |
| `MST_TireTypes` | Tire type lookup | TireTypeID, TireTypeName, TireTypeNameFR |
| `MST_TireSize_BreakUP` | Parsed tire size components | Width, Aspect, Rim, SizeString |
| `MST_SideWall_Abbreviations` | Sidewall code lookup | Abbreviation, Description |
| `MST_Speed_Symbols` | Speed rating symbols | Symbol, MaxSpeedKPH |
| `MST_Attributes_ToolTips` | UI tooltip text | AttributeCode, TooltipEN, TooltipFR |

#### Inventory & Pricing Tables

| Table | Purpose | Key Columns |
|---|---|---|
| `MST_Tire_WareHouse_Price` | Per-warehouse tire pricing | `SkuNumber`, `WarehouseCode`, `Price`, `Currency`, `IsAvailable`, `Available_Qty`, `SafetyStockLimit`, `NextAvailableDate`, `MaxBuyingQty` |
| `MST_Tire_WareHouse_Inventory` | Inventory levels | `SkuNumber`, `WarehouseCode`, `OnHand`, `OnOrder` |
| `AdditionalFeeDetails` | Additional fees per warehouse | `WarehouseCode`, `FeeAmount`, `FeeType`, `ItemTypeId` |
| `ShipToHomeDetails` | Ship-to-home eligibility | `SkuNumber`, `IsShipToHome`, `ShipCost` |
| `SpecialConsiderations` | Special handling flags | `SkuNumber`, `WarehouseCode`, `Consideration` |

#### Fitment Tables

| Table | Purpose |
|---|---|
| `Tire Fitment` | OE fitment data (year/make/model/option → SkuNumber) |
| `Tire Fitment_Updated_Redesign` | Active redesigned fitment |
| `Tire Fitment_French_Updated_Redesign` | French locale fitment |
| `MST_Wheel_Specs` | Wheel specifications |

#### Promotion Tables

| Table | Purpose |
|---|---|
| `mst_promotions_redesign` | Active promotion header |
| `mst_promotions_detail_redesign` | Promotion details per SKU/vendor |
| `mst_promotions_text_redesign` | Promotion display text (EN/FR) |
| `mst_promotions_header_redesign` | Promotion header banners |

---

### 2. Battery Domain

| Table | Purpose | Key Columns |
|---|---|---|
| `Battery_Details` | Battery catalog | `BatteryID`, `SkuNumber`, `BatteryName`, `Category` (Automotive/Powersport), `is_sold_online` |
| `Battery_Pricing` | Battery pricing per warehouse | `SkuNumber`, `WarehouseCode`, `Price`, `IsAvailable` |
| `Battery_Inventory` | Battery inventory | `SkuNumber`, `WarehouseCode`, `OnHand` |
| `BatteryFitment` | OE battery fitment mapping | Year/Make/Model/Engine → SkuNumber |
| `BatteryFitment_Automotive` | Automotive battery fitment | — |
| `BatteryFitment_Powersport` | Powersport battery fitment | — |
| `BT_BatteryTypes` | Battery type lookup | BatteryTypeID, Description |
| `Battery_AdditionalFeeDetails` | Battery additional fees | — |
| `battery_category` | Battery categories | CategoryID, CategoryName |
| `Battery_Footnotes` | Battery footnote text | — |
| `battery_highlights` | Feature highlights | — |
| `battery_images` | Battery images | — |
| `Battery_Specification` | Technical specs | — |
| `Battery_SupplierDetails` | Supplier info | — |
| `BT_promotions_details` | Battery promotions | — |

---

### 3. Vehicle / Fitment

| Table | Purpose |
|---|---|
| `tblBrandName` / `tblBrandName_V1` | Vehicle make/brand lookup |
| `French_Lookup` | French translation for car makes |
| `StateMaster` | US State list (code, name) |

---

### 4. Warehouse

| Table | Purpose | Key Columns |
|---|---|---|
| `tbl_CostcoWarehoseInfo` | Warehouse master | `WarehouseCode`, `CityName`, `StateAbbreviation`, `PostalCode`, `Country`, `Latitude`, `Longitude` |
| `MST_WarehouseTiming` | Warehouse hours | `WarehouseCode`, `WeekDaysTiming`, `SaturdayTiming`, `SundayTiming` |
| `WarehouseVendorList` | Vendor-to-warehouse mapping | — |

---

### 5. Configuration

| Table | Purpose |
|---|---|
| `Mst_GeneralConfiguration` | Runtime feature flags and config values (DB-driven config) |
| `MST_KeySettings` | Per-culture key-value settings (promotion prices, URLs, etc.) |
| `MST_WebConfigKeySettings_V1` | Web config key overrides from DB |
| `MST_SortingConfiguration` | Search result sort options |
| `MST_Currency` | Currency info per locale |
| `MST_Holidays` | Holiday closure dates |
| `MST_SupplierDetails` | Supplier info |
| `MST_API_Configuration` | External API endpoint config |

---

### 6. Admin & Logging

| Table | Purpose |
|---|---|
| `admin_users` | Admin portal users (also ASP.NET Identity `AspNetUsers`) |
| `admin_activity_log` | Admin action audit trail |
| `admin_request_assets` | Asset management requests |
| `admin_request_promotions` | Promotion requests |
| `mst_tires_whs_event_logs` | Warehouse event logging |
| `MST_SearchDetails` | Search audit log (query terms, results count, timestamp) |

---

### 7. WebJobs / Feed Processing

| Table | Purpose |
|---|---|
| `MST_WebJob_Configuration` | WebJob schedule and config |
| `MST_WebJob_FeedStack` | Feed processing queue |
| `MST_WebJob_Logs` | WebJob execution logs |
| `MST_BAZAARVOICE_FEEDSTACK` | Bazaarvoice product feed queue |
| `MST_PRODUCT_FEED_FEEDSTACK` | Product feed queue |
| `GeneralFeedStack` | General feed processing queue |

---

## Key Domain Models (C# Classes)

### ITireInfo / TireInfo

```
SkuNumber, CostcoProductId, TireID, TireName, TireSize, Sidewall
Price (double), Currency, PromoPrice, DiscountedPrice, MemberOnlyPrice
IsAvailable (bool), Available_Qty, SafetyStockLimit, NextAvailableDate, MaxBuyingQty
STD_OR_OPT, LoadIndex, SpeedRating, FitmentTireSize, FRB
IsStaggeredTire (bool), ClubbedId, StaggeredTirePosition
IsItemNew, DisplayOnWebsite, ZeroPressure, IsEVCompatible, IsEVDedicated
SupplierNumber, SupplierID, MfgWarranty, MfgWarrantyKM
List<IAdditionalFees> additionalFees
List<ITireImages> TireImages
List<ITireFeatures> TireFeatures
```

### ISearch / Search (search criteria)

```
Year, Make, Model, Option           ← vehicle search
Width, Aspect, Rim                  ← size search
ItemNo, PartNo                      ← item/part search
Culture, CountryCode, SearchLang
WareHouseCode, StateCode
BrandNames (comma-sep), SeasonNames
FilterType (1=vehicle, 2=custom, 3=size, 4=item)
LoadRatingDeviation, SpeedRatingDeviation
FRB, SpecialMake
```

### ISearchResults / SearchResults

```
List<IFitmentTireInfo> FitmentTireInfo   ← OE fitment tires
List<ITireInfo> TireInfo                 ← matching tire products
ISearch Search                           ← original criteria
```

### IQuantityAndCart / QuantityAndCart

```
Quantity, QuantityTextId, AddtoCartButtonId
NetPricePerTire, FourTirePrice, FinalDiscountPrice, PriceforSingleTire
ActualScratchedPriceValue, PromoPrice
IsPromoMember, IsNonPromoMember, IsCostcoMember
SetPromotionPrice, SetOriginalPrice
IsShippingContent, IsNonPromoDeliveryInfo, IsPromoDeliveryInfo
IsTemplyOutofStockUsInfo, IsSpecficWarehosueIdforAddtionalFee
```

### ITireWarehousePrice / TireWarehousePrice

```
WarehouseCode, PostalCode, CityName, StateAbbreviation, Country
WeekDaysTiming, SaturdayTiming, SundayTiming
OffSet (timezone)
```

---

## Key Stored Procedures

| Procedure | Domain | Purpose |
|---|---|---|
| `SP_GetSearchResultsBySize_Redesign_V8` | Tire | Custom fitment / size search |
| `SP_GetSearchResultsByItemNo_Redesign_V*` | Tire | Item/part number search |
| `SP_GetVehicleMakeList_V3` | Vehicle | Year/make dropdown |
| `SP_GetVehicleModelList_Redesign_V3` | Vehicle | Model dropdown |
| `SP_GetVehicleOptionList_Redesign_V4` | Vehicle | Option/trim dropdown |
| `SP_GetTireWarehouseInfo_V1` | Warehouse | Warehouse info for tire |
| `SP_GetTireWarehouseTimingInfo` | Warehouse | Opening hours |
| `SP_GetWarehouseShipToHomeInfo_Redesign` | Warehouse | Ship-to-home eligibility |
| `SP_GetConfigurableItemsAndAttributes_Redesign_V*` | Config | UI filter options |
| `SP_GetWebConfigKeySettings_Redesign_v1` | Config | DB-driven config keys |
| `SP_GetSortingConfiguration` | Config | Sort options |
| `SP_Admin_SavePromotion_Redesign` | Admin | Save/update promotion |
| `SP_Admin_GetPromotionList_Redesign` | Admin | List promotions |
| `SP_GetTireWarranty_Redesign_v3` | Product | Warranty text |
| `BT_Get_SearchResultsByVehicle` | Battery | Battery vehicle search |
| `BT_GetBatterySearchWidgetList` | Battery | Battery dropdowns |
| `BT_GetBatteryTypes` | Battery | Battery type list |
| `API_SP_GetWarehouseInfo_v1` | Warehouse | API warehouse info |
