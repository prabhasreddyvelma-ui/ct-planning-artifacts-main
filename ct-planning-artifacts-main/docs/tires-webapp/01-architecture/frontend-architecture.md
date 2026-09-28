# Costco Tires Web App — Frontend Architecture

> **Version:** 1.0
>
> **Author:** Rupesh.Jain
>
> **Date:** 2026-05-12
>
> **Purpose:** Defines the frontend architecture for the `gdx-ux-tires` repository — the Next.js application powering `tires.costco.com` and `tires.costco.ca`. This document establishes:
> 1. How the architecture **inherits** from the `gdx-ux-cnsw-search-catalog` reference repo (patterns, tooling, component library, coding rules).
> 2. Where tires **diverges** from catalog search — and precisely why.
> 3. The complete folder structure, component catalogue, state strategy, data flow, and integration contracts for developers joining the project.
>
> **Reference architecture:** `docs/tires-webapp/01-architecture/catalog-search-architecture.md`
> All patterns not explicitly overridden here follow the catalog search reference document.

---

## Table of Contents

1. [Architecture Alignment — Inherit vs Diverge](#1-architecture-alignment--inherit-vs-diverge)
2. [Technology Stack](#2-technology-stack)
3. [High-Level Architecture](#3-high-level-architecture)
4. [Rendering Strategy](#4-rendering-strategy)
5. [Key Architectural Decisions](#5-key-architectural-decisions)
6. [Folder Structure](#6-folder-structure)
7. [Component Catalogue](#7-component-catalogue)
   - 7.1 [Reused from Catalog Search](#71-reused-from-catalog-search)
   - 7.2 [Adapted from Catalog Search](#72-adapted-from-catalog-search)
   - 7.3 [Net-New Tires Components](#73-net-new-tires-components)
8. [State Management — Redux Toolkit](#8-state-management--redux-toolkit)
9. [Data Flow — Request Lifecycle](#9-data-flow--request-lifecycle)
10. [Key Cross-Cutting Concerns](#10-key-cross-cutting-concerns)
    - 10.1 [Internationalisation (i18n)](#101-internationalisation-i18n)
    - 10.2 [Content Management (ContentStack)](#102-content-management-contentstack)
    - 10.3 [Tire Center & Bing Maps](#103-tire-center--bing-maps)
    - 10.4 [Authentication & Consent](#104-authentication--consent)
    - 10.5 [Analytics — Adobe Analytics](#105-analytics--adobe-analytics)
    - 10.6 [Feature Flags](#106-feature-flags)
    - 10.7 [Observability](#107-observability)
    - 10.8 [Caching Strategy](#108-caching-strategy)
11. [Style Guide](#11-style-guide)
12. [TypeScript Patterns & Conventions](#12-typescript-patterns--conventions)
13. [Testing Strategy](#13-testing-strategy)
14. [API Routes](#14-api-routes)
15. [Environment Variables](#15-environment-variables)
16. [Build, Deploy & Runtime](#16-build-deploy--runtime)
17. [Path Aliases](#17-path-aliases)
18. [Quick Reference](#18-quick-reference)

---

## 1. Architecture Alignment — Inherit vs Diverge

The tires webapp is a **sibling application** to `gdx-ux-cnsw-search-catalog`. It is built on the same Next.js 14 App Router foundation, uses the same Costco Forge design system, follows the same coding conventions, and reuses entire directories (services/content, observability, i18n, CSP, consent). The divergences exist only where the tires domain requires them.

### Summary Table

| Concern | Catalog Search | Tires Webapp | Action |
|---|---|---|---|
| Framework | Next.js 14 App Router | **Same** | Copy config |
| Language | TypeScript 5 strict | **Same** | Copy tsconfig |
| Component library | Forge (`@nxt-costco-com/forge-*`) | **Same** | Reuse |
| Layout primitives | MUI v6 (Grid, Box) | **Same** | Reuse |
| Global styles | SCSS modules | **Same** | Copy global.scss |
| CMS | ContentStack (2 stacks) | **Same** | Copy services/content/ |
| i18n routing | `next-i18n-router` | **Same** | Copy middleware pattern |
| Auth | MSAL via Forge | **Same** | Copy layout providers |
| Consent | Transcend via Forge | **Same** | Copy layout providers |
| Analytics | Adobe Analytics + Criteo | **Same** (Adobe only; Criteo not applicable) | Adapt |
| Observability | OpenTelemetry + Prometheus | **Same** | Copy exactly |
| Logging | Pino | **Same** | Copy exactly |
| Testing | Vitest + RTL + MSW + Playwright | **Same** | Copy config |
| Linting | oxlint | **Same** | Copy config |
| Build / Docker | Standalone Next.js output | **Same** | Copy Dockerfile |
| **Client state** | React Context (`SearchProvider`) + URL params | **Redux Toolkit** (fitment/location context persists across sessions) | **DIVERGE — see §5 & §8** |
| **Maps** | Google Maps (WarehouseSelector) | **Bing Maps** (Tire Center Locator) | **DIVERGE — see §10.3** |
| **Search backend** | GRS / LucidWorks | **Spring Boot 3.2 BFF** (Strangler Fig from .NET 4.8) | **DIVERGE — see §9** |
| **State persistence** | URL params only (no localStorage) | Redux + localStorage for fitment context and Tire Center | **DIVERGE — see §8** |

> **Rule:** When a pattern is not listed as DIVERGE in this table, default to the catalog search reference architecture. Do not invent alternatives.

---

## 2. Technology Stack

| Layer | Library | Version | Source |
|---|---|---|---|
| Framework | Next.js (App Router) | ^14.2.35 | Same as catalog search |
| Language | TypeScript | 5 (strict) | Same as catalog search |
| UI Primitives | MUI | v6 | Same as catalog search |
| **Design System** | `@nxt-costco-com/forge-components` | ^3.8.1 | **Reuse — primary component library** |
| **Design Tokens** | `@nxt-costco-com/forge-design-tokens` | ^3.0.2 | **Reuse — brand colours, spacing, typography** |
| **Digital Components** | `@nxt-costco-com/forge-digital-components` | ^3.14.4 | **Reuse — Header, Footer, ProductTile, Analytics** |
| **Icons** | `@nxt-costco-com/forge-icons` | ^2.0.3 | **Reuse — SVG icon set** |
| CMS | ContentStack SDK | ^3.25.1 | Same as catalog search |
| i18n | `next-i18n-router` | ^5.4.2 | Same as catalog search |
| **State** | **Redux Toolkit** | **^2.x** | **DIVERGE: fitment/location context requires cross-session persistence** |
| Forms | Zod + `react-hook-form` | 3.22.4 | Same as catalog search |
| Styling | SCSS modules + Emotion | — | Same as catalog search |
| **Maps** | **Bing Maps JavaScript API** | **latest** | **DIVERGE: Tire Center Locator uses Bing Maps per Costco standard** |
| Session tracking | LogRocket | ^7.0.0 | Same as catalog search |
| Testing (unit) | Vitest | — | Same as catalog search |
| Testing (component) | React Testing Library | ^14.3.1 | Same as catalog search |
| Testing (API mock) | MSW | — | Same as catalog search |
| Testing (E2E) | Playwright | ^1.40.0 | Same as catalog search |
| Observability | OpenTelemetry + prom-client | ^1.30.0 / ^15.1.3 | Same as catalog search |
| Logging | Pino | ^9.6.0 | Same as catalog search |
| Session cache | Redis via `@neshca/cache-handler` | ^1.9.0 | Same as catalog search |
| Auth | MSAL via Forge | — | Same as catalog search |
| Consent | Transcend via Forge | — | Same as catalog search |
| Analytics | Adobe Analytics via Forge | — | Same as catalog search |
| HTTP client | undici | ^6.21.3 | Same as catalog search |
| Env validation | `@t3-oss/env-nextjs` + Zod | 0.7.1 | Same as catalog search |
| Bundle analysis | `@next/bundle-analyzer` | ^14.2.24 | Same as catalog search |
| Linting | oxlint (Rust) | — | Same as catalog search |

### Node & Runtime Requirements

```
Node.js >= 22.11.0   (enforced via package.json "engines")
Redis               (required for ISR cache in production)
Docker              (containerised deployment)
PM2                 (process manager — ecosystem.config.js)
```

---

## 3. High-Level Architecture

```
Browser / CDN
  │
  ▼
Next.js Edge Middleware  (src/middleware.ts)
  │  ┌─ i18n routing: /[lang]/  /[lang]/tires  /[lang]/tire/[id]  etc.
  │  ├─ URL rewrites (legacy path support)
  │  ├─ Cache-tag response headers (CDN purge groups)
  │  ├─ CSP header injection from process.env.CONTENT_SECURITY_POLICIES
  │  ├─ Queue-It bot/load management check
  │  └─ Prometheus edge metrics (inbound + latency counters)
  ▼
Next.js 14 App Router  (src/app/)
  │
  ├─ Server Components  (RSC — default)
  │    ├─ Fetches ContentStack CMS layout config (hero content, carousel data, labels)
  │    ├─ Fetches feature flag state
  │    └─ Passes serialisable data as props to client subtree
  │
  └─ Client Components  ('use client')
       ├─ Redux Store — vehicle selection, Tire Center, search criteria, UI state
       ├─ Tire Search Form — 4 tabs, form state, validation, routing
       ├─ Tire Center Locator — Bing Maps + results list
       └─ Analytics event firing
       
       ┌──────────────────────────────────────────────────────────────┐
       │  External Services Called at Runtime                        │
       │                                                              │
       │  ┌───────────────────┐   ┌──────────────────────────────┐  │
       │  │  ContentStack CMS │   │  Spring Boot 3.2 BFF         │  │
       │  │  (US / CA)        │   │  (Strangler Fig from .NET 4.8│  │
       │  │  Config stack     │   │  133 API migration in flight) │  │
       │  │  Content stack    │   └──────────────────────────────┘  │
       │  └───────────────────┘   ┌──────────────────────────────┐  │
       │                          │  Mitchell1 VIN decode service│  │
       │  ┌───────────────────┐   │  (License Plate → vehicle)   │  │
       │  │  Redis Cache      │   └──────────────────────────────┘  │
       │  │  (ISR handler)    │   ┌──────────────────────────────┐  │
       │  └───────────────────┘   │  Bing Maps JavaScript API    │  │
       │                          │  (Tire Center Locator map)   │  │
       │  ┌───────────────────┐   └──────────────────────────────┘  │
       │  │  Prometheus       │   ┌──────────────────────────────┐  │
       │  │  metrics :9200    │   │  Adobe Analytics             │  │
       │  └───────────────────┘   │  LogRocket (session replay)  │  │
       │                          └──────────────────────────────┘  │
       └──────────────────────────────────────────────────────────────┘
```

---

## 4. Rendering Strategy

| Page | Strategy | Revalidation | Notes |
|---|---|---|---|
| Home page (`/`) | ISR | `revalidate = 3600` (1 hour) | CMS-driven carousel and hero content |
| Tire Center Locator (`/TireCenterLocator`) | SSR (dynamic) | n/a — always fresh | Warehouse availability is real-time |
| Size Selection Page (`/sizeSelection`) | SSR (dynamic) | n/a | Depends on vehicle params in URL |
| Products Listing Page (`/SearchResultsByVehicle`, `/SearchResultsBySize`) | SSR (dynamic) | n/a | Real-time inventory; no caching |
| Product Detail Page (`/Product/[id]`) | ISR | `revalidate = 1800` (30 min) | Product data changes infrequently |
| Compare Tires (`/Tirecomparison`) | SSR (dynamic) | n/a | User-selected comparison set |
| Info pages (FAQ, TireBasics, etc.) | ISR | `revalidate = 86400` (24 hours) | Rarely changes |
| New CMS entries | On-demand ISR (`dynamicParams = true`) | Webhook → `/api/revalidate` | Same as catalog search |
| Static locale params | `generateStaticParams` | Build time | en-US, en-CA, fr-CA |
| API routes | Dynamic (always fresh) | n/a | — |

---

## 5. Key Architectural Decisions

### 1. Redux Toolkit for Cross-Session State (DIVERGE from catalog search)

The catalog search app uses React Context (`SearchProvider`) + URL params exclusively — no localStorage persistence. The tires webapp **requires Redux Toolkit** because:

- **Fitment context** (vehicle make/model/year/trim, selected tire size) must persist across pages and sessions. A member who selects their vehicle on the homepage must not re-enter it on the Size Selection or PLP page.
- **Tire Center selection** must persist across sessions (30-day retention for signed-in members, 24 hours for guests) — this cannot be encoded in URL params.
- **Cross-page context continuity** (the Change Vehicle Modal, Tire Center Locator) require shared writable state that neither URL params nor a single-page Context handles cleanly.

Redux Toolkit slices:

| Slice | State held | Persisted? |
|---|---|---|
| `vehicleSlice` | Year, Make, Model, Trim, active search tab | localStorage (fitment context only) |
| `tireCenterSlice` | Selected Tire Center ID, name, address | localStorage (Tire Center ID only) |
| `searchCriteriaSlice` | Tire size, item number, same/staggered toggle | localStorage (search criteria) |
| `uiSlice` | Active tab, modal open/closed, loading states | No — session only |

**Security rule:** No auth tokens, member PII, or payment data ever enters localStorage. Only fitment context (vehicle, Tire Center, tire size) is persisted.

### 2. Spring Boot 3.2 BFF as the Sole Backend (DIVERGE from catalog search)

The catalog search calls GRS/LucidWorks directly from the Next.js app. The tires webapp routes **all backend calls through a Spring Boot 3.2 BFF** — a Strangler Fig migration from the .NET 4.8 monolith (133 APIs, 20-week migration plan). The Next.js app has no direct connection to any backend service other than the BFF.

During development, **MSW (Mock Service Worker)** is used to simulate BFF responses before live APIs are available. BFF contract sign-off is required before sprint 1 begins.

### 3. Bing Maps for Tire Center Locator (DIVERGE from catalog search)

The catalog search architecture uses Google Maps for warehouse lookup. The tires webapp uses **Bing Maps JavaScript API** — Costco's standard map provider for tires.costco.com. The Bing Maps API key is managed via environment config and must not be hardcoded.

### 4. RSC-First — Same as catalog search

Every component defaults to a Server Component. Only add `'use client'` when browser APIs, event handlers, or React hooks (`useState`, `useEffect`) are required. The Redux store provider must be wrapped in a `'use client'` boundary — following the same pattern as `SearchProvider` in the catalog search.

### 5. CMS-Driven Content — Same as catalog search

ContentStack drives all configurable content: homepage hero, carousel slides, feature flags, CSP, labels. The tires webapp uses the **same two-stack pattern** (Content Stack + Config Stack) as catalog search. The `services/content/` directory is copied without modification.

### 6. Tire Search Form as Reusable Component

The Tire Search Form (`TireSearchForm`) is the most critical component. It renders in two modes controlled by the `isModalContent` prop:

- `isModalContent={false}` — standalone on the homepage
- `isModalContent={true}` — inside the Change Vehicle Modal (subsequent pages)

This is the same `isModalContent` pattern used for the Tire Center Locator component.

---

## 6. Folder Structure

```
gdx-ux-tires/
├── env.mjs                          # Zod-validated env vars (tire-specific vars added)
├── next.config.mjs                  # Copy from catalog search + tire-specific route rewrites
├── tsconfig.json                    # Copy from catalog search (update path aliases)
├── tsconfig.test.json               # Copy exactly from catalog search
├── vitest.config.mts                # Copy from catalog search (update aliases)
├── vitest.setup.ts                  # Copy exactly from catalog search
├── playwright.config.ts             # Copy from catalog search (update base URL)
├── Dockerfile                       # Copy exactly from catalog search
├── ecosystem.config.js              # Copy + rename app: 'tires'
├── oxlintrc.json                    # Copy exactly from catalog search
├── global.d.ts                      # Copy from catalog search (add Bing Maps window types)
├── start.js
├── public/
│   └── tiresAssets/
│       └── fonts/                   # Self-hosted Costco fonts (WOFF2)
├── grafana/
│   ├── docker-compose.yml           # Copy exactly
│   └── prometheus.yml               # Copy exactly
└── src/
    ├── instrumentation.ts           # Copy exactly from catalog search
    ├── middleware.ts                 # Adapt from catalog search (tire-specific URL rewrites)
    ├── metrics.ts                   # Copy exactly from catalog search
    ├── metrics-server.ts            # Copy exactly from catalog search
    │
    ├── actions/
    │   └── auth.ts                  # Copy from catalog search
    │
    ├── app/
    │   ├── _lib/
    │   │   └── types.ts             # LocalizedLayoutProps, TirePageProps
    │   ├── [lang]/
    │   │   ├── (customers)/
    │   │   │   ├── layout.tsx       # Root <html> shell + consent + Queue-It
    │   │   │   ├── page.tsx         # Home page  →  /[lang]/
    │   │   │   ├── sizeSelection/
    │   │   │   │   └── page.tsx     # Size Selection  →  /[lang]/sizeSelection
    │   │   │   ├── SearchResultsByVehicle/
    │   │   │   │   └── page.tsx     # PLP (vehicle)  →  /[lang]/SearchResultsByVehicle
    │   │   │   ├── SearchResultsBySize/
    │   │   │   │   └── page.tsx     # PLP (size)  →  /[lang]/SearchResultsBySize
    │   │   │   ├── Product/
    │   │   │   │   └── page.tsx     # PDP  →  /[lang]/Product?id=[itemNumber]
    │   │   │   ├── Tirecomparison/
    │   │   │   │   └── page.tsx     # Compare  →  /[lang]/Tirecomparison
    │   │   │   ├── TireCenterLocator/
    │   │   │   │   └── page.tsx     # Tire Center Locator standalone  →  /[lang]/TireCenterLocator
    │   │   │   ├── [brandSlug]/
    │   │   │   │   └── page.tsx     # Brand landing  →  /[lang]/Michelin_tires
    │   │   │   └── [infoSlug]/
    │   │   │       └── page.tsx     # Info pages  →  /[lang]/FAQ  /[lang]/TireBasics  etc.
    │   │   └── (businessUsers)/
    │   │       ├── layout.tsx
    │   │       └── livepreview/
    │   │           └── page.tsx
    │   └── api/
    │       ├── health/route.ts
    │       ├── build-health/route.ts
    │       ├── metrics/route.ts
    │       ├── revalidate/route.ts
    │       ├── contentStack/route.ts
    │       └── tires/
    │           ├── vehicle-lookup/route.ts      # GET: plate + state → vehicle record (via BFF)
    │           ├── vehicle-catalog/
    │           │   ├── makes/route.ts            # GET: year → makes[]
    │           │   ├── models/route.ts           # GET: year + make → models[]
    │           │   ├── trims/route.ts            # GET: year + make + model → trims[]
    │           │   └── sizes/route.ts            # GET: year + make + model + trim → sizes[]
    │           └── tire-centers/route.ts         # GET: zip → tire centers[] (via BFF)
    │
    ├── components/
    │   │
    │   │  ── Copied directly from catalog search (no changes) ──
    │   ├── Analytics/
    │   ├── BreadCrumb/
    │   ├── CenteredBox/
    │   ├── ErrorBoundary/
    │   ├── ErrorServiceUnavailable/
    │   ├── Feedback/                            # OpinionLab survey
    │   ├── KMSI/
    │   ├── LivePreview/
    │   ├── NotFoundUI/
    │   ├── OneTag/
    │   ├── TranscendOverride/
    │   │
    │   │  ── Adapted from catalog search ──
    │   ├── SortSelector/                        # Adapt from catalog search SortSelector
    │   ├── TirePagination/                      # Adapt from SearchResultsPagination
    │   ├── TireResultsGrid/                     # Adapt from SearchResultsGrid
    │   │   ├── index.tsx
    │   │   ├── index.test.tsx
    │   │   ├── TireProductTile.tsx              # Adapt from Forge ProductTileUI
    │   │   ├── TireProductTile.test.tsx
    │   │   └── LoadingTiresGrid.tsx             # Adapt from LoadingResultsGrid
    │   ├── TireFilters/                         # Adapt from SearchResultsFacets / FilterGroup
    │   │   ├── index.tsx
    │   │   ├── BrandFilter.tsx
    │   │   ├── TireTypeFilter.tsx
    │   │   ├── PriceRangeFilter.tsx
    │   │   ├── SpeedRatingFilter.tsx
    │   │   └── MileageWarrantyFilter.tsx
    │   │
    │   │  ── Net-new tires components ──
    │   ├── TireSearchForm/
    │   │   ├── index.tsx                        # Tab container; isModalContent prop
    │   │   ├── index.test.tsx
    │   │   ├── VehicleTab.tsx                   # Year/Make/Model/Trim cascading dropdowns
    │   │   ├── VehicleTab.test.tsx
    │   │   ├── LicensePlateTab.tsx              # Plate + State (en-US only)
    │   │   ├── LicensePlateTab.test.tsx
    │   │   ├── TireSizeTab.tsx                  # Width/Ratio/Rim + same/staggered toggle
    │   │   ├── TireSizeTab.test.tsx
    │   │   ├── ItemNumberTab.tsx
    │   │   ├── ItemNumberTab.test.tsx
    │   │   ├── ZipCodeField.tsx                 # Conditional ZIP field (no Tire Center stored)
    │   │   ├── ZipCodeField.test.tsx
    │   │   ├── types.ts                         # SearchMode, TireSearchFormProps, etc.
    │   │   └── constants.ts
    │   ├── TireCenterLocator/
    │   │   ├── index.tsx                        # isModalContent prop; standalone + modal modes
    │   │   ├── index.test.tsx
    │   │   ├── TireCenterMap.tsx                # Bing Maps integration; marker → list highlight
    │   │   ├── TireCenterMap.test.tsx
    │   │   ├── TireCenterList.tsx               # Results list with Set as My Tire Center button
    │   │   ├── TireCenterList.test.tsx
    │   │   ├── TireCenterCard.tsx               # Individual tire center result card
    │   │   ├── TireCenterSearchInput.tsx        # City / state / ZIP search input
    │   │   ├── types.ts                         # TireCenter, TireCenterLocatorProps
    │   │   └── constants.ts
    │   ├── DealsCarousel/
    │   │   ├── index.tsx                        # Brand slides + play/pause + external links
    │   │   ├── index.test.tsx
    │   │   └── DealSlide.tsx
    │   ├── TireInfoSlider/
    │   │   ├── index.tsx                        # 8 educational slides; click → external URL
    │   │   ├── index.test.tsx
    │   │   └── InfoSlide.tsx
    │   ├── WhyBuyAtCostco/
    │   │   ├── index.tsx                        # Static benefits content + Schedule Appointment CTA
    │   │   └── index.test.tsx
    │   ├── BlogSection/
    │   │   ├── index.tsx                        # Full-width editorial article
    │   │   └── index.test.tsx
    │   ├── TireDetailSpec/
    │   │   ├── index.tsx                        # Spec table (size, load index, speed rating, etc.)
    │   │   ├── index.test.tsx
    │   │   └── types.ts
    │   ├── TireDetailWarranty/
    │   │   ├── index.tsx                        # Road hazard + manufacturer warranty section
    │   │   └── index.test.tsx
    │   ├── ExternalLinkModal/
    │   │   ├── index.tsx                        # Warning modal when leaving site; Cancel + Visit buttons
    │   │   └── index.test.tsx
    │   ├── SelectedVehicleBanner/
    │   │   ├── index.tsx                        # Shows selected vehicle/size below nav; Change link
    │   │   └── index.test.tsx
    │   ├── ChangeVehicleModal/
    │   │   ├── index.tsx                        # Reuses <TireSearchForm isModalContent={true} />
    │   │   └── index.test.tsx
    │   ├── ScheduleAppointmentModal/
    │   │   ├── index.tsx                        # Scenario 1 (TC selected) + Scenario 2 (no TC)
    │   │   ├── index.test.tsx
    │   │   └── types.ts
    │   └── TireCompatibilityCheckModal/
    │       ├── index.tsx                        # Post-size-search add-to-cart compatibility check
    │       └── index.test.tsx
    │
    ├── containers/
    │   ├── common/
    │   │   ├── header.tsx                       # Forge Header + Tire Center header link logic
    │   │   └── footer-full.tsx                  # Copy from catalog search
    │   ├── home/
    │   │   ├── index.tsx                        # HomeContainer (async RSC)
    │   │   ├── HeroSection.tsx                  # Hero + TireSearchForm
    │   │   ├── DealsCarouselSection.tsx
    │   │   ├── WhyBuySection.tsx
    │   │   ├── TireInfoSliderSection.tsx
    │   │   └── BlogSection.tsx
    │   ├── tireSearch/
    │   │   ├── index.tsx                        # TireSearchContainer (async RSC)
    │   │   ├── LeftColumn.tsx                   # TireFilters sidebar
    │   │   ├── MainColumn.tsx                   # TireResultsGrid + TirePagination
    │   │   ├── NoResultsComponent.tsx
    │   │   ├── utils.ts
    │   │   └── constants.ts
    │   ├── sizeSelection/
    │   │   ├── index.tsx                        # SizeSelectionContainer (async RSC)
    │   │   └── utils.ts
    │   ├── tireDetail/
    │   │   ├── index.tsx                        # TireDetailContainer (async RSC)
    │   │   ├── ImageGallery.tsx
    │   │   ├── PurchasePanel.tsx
    │   │   ├── SpecSection.tsx
    │   │   └── WarrantySection.tsx
    │   └── brandLanding/
    │       └── index.tsx
    │
    ├── redux/
    │   ├── store.ts                             # Redux Toolkit configureStore; persistReducer config
    │   ├── hooks.ts                             # Typed useAppDispatch, useAppSelector
    │   ├── ReduxProvider.tsx                    # 'use client' — wraps app with <Provider store>
    │   └── slices/
    │       ├── vehicleSlice.ts                  # Year/Make/Model/Trim; active search tab
    │       ├── vehicleSlice.test.ts
    │       ├── tireCenterSlice.ts               # Selected Tire Center ID, name, address
    │       ├── tireCenterSlice.test.ts
    │       ├── searchCriteriaSlice.ts           # Tire size, item number, same/staggered
    │       ├── searchCriteriaSlice.test.ts
    │       ├── uiSlice.ts                       # Modal open/closed, active tab, loading flags
    │       └── uiSlice.test.ts
    │
    ├── constants/
    │   ├── index.ts                             # Cookie names, locale IDs, delimiter strings
    │   ├── language/
    │   │   └── index.ts                         # i18nConfig — copy from catalog search
    │   ├── tires.ts                             # Tire widths/ratios/rims, brands, speed ratings
    │   ├── contentStack.ts                      # CMS content type UIDs for tires
    │   ├── routes.ts                            # Page route constants (/sizeSelection, /TireCenterLocator, etc.)
    │   ├── styles.ts                            # z-index values, breakpoints
    │   └── errorMessages/
    │       └── index.ts                         # User-facing error strings (centralised for i18n)
    │
    ├── hooks/
    │   ├── useCheckScreen.ts                    # Copy from catalog search
    │   ├── useQueryString.ts                    # Copy from catalog search
    │   ├── useVehicleDropdowns.ts               # Cascading Year/Make/Model/Trim dropdown state
    │   ├── useVehicleDropdowns.test.ts
    │   ├── useTireCenterLocator.ts              # Bing Maps init, search, marker interaction
    │   └── useTireCenterLocator.test.ts
    │
    ├── mock/
    │   ├── handlers.ts                          # MSW v2 handlers for BFF API routes
    │   └── server.ts                            # Copy from catalog search
    │
    ├── providers/
    │   └── mui/
    │       └── ThemeRegistry.tsx                # Copy exactly from catalog search
    │
    ├── services/
    │   ├── logger.ts                            # Copy exactly from catalog search
    │   ├── content/                             # Copy entire directory from catalog search
    │   │   ├── stackClient.ts
    │   │   ├── config.lib.ts
    │   │   ├── featureFlag.ts
    │   │   ├── livePreviewClient.ts
    │   │   ├── unstable_cache.ts
    │   │   └── constants.ts
    │   └── bff/                                 # Spring Boot BFF API client layer
    │       ├── client.ts                        # Centralised fetch wrapper; base URL from env; error handling
    │       ├── vehicleLookup.ts                 # License Plate + State → vehicle (Mitchell1 via BFF)
    │       ├── vehicleLookup.test.ts
    │       ├── vehicleCatalog.ts                # Year/Make/Model/Trim dropdown data
    │       ├── vehicleCatalog.test.ts
    │       ├── tireCenters.ts                   # ZIP → tire centers list (for Tire Center Locator)
    │       ├── tireCenters.test.ts
    │       ├── types.ts                         # BFF response shapes
    │       └── constants.ts                     # BFF endpoint paths
    │
    ├── styles/
    │   ├── global.scss                          # CSS reset, root variables — copy from catalog search
    │   ├── colors.scss                          # Brand colour CSS custom properties
    │   └── fonts.scss                           # @font-face declarations
    │
    ├── types/
    │   ├── types.d.ts                           # Copy from catalog search + Bing Maps window types
    │   ├── cookies.ts                           # Copy from catalog search
    │   ├── metrics.ts                           # Copy from catalog search
    │   ├── contentStack/
    │   │   ├── TireHomePage.ts
    │   │   ├── TireSearchConfig.ts
    │   │   ├── TireBrandPage.ts
    │   │   └── TireInfoPage.ts
    │   ├── layouts/
    │   │   └── index.ts                         # LocalizedLayoutProps
    │   ├── pages/
    │   │   └── tire.ts                          # TirePageType enum map, page prop types
    │   └── tires/
    │       ├── vehicle.ts                       # Vehicle, VehicleYear, VehicleMake, etc.
    │       ├── tireCenter.ts                    # TireCenter, TireCenterSearchResult
    │       ├── tireProduct.ts                   # TireProductTile, TireSpec, TireFilter
    │       └── search.ts                        # TireSearchParams, SearchMode, SearchRouting
    │
    └── utils/
        ├── index.ts                             # Copy from catalog search
        ├── lib.ts                               # Copy from catalog search
        ├── cookieUtils.ts                       # Copy from catalog search
        ├── middleware.ts                        # Adapt from catalog search (tire-specific routes)
        ├── metrics.ts                           # Copy from catalog search
        ├── metricsHelper.ts                     # Copy from catalog search
        ├── metricsClassifier.ts                 # Adapt — add tire route patterns
        ├── inboundMetricsClassifier.ts          # Copy from catalog search
        ├── fetchContentStackPolicies.ts         # Copy from catalog search
        ├── next.ts                              # Copy from catalog search
        ├── tireSize.ts                          # Parse/format "215/55R17" — validation, conversion
        ├── tireSize.test.ts
        ├── vehicleUtils.ts                      # Format vehicle display strings
        ├── vehicleUtils.test.ts
        └── links.ts                             # Copy + add tire-specific URL builders
```

---

## 7. Component Catalogue

### 7.1 Reused from Catalog Search

Copy these directories without modification:

| Component | Source | Notes |
|---|---|---|
| `Analytics/` | catalog search | Adobe Analytics `window.digitalData` population |
| `BreadCrumb/` | catalog search | Server Component — ContentStack data |
| `CenteredBox/` | catalog search | MUI layout utility |
| `ErrorBoundary/` | catalog search | Client boundary — catches render errors |
| `ErrorServiceUnavailable/` | catalog search | 503-style error page |
| `Feedback/OpinionLab` | catalog search | Survey script — locale-conditional |
| `KMSI/` | catalog search | Keep Me Signed In — MSAL |
| `LivePreview/` | catalog search | ContentStack live preview overlay |
| `NotFoundUI/` | catalog search | 404 page content |
| `OneTag/` | catalog search | Criteo OneTag pixel |
| `TranscendOverride/` | catalog search | Consent per-page override |

### 7.2 Adapted from Catalog Search

| Component | Adapted From | Key Changes |
|---|---|---|
| `TireResultsGrid/` | `SearchResultsGrid` | Remove Criteo sponsored tile injection; add tire-specific badge rendering (M+S, 3PMSF, Run-Flat); replace product tile with `TireProductTile` |
| `TireProductTile/` | Forge `ProductTileUI` | Add: mileage warranty, speed rating, seasonal badges; price displayed per-tire AND as set-of-4 |
| `TireFilters/` | `SearchResultsFacets` / `FilterGroup` | Replace generic facet checkboxes with typed tire filter groups (Brand, TireType, Price Range, Speed Rating, Mileage Warranty) |
| `TirePagination/` | `SearchResultsPagination` | Minimal adaptation — same pagination logic, tire-themed styles |
| `SortSelector/` | `SortSelector` | Copy directly; tire-specific sort options (Featured, Price Asc/Desc, Rating, Mileage Warranty) |
| `LoadingTiresGrid/` | `LoadingResultsGrid` | Adapt skeleton dimensions to tire product tile shape |

### 7.3 Net-New Tires Components

| Component | Description | Key Props / Behaviour |
|---|---|---|
| `TireSearchForm` | 4-tab search form — Vehicle, License Plate, Tire Size, Item No. | `isModalContent: boolean` — controls layout for standalone (homepage) vs modal (Change Vehicle) rendering |
| `TireCenterLocator` | Tire Center search — city/state/ZIP → list + Bing Maps | `isModalContent: boolean` — standalone page vs modal; map marker click highlights list item |
| `DealsCarousel` | Promotional brand slides with play/pause | Auto-play; pause on hover/focus; external link via `ExternalLinkModal`; 3 slides (Michelin, Firestone, Bridgestone) |
| `TireInfoSlider` | 8 educational content slides, each linking to external URL | Same carousel pattern as `DealsCarousel`; click → `ExternalLinkModal` |
| `WhyBuyAtCostco` | Static benefits content + Schedule Appointment CTA | CTA opens `ScheduleAppointmentModal` |
| `BlogSection` | Full-width editorial article | SSR-rendered; CMS-driven content |
| `ExternalLinkModal` | Intercepts external link clicks with a warning modal | "Cancel" + "Visit External Link" buttons; applies to all external URLs sitewide |
| `SelectedVehicleBanner` | Displays selected vehicle or tire size below header navigation | Hidden until search performed; "Change" link opens `ChangeVehicleModal` |
| `ChangeVehicleModal` | Update search criteria from any page | Wraps `<TireSearchForm isModalContent={true} />` — component reuse, not duplication |
| `ScheduleAppointmentModal` | Schedule appointment at Tire Center | Scenario 1: TC pre-selected → vehicle form → redirect to Waitwhile; Scenario 2: no TC → vehicle + ZIP → opens Tire Center Locator modal → redirect to Waitwhile |
| `TireCompatibilityCheckModal` | Verify tire-to-vehicle compatibility after tire size search add-to-cart | Scenario 1: compatible → confirm Add to Cart; Scenario 2: incompatible → warning + Change Vehicle / Continue Browsing / Add to Cart Anyway |
| `TireDetailSpec` | Spec table (size, load index, speed rating, UTQG, weight, etc.) | Server Component — structured data |
| `TireDetailWarranty` | Road hazard + manufacturer warranty display | CMS-driven content per brand |
| `ZipCodeField` | Conditional ZIP input within TireSearchForm | Rendered only when no Tire Center stored in Redux; absent from DOM (not hidden) when TC is present |

---

## 8. State Management — Redux Toolkit

### Store Configuration

```ts
// src/redux/store.ts
import { configureStore } from '@reduxjs/toolkit'
import { persistStore, persistReducer } from 'redux-persist'
import storage from 'redux-persist/lib/storage'  // localStorage
import vehicleReducer from './slices/vehicleSlice'
import tireCenterReducer from './slices/tireCenterSlice'
import searchCriteriaReducer from './slices/searchCriteriaSlice'
import uiReducer from './slices/uiSlice'

const persistConfig = {
  key: 'tires-root',
  storage,
  whitelist: ['vehicle', 'tireCenter', 'searchCriteria'],  // ui NOT persisted
  // blacklist: ['ui']  — loading states, modal open/closed never persisted
}
```

### Slice Definitions

#### `vehicleSlice`

```ts
interface VehicleState {
  year: string | null
  make: string | null
  model: string | null
  trim: string | null
  activeSearchTab: 'vehicle' | 'licensePlate' | 'tireSize' | 'itemNumber'
}
```

#### `tireCenterSlice`

```ts
interface TireCenterState {
  id: string | null          // Warehouse ID — persisted
  name: string | null        // Display name for header link
  address: string | null
}
// On selection: header "Tire Center" link updates to show name
// Absence: ZIP Code field renders in TireSearchForm; header shows "Tire Center" link text
```

#### `searchCriteriaSlice`

```ts
interface SearchCriteriaState {
  tireWidth: string | null
  tireRatio: string | null
  tireDiameter: string | null
  isSameFrontRear: boolean
  rearTireWidth: string | null
  rearTireRatio: string | null
  rearTireDiameter: string | null
  itemNumber: string | null
}
```

#### `uiSlice` (not persisted)

```ts
interface UIState {
  isTireCenterLocatorOpen: boolean
  isChangeVehicleModalOpen: boolean
  isScheduleAppointmentModalOpen: boolean
  isTireCompatibilityModalOpen: boolean
  isExternalLinkModalOpen: boolean
  externalLinkTarget: string | null
  isLoading: boolean
}
```

### Provider Setup

```tsx
// src/redux/ReduxProvider.tsx
'use client'
import { Provider } from 'react-redux'
import { PersistGate } from 'redux-persist/integration/react'
import { store, persistor } from './store'

export function ReduxProvider({ children }: { children: React.ReactNode }) {
  return (
    <Provider store={store}>
      <PersistGate loading={null} persistor={persistor}>
        {children}
      </PersistGate>
    </Provider>
  )
}
```

`ReduxProvider` is placed in the tires layout — inside the `ThemeRegistry`, outside all page-level containers, following the same nesting order as `SearchProvider` in the catalog search.

### Typed Hooks

```ts
// src/redux/hooks.ts
import { useDispatch, useSelector } from 'react-redux'
import type { RootState, AppDispatch } from './store'

// Use these throughout the app — never plain useDispatch/useSelector
export const useAppDispatch = () => useDispatch<AppDispatch>()
export const useAppSelector = <T>(selector: (state: RootState) => T) =>
  useSelector(selector)
```

---

## 9. Data Flow — Request Lifecycle

### Homepage Load

```
Member visits /en-US/
     │
     ▼ ── EDGE ──────────────────────────────────────────────
 middleware.ts
   ├─ Locale detected → 'en-US'
   ├─ CSP header injected
   └─ next() → App Router

     ▼ ── SERVER ────────────────────────────────────────────
 HomeContainer (async RSC)
   ├─ fetchHomepageContent(locale)
   │    → ContentStack Content stack: hero, carousel slides, blog content
   │    → unstable_cache with tag: 'tire-home-en-US'
   │
   ├─ getFeatureFlagState('license_plate_search', 'tires')
   │    → ContentStack Config stack: is LP tab enabled for locale?
   │
   └─ Returns JSX:
        <HeroSection tireSearchFormConfig={...} />   ← RSC; client boundary at TireSearchForm
        <DealsCarouselSection slides={...} />         ← Client (carousel needs browser events)
        <WhyBuySection content={...} />               ← RSC
        <TireInfoSliderSection slides={...} />        ← Client
        <BlogSection article={...} />                 ← RSC

     ▼ ── CLIENT ───────────────────────────────────────────
 ReduxProvider hydrates (store rehydrated from localStorage)
   ├─ vehicleSlice.year → null (new visitor) OR '2021' (returning visitor)
   ├─ tireCenterSlice.id → null OR 'WH-0481'
   │
 TireSearchForm mounts ('use client')
   ├─ reads Redux: tireCenterSlice.id
   ├─ if id is null → renders ZipCodeField
   ├─ if id present → ZipCodeField absent from DOM
   ├─ reads featureFlag (LP tab) → renders/suppresses License Plate tab
   └─ restores active tab from vehicleSlice.activeSearchTab
```

### Tire Search — Vehicle Flow

```
Member selects Vehicle tab → fills Year/Make/Model/Trim → submits
     │
 TireSearchForm dispatches:
   vehicleSlice.setVehicle({ year, make, model, trim })
   searchCriteriaSlice.clear()
     │
 if no Tire Center stored:
   uiSlice.openTireCenterLocator()
   → TireCenterLocator opens (standalone page on first-ever use, modal on subsequent)
   → Member searches by ZIP → Bing Maps renders markers + list
   → Member clicks "Set as My Tire Center"
   → tireCenterSlice.setTireCenter({ id, name, address })
   → TireCenterLocator closes
     │
 Next.js router.push('/sizeSelection?year=2021&make=Toyota&model=RAV4&trim=XLE')
     │
 SizeSelectionContainer (async RSC)
   → GET /api/tires/vehicle-catalog/sizes?year=2021&make=Toyota&model=RAV4&trim=XLE
   → Next.js API route → Spring Boot BFF → returns compatible sizes[]
   → Page renders with size options
```

### BFF API Client Pattern

All BFF calls are routed through Next.js API routes (never called directly from client components). This preserves server-side secrets, enables rate limiting, and allows MSW to intercept during development.

```ts
// src/services/bff/client.ts
const BFF_BASE_URL = process.env.BFF_BASE_URL  // never exposed to client bundle

export async function bffFetch<T>(
  path: string,
  options?: RequestInit
): Promise<T> {
  const response = await fetch(`${BFF_BASE_URL}${path}`, {
    ...options,
    headers: {
      'Content-Type': 'application/json',
      ...options?.headers,
    },
  })
  if (!response.ok) {
    throw new BFFError(response.status, path)
  }
  return response.json() as Promise<T>
}
```

---

## 10. Key Cross-Cutting Concerns

### 10.1 Internationalisation (i18n)

Identical to catalog search. See catalog search architecture §5.1.

| Aspect | Value |
|---|---|
| Locales | `en-US`, `en-CA`, `fr-CA` |
| Routing | `/[lang]/…` — locale prefix on all routes |
| Detection | `next-i18n-router` — URL prefix first, then `Accept-Language` |
| Config | `src/constants/language/index.ts` — copy from catalog search |
| fr-CA requirement | Legal obligation under Quebec Bill 96 — fr-CA strings must ship simultaneously with en-CA |
| License Plate tab | Suppressed for `en-CA` and `fr-CA` — feature flag + locale check |
| UI strings | Fetched from ContentStack `key_value_pair` entries — no hardcoded English strings in component code |

### 10.2 Content Management (ContentStack)

Identical two-stack pattern as catalog search. `services/content/` copied without modification.

| Stack | Purpose |
|---|---|
| Content Stack | Homepage content, carousel slides, info pages, labels |
| Config Stack | Feature flags, CSP, app configuration |

**Tires-specific ContentStack content types:**

| Content Type UID | Purpose |
|---|---|
| `tire_home_page` | Hero content, carousel slides, blog article |
| `tire_search_config` | Results per page, sort options, enabled filters |
| `tire_brand_page` | Brand landing page content |
| `tire_info_page` | FAQ, TireBasics, TireTips, Warranty, Glossary, Costco Advantage |
| `feature_toggle` | Feature flags — same as catalog search |
| `configuration_setting` | CSP, app config — same as catalog search |
| `key_value_pair` | UI labels and strings — same as catalog search |

### 10.3 Tire Center & Bing Maps

The Tire Center Locator uses **Bing Maps JavaScript API** — not Google Maps.

```
useTireCenterLocator hook ('use client')
  ├─ Loads Bing Maps JS script dynamically (lazy — only on Locator page/modal render)
  ├─ Initialises Microsoft.Maps.Map on container div ref
  ├─ On search submit:
  │    GET /api/tires/tire-centers?zip={zip}
  │    → Next.js API route → Spring Boot BFF → returns TireCenter[]
  │    → Places pushpin markers on map for each result
  │    → Populates TireCenterList with results
  ├─ On marker click:
  │    → Highlights corresponding TireCenterCard in list
  │    → Scrolls list to that card
  ├─ On "Set as My Tire Center" click:
  │    → Dispatches tireCenterSlice.setTireCenter({ id, name, address })
  │    → Persisted to localStorage
  │    → Header Tire Center link updates to show selected name
  └─ Scenario: no results → clears all map markers; shows "We were unable to find..." message
```

**Bing Maps API key** is set via `NEXT_PUBLIC_BING_MAPS_KEY` environment variable — never hardcoded.

**Modal vs standalone page:** The `TireCenterLocator` component accepts `isModalContent: boolean`. First-time selection renders as a full standalone page (`/TireCenterLocator`). Subsequent changes or access from the Schedule Appointment flow render inside a modal using the same component.

### 10.4 Authentication & Consent

Identical to catalog search. See catalog search architecture §5.5.

- **MSAL** via Forge — Azure AD / Costco member SSO
- **Transcend** — GDPR/CCPA consent management
- Provider components in root layout `<head>` — same pattern as catalog search

### 10.5 Analytics — Adobe Analytics

Adobe Analytics is implemented via Forge's `window.digitalData` layer — same pattern as catalog search. Populate on each page render.

Criteo OneTag is **not applicable** to the tires webapp (no sponsored product tiles in tire search results).

### 10.6 Feature Flags

Identical to catalog search — ContentStack Config stack entries of type `feature_toggle`. See catalog search architecture §5.7.

| Flag key | Purpose |
|---|---|
| `license_plate_search` | Enable/disable License Plate tab (en-US only by market; can also be toggled per-site) |
| `tire_center_locator_v2` | Feature flag for enhanced Tire Center Locator (Phase 2) |
| `schedule_appointment` | Enable Schedule Appointment modal CTA |

### 10.7 Observability

Identical to catalog search. Copy these files exactly:

- `src/instrumentation.ts` — OpenTelemetry setup
- `src/metrics.ts` — Prometheus counter/histogram declarations
- `src/metrics-server.ts` — standalone metrics HTTP server on port `:9200`
- `grafana/` — local Prometheus + Grafana stack

See catalog search architecture §5.4 for full details.

### 10.8 Caching Strategy

| Cache Layer | Technology | TTL | What is cached |
|---|---|---|---|
| Homepage | Next.js ISR | 1 hour | Rendered HTML — hero, carousels |
| Product Detail pages | Next.js ISR | 30 minutes | Tire product HTML |
| Info pages | Next.js ISR | 24 hours | FAQ, TireBasics, etc. |
| ContentStack data | `unstable_cache` (tagged) | Per call | CMS entry responses |
| Shared ISR cache | Redis (`@neshca/cache-handler`) | Same as ISR TTL | Shared across server instances |
| CDN cache | Cache-Tag headers | Until purged | Full pages at CDN edge |
| On-demand purge | `/api/revalidate` + ContentStack webhook | Immediate | Specific paths or tags |

PLP, Size Selection, and Tire Center Locator pages are **not cached** — they depend on real-time vehicle fitment and warehouse data.

---

## 11. Style Guide

> **⚠ Status: Work in Progress**
>
> The official tires webapp style guide is under development. This section will be updated when the style guide is finalised.
>
> Until then, all visual design decisions must reference:
> - `docs/tires-webapp/02-design/` — Mobile and desktop design comps for all pages
> - `@nxt-costco-com/forge-design-tokens` — Brand colours, spacing, typography (source of truth for all design tokens)
> - `@nxt-costco-com/forge-components` — Costco shared component library (use Forge components before building custom)
>
> **Interim rules (in effect until style guide is published):**
> - All CSS custom properties must reference Forge design tokens (e.g. `var(--forge-color-brand-primary)`)
> - No hardcoded hex colour values in component code
> - Global styles in `src/styles/global.scss` — no inline styles on any component
> - Component-level styles via MUI `sx` prop or co-located `styles.ts` with `styled()` — same as catalog search

---

## 12. TypeScript Patterns & Conventions

All conventions from the catalog search reference architecture apply without change. Key rules:

```ts
// ✅ Interfaces over types
interface TireSearchResult { ... }

// ✅ Named exports — no default exports for components
export function TireSearchForm({ isModalContent }: TireSearchFormProps) { ... }

// ✅ RORO pattern
function buildVehicleSearchParams({ year, make, model, trim }: VehicleArgs): URLSearchParams { ... }

// ✅ Early returns for guard conditions
function parseTireSize(input: string): TireSize | null {
  if (!input) return null
  const match = input.match(/^(\d{3})\/(\d{2})R(\d{2})$/i)
  if (!match) return null
  return { width: match[1], ratio: match[2], diameter: match[3] }
}

// ✅ const maps — no enums
const SearchMode = {
  vehicle: 'vehicle',
  licensePlate: 'licensePlate',
  tireSize: 'tireSize',
  itemNumber: 'itemNumber',
} as const
type SearchMode = typeof SearchMode[keyof typeof SearchMode]

// ❌ Never: any, relative imports > 1 level, default exports for components
```

See catalog search architecture §7 for the full conventions reference.

### Zod Validation — API Routes

All Next.js API route inputs validated with Zod before calling BFF:

```ts
// src/app/api/tires/vehicle-lookup/route.ts
const VehicleLookupSchema = z.object({
  plate: z.string().min(2).max(10).regex(/^[A-Z0-9 -]+$/i),
  state: z.string().length(2).regex(/^[A-Z]{2}$/),
})

export async function GET(request: NextRequest) {
  const parsed = VehicleLookupSchema.safeParse(
    Object.fromEntries(request.nextUrl.searchParams)
  )
  if (!parsed.success) {
    return NextResponse.json({ error: parsed.error.flatten() }, { status: 400 })
  }
  // call BFF with validated data only
}
```

---

## 13. Testing Strategy

Same tooling and patterns as catalog search. See catalog search architecture §8 for full details.

| Layer | Tool | What it tests |
|---|---|---|
| Unit | Vitest | Redux slices, util functions (tireSize, vehicleUtils), hooks |
| Component | Vitest + RTL | TireSearchForm, TireCenterLocator, DealsCarousel, modal components |
| Integration | Vitest + MSW | Components + BFF API routes via MSW handlers |
| E2E | Playwright | Homepage search flows, Tire Center Locator, error recovery |

### Priority test coverage (MVP 1)

The following must have unit tests before MVP 1 ships:

1. **`vehicleSlice`** — all actions: setVehicle, clearVehicle, setActiveTab
2. **`tireCenterSlice`** — setTireCenter, clearTireCenter
3. **`searchCriteriaSlice`** — setTireSize, toggleSameFrontRear, setItemNumber, clear
4. **`TireSearchForm`** — all 4 tabs render, validation errors, market-conditional tab suppression (License Plate hidden for en-CA/fr-CA), ZipCodeField conditional rendering
5. **`tireSize.ts`** — parseTireSize valid/invalid formats

---

## 14. API Routes

All routes under `src/app/api/`. Routes in the `tires/` subfolder proxy to the Spring Boot BFF.

| Route | Method | Inputs | BFF Call | Response |
|---|---|---|---|---|
| `/api/health` | GET | — | None | `200 OK` |
| `/api/build-health` | GET | — | None | `{ version, buildTime, gitSha }` |
| `/api/metrics` | GET | — | None | Prometheus text format |
| `/api/revalidate` | POST | `secret`, `path` | None | `{ revalidated: true }` |
| `/api/contentStack` | POST | Webhook payload | None | Cache purge trigger |
| `/api/tires/vehicle-lookup` | GET | `plate`, `state` | BFF `/vehicle/lookup` | `{ year, make, model, trim }` or error |
| `/api/tires/vehicle-catalog/makes` | GET | `year` | BFF `/vehicle/makes` | `{ makes: string[] }` |
| `/api/tires/vehicle-catalog/models` | GET | `year`, `make` | BFF `/vehicle/models` | `{ models: string[] }` |
| `/api/tires/vehicle-catalog/trims` | GET | `year`, `make`, `model` | BFF `/vehicle/trims` | `{ trims: string[] }` |
| `/api/tires/vehicle-catalog/sizes` | GET | `year`, `make`, `model`, `trim` | BFF `/vehicle/sizes` | `{ sizes: TireSize[] }` |
| `/api/tires/tire-centers` | GET | `zip` | BFF `/tire-centers/search` | `{ tireCenters: TireCenter[] }` |

**MSW during development:** All `tires/` routes are mocked by handlers in `src/mock/handlers.ts` using MSW v2. Stubs must match agreed BFF contract schemas — sign-off required before sprint 1.

---

## 15. Environment Variables

Declared and Zod-validated in `env.mjs`. Build fails on any undeclared or missing required variable.

```ts
// env.mjs — tire-specific variables (additions to catalog search base)
server: {
  // Spring Boot BFF
  BFF_BASE_URL: z.string().url(),

  // ContentStack — Tires Content stack
  CONTENTSTACK_TIRES_API_KEY: z.string().min(1),
  CONTENTSTACK_TIRES_DELIVERY_TOKEN: z.string().min(1),
  CONTENTSTACK_TIRES_ENVIRONMENT: z.string().min(1),
  CONTENTSTACK_TIRES_REGION: z.string().default('US'),

  // ContentStack — Config stack (shared with catalog search)
  CONTENTSTACK_CONFIG_API_KEY: z.string().min(1),
  CONTENTSTACK_CONFIG_DELIVERY_TOKEN: z.string().min(1),
  CONTENTSTACK_CONFIG_ENVIRONMENT: z.string().min(1),

  // Observability
  OTEL_EXPORTER_OTLP_ENDPOINT: z.string().url().optional(),

  // Cache
  REDIS_URL: z.string().optional(),

  // Site identification (feature flag scoping)
  SITE: z.string().optional(),
  NODE_ENV: z.enum(['development', 'production', 'test']).default('development'),
},
client: {
  NEXT_PUBLIC_BASE_PATH: z.string().optional(),
  NEXT_PUBLIC_BING_MAPS_KEY: z.string().min(1),  // Bing Maps — client-side API key
}
```

> **Security rule:** `BFF_BASE_URL` and all server-side secrets are never exposed to the client bundle. Only `NEXT_PUBLIC_*` variables reach the browser. Bing Maps key is `NEXT_PUBLIC_` because it is loaded by the Bing Maps JS script in the browser — this is standard practice; restrict key by domain in the Bing Maps developer portal.

---

## 16. Build, Deploy & Runtime

Identical to catalog search. See catalog search architecture §9 for full details.

```bash
# Development
npm run dev              # Next.js dev server + MSW active

# Type check (run separately from build)
npm run tsc

# Lint
npm run lint:oxlint

# Tests
npm run test             # Vitest with coverage
npm run e2e:headless     # Playwright headless

# Production build
npm run build            # next build → .next/standalone/

# Start
npm start                # Copies static assets + starts standalone server
```

Docker and PM2 configuration: copy `Dockerfile` and `ecosystem.config.js` from catalog search; rename app to `tires` in ecosystem config.

---

## 17. Path Aliases

Configured in `tsconfig.json` and mirrored in `vitest.config.mts`:

| Alias | Resolves to |
|---|---|
| `@/components/*` | `src/components/*` |
| `@/constants/*` | `src/constants/*` |
| `@/services/*` | `src/services/*` |
| `@/types/*` | `src/types/*` |
| `@/utils/*` | `src/utils/*` |
| `@/redux/*` | `src/redux/*` |
| `@/hooks/*` | `src/hooks/*` |
| `@/mock/*` | `src/mock/*` |

> **Rule:** Never use relative paths going up more than one level (`../../`). Use path aliases instead.

---

## 18. Quick Reference

| Question | Answer |
|---|---|
| Where does routing work? | `src/middleware.ts` + `src/app/[lang]/` |
| Where is client state managed? | `src/redux/slices/` + `src/redux/store.ts` |
| Where are BFF calls made? | `src/services/bff/` (server) + `src/app/api/tires/` (API routes) |
| Where is the Tire Search Form? | `src/components/TireSearchForm/` |
| Where is Tire Center Locator? | `src/components/TireCenterLocator/` (component) + `src/app/.../TireCenterLocator/page.tsx` (standalone page) |
| Where are BFF mocks for dev? | `src/mock/handlers.ts` |
| Where are ContentStack calls? | `src/services/content/` (copied from catalog search) |
| Where are feature flags resolved? | `src/services/content/featureFlag.ts` |
| Where is the Bing Maps hook? | `src/hooks/useTireCenterLocator.ts` |
| Where are design references? | `docs/tires-webapp/02-design/` |
| Where is the style guide? | `docs/tires-webapp/01-architecture/frontend-architecture.md` §11 — **WIP** |
| How does CSP work? | Same as catalog search — `src/instrumentation.ts` + `src/middleware.ts` |
| How does i18n work? | `src/middleware.ts` + `src/constants/language/index.ts` — same as catalog search |
| Catalog search reference | `docs/tires-webapp/01-architecture/catalog-search-architecture.md` |
