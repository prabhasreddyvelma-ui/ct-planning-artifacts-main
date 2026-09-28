# Costco Search & Catalog — Architecture & Repo Structure Guide

> **Version:** 2.0 — Full deep-dive edition
>
> **Purpose:** This document explains every layer of the `gdx-ux-cnsw-search-catalog` repository so a developer can:
> 1. Understand **why** each folder/file exists and how it connects to others.
> 2. Scaffold a **new, similar React/Next.js 14 application** — specifically the `tires.costco.com` redesign — following exactly the same patterns.
> 3. Know which **conventions, patterns, and rules** to respect when adding new features.
> 4. Understand the **live site** at `tires.costco.com` well enough to map each UI section to a concrete React component.

---

## Table of Contents

1. [High-Level Architecture](#1-high-level-architecture)
2. [Technology Stack](#2-technology-stack)
3. [Root-Level Files — Deep Dive](#3-root-level-files--deep-dive)
4. [Folder-by-Folder Breakdown](#4-folder-by-folder-breakdown)
   - 4.1 [src/app — Next.js 14 App Router](#41-srcapp--nextjs-14-app-router)
   - 4.2 [src/components — Presentational UI](#42-srccomponents--presentational-ui)
   - 4.3 [src/containers — Smart / Page-Level Components](#43-srccontainers--smart--page-level-components)
   - 4.4 [src/services — Business Logic & External APIs](#44-srcservices--business-logic--external-apis)
   - 4.5 [src/hooks — Shared Custom React Hooks](#45-srchooks--shared-custom-react-hooks)
   - 4.6 [src/providers — Context / Theme Providers](#46-srcproviders--context--theme-providers)
   - 4.7 [src/constants — App-Wide Constants](#47-srcconstants--app-wide-constants)
   - 4.8 [src/types — TypeScript Type Definitions](#48-srctypes--typescript-type-definitions)
   - 4.9 [src/utils — Pure Utility Functions](#49-srcutils--pure-utility-functions)
   - 4.10 [src/styles — Global SCSS](#410-srcstyles--global-scss)
   - 4.11 [src/mock — MSW Test Mocks](#411-srcmock--msw-test-mocks)
   - 4.12 [src/actions — Next.js Server Actions](#412-srcactions--nextjs-server-actions)
   - 4.13 [src/app/api — API Route Handlers](#413-srcappapi--api-route-handlers)
   - 4.14 [public/ — Static Assets](#414-public--static-assets)
   - 4.15 [grafana/ — Observability Stack](#415-grafana--observability-stack)
5. [Key Cross-Cutting Concerns](#5-key-cross-cutting-concerns)
   - 5.1 [Internationalisation (i18n)](#51-internationalisation-i18n)
   - 5.2 [Content Management (ContentStack)](#52-content-management-contentstack)
   - 5.3 [Search Providers — GRS vs LucidWorks Deep Dive](#53-search-providers--grs-vs-lucidworks-deep-dive)
   - 5.4 [Observability (OpenTelemetry + Prometheus)](#54-observability-opentelemetry--prometheus)
   - 5.5 [Authentication & Consent](#55-authentication--consent)
   - 5.6 [Analytics — Adobe Analytics + OneTag + Criteo](#56-analytics--adobe-analytics--onetag--criteo)
   - 5.7 [Feature Flags — Implementation Details](#57-feature-flags--implementation-details)
   - 5.8 [Content Security Policy (CSP)](#58-content-security-policy-csp)
   - 5.9 [Cookie System](#59-cookie-system)
   - 5.10 [Caching Strategy](#510-caching-strategy)
6. [Data Flow — Complete Request Lifecycle](#6-data-flow--complete-request-lifecycle)
7. [TypeScript Patterns & Conventions](#7-typescript-patterns--conventions)
8. [Testing Strategy — Detailed Patterns](#8-testing-strategy--detailed-patterns)
9. [Build, Deploy & Runtime](#9-build-deploy--runtime)
10. [Path Aliases](#10-path-aliases)
11. [tires.costco.com — Live Site Analysis](#11-tirescostcocom--live-site-analysis)
    - 11.1 [Site Navigation Structure](#111-site-navigation-structure)
    - 11.2 [Home Page Sections](#112-home-page-sections)
    - 11.3 [Search Widget — 4 Search Modes](#113-search-widget--4-search-modes)
    - 11.4 [Tire Results Page](#114-tire-results-page)
    - 11.5 [Tire Detail Page](#115-tire-detail-page)
    - 11.6 [Information Pages](#116-information-pages)
    - 11.7 [The Costco Advantage](#117-the-costco-advantage)
12. [Creating the Tires Repo — Complete Guide](#12-creating-the-tires-repo--complete-guide)
    - 12.1 [Scaffold Checklist](#121-scaffold-checklist)
    - 12.2 [Environment Variables](#122-environment-variables)
    - 12.3 [Recommended Folder Structure](#123-recommended-folder-structure)
    - 12.4 [Key Files — Code Templates](#124-key-files--code-templates)
    - 12.5 [Tire-Specific Component Designs](#125-tire-specific-component-designs)
    - 12.6 [API Routes Required](#126-api-routes-required)
    - 12.7 [ContentStack Content Types Needed](#127-contentstack-content-types-needed)
    - 12.8 [Feature-to-File Mapping](#128-feature-to-file-mapping)
13. [Coding Rules & Conventions](#13-coding-rules--conventions)
14. [Quick Reference — Where to Find Things](#14-quick-reference--where-to-find-things)

---

## 1. High-Level Architecture

```
Browser / CDN
  │
  ▼
Next.js Edge Middleware  (src/middleware.ts)
  │  ┌─ i18n routing: /[lang]/s/…  /[lang]/c/…
  │  ├─ URL rewrites: /CatalogSearch → /s  |  /s?dept=X → category path
  │  ├─ Cache-tag response headers (CDN purge groups)
  │  ├─ CSP header injection from process.env.CONTENT_SECURITY_POLICIES
  │  ├─ Queue-It bot/load management check
  │  └─ Prometheus edge metrics (inbound + latency counters)
  ▼
Next.js App Router  (src/app/)
  │
  ├─ Server Components  (RSC — default, zero client JS)
  │    ├─ Fetches ContentStack CMS layout config
  │    ├─ Fetches search rules (keyword → search config mapping)
  │    └─ Passes data as props to client subtree
  │
  └─ Client Components  ('use client')
       ├─ SearchProvider — React context holding search state
       ├─ Filter interaction, pagination, sort
       └─ Analytics event firing
       
       ┌──────────────────────────────────────────────────────┐
       │  External Services Called at Runtime                 │
       │                                                      │
       │  ┌─────────────────┐   ┌──────────────────────────┐  │
       │  │  ContentStack   │   │  GRS (Global Relevance   │  │
       │  │  CMS (US / CA)  │   │  Service) — internal     │  │
       │  │  Config stack   │   │  Costco search engine    │  │
       │  │  Content stack  │   └──────────────────────────┘  │
       │  └─────────────────┘   ┌──────────────────────────┐  │
       │                        │  LucidWorks — external   │  │
       │  ┌─────────────────┐   │  SaaS search (A/B test)  │  │
       │  │  Redis Cache    │   └──────────────────────────┘  │
       │  │  (ISR handler)  │                                  │
       │  └─────────────────┘   ┌──────────────────────────┐  │
       │                        │  Adobe Analytics         │  │
       │  ┌─────────────────┐   │  Criteo OneTag           │  │
       │  │  Prometheus     │   │  LogRocket (session)     │  │
       │  │  metrics server │   └──────────────────────────┘  │
       │  │  port :9200     │                                  │
       │  └─────────────────┘                                  │
       └──────────────────────────────────────────────────────┘
```

### Rendering Strategy

| Page Type | Strategy | Revalidation |
|---|---|---|
| Search results (`/s/[id]`) | ISR | `revalidate = 10800` (3 hours) |
| Category pages (`/c/[id]`) | ISR | `revalidate = 10800` (3 hours) |
| New CMS entries | On-demand ISR (`dynamicParams = true`) | Webhook → `/api/revalidate` |
| Static locale params | `generateStaticParams` | Build time |
| API routes | Dynamic (always fresh) | n/a |

### Key Architectural Decisions

1. **No global client state library** (no Redux, no Zustand). State lives in React Context (`SearchProvider`) and URL query params. This enables deep-link sharing of any search state.
2. **CMS-driven everything**. Page layout, facet config, search rules, CSP, feature flags — all come from ContentStack. Developers rarely need to deploy code to change page behaviour.
3. **Dual search backends**. GRS and LucidWorks can run in parallel for A/B testing. The `SearchProvider` selects the backend at runtime based on cookies/query params.
4. **RSC-first**. Every component defaults to a Server Component. Only add `'use client'` when browser APIs (event handlers, hooks like `useState`) are required.
5. **Standalone output**. The Next.js `output: 'standalone'` mode produces a minimal Docker image that doesn't need `node_modules` at runtime.

---

## 2. Technology Stack

| Layer | Library | Version | Why |
|---|---|---|---|
| Framework | Next.js (App Router) | ^14.2.35 | ISR, RSC, middleware, standalone output |
| Language | TypeScript | 5 (strict) | `noImplicitAny`, `noUncheckedIndexedAccess` |
| UI Primitives | MUI | v6 | Grid, Box, styling via Emotion |
| Design System | `@nxt-costco-com/forge-components` | ^3.8.1 | Costco shared component library |
| Design Tokens | `@nxt-costco-com/forge-design-tokens` | ^3.0.2 | Brand colours, spacing, typography |
| Digital Components | `@nxt-costco-com/forge-digital-components` | ^3.14.4 | Header, Footer, ProductTile, Analytics providers |
| Icons | `@nxt-costco-com/forge-icons` | ^2.0.3 | SVG icon set |
| CMS | ContentStack SDK | ^3.25.1 | Headless CMS — drives all page config |
| Search (primary) | GRS | internal | Costco's internal relevance engine |
| Search (A/B) | LucidWorks | external | SaaS search for comparison testing |
| i18n | `next-i18n-router` | ^5.4.2 | URL-prefix locale routing |
| State | React Context | built-in | `SearchProvider` for search state |
| Forms | Zod + `react-hook-form` | 3.22.4 | Type-safe form validation |
| Styling | SCSS modules + Emotion | — | Global SCSS + component-level Emotion |
| Session tracking | LogRocket | ^7.0.0 | Session replay for debugging |
| Testing (unit) | Vitest | — | Vite-native, fast, no separate config |
| Testing (component) | React Testing Library | ^14.3.1 | DOM-based component tests |
| Testing (API mock) | MSW | — | Mock Service Worker |
| Testing (E2E) | Playwright | ^1.40.0 | Browser automation tests |
| Observability | OpenTelemetry + prom-client | ^1.30.0 / ^15.1.3 | Traces + Prometheus metrics |
| Logging | Pino | ^9.6.0 | Structured JSON logging |
| Session cache | Redis via `@neshca/cache-handler` | ^1.9.0 | ISR shared cache across instances |
| Auth | MSAL (Microsoft Authentication Library) | via Forge | Azure AD / Costco member auth |
| Consent | Transcend | via Forge | GDPR / CCPA consent management |
| Ads | Criteo | via Forge | Sponsored product tiles in search grid |
| Analytics | Adobe Analytics | via Forge | `window.digitalData` + AJO personalisation |
| HTTP client | undici (global dispatcher) | ^6.21.3 | Outbound request metrics + logging |
| Env validation | `@t3-oss/env-nextjs` | 0.7.1 | Zod-validated env vars at startup |
| Bundle analysis | `@next/bundle-analyzer` | ^14.2.24 | `npm run analyze` |
| Linting | oxlint (Rust) | — | Primary linter — `npm run lint:oxlint` |

### Node & Runtime Requirements

```
Node.js >= 22.11.0   (enforced via package.json "engines")
Redis               (required for ISR cache in production)
Docker              (for containerised deployment)
PM2                 (process manager — ecosystem.config.js)
```

---

## 3. Root-Level Files — Deep Dive

| File | Purpose | Notes |
|---|---|---|
| `next.config.mjs` | Next.js configuration | Fetches CSP from ContentStack at build time as fallback; webpack aliases for `node:` protocol (`node:assert` → `assert`, etc.); bundle analyser toggle via `ANALYZE=true` |
| `env.mjs` | Type-safe environment variables | Uses `@t3-oss/env-nextjs` + Zod. **Every** env var must be declared here — the build fails if an undeclared var is accessed |
| `tsconfig.json` | TypeScript configuration | `strict: true`, `noImplicitAny: true`, `noUncheckedIndexedAccess: true`, path aliases, `moduleResolution: bundler` |
| `tsconfig.test.json` | Test-specific TypeScript config | Looser settings for test files; references `vitest/globals` and `@testing-library/jest-dom` types |
| `vitest.config.mts` | Vitest test runner configuration | Loads `.env` + `.env.local` files dynamically; sets up vite-tsconfig-paths and `@vitejs/plugin-react` |
| `vitest.setup.ts` | Vitest global setup file | Starts and tears down the MSW server before/after each test suite |
| `jest.setup.js` | Legacy Jest setup | Kept for any remaining Jest-based tests; imports `@testing-library/jest-dom` |
| `playwright.config.ts` | E2E test configuration | Configures browser projects, base URL, trace collection |
| `ecosystem.config.js` | PM2 process manager config | Defines app name, script (`start.js`), instance count, env vars for production runtime |
| `start.js` / `start.sh` | Custom production server | Copies `build/static` and `public/` into `build/standalone/` before starting Node server |
| `Dockerfile` | Multi-stage Docker build | Stage 1: install deps; Stage 2: build; Stage 3: minimal `node:22-alpine` runtime image |
| `oxlintrc.json` | Rust-based linter config | Primary linter — significantly faster than ESLint; run with `npm run lint:oxlint` |
| `global.d.ts` | Global TypeScript declarations | Declares global types that don't belong to any specific module (e.g. `window.digitalData`) |
| `test-globals.ts` | Test global type declarations | Vitest global type augmentations |

### `next.config.mjs` — Key Configuration Details

```js
// CSP fetched from ContentStack at build time (fallback only)
// Primary CSP is loaded at runtime in instrumentation.ts

// webpack aliases solve the "node: protocol" issue with undici
config.resolve.alias = {
  'node:assert': 'assert',
  'node:util': 'util',
  ...
}

// ISR configuration
output: 'standalone'          // Minimal Docker output
staticPageGenerationTimeout: 180  // 3 minutes for slow CMS fetches
```

### `env.mjs` — Environment Variable Categories

All variables are validated against a Zod schema at startup. Missing required vars crash the build/server immediately, preventing silent runtime failures.

| Category | Variables |
|---|---|
| ContentStack (Content) | `CONTENTSTACK_API_KEY`, `CONTENTSTACK_DELIVERY_TOKEN`, `CONTENTSTACK_ENVIRONMENT`, `CONTENTSTACK_REGION`, `CONTENTSTACK_BRANCH` |
| ContentStack (Config) | `CONTENTSTACK_CONFIG_API_KEY`, `CONTENTSTACK_CONFIG_DELIVERY_TOKEN`, `CONTENTSTACK_CONFIG_ENVIRONMENT` |
| Search (GRS) | `GRS_BASE_URL`, `GRS_API_KEY` |
| Search (LucidWorks) | `LW_BASE_URL`, `LW_API_KEY` |
| App | `NEXT_PUBLIC_BASE_PATH`, `SITE`, `NODE_ENV`, `ANALYZE` |
| Observability | `OTEL_EXPORTER_OTLP_ENDPOINT` |
| Cache | `REDIS_URL` |

---

## 4. Folder-by-Folder Breakdown

### 4.1 `src/app` — Next.js 14 App Router

```
src/app/
├── _lib/
│   └── types.ts                   # LocalizedLayoutProps and other shared layout types
│
├── [lang]/                        # Dynamic locale segment (en-US, en-CA, fr-CA)
│   │
│   ├── (customers)/               # Route GROUP — public storefront (no URL impact)
│   │   ├── layout.tsx             # ROOT layout: <html lang>, <head> scripts
│   │   │                          #   Injects: OpinionLab, ConsentScripts,
│   │   │                          #   BrowseContextProvider, TranscendIntegrationManager,
│   │   │                          #   ConsentManagement, SalesTargeting, QueueIt
│   │   │                          #   revalidate = NOT SET (inherits from children)
│   │   │
│   │   ├── s/                     # SEARCH pages
│   │   │   ├── layout.tsx         # Search layout (revalidate = 10800)
│   │   │   │                      #   Providers: AppRouterCacheProvider, CustomerTelemetry,
│   │   │   │                      #   ThemeRegistry, QueryParamsProvider, MSALContextProvider,
│   │   │   │                      #   BrowseContextProvider, CartContextProvider,
│   │   │   │                      #   GroceryProvider, AgentProvider, ImageConfigProvider,
│   │   │   │                      #   ProductTileConfigProvider, ExternalPopupConfigProvider,
│   │   │   │                      #   StwInfoContextProvider, AJOPersonalizationProvider
│   │   │   │                      #   Components: KMSI, Header (from Forge), FooterFull,
│   │   │   │                      #   ErrorServiceUnavailable, ToggleChildrenBySearchParams
│   │   │   │
│   │   │   ├── no-results.tsx     # Rendered when search returns 0 results
│   │   │   ├── constants.ts       # Search-page-level constants
│   │   │   │
│   │   │   └── [contentStackEntryId]/
│   │   │       ├── page.tsx       # Search results page — calls MainContainer
│   │   │       │                  # Receives: params.lang, params.contentStackEntryId
│   │   │       │                  #           searchParams (keyword, page, sort, refine)
│   │   │       └── [categoryPageId]/
│   │   │           └── page.tsx   # Category drill-down — sub-category pages
│   │   │
│   │   ├── c/                     # CATEGORY pages (CLP — Category Landing Pages)
│   │   │   ├── layout.tsx         # Category layout (thin — same provider stack)
│   │   │   └── [pageId]/
│   │   │       └── page.tsx       # CLP page — calls MainContainer with category config
│   │   │
│   │   └── admin/                 # Internal admin tools (gated — not customer-facing)
│   │
│   └── (businessUsers)/           # Route GROUP — B2B / internal users
│       ├── layout.tsx             # Thin layout without full storefront providers
│       ├── not-found.tsx          # Custom 404 for business routes
│       └── livepreview/
│           └── page.tsx           # ContentStack Live Preview sandbox
│
└── api/                           # API Route Handlers (server-only, never bundled)
    ├── health/route.ts            # GET → 200 OK (K8s liveness probe)
    ├── build-health/route.ts      # GET → { sha, version, buildTime }
    ├── metrics/route.ts           # GET → Prometheus text exposition format
    ├── contentStack/route.ts      # POST → CMS webhook receiver + purge trigger
    ├── revalidate/route.ts        # POST → On-demand ISR revalidation (secret-gated)
    ├── findSearchRuleId/route.ts  # GET → keyword → contentStackEntryId
    ├── getAllSearchRules/route.ts  # GET → full list of search routing rules
    ├── getAllCategoryPageRoutes/   # GET → seed data for generateStaticParams
    ├── getFeatureFlagState/       # GET → debug endpoint for flag inspection
    └── crossRegionCall/           # POST → replicates revalidation to other regions
```

#### Naming Conventions in App Router

| Pattern | Meaning | Example |
|---|---|---|
| `(groupName)/` | Route group — nests layout but adds nothing to URL | `(customers)/s/…` renders at `/s/…` |
| `[param]/` | Dynamic segment — value accessible via `params.param` | `[lang]` → `params.lang = 'en-US'` |
| `layout.tsx` | Layout wrapper — renders for all children | Shared Header + Footer |
| `page.tsx` | Leaf page — rendered at exact path | Actual search results view |
| `_lib/` | Collocated private files — not treated as routes | Shared types, helpers |

#### `generateStaticParams` Pattern

Every layout that needs locale-specific static paths exports `generateStaticParams`:

```ts
// src/app/[lang]/(customers)/layout.tsx
export async function generateStaticParams() {
  return i18nConfig.locales.map(lang => ({ lang }))
  // → [{ lang: 'en-US' }, { lang: 'en-CA' }, { lang: 'fr-CA' }]
}
```

This seeds the build with 3 static locale paths. Combined with `dynamicParams = true`, new CMS entries are generated on-demand after the first visit.

---

### 4.2 `src/components` — Presentational UI

**Rules for every component folder:**
```
src/components/ComponentName/
├── index.tsx          # Main export — always a named export
├── index.test.tsx     # Vitest unit test
├── types.ts           # Local TypeScript interfaces (optional)
├── styles.ts          # MUI sx objects or styled-components (optional)
├── constants.ts       # Component-local constants (optional)
└── views/             # Sub-view variants, e.g. Mobile.view.tsx (optional)
```

**Complete component catalogue:**

| Component | Type | What it renders | Key props/hooks |
|---|---|---|---|
| `SearchResultsGrid` | Client | Product tile grid — injects Criteo ads at configured positions; shows skeleton on load | `useSearchContext()`, Criteo `useCriteo()` |
| `SearchResultsFacets` | Client | Left-rail filter panel containing all `FilterGroup` components | `useSearchContext()`, `useRefinement()` |
| `FilterGroup` | Client | Single collapsible accordion filter group with checkboxes | `facet`, `onRefine` callback |
| `FilterGroup/ProgramEligibility` | Client | Special filter for program eligibility (ShipIt, InWarehouse, 2DayDelivery) | Custom facet handling |
| `FilterGroup/ProgramEligibilityV2` | Client | Updated version with additional display logic | — |
| `FilterPopup` | Client | Mobile full-screen filter drawer (renders all FilterGroups) | `useCheckScreen()` for mobile detection |
| `SelectedFilters` | Client | Row of active-filter chips with remove buttons | `useQueryString()` |
| `ClearAllFilters` | Client | "Clear All" button — resets all refinements in URL | `useQueryString()` |
| `SortSelector` | Client | Sort-by dropdown (Relevance, Price Asc/Desc, Rating) | URL-based sort param |
| `SearchResultsPagination` | Client | Prev/Next + numbered page controls | `useSearchContext()` for total count |
| `ResultsCount` | Client | "Showing X–Y of Z results for [keyword]" | `useSearchContext()` |
| `BreadCrumb` | Server | Category breadcrumb trail from CMS data | ContentStack entry data |
| `DepartmentList` | Server | Left-nav department link list | ContentStack entry data |
| `DisplayGroup` | Server | CMS-driven content block (text, images, buttons) | ContentStack composer entry |
| `DisplayGroupList` | Server | Renders a list of `DisplayGroup` items | ContentStack composer data |
| `QueryControls` | Client | Search keyword input / "did you mean" controls | URL query params |
| `Headings` | Server | Accessible `<h1>`–`<h6>` hierarchy from CMS | ContentStack data |
| `ErrorBoundary` | Client | React error boundary — catches render errors | `fallback` prop |
| `ErrorServiceUnavailable` | Server | 503-style error page with retry messaging | — |
| `NotFoundUI` | Server | 404 page content | — |
| `Analytics/SearchPageDigitalDataLayer` | Client | Populates `window.digitalData` on search render | `useSearchContext()` |
| `AnalyticsObserver` | Client | `MutationObserver` watches DOM changes → fires analytics | — |
| `Adobe/AdobeTarget` | Server | Injects Adobe Target personalisation `<script>` | locale, page type |
| `OneTag` | Server | Criteo OneTag `<script>` pixel | — |
| `OneTag/OneTagSearchPageEventScript` | Client | Fires Criteo event on search result render | `useSearchContext()` |
| `KMSI` | Client | "Keep Me Signed In" modal (Microsoft MSAL) | — |
| `CookieBasedContentLoader` | Client | Conditionally renders children based on cookie value | `cookieName`, `cookieValue` |
| `CostcoGroceryLabel` | Server | "Costco Grocery" badge shown on eligible product tiles | — |
| `Feedback/OpinionLab` | Server | Injects OpinionLab survey `<script>` from `public/` | `locale` prop selects US/CA script |
| `LivePreview` | Client | ContentStack live-preview overlay + highlight mode | Live preview hash |
| `Legacy` | Server | Backwards-compatible wrapper for legacy URL patterns | — |
| `TranscendOverride` | Client | Consent override UI (Transcend) | — |
| `FilterAwareVisibilityToggle` | Client | Shows/hides content when specific filters are active | `filterKey`, `filterValue` |
| `ToggleChildrenBySearchParams` | Client | Renders children only when URL params match condition | `param`, `value` |
| `CenteredBox` | Server | Layout utility — MUI Box with centering styles | — |
| `SearchResultRegion` | Client | ARIA `role="region"` wrapper for screen reader announcement | `aria-label` |

---

### 4.3 `src/containers` — Smart / Page-Level Components

Containers **own page-level state** and **orchestrate** multiple components. They are the bridge between CMS data (RSC) and interactive client components.

```
src/containers/
├── main/                              # Primary search results page orchestrator
│   ├── index.tsx                      # MainContainer — async RSC
│   ├── LeftColumn.tsx                 # Facets sidebar
│   ├── LeftColumn.test.tsx
│   ├── MainColumn.tsx                 # Product grid + pagination + results count
│   ├── NoResultsComponent.tsx         # Zero-results state with suggestions
│   ├── ResultsAnnouncer.tsx           # ARIA live-region for screen reader announcements
│   ├── SearchAdTargetingProvider.tsx  # Criteo ad targeting context
│   ├── SearchAdTargetingProvider.test.tsx
│   ├── SearchVisibilityToggle.tsx     # Feature-flag gated visibility experiment
│   ├── SearchVisibilityToggle.test.tsx
│   ├── CriteoProviderWrapper.tsx      # Initialises Criteo SDK with config
│   ├── utils.ts                       # getPageConfigs, mergeComposerData, isFilterEnabled
│   ├── utils.test.ts
│   ├── styles.ts                      # MUI sx / styled wrappers for layout
│   └── constants.ts                   # A11Y strings, display style constants, token names
├── common/                            # Shared layout containers
│   ├── footer-full.tsx                # Full footer (Forge Footer component)
│   └── header.tsx                     # (if customised beyond Forge default)
└── compareProducts/                   # Product comparison drawer (up to 3 products)
```

#### `MainContainer` — The Orchestrator

`MainContainer` is the most important component in the codebase. It is an **async Server Component** (no `'use client'`).

**Execution flow:**
```
MainContainer(async RSC)
  │
  ├─ 1. fetchPageRules(contentStackEntryId, locale)
  │      → Returns: SearchRuleEntry | null
  │      → Determines: keyword, page type, GRS/LW config
  │
  ├─ 2. getPageConfigs(entry, locale)
  │      → Returns: SearchPageConfigProperties
  │      → Contains: composerData, facetConfig, featureFlags, siteSettings
  │
  ├─ 3. getComposerDataFromEntry(entry) + mergeComposerData()
  │      → Returns: ComposerData (layout blocks from CMS)
  │
  ├─ 4. isFilterEnabled(pageConfig) → boolean
  │
  ├─ 5. isSearchEnabled(pageConfig) → boolean
  │
  └─ 6. Returns JSX:
         <SearchProvider config={pageConfig}>        // Client boundary
           <SearchAdTargetingProvider>               // Client
             <CriteoProviderWrapper>                 // Client
               <SearchVisibilityToggle>              // Client
                 <LeftColumn />                      // Client (facets)
                 <MainColumn />                      // Client (grid)
               </SearchVisibilityToggle>
             </CriteoProviderWrapper>
           </SearchAdTargetingProvider>
         </SearchProvider>
```

#### `utils.ts` — Key Functions

| Function | Purpose |
|---|---|
| `getPageConfigs(entry, locale)` | Fetches the full page config from CMS, including facet config, feature flags, GRS config |
| `isFilterEnabled(config)` | Returns `true` if the filter panel should be shown for this page |
| `isSearchEnabled(config)` | Returns `true` if the search box is active |
| `mergeComposerData(base, override)` | Merges CMS composer slot data — override wins on conflicts |
| `getComposerDataFromEntry(entry)` | Extracts the composer data array from a raw CMS entry |

---

### 4.4 `src/services` — Business Logic & External APIs

```
src/services/
├── logger.ts                        # Pino logger singleton — structured JSON output
│
├── content/                         # ContentStack CMS layer
│   ├── stackClient.ts               # SDK initialisation — creates US and CA stacks
│   │                                # Uses undici LoggingDispatcher for outbound metrics
│   ├── config.lib.ts                # Generic CMS fetcher helpers
│   │                                # getConfigurationSettingByDevelopmentKey()
│   │                                # getConfigurationSettingByConfigKey()
│   ├── featureFlag.ts               # Feature flag resolution
│   │                                # getFeatureFlagState(featureKey, appName)
│   ├── livePreviewClient.ts         # Live-preview stack (bypasses cache)
│   ├── business.client.ts           # B2B-specific CMS queries
│   ├── unstable_cache.ts            # Wraps Next.js unstable_cache with typed helpers
│   ├── constants.ts                 # Content type UIDs, config keys
│   └── data/                        # Content-type-specific data fetchers
│       ├── index.ts                 # findByEntryId, getKeyValue, etc.
│       └── [contentType].ts         # Per-content-type fetch utilities
│
└── search/                          # Search engine abstraction layer
    ├── SearchProvider.tsx            # 'use client' — React context + all search logic
    ├── SearchProvider.test.tsx
    ├── searchRules.ts               # fetchPageRules() — keyword → search config
    ├── searchRules.test.ts
    ├── types.ts                     # SearchResult, SearchServiceProps, WarehouseCookieValue
    ├── utils.ts                     # URL ↔ search param conversion helpers
    │                                # getCurrentPage(), getNumberOfPages()
    │                                # getFacetKeyFromQueryParamKey()
    ├── utils.test.ts
    ├── constants.ts                 # DEFAULT_USER_STATE, provider keys
    └── services/
        ├── Grs.ts                   # GRS query builder + response parser
        │                            # GrsQuery(), isSearchResultGRS(), getAdTargetingFlags()
        ├── GRS.types.ts             # SearchResultGRS, WarehouseSearchParams, AdTargetingFlags
        ├── grs.constants.ts         # GRS facet key constants
        ├── LucidWorks.ts            # LucidWorks query builder + response parser
        │                            # LucidWorksQuery(), isSearchResultLW(), isWAPOEnabledForWarehouse()
        └── __mock__/                # Fixture JSON files for tests
```

#### `stackClient.ts` — ContentStack Initialisation

Two stacks are initialised per region:
- **Content Stack** — actual page content (search pages, category pages, labels)
- **Config Stack** — configuration data (feature flags, search rules, CSP, facet config)

The `configureStacks()` function is called once per server startup in `layout.tsx`. It sets up a custom `undici` `LoggingDispatcher` that:
- Records outbound HTTP metrics (counter + duration histogram)
- Logs request failures with sanitised error information (no URLs in labels to prevent high cardinality)

#### `SearchProvider.tsx` — The Search Engine

This is the **most complex client component** in the repo. It:

1. Reads all URL query params via `useQueryParams()` (keyword, refine, page, sort, etc.)
2. Reads cookies: warehouse location (`C_LOC`), postal code (`invCheckPostalCode`), provider preference
3. Selects backend: GRS (default) or LucidWorks (A/B — based on `provider` cookie or query param)
4. Constructs the search query using `GrsQuery()` or `LucidWorksQuery()`
5. Fires the search request
6. Parses the response into a normalised `SearchResult` shape
7. Exposes all state via `useSearchContext()` hook

**Context values exposed:**
```ts
{
  searchResults,        // Product tiles
  facets,               // Available filters
  totalResults,         // Total count
  currentPage,
  resultsPerPage,
  isLoading,
  pageType,             // 'search' | 'category' | 'noResults'
  searchService,        // 'grs' | 'lw'
  keyword,
  breadcrumbs,
  adTargetingFlags,     // Criteo targeting data
}
```

#### `featureFlag.ts` — Flag Resolution Logic

```ts
getFeatureFlagState(featureKey: string, appName: AppName): Promise<FeatureFlagState>
```

- Fetches the flag entry from ContentStack Config stack
- If `enabled_applications` is empty → flag applies to ALL sites
- If `enabled_applications` is set → flag only applies if `process.env.SITE` matches
- `AppName` is `'browse' | 'search'` — maps to site environment identifiers like `browse-cabc`

---

### 4.5 `src/hooks` — Shared Custom React Hooks

All hooks are `'use client'` — they use browser APIs or React state.

| Hook | File | Purpose | Details |
|---|---|---|---|
| `useCheckScreen` | `useCheckScreen.ts` | Responsive breakpoint detection | Returns `{ isMobile, isTablet, isDesktop }` — used to switch between `FilterGroup` (desktop) and `FilterPopup` (mobile) |
| `useQueryString` | `useQueryString.ts` | Read/write URL query params | Wraps Next.js `useSearchParams` + `useRouter`; batches multiple param updates into one history push to avoid re-render storms |
| `useRefinement` | `useRefinement.ts` | Toggle filter refinements in URL | Calls `updateRefinements()` which encodes/decodes the `refine` query param in the format `facetKey=value\|\|facetKey2=value2` |

#### `useRefinement` — Refinement Encoding

The refinement state lives entirely in the URL. Each active filter is encoded as:

```
GRS:  /s/[id]?refine=item_category=Laptops||item_location_availability=in+stock
LW:   /s/[id]?refineByLW=item_category%3DLaptops||item_rating_value%3D4
```

The delimiter `||` (`REFINE_QUERY_DELIMITER` constant) separates multiple active filters. The `=` vs `-` separator differs between GRS and LucidWorks, handled by `GRS_REFINE_FILTER_KEY_VALUE_DELIMITER` and `LW_REFINE_FILTER_KEY_VALUE_DELIMITER` constants.

---

### 4.6 `src/providers` — Context / Theme Providers

```
src/providers/
└── mui/
    └── ThemeRegistry.tsx    # MUI emotion cache + theme provider (SSR-safe)
```

`ThemeRegistry` wraps both `AppRouterCacheProvider` (Emotion SSR cache) and MUI's `ThemeProvider`. It prevents the "flash of unstyled content" that occurs when Emotion styles haven't hydrated yet in Next.js App Router. This must wrap any component that uses MUI's `sx` prop or `styled()`.

**Usage in layout:**
```tsx
// src/app/[lang]/(customers)/s/layout.tsx
<AppRouterCacheProvider>
  <ThemeRegistry>
    {children}
  </ThemeRegistry>
</AppRouterCacheProvider>
```

---

### 4.7 `src/constants` — App-Wide Constants

| File | Key exports | Notes |
|---|---|---|
| `index.ts` | `ITEM_CATEGORY_FACET_KEY`, `ITEM_LOCATION_FACET_KEY`, `REFINEMENT_KEY_DELIVERY`, `REFINE_QUERY_DELIMITER`, `EN_US_LANG_ID`, `PageTypes`, `PUBLIC_DIRECTORY` | Facet keys, cookie names, locale IDs, delimiter strings |
| `grs.ts` | `SEARCH_SERVICE_PROVIDERS`, `GRS_SEARCH_PROVIDER_QUERY_PARAM_VALUE`, `SEARCH_PROVIDER_QUERY_PARAM_KEY` | GRS vs LucidWorks provider identification constants |
| `contentStack.ts` | `SEARCH_V2_FEATURE_TOGGLE`, content type UIDs | ContentStack content type UIDs used in API queries |
| `styles.ts` | z-index values, breakpoint overrides | Shared styling constants |
| `language/index.ts` | `i18nConfig` | `{ locales: ['en-US', 'en-CA', 'fr-CA'], defaultLocale: 'en-US' }` |
| `language/constants.tsx` | Locale → display label map | `{ 'en-US': 'English', 'en-CA': 'English (Canada)', 'fr-CA': 'Français' }` |
| `errorMessages/` | User-facing error strings | Centralised so they can be CMS-driven in future |

#### Facet Key Constants (Critical for Tires Repo)

The search facet system relies entirely on these key constants. Different backends use different key names for the same concept:

| Concept | GRS key | LucidWorks key |
|---|---|---|
| Category | `item_category` | `item_category` |
| Availability | `item_location_availability` | `item_location_availability` |
| Program eligibility | `item_program_eligibility` | `item_program_eligibility` |
| Price | `item_location_pricing_salePrice` (US) | `item_location_pricing_salePrice` |
| Rating | `item_rating_value` | `item_rating_value` |
| In warehouse | `item_location_bopiw` | `item_location_bopiw` |
| BOPIM (order in warehouse) | `item_location_bopim` | `item_location_bopim` |

---

### 4.8 `src/types` — TypeScript Type Definitions

```
src/types/
├── types.d.ts                    # Global ambient types (e.g. window extensions)
├── react-tracking.d.ts           # Module augmentation for react-tracking library
├── cookies.ts                    # CookieData interface (invCheckPostalCode, STORELOCATION, etc.)
├── metrics.ts                    # Prometheus metric label types
├── contentModels/                # Base ContentStack content model interfaces
│   └── *.ts                      # One file per CMS content model
├── contentStack/                 # Full CMS entry type definitions
│   ├── index.ts                  # Re-exports all types
│   ├── SearchPageConfigProperties.ts
│   ├── CategoryLandingPage.ts
│   ├── ComposerData.ts           # SearchComposerData, SearchComposerV2Data
│   ├── GrsConfig.ts              # GrsConfig, GRSFacetConfig, GRSFacetConfigProperties
│   ├── SearchServiceConfig.ts
│   └── ...
├── generics/                     # Utility generic types
│   └── index.ts                  # DeepPartial<T>, Nullable<T>, etc.
├── grs/                          # GRS API response types
│   └── index.ts                  # GRSSearchResult, GRSFacet, GRSBreadcrumb
├── layouts/                      # Next.js layout prop types
│   └── index.ts                  # LocalizedLayoutProps { params: { lang: string }, children }
├── pages/                        # Page-level prop types
│   └── search.ts                 # PageType enum map, SearchPageProps
└── searchQuery/                  # Search URL query parameter types
    └── index.ts                  # SearchParams, LWRefinement, GRSRefinement
```

#### Critical Type: `SearchPageConfigProperties`

This is the central type for all page configuration. It contains everything the page needs:

```ts
interface SearchPageConfigProperties {
  composerData: ComposerData          // CMS-driven layout blocks
  facetConfig: GRSFacetConfigProperties  // Which filters to show
  grsConfig: GrsConfig                // GRS endpoint + params
  searchServiceConfig: SearchServiceConfig  // Backend config
  siteSettings: SiteSettingsContentStackData  // Global site config
  featureFlags: Record<string, boolean>  // Runtime feature flags
}
```

---

### 4.9 `src/utils` — Pure Utility Functions

Every util file has a co-located `.test.ts` file. Utils are **pure functions** — no side effects, no React imports.

| File | Key functions | Purpose |
|---|---|---|
| `index.ts` | `stringToBoolean()`, `clamp()` | General JS helpers |
| `lib.ts` | `addEdgeCacheTagHeader()` | Adds `Cache-Tag` header for CDN purge groups |
| `middleware.ts` | `validateCategorySlug()`, `addMetaHeaders()`, `fetchIsSearchV2Enabled()`, `updateEntryIdToUseByKeywordToCSSearchPageEntry()` | Logic used inside `src/middleware.ts` |
| `cookieUtils.ts` | `getLocalWarehouseFromCookies(cookies, {isBD})`, `getCookie(name)` | Type-safe cookie read; `isBD` flag selects B2B vs consumer cookie names |
| `grs.ts` | `getAdobeSessionId()`, `getIsGRSSearchProviderSessionStorageSet()`, `setGRSSearchProviderSessionStorage()` | GRS session storage helpers for Adobe session ID tracking |
| `searchFacets.ts` | `splitByCommaOutsideParentheses()` | Splits facet value strings while respecting parenthetical commas (e.g. brand names) |
| `contentEntries.ts` | Entry URL builders | Constructs ContentStack API query URLs for specific entry IDs |
| `links.ts` | `modernizeCategoryLink(path, link)` | Converts legacy `/c/` URLs to modern format; handles edge cases (empty, search paths) |
| `livePreview.ts` | Live preview hash utilities | Parses ContentStack live-preview query parameters |
| `brandfolder.ts` | Brandfolder asset URL builder | Constructs Brandfolder CDN URLs with size/format params |
| `metrics.ts` | Prometheus label extractors | Extracts method, route, status code for metric labels |
| `metricsHelper.ts` | `createHttpMetric()`, `sendMetricsAsync()`, `DEFAULT_CONFIG` | Creates metric payloads from request data; `sendMetricsAsync` is non-blocking |
| `metricsClassifier.ts` | Route pattern classifier | Groups routes into metric label categories to prevent high cardinality |
| `inboundMetricsClassifier.ts` | Inbound path classifier | Classifies incoming request paths for inbound metrics |
| `fetchContentStackPolicies.ts` | `api.getContentSecurityPolicies()` | Fetches CSP string from ContentStack Config stack at runtime |
| `next.ts` | Next.js cache helpers | `revalidatePath()` wrappers |
| `twoWayMap.ts` | `TwoWayMap<K, V>` class | Bidirectional lookup map — used for facet key translation between GRS and LW |
| `build-server-health.ts` | Health check JSON builder | Returns `{ version, buildTime, gitSha }` for `/api/build-health` |

---

### 4.10 `src/styles` — Global SCSS

| File | Purpose | Notes |
|---|---|---|
| `global.scss` | CSS reset, root variables, body defaults | Imported once in `layout.tsx` |
| `colors.scss` | CSS custom properties for brand colours | e.g. `--color-primary: #005DAA` |
| `fonts.scss` | `@font-face` declarations | Self-hosted fonts from `public/nextGSearchAssets/fonts/` |

These are **global** styles. Component-level styles use either:
- MUI's `sx` prop (for Emotion-based styling)
- A co-located `styles.ts` with `styled()` components

---

### 4.11 `src/mock` — MSW Test Mocks

```
src/mock/
├── handlers.ts                        # MSW v2 request handlers
│                                      # http.get('/api/...', resolver)
│                                      # http.post('/api/...', resolver)
├── server.ts                          # setupServer(...handlers) — Node.js MSW server
├── TestMswMockingComponent.tsx        # Helper: renders children inside MSW-ready render
└── TestMswMockingComponent.test.tsx   # Self-test verifying MSW interceptors work
```

**How MSW is used:**
```ts
// vitest.setup.ts
import { server } from '@/mock/server'
beforeAll(() => server.listen())
afterEach(() => server.resetHandlers())
afterAll(() => server.close())
```

In individual tests, override specific handlers:
```ts
server.use(
  http.get('/api/health', () => HttpResponse.json({ status: 'ok' }))
)
```

---

### 4.12 `src/actions` — Next.js Server Actions

```
src/actions/
└── auth.ts    # MSAL-based authentication server actions
```

**Rules for server actions (enforced by copilot-instructions.md):**
- All inputs validated with Zod
- Use `next-safe-action`'s `action()` function
- Always return `ActionResponse` type
- Never use `try/catch` for expected errors — return them as values
- Use `useActionState` on the client to consume the response

---

### 4.13 `src/app/api` — API Route Handlers

Each route is in its own folder: `src/app/api/[name]/route.ts`

| Route | Method | Purpose | Auth |
|---|---|---|---|
| `/api/health` | GET | Kubernetes liveness probe — returns `200 OK` immediately | None |
| `/api/build-health` | GET | Returns `{ version, buildTime, gitSha }` | None |
| `/api/metrics` | GET | Prometheus text exposition — scraped by Prometheus server | Internal only |
| `/api/contentStack` | POST | Receives CMS webhook events, triggers cache purge | Webhook secret |
| `/api/revalidate` | POST | On-demand ISR revalidation for a specific path or tag | Secret token |
| `/api/findSearchRuleId` | GET | Given a keyword, returns the matching CMS entry ID | None |
| `/api/getAllSearchRules` | GET | Returns all configured search routing rules | None |
| `/api/getAllCategoryPageRoutes` | GET | Returns all category page route IDs for `generateStaticParams` | None |
| `/api/getFeatureFlagState` | GET | Debug: returns current state of a named feature flag | Dev only |
| `/api/crossRegionCall` | POST | Replicates a revalidation call to the other CDN region | Internal secret |
| `/api/build` | GET | Build metadata endpoint | None |

---

### 4.14 `public/` — Static Assets

```
public/
├── robots933456.txt              # Azure App Service health check verification file
├── 1234.htm                      # Legacy health probe page
└── nextGSearchAssets/
    ├── opinionlab_us.min.js      # OpinionLab survey widget — US version
    ├── opinionlab_ca.min.js      # OpinionLab survey widget — CA version
    └── fonts/                    # Self-hosted web fonts (WOFF2 format)
        ├── CostcoFontBold.woff2
        ├── CostcoFontRegular.woff2
        └── ...
```

The `opinionlab_*.min.js` files are served from `PUBLIC_DIRECTORY = '/nextGSearchAssets'` and loaded conditionally by the `OpinionLab` component based on locale.

---

### 4.15 `grafana/` — Observability Stack

```
grafana/
├── docker-compose.yml    # Prometheus (port 9090) + Grafana (port 3001)
└── prometheus.yml        # Scrape config:
                          #   - targets: ['host.docker.internal:9200']
                          #   - scrape_interval: 15s
```

**Local development:**
```bash
docker compose -f grafana/docker-compose.yml up
# Prometheus: http://localhost:9090
# Grafana:    http://localhost:3001 (admin/admin)
```

**Metrics available:**
- `http_requests_inbound_total` — inbound by method/status/statusGroup
- `http_requests_outbound_total` — outbound by method/target_host
- `http_request_duration_seconds` — request latency histogram
- `http_errors_total` — error counter by errorType
- All default Node.js process metrics (memory, CPU, event loop lag)

---

## 5. Key Cross-Cutting Concerns

### 5.1 Internationalisation (i18n)

| Aspect | Detail |
|---|---|
| Locales | `en-US`, `en-CA`, `fr-CA` |
| Routing | `/[lang]/s/…` — locale prefix on all routes |
| Detection | `next-i18n-router` — checks URL prefix first, then `Accept-Language` header |
| Config | `src/constants/language/index.ts` exports `i18nConfig` |
| Default locale | `en-US` — used when no prefix detected |
| Static seeding | `generateStaticParams` in each layout returns `[{lang:'en-US'}, {lang:'en-CA'}, {lang:'fr-CA'}]` |
| Labels | All UI strings fetched from ContentStack — **not** local JSON files |
| Currency | Determined by locale (USD for en-US, CAD for en-CA/fr-CA) |
| Lang ID | ContentStack uses numeric IDs: `en-US = '-1'`, `en-CA = '-2'`, `fr-CA = '-3'` |

**Middleware handles redirect:**
```
GET /s/[id]           → redirects to /en-US/s/[id]  (Accept-Language: en-US)
GET /s/[id]           → redirects to /fr-CA/s/[id]  (Accept-Language: fr)
GET /en-CA/s/[id]     → passes through (locale already in URL)
```

---

### 5.2 Content Management (ContentStack)

ContentStack is used as a **headless CMS** that drives all configurable aspects of the application. There are **two separate stacks**:

| Stack | Purpose | Env vars prefix |
|---|---|---|
| Content Stack | Page content — search pages, category pages, labels, promotional blocks | `CONTENTSTACK_` |
| Config Stack | Application configuration — feature flags, search rules, facet config, CSP | `CONTENTSTACK_CONFIG_` |

**What ContentStack controls:**

| Feature | ContentStack Content Type |
|---|---|
| Search page layout | `search_page` |
| Category page layout | `category_landing_page` |
| Filter/facet config | `grs_facet_config` |
| Search routing rules | `search_rule` |
| Feature toggles | `feature_toggle` |
| UI labels | `key_value_pair` |
| Content Security Policy | `configuration_setting` (configKey: `content_security_policy`) |
| Header/Footer nav | Via Forge digital components' own CMS queries |
| Promotional blocks | `composer_data` slots |

**Stack initialisation** (`stackClient.ts`):
```ts
// Called once per server startup in layout.tsx
configureStacks()

// Under the hood:
ForgeContentStack.configure({ api_key, delivery_token, environment, region })
ForgeConfigStack.configure({ api_key, delivery_token, environment, region })
```

**Caching:** All ContentStack calls are wrapped with `unstable_cache` from Next.js with cache tags. When a CMS entry is published, the webhook hits `/api/contentStack` → triggers `revalidateTag(cacheTag)` → that page ISR is re-generated on next request.

---

### 5.3 Search Providers — GRS vs LucidWorks Deep Dive

#### Architecture

```
SearchProvider ('use client')
    │
    ├─ reads: URL params (keyword, refine, page, sort)
    ├─ reads: cookies (C_LOC, invCheckPostalCode, provider preference)
    ├─ reads: session storage (A/B test assignment)
    │
    ├─ DECISION: which backend?
    │   ├─ Query param ?provider=grs  → GRS
    │   ├─ Query param ?provider=lw   → LucidWorks
    │   ├─ Session storage key set    → LucidWorks (A/B)
    │   └─ Default                    → GRS
    │
    ├─ GRS path:
    │   GrsQuery(keyword, refinements, wareouseParams, grsConfig, locale)
    │   → HTTP POST to GRS endpoint
    │   → response parsed by isSearchResultGRS() + normalise
    │
    └─ LucidWorks path:
        LucidWorksQuery(keyword, refinements, page, sort, warehouseParams)
        → HTTP GET to LucidWorks endpoint
        → response parsed by isSearchResultLW() + normalise
```

#### GRS Facet System

GRS uses a complex facet key system with boolean and range facets:

```ts
// Boolean facets (presence/absence)
GRS_SHOW_OUT_OF_STOCK_FACET_KEY  → 'SHOW_OUT_OF_STOCK'
GRS_HIDE_OUT_OF_STOCK_FACET_KEY  → 'HIDE_OUT_OF_STOCK'
GRS_WAREHOUSE_PICKUP_FACET_KEY   → warehouse-based filter

// Range facets (price ranges)
grsRangeValueFacets = [GRS_US_PRICE_FACET_KEY, GRS_CA_PRICE_FACET_KEY]
// Encoded as: facetKey=(low,high)

// Boolean value facets (true/false)
grsBooleanValueFacets = [GRS_BUY_IN_WAREHOUSE_FACET_KEY, ...]
```

#### Warehouse Location Cookies

The warehouse location system uses cookies to personalise search results (inventory availability, pricing):

| Cookie | B2C | B2B (`isBD=true`) |
|---|---|---|
| Postal code | `invCheckPostalCode` | `WC_BD_ZIP` |
| Warehouse | `STORELOCATION.storeLocation.city` | `WC_BD_WHS` (pipe-delimited) |

Both are read by `getLocalWarehouseFromCookies(cookies, { isBD })` in `cookieUtils.ts`.

---

### 5.4 Observability (OpenTelemetry + Prometheus)

#### Metrics Collection Points

| Location | What is measured |
|---|---|
| `src/middleware.ts` | Every inbound request: method, status, latency, route classification |
| `src/services/content/stackClient.ts` | Every outbound HTTP call via undici: method, target_host, status, latency, errors |
| `src/metrics-server.ts` | Standalone HTTP server on port `9200` exposing `/metrics` |
| `src/app/api/metrics/route.ts` | Same metrics accessible via Next.js API route |

#### Metrics Architecture

```
Application Server (port 3000)
    │
    ├─ Middleware records: httpInboundCounter, httpInboundDuration
    └─ undici Dispatcher records: httpOutboundCounter, httpOutboundDuration, httpErrorCounter

Metrics Server (port 9200)
    └─ /metrics → Prometheus text format

Prometheus (port 9090)
    └─ scrapes :9200/metrics every 15s

Grafana (port 3001)
    └─ Queries Prometheus → dashboards
```

#### OpenTelemetry Traces

Configured via `@vercel/otel` in `src/instrumentation.ts`:
- Traces exported via OTLP HTTP to `OTEL_EXPORTER_OTLP_ENDPOINT`
- `@opentelemetry/instrumentation-http` auto-instruments all HTTP calls
- Trace context propagated across service boundaries via W3C Trace Context headers

---

### 5.5 Authentication & Consent

#### Authentication Flow

```
User visits /[lang]/s/[id]
    │
    ├─ MSALContextProvider initialises Azure AD client
    ├─ KMSI component checks for "keep me signed in" preference
    ├─ If not authenticated → silent token refresh attempt
    │   ├─ Success → user is signed in (no redirect)
    │   └─ Failure → user remains anonymous (search still works)
    └─ Auth state available via Forge's useAuth() hook in child components
```

#### Consent Management (Transcend)

Transcend manages GDPR/CCPA compliance:

| Component | When rendered | Purpose |
|---|---|---|
| `ConsentScripts` | `<head>` in root layout | Loads Transcend consent banner JS |
| `TranscendIntegrationManager` | `<head>` | Integrates Transcend with other tracking scripts |
| `ConsentManagement` | `<head>` | Manages consent state (opt-in/opt-out) |
| `CostcoConsentManagerContainer` | `<body>` | Renders the visible consent UI (banner/modal) |
| `TranscendOverride` | Search layout | Allows per-page consent overrides |

#### Queue-It Bot Protection

`<QueueIt />` is injected in `<head>` to manage bot traffic and load spikes. It intercepts requests and redirects to a waiting room if thresholds are exceeded.

---

### 5.6 Analytics — Adobe Analytics + OneTag + Criteo

#### Adobe Analytics (`window.digitalData`)

The standard digital data layer:

```ts
// Populated by SearchPageDigitalDataLayer on every search render
window.digitalData = {
  page: { pageInfo: { pageType, pageName, language } },
  search: {
    keyword,
    totalResults,
    currentPage,
    facetsApplied: [...activeFacets],
  },
  product: [...productImpressions],
}
```

#### AJO (Adobe Journey Optimizer) Personalisation

`AJOPersonalizationProvider` (from Forge) wraps the layout to enable Adobe Target personalisation — product tile ordering, promotional banners, featured items.

#### Criteo OneTag

Criteo tracks product impressions for sponsored placement:

```
OneTag script loaded in <head>   ← OneTag component (server)
    │
    └─ OneTagSearchPageEventScript fires on search result render (client)
       → sends product impression data to Criteo
       → Criteo returns sponsored products
       → injected into search grid at configured positions by SearchResultsGrid
```

#### LogRocket Session Recording

`logrocket` + `logrocket-react` record user sessions for debugging. Initialised once per page load. PII is scrubbed before transmission.

---

### 5.7 Feature Flags — Implementation Details

Feature flags are ContentStack entries of type `feature_toggle`.

**Resolution logic:**
```ts
const { isEnabled } = await getFeatureFlagState('search_v2', 'search')

// Inside getFeatureFlagState:
// 1. Fetches ContentStack Config entry by development key
// 2. Checks feature_on: boolean
// 3. If enabled_applications is empty → applies to all
// 4. If enabled_applications has entries → checks if current SITE env var matches
//    e.g. SITE=CABC → checks for 'browse-cabc' in enabled_applications list
// 5. Returns { isEnabled: boolean, lastUpdated: Date | null }
```

**Flags currently in use:**
| Flag key | Purpose |
|---|---|
| `search_v2` (SEARCH_V2_FEATURE_TOGGLE) | Enables new search backend |
| GRS provider beta | Enables GRS beta features for specific sites |
| Filter visibility flags | Show/hide specific filter groups |
| WAPO (warehouse purchase order) | Enable warehouse ordering mode |

---

### 5.8 Content Security Policy (CSP)

CSP is loaded in **two phases** to ensure it's always available:

```
Phase 1 — Build time (fallback)
  next build runs next.config.mjs
  → Fetches CSP entry from ContentStack Config stack
  → Stores as CONTENT_SECURITY_POLICIES_FALLBACK env var in build output

Phase 2 — Runtime (primary)
  Server starts → instrumentation.ts register() runs
  → Fetches CSP entry from ContentStack at runtime
  → Stores in process.env.CONTENT_SECURITY_POLICIES
  → If fetch fails → uses CONTENT_SECURITY_POLICIES_FALLBACK

Each request:
  middleware.ts reads process.env.CONTENT_SECURITY_POLICIES
  → Sets Content-Security-Policy response header
```

The CSP string is assembled from ContentStack fields:
`DEFAULT_SRC`, `SCRIPT_SRC`, `STYLE_SRC`, `IMG_SRC`, `MEDIA-SRC`, `FONT_SRC`, `CONNECT_SRC`, etc.

---

### 5.9 Cookie System

The application reads several cookies for personalisation:

| Cookie | Value | Used for |
|---|---|---|
| `invCheckPostalCode` | ZIP/postal code | Inventory availability in search results |
| `C_LOC` | Store location | Warehouse-based pricing and availability |
| `STORELOCATION` | JSON `{storeLocation:{city}}` | B2C warehouse city name |
| `WC_BD_ZIP` | ZIP code | B2B delivery zip |
| `WC_BD_WHS` | Warehouse code | B2B warehouse identifier (pipe-delimited) |
| `WAREHOUSEDELIVERY_WHS` | Warehouse code | Warehouse delivery cookie |
| `provider` | `'grs'` or `'lw'` | Force search backend (dev/test) |

Cookies are read server-side via the Next.js `cookies()` function and client-side via `js-cookie` or `document.cookie`.

---

### 5.10 Caching Strategy

| Cache Layer | Technology | TTL | What is cached |
|---|---|---|---|
| Page cache | Next.js ISR | 3 hours (`revalidate = 10800`) | Rendered HTML of search/category pages |
| Data cache | Next.js `unstable_cache` | Per call, tagged | ContentStack API responses |
| Shared ISR cache | Redis (`@neshca/cache-handler`) | Same as ISR TTL | Shared across multiple server instances |
| CDN cache | Cache-Tag headers | Until purged | Full pages at CDN edge |
| On-demand purge | `/api/revalidate` + `/api/contentStack` webhook | Immediate | Specific paths or tags |

**Cache tag strategy:** Each page is tagged with its ContentStack entry ID. When an editor publishes a change in ContentStack, the webhook sends the entry ID → `revalidateTag(entryId)` → all pages using that entry are regenerated on next request.

---

## 6. Data Flow — Complete Request Lifecycle

```
User visits /en-US/s/abc123?keyword=laptop&refine=item_category=Laptops&page=2
     │
     ▼ ── EDGE (sub-millisecond) ──────────────────────────────────────────
     │
 middleware.ts
   ├─ Parse URL → extract lang='en-US', path='/s/abc123'
   ├─ Validate locale → matches i18nConfig.locales ✓
   ├─ Add response headers:
   │    Cache-Tag: abc123                    (CDN cache group)
   │    Content-Security-Policy: [from env] (security header)
   ├─ Start metrics timer
   ├─ Decide: pass through (not a redirect candidate)
   └─ next() → continue to App Router
     │
     ▼ ── SERVER (Node.js) ────────────────────────────────────────────────
     │
 Next.js App Router
   matches: src/app/[lang]/(customers)/s/[contentStackEntryId]/page.tsx
   params = { lang: 'en-US', contentStackEntryId: 'abc123' }
   searchParams = { keyword: 'laptop', refine: 'item_category=Laptops', page: '2' }
     │
     ▼
 page.tsx (RSC — Server Component, async)
   └─ calls MainContainer({ contentStackEntryId: 'abc123', lang: 'en-US', searchParams })
     │
     ▼
 MainContainer (RSC — async Server Component)
   │
   ├─ 1. fetchPageRules('abc123', 'en-US')
   │      → ContentStack Config: finds SearchRuleEntry for this entry ID
   │      → Returns: { keyword: 'laptop', pageType: 'search', grsConfig: {...} }
   │      → Cached with unstable_cache, tag: 'abc123'
   │
   ├─ 2. getPageConfigs(entry, 'en-US')
   │      → ContentStack Content: fetches full SearchPageConfigProperties
   │      → Returns: composerData, facetConfig, siteSettings, featureFlags
   │      → Cached with unstable_cache, tag: 'abc123-config'
   │
   ├─ 3. getComposerDataFromEntry(entry) + mergeComposerData()
   │      → Returns ordered list of content blocks
   │
   ├─ 4. isFilterEnabled(config) → true
   │
   └─ 5. Returns JSX:
          <SearchProvider config={pageConfig} params={searchParams}>
            <LeftColumn />
            <MainColumn />
          </SearchProvider>
     │
     ▼ ── CLIENT (Browser) ────────────────────────────────────────────────
     │
 SearchProvider hydrates ('use client')
   │
   ├─ useQueryParams() reads URL: keyword='laptop', refine='item_category=Laptops', page=2
   ├─ Reads cookies: C_LOC, invCheckPostalCode
   ├─ Decision: use GRS (default, no provider override)
   ├─ Calls GrsQuery('laptop', refinements, warehouseParams, grsConfig, 'en-US')
   │    → POST to GRS endpoint
   │    → Response: { products: [...], facets: [...], totalCount: 847 }
   ├─ Normalises response into SearchResult shape
   └─ Stores in React context
     │
     ▼
 Child components re-render with search data:
   │
   ├─ SearchResultsGrid
   │    ├─ Renders 24 ProductTileUI components
   │    ├─ Calls findCriteoPlacement() → injects 2 sponsored tiles
   │    └─ Shows LoadingResultsGrid skeleton during loading
   │
   ├─ SearchResultsFacets (LeftColumn)
   │    ├─ Renders FilterGroup for each facet
   │    ├─ "Category (12)" accordion
   │    ├─ "Availability (3)" accordion  
   │    └─ Active filter "Laptops ×" chip
   │
   ├─ ResultsCount → "Showing 25–48 of 847 results for 'laptop'"
   ├─ SearchResultsPagination → page 2 of 36
   │
   └─ Analytics events:
        SearchPageDigitalDataLayer → window.digitalData.search
        OneTagSearchPageEventScript → Criteo impression event
     │
     ▼
 Middleware records metrics (async, non-blocking):
   httpInboundCounter.inc({ method: 'GET', statusCode: 200, statusGroup: '2xx' })
   httpInboundDuration.observe(latencyMs)
```

---

## 7. TypeScript Patterns & Conventions

These rules are enforced by `copilot-instructions.md` and `tsconfig.json`:

### Core Rules

```ts
// ✅ Use interfaces over types
interface SearchResult { ... }

// ✅ Use function keyword for pure functions
function getCurrentPage(start: number, resultsPerPage: number) { ... }

// ✅ Named exports only — no default exports for components
export function SearchResultsGrid({ results }: Props) { ... }

// ✅ RORO pattern (Receive Object, Return Object)
function getPageConfigs({ entry, locale }: GetPageConfigsArgs): PageConfig { ... }

// ✅ Early returns for error conditions
function validateCategorySlug(slug: string): string | null {
  if (!slug) return null
  if (slug.length > 200) return null
  return slug
}

// ❌ Never use 'any'
const data: any = response  // FORBIDDEN

// ❌ No enums — use const maps
const PageTypes = { search: 'search', category: 'category' } as const
type PageType = typeof PageTypes[keyof typeof PageTypes]
```

### Component Patterns

```tsx
// ✅ Server Component (default — no directive needed)
export function BreadCrumb({ entry, locale }: BreadCrumbProps) {
  const crumbs = parseBreadCrumbs(entry)
  return <nav aria-label="breadcrumb">...</nav>
}

// ✅ Client Component (only when needed)
'use client'
export function SortSelector({ options }: SortSelectorProps) {
  const { queryParams, setQueryParams } = useQueryParams()
  ...
}

// ✅ Async Server Component
export default async function MainContainer({ contentStackEntryId, lang }: Props) {
  const config = await getPageConfigs({ entry, locale: lang })
  return <SearchProvider config={config}>...</SearchProvider>
}
```

### noUncheckedIndexedAccess

TypeScript's `noUncheckedIndexedAccess` is enabled. Array and object indexing returns `T | undefined`:

```ts
// ✅ Must guard array access
const first = results[0]
if (first) { ... }

// ✅ Or use optional chaining
const name = results[0]?.name
```

---

## 8. Testing Strategy — Detailed Patterns

| Layer | Tool | What it tests | Location |
|---|---|---|---|
| Unit | Vitest | Pure functions, hooks | `*.test.ts` co-located |
| Component | Vitest + RTL | React components in isolation | `*.test.tsx` co-located |
| Integration | Vitest + MSW | Components that call APIs | `*.test.tsx` with MSW handlers |
| E2E | Playwright | Full browser flows | `playwright.config.ts` |
| Coverage | v8 via Vitest | Line/branch coverage gate | CI enforced |

### Writing Component Tests

```tsx
// ✅ Pattern: render → interact → assert
import { render, screen } from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import { describe, it, expect } from 'vitest'

describe('SortSelector', () => {
  it('updates URL when user changes sort', async () => {
    const user = userEvent.setup()
    render(<SortSelector options={mockOptions} />)
    
    await user.selectOptions(screen.getByRole('combobox'), 'price-asc')
    
    expect(window.location.search).toContain('sort=price-asc')
  })
})
```

### Writing MSW Handlers

```ts
// src/mock/handlers.ts
import { http, HttpResponse } from 'msw'

export const handlers = [
  http.post('https://grs.costco.com/search', () => {
    return HttpResponse.json(mockSearchResponse)
  }),
  http.get('https://api.contentstack.io/v3/content_types/*', () => {
    return HttpResponse.json(mockContentStackEntry)
  }),
]
```

### Test Commands

```bash
npm run test          # Run all tests with coverage
npm run test:ci       # CI mode (no watch, fail on missing coverage)
npm run e2e:headless  # Playwright headless
npm run e2e:ui        # Playwright with UI mode
```

---

## 9. Build, Deploy & Runtime

### Build Pipeline

```bash
npm run build
# 1. npx rimraf build          — clean previous build
# 2. CONTAINER_BUILD=true next build
#    ├─ Fetches CSP from ContentStack (fallback)
#    ├─ Runs generateStaticParams for all layouts
#    ├─ Generates .next/standalone/   ← minimal runtime
#    ├─ TypeScript errors: IGNORED (CONTAINER_BUILD=true)
#    └─ ESLint errors: IGNORED (CONTAINER_BUILD=true)
#    ⚠ Type checking should be run separately: npm run tsc

npm start
# 1. cp -r build/static build/standalone/build
# 2. cp -r public build/standalone/
# 3. node start.js   (wraps standalone/server.js)
```

### Docker Build

```dockerfile
# Stage 1: Install dependencies
FROM node:22-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci

# Stage 2: Build
FROM node:22-alpine AS builder
COPY . .
COPY --from=deps /app/node_modules ./node_modules
RUN npm run build

# Stage 3: Minimal runtime
FROM node:22-alpine AS runner
COPY --from=builder /app/build/standalone ./
# public/ and build/static/ copied by start.js
```

### PM2 Runtime (`ecosystem.config.js`)

```js
module.exports = {
  apps: [{
    name: 'search-catalog',
    script: 'start.js',
    instances: 'max',     // One per CPU core
    exec_mode: 'cluster',
    env: { NODE_ENV: 'production' }
  }]
}
```

### Environment Variables Summary

```bash
# Node runtime
NODE_ENV=production
NEXT_PUBLIC_BASE_PATH=/                    # CDN base path

# ContentStack — Content
CONTENTSTACK_API_KEY=...
CONTENTSTACK_DELIVERY_TOKEN=...
CONTENTSTACK_ENVIRONMENT=production
CONTENTSTACK_REGION=US
CONTENTSTACK_BRANCH=main

# ContentStack — Config
CONTENTSTACK_CONFIG_API_KEY=...
CONTENTSTACK_CONFIG_DELIVERY_TOKEN=...
CONTENTSTACK_CONFIG_ENVIRONMENT=production

# Search
GRS_BASE_URL=https://grs.costco.internal/...
LW_BASE_URL=https://lw.costco.com/...

# Observability
OTEL_EXPORTER_OTLP_ENDPOINT=https://otel.costco.internal

# Cache
REDIS_URL=redis://cache.costco.internal:6379

# Site identification
SITE=CABC                                  # Determines feature flag scope
```

---

## 10. Path Aliases

Configured in `tsconfig.json` and mirrored exactly in `vitest.config.mts`:

| Alias | Resolves to | Example usage |
|---|---|---|
| `@/components/*` | `src/components/*` | `import { BreadCrumb } from '@/components/BreadCrumb'` |
| `@/constants/*` | `src/constants/*` | `import { REFINE_QUERY_DELIMITER } from '@/constants/index'` |
| `@/services/*` | `src/services/*` | `import { getFeatureFlagState } from '@/services/content/featureFlag'` |
| `@/types/*` | `src/types/*` | `import type { SearchResult } from '@/types/searchQuery'` |
| `@/utils/*` | `src/utils/*` | `import { getLocalWarehouseFromCookies } from '@/utils/cookieUtils'` |
| `@/src/*` | `src/*` | `import { logger } from '@/src/services/logger'` (use sparingly) |
| `@/hooks/*` | `src/hooks/*` | (via tsconfig paths) |

> **Rule:** Never use relative paths that go up more than one level (`../../`). Use path aliases instead.

---

## 11. tires.costco.com — Live Site Analysis

This section documents the actual live site at `https://tires.costco.com` so developers know exactly what to build.

### 11.1 Site Navigation Structure

```
Header
  ├─ Costco.com logo (links back to costco.com)
  ├─ "Tires will be shipped to and installed at [Tire Center]" — warehouse selector
  ├─ Sign In (links to costco.com LogonForm with return URL)
  └─ View Cart (links to costco.com OrderItemDisplay)

Primary Navigation
  ├─ Tires Home         → /
  ├─ Wheels             → veloxwheel.com (external, opens dialog)
  ├─ Batteries          → costco.com/automotive-batteries.html
  ├─ Care & Maintenance → /CostcoAdvantage
  ├─ Warranty           → /Warranty
  ├─ Go to Costco.com   → costco.com
  └─ Schedule an Appointment → / (opens scheduling dialog)

Footer
  ├─ FAQ                → /FAQ
  ├─ Tire Glossary      → /TireGlossary
  ├─ Tire Basics        → /TireBasics
  ├─ Tire Tips          → /TireTips
  ├─ Warranty           → /Warranty
  ├─ Costco Advantage   → /CostcoAdvantage
  ├─ Disclaimers        → /Disclaimers
  ├─ Privacy Policy     → costco.com/privacy-policy.html
  ├─ Terms & Conditions → costco.com/terms-and-conditions-of-use.html
  ├─ Your Privacy Choices (CCPA) → costco.com/DNSMIView
  ├─ Consumer Health Data → costco.com/consumer-health-data-privacy-policy.html
  ├─ Cookie Settings    → Transcend consent modal
  └─ Site Map           → /SiteMapDisplayView
```

---

### 11.2 Home Page Sections

The home page (`/` or `/?lang=en-US`) contains these sections in order:

#### Section 1 — Hero / Search Widget
- Large hero area with "Shop for Tires" headline
- Animated slideshow (auto-play with pause control)
- The 4-tab **TireSearchWidget** (see 11.3)

#### Section 2 — "Why Costco?" Slider
- Auto-play image carousel with 5 slides
- Stop/pause control for accessibility
- Each slide promotes a Costco tire advantage

#### Section 3 — "How It Works" — 4 Steps
```
Step 1: Search  → "Search for tires by entering Vehicle, License Plate, or Tire Size"
Step 2: Select  → "Select your tires based on your driving needs and rim size"
Step 3: Schedule → "Schedule your installation at your chosen Costco Tire Center"
Step 4: Free Shipping → "Tires delivered to your Costco Tire Center free of charge"
```
Each step has an icon, number, title, and description. CTA: "Start Your Tire Search" button.

#### Section 4 — Specialty Tires / Brand Carousel
- Auto-play carousel of brand promotional images
- Brands shown: Pirelli, BFGoodrich, Firestone, Bridgestone, Michelin, Velox Wheels, Interstate Batteries, TOW Master
- Each brand links to its brand landing page (e.g. `/Pirelli_tires`)
- Active promotions shown as overlay text (e.g. "Save $60 on any set of 4 BFGoodrich tires")

#### Section 5 — Tire Information Links
Three editorial cards linking to:
- **How To Buy Tires** → `/TireTips?lang=en-US#HowToBuy`
- **When To Buy Tires** → `/TireTips?lang=en-US#WhenToBuy`
- **Tire Care** → `/CostcoRoadHazardWarranty`

#### Section 6 — SEO Content Block
- "Costco - The Best Place to Buy Tires" heading
- Long-form descriptive text about tire selection, service, seasonal options
- Footer links: FAQ, Tire Glossary, Tire Basics, Tire Tips

---

### 11.3 Search Widget — 4 Search Modes

The `TireSearchWidget` is the most critical UX component. It is a tabbed interface:

#### Tab 1: License Plate (Preferred / Default Tab)
```
Fields:
  ├─ License Plate Number  (text input)
  ├─ State                 (dropdown — all US states)
  └─ ZIP Code              (text input)

Note: "Match Tires to Your State's Public Vehicle Records"
Warning: "If your vehicle is manufactured before 2001, please call or visit a warehouse for assistance"

On Submit:
  → Queries DMV/vehicle lookup API with plate + state
  → Returns: year, make, model, trim
  → Proceeds to tire size lookup
```

#### Tab 2: Search by Vehicle
```
Fields (cascading dropdowns — each enables the next):
  ├─ Year   (dropdown: current year down to ~2001)
  ├─ Make   (dropdown: Honda, Toyota, Ford, etc.)
  ├─ Model  (dropdown: Civic, Accord, etc.)
  └─ Trim   (dropdown: LX, EX, Sport, etc.)

On Submit:
  → Queries tire size database for vehicle
  → Returns: recommended tire sizes
  → Shows tire results grid filtered by those sizes
```

#### Tab 3: Search by Tire Size
```
Fields:
  ├─ Width  (dropdown: 155, 165, 175, 185, 195, 205, 215, 225, 235, 245, 255, ...)
  ├─ Ratio  (dropdown: 30, 35, 40, 45, 50, 55, 60, 65, 70, 75, 80, ...)
  └─ Rim    (dropdown: 14", 15", 16", 17", 18", 19", 20", 21", 22", ...)

Tire size format: WIDTH/RATIO R RIM  e.g. 215/55R17

On Submit:
  → Queries tire catalogue with exact size
  → Shows results grid filtered by size
```

#### Tab 4: Search by Item #
```
Fields:
  └─ Item Number (text input — Costco item number)

On Submit:
  → Direct product lookup by item ID
  → Navigates to tire detail page
```

---

### 11.4 Tire Results Page

URL pattern: `/tires?vehicle=2022-Honda-Civic` or `/tires?size=215-55R-17`

#### Layout
```
┌─────────────────────────────────────────────────────┐
│  Results: "Showing 24 of 86 tires for 215/55R17"   │
│  Sort: [Featured ▾]    Filter: [Brand] [Type] [Price]│
├──────────────────┬──────────────────────────────────┤
│  LEFT RAIL       │  PRODUCT GRID                    │
│                  │                                  │
│  Filters:        │  [Tire Tile] [Tire Tile] [Tile]  │
│  □ Brand         │  [Tire Tile] [Tire Tile] [Tile]  │
│    □ Michelin    │  [Tire Tile] [Tire Tile] [Tile]  │
│    □ Bridgestone │  [Tire Tile] [Tire Tile] [Tile]  │
│    □ Goodyear    │                                  │
│    □ Pirelli     │  [Sponsored] [Tire Tile] [Tile]  │
│  □ Type          │                                  │
│    □ All-Season  │  < 1 2 3 4 >                    │
│    □ Winter      │                                  │
│    □ Summer      │                                  │
│  □ Price Range   │                                  │
│  □ Speed Rating  │                                  │
│  □ Load Index    │                                  │
└──────────────────┴──────────────────────────────────┘
```

#### Tire Product Tile Contents
Each product tile shows:
- Brand logo
- Product image
- Product name (e.g. "Michelin Defender T+H")
- Size (e.g. 215/55R17)
- Price (e.g. "$189.99/each" or "$759.96/set of 4")
- Star rating + review count
- Mileage warranty (e.g. "80,000 mile warranty")
- Key badges: All-Season, Run-Flat, M+S, 3PMSF
- "Add to Cart" button
- Quantity selector

#### Filters Available
| Filter | Type | Options |
|---|---|---|
| Brand | Checkbox | Michelin, Bridgestone, Goodyear, Pirelli, Firestone, BFGoodrich, Continental, Yokohama, etc. |
| Tire Type | Checkbox | All-Season, Winter/Snow, Summer, All-Terrain, Highway, Touring |
| Price Range | Range slider | $50–$500+ |
| Speed Rating | Checkbox | H (130mph), V (149mph), W (168mph), Y (186mph), T (118mph) |
| Load Index | Checkbox | Standard Load, Extra Load, Light Truck |
| Run-Flat | Checkbox | Run-Flat capable |
| Mileage Warranty | Checkbox | 40K, 50K, 60K, 70K, 80K, 90K+ miles |

---

### 11.5 Tire Detail Page

URL: `/tire/[itemNumber]`

Contains:
- Full-size product image gallery (multiple angles)
- Brand logo
- Product name + size
- Price (per tire + set of 4)
- Mileage warranty badge
- "Add to Cart" + quantity selector
- "Schedule Installation" CTA
- Spec table:
  | Spec | Example Value |
  |---|---|
  | Tire Size | 215/55R17 |
  | Section Width | 215mm |
  | Aspect Ratio | 55 |
  | Rim Size | 17 inches |
  | Load Index | 94 |
  | Speed Rating | V |
  | UTQG Treadwear | 700 |
  | UTQG Traction | A |
  | UTQG Temperature | A |
  | Ply Rating | 4 ply |
  | Max Load | 1477 lbs |
  | Max Inflation | 51 psi |
  | Tread Depth | 10/32" |
  | Weight | 21 lbs |
- Road hazard warranty details
- Manufacturer warranty
- Customer reviews section

---

### 11.6 Information Pages

| Page | URL | Content |
|---|---|---|
| FAQ | `/FAQ` | 13 questions covering: pair placement, inflation, alignment, vibration, balancing, mixing, wear indicators, rotation |
| Tire Basics | `/TireBasics` | Technical deep-dive: Balancing (why, when, how), Rotation tips (every 10,000–12,000 km), Alignment (Camber, Caster, Toe), Inflation Pressure (checking frequency, recommended pressure, valve care) |
| Tire Tips | `/TireTips` | Practical tips: HowToBuy, WhenToBuy sections |
| Tire Glossary | `/TireGlossary` | A–Z definitions of tire terminology |
| Costco Advantage | `/CostcoAdvantage` | Full description of included services: inflation checks, balancing, rotation, flat repairs, nitrogen inflation, rubber valve stems, road hazard warranty |
| Warranty | `/Warranty` | Links to: Costco Road Hazard Warranty PDF, Michelin warranty PDF, BFGoodrich warranty PDF, Bridgestone, Firestone, Pirelli, trailer tire warranties |
| Road Hazard Warranty | `/CostcoRoadHazardWarranty` | Full warranty terms |
| Disclaimers | `/Disclaimers` | Legal disclaimers |
| Site Map | `/SiteMapDisplayView` | All page links |

---

### 11.7 The Costco Advantage

Key selling points that **must** be prominently displayed (drives purchase decisions):

1. **Free Installation** — Tire purchase includes installation at no charge
2. **5-Year Road Hazard Warranty** — Covers damage from road hazards
3. **Lifetime Tire Rotation** — Free rotations throughout tire life
4. **Lifetime Balancing** — Free balancing throughout tire life
5. **Lifetime Flat Repairs** — Free flat repairs throughout tire life
6. **Inflation Pressure Checks** — Free pressure checks
7. **Nitrogen Inflation** — Nitrogen instead of compressed air (retains pressure better)
8. **New Rubber Valve Stems** — Installed with every tire service
9. **Free Shipping** — Tires shipped free to chosen Costco Tire Center
10. **Competitive Pricing** — Published retail pricing includes all shipping/handling

> Note: TPMS (Tire Pressure Monitoring System) valve stems and accessories are **additional charge**. Not available for all vehicles/tires.

---

## 12. Creating the Tires Repo — Complete Guide

### 12.1 Scaffold Checklist

```
□ 1. Bootstrap Next.js 14 App Router project
     npx create-next-app@14 gdx-ux-tires \
       --typescript --app --src-dir --no-tailwind \
       --import-alias "@/*"

□ 2. Install Costco Forge design system
     npm install \
       @nxt-costco-com/forge-components \
       @nxt-costco-com/forge-design-tokens \
       @nxt-costco-com/forge-digital-components \
       @nxt-costco-com/forge-icons

□ 3. Install core runtime dependencies
     npm install \
       next-i18n-router \
       contentstack \
       @t3-oss/env-nextjs \
       zod \
       @emotion/react @emotion/styled @emotion/cache \
       @mui/material-nextjs @mui/system @mui/utils \
       pino \
       js-cookie react-cookie \
       @types/js-cookie \
       redis @neshca/cache-handler \
       sharp \
       prom-client \
       @vercel/otel \
       @opentelemetry/core \
       @opentelemetry/exporter-trace-otlp-http \
       @opentelemetry/instrumentation \
       @opentelemetry/instrumentation-http \
       @opentelemetry/sdk-logs \
       undici \
       logrocket logrocket-react

□ 4. Install dev dependencies
     npm install -D \
       vitest @vitest/coverage-v8 \
       @vitejs/plugin-react \
       vite-tsconfig-paths \
       @testing-library/react \
       @testing-library/user-event \
       @testing-library/jest-dom \
       @testing-library/dom \
       msw \
       @playwright/test \
       prettier \
       @trivago/prettier-plugin-sort-imports \
       eslint \
       oxlint \
       dotenv \
       cross-env \
       @next/bundle-analyzer \
       next-compose-plugins

□ 5. Copy config files from search-catalog repo:
     cp tsconfig.json                  → (update paths, remove search-specific aliases)
     cp tsconfig.test.json             → (no changes needed)
     cp vitest.config.mts              → (update aliases)
     cp vitest.setup.ts                → (no changes needed)
     cp playwright.config.ts           → (update base URL)
     cp oxlintrc.json                  → (no changes needed)
     cp ecosystem.config.js            → (update app name)
     cp Dockerfile                     → (no changes needed)
     cp grafana/docker-compose.yml     → (no changes needed)
     cp grafana/prometheus.yml         → (no changes needed)

□ 6. Create env.mjs — declare all tire-specific env vars with Zod
□ 7. Create src/constants/language/index.ts — i18nConfig
□ 8. Create src/middleware.ts — copy + adapt from search repo
□ 9. Create src/instrumentation.ts — copy exactly from search repo
□ 10. Create src/metrics.ts — copy exactly from search repo
□ 11. Create src/metrics-server.ts — copy exactly from search repo
□ 12. Create src/mock/handlers.ts + server.ts
□ 13. Create src/providers/mui/ThemeRegistry.tsx — copy exactly
□ 14. Create src/services/logger.ts — copy exactly
□ 15. Copy src/services/content/ — entire directory, no changes
□ 16. Create src/app/[lang]/(customers)/layout.tsx — root layout
□ 17. Create src/app/[lang]/(customers)/page.tsx — home page
□ 18. Build TireSearchWidget (most complex new component)
□ 19. Set up ContentStack content types (see 12.7)
```

---

### 12.2 Environment Variables

Create `env.mjs` with these Zod-validated variables:

```js
import { createEnv } from '@t3-oss/env-nextjs'
import { z } from 'zod'

export const env = createEnv({
  server: {
    // ContentStack — Tire Content
    CONTENTSTACK_TIRES_API_KEY: z.string().min(1),
    CONTENTSTACK_TIRES_DELIVERY_TOKEN: z.string().min(1),
    CONTENTSTACK_TIRES_ENVIRONMENT: z.string().min(1),
    CONTENTSTACK_TIRES_REGION: z.string().default('US'),

    // ContentStack — Config
    CONTENTSTACK_CONFIG_API_KEY: z.string().min(1),
    CONTENTSTACK_CONFIG_DELIVERY_TOKEN: z.string().min(1),
    CONTENTSTACK_CONFIG_ENVIRONMENT: z.string().min(1),

    // Tire Catalogue API
    TIRE_CATALOGUE_BASE_URL: z.string().url(),
    TIRE_CATALOGUE_API_KEY: z.string().min(1),

    // Vehicle Lookup API (license plate → vehicle)
    VEHICLE_LOOKUP_BASE_URL: z.string().url(),
    VEHICLE_LOOKUP_API_KEY: z.string().min(1),

    // Warehouse API
    WAREHOUSE_API_BASE_URL: z.string().url(),

    // Appointment Scheduling
    APPOINTMENT_API_BASE_URL: z.string().url(),

    // Redis cache
    REDIS_URL: z.string().optional(),

    // Observability
    OTEL_EXPORTER_OTLP_ENDPOINT: z.string().url().optional(),

    // Site
    SITE: z.string().optional(),
    NODE_ENV: z.enum(['development', 'production', 'test']).default('development'),
  },
  client: {
    NEXT_PUBLIC_BASE_PATH: z.string().optional(),
    NEXT_PUBLIC_GOOGLE_MAPS_KEY: z.string().optional(), // For warehouse map
  },
  runtimeEnv: {
    CONTENTSTACK_TIRES_API_KEY: process.env.CONTENTSTACK_TIRES_API_KEY,
    // ... etc
  },
})
```

---

### 12.3 Recommended Folder Structure

```
gdx-ux-tires/
├── env.mjs
├── next.config.mjs
├── tsconfig.json
├── tsconfig.test.json
├── vitest.config.mts
├── vitest.setup.ts
├── playwright.config.ts
├── Dockerfile
├── ecosystem.config.js
├── oxlintrc.json
├── global.d.ts
├── start.js
├── public/
│   ├── robots933456.txt
│   └── tiresAssets/
│       └── fonts/
├── grafana/
│   ├── docker-compose.yml
│   └── prometheus.yml
└── src/
    ├── instrumentation.ts
    ├── middleware.ts
    ├── metrics.ts
    ├── metrics-server.ts
    │
    ├── actions/
    │   └── auth.ts
    │
    ├── app/
    │   ├── _lib/
    │   │   └── types.ts                    # LocalizedLayoutProps, TirePageProps
    │   ├── [lang]/
    │   │   ├── (customers)/
    │   │   │   ├── layout.tsx              # Root <html> shell + consent scripts
    │   │   │   ├── page.tsx                # Home page  →  /[lang]/
    │   │   │   ├── tires/
    │   │   │   │   ├── layout.tsx          # Header, Footer, all providers
    │   │   │   │   └── page.tsx            # Search results  →  /[lang]/tires
    │   │   │   ├── tire/
    │   │   │   │   └── [itemNumber]/
    │   │   │   │       └── page.tsx        # Tire detail  →  /[lang]/tire/123456
    │   │   │   ├── brands/
    │   │   │   │   └── [brandSlug]/
    │   │   │   │       └── page.tsx        # Brand page  →  /[lang]/brands/Michelin_tires
    │   │   │   ├── appointment/
    │   │   │   │   └── page.tsx            # Appointment scheduler
    │   │   │   ├── info/
    │   │   │   │   └── [slug]/
    │   │   │   │       └── page.tsx        # FAQ, TireBasics, TireTips, Warranty, etc.
    │   │   │   └── (legacy)/
    │   │   │       └── [...path]/          # Catch-all for old /TireTips, /FAQ etc.
    │   │   │           └── page.tsx
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
    │           ├── search/route.ts         # GET /api/tires/search?size=215-55R17
    │           ├── vehicle-lookup/route.ts # GET /api/tires/vehicle-lookup?plate=ABC123&state=CA&zip=90210
    │           ├── vehicle-sizes/route.ts  # GET /api/tires/vehicle-sizes?year=2022&make=Honda&model=Civic&trim=LX
    │           ├── warehouses/route.ts     # GET /api/tires/warehouses?zip=90210&radius=50
    │           └── appointment/route.ts   # POST /api/tires/appointment
    │
    ├── components/
    │   │   ── Reused from search-catalog (copy as-is) ──
    │   ├── Analytics/
    │   ├── BreadCrumb/
    │   ├── CenteredBox/
    │   ├── ErrorBoundary/
    │   ├── ErrorServiceUnavailable/
    │   ├── Feedback/
    │   ├── KMSI/
    │   ├── LivePreview/
    │   ├── NotFoundUI/
    │   ├── OneTag/
    │   ├── TranscendOverride/
    │   │
    │   │   ── Tire-specific NEW components ──
    │   ├── TireSearchWidget/
    │   │   ├── index.tsx                   # Tab container
    │   │   ├── index.test.tsx
    │   │   ├── LicensePlateTab.tsx         # Tab 1: plate + state + zip
    │   │   ├── LicensePlateTab.test.tsx
    │   │   ├── VehicleTab.tsx              # Tab 2: year/make/model/trim cascading dropdowns
    │   │   ├── VehicleTab.test.tsx
    │   │   ├── TireSizeTab.tsx             # Tab 3: width/ratio/rim dropdowns
    │   │   ├── TireSizeTab.test.tsx
    │   │   ├── ItemNumberTab.tsx           # Tab 4: item number text input
    │   │   ├── ItemNumberTab.test.tsx
    │   │   ├── types.ts                    # SearchMode, VehicleSearchParams, etc.
    │   │   └── constants.ts               # Tire width/ratio/rim options arrays
    │   ├── TireResultsGrid/
    │   │   ├── index.tsx                   # Grid of TireProductTile components
    │   │   ├── index.test.tsx
    │   │   ├── TireProductTile.tsx         # Individual tire card
    │   │   ├── TireProductTile.test.tsx
    │   │   ├── LoadingTiresGrid.tsx        # Skeleton loader
    │   │   └── types.ts
    │   ├── TireFilters/
    │   │   ├── index.tsx                   # Filter panel container
    │   │   ├── BrandFilter.tsx             # Brand checkbox group
    │   │   ├── TireTypeFilter.tsx          # All-Season / Winter / Summer
    │   │   ├── PriceRangeFilter.tsx        # Price range slider
    │   │   ├── SpeedRatingFilter.tsx       # H/V/W/Y/T checkboxes
    │   │   ├── MileageWarrantyFilter.tsx   # Warranty mileage checkboxes
    │   │   └── types.ts
    │   ├── TireDetailSpec/
    │   │   ├── index.tsx                   # Spec table component
    │   │   ├── index.test.tsx
    │   │   └── types.ts                    # TireSpec interface
    │   ├── TireDetailWarranty/
    │   │   ├── index.tsx                   # Warranty section (road hazard + manufacturer)
    │   │   └── index.test.tsx
    │   ├── BrandCarousel/
    │   │   ├── index.tsx                   # Auto-play brand image carousel
    │   │   ├── index.test.tsx
    │   │   └── BrandSlide.tsx              # Individual brand slide
    │   ├── HowItWorks/
    │   │   ├── index.tsx                   # 4-step process section
    │   │   ├── index.test.tsx
    │   │   └── Step.tsx                    # Individual step card
    │   ├── CostcoAdvantage/
    │   │   ├── index.tsx                   # Advantage checklist component
    │   │   └── index.test.tsx
    │   ├── AppointmentScheduler/
    │   │   ├── index.tsx                   # Multi-step scheduling wizard
    │   │   ├── index.test.tsx
    │   │   ├── WarehouseStep.tsx           # Step 1: select tire center
    │   │   ├── DateTimeStep.tsx            # Step 2: pick date + time
    │   │   ├── ConfirmStep.tsx             # Step 3: confirm details
    │   │   └── types.ts
    │   ├── WarehouseSelector/
    │   │   ├── index.tsx                   # Zip-based warehouse search
    │   │   ├── index.test.tsx
    │   │   ├── WarehouseMap.tsx            # Google Maps integration
    │   │   ├── WarehouseList.tsx           # List view of results
    │   │   └── WarehouseCard.tsx           # Individual warehouse card
    │   ├── SortSelector/                   # Reuse from search-catalog
    │   └── TirePagination/                 # Adapted from SearchResultsPagination
    │
    ├── containers/
    │   ├── common/
    │   │   ├── footer-full.tsx
    │   │   └── header.tsx
    │   ├── home/
    │   │   ├── index.tsx                   # HomeContainer (async RSC)
    │   │   ├── HeroSection.tsx             # Hero + TireSearchWidget
    │   │   ├── WhyCostcoSection.tsx        # "Why Costco?" slider
    │   │   ├── HowItWorksSection.tsx       # 4-step section
    │   │   ├── BrandsSection.tsx           # Brand carousel
    │   │   ├── TireInfoSection.tsx         # 3 editorial cards
    │   │   └── SEOContentSection.tsx       # Bottom text block
    │   ├── tireSearch/
    │   │   ├── index.tsx                   # TireSearchContainer (async RSC)
    │   │   ├── LeftColumn.tsx              # Filter sidebar
    │   │   ├── MainColumn.tsx              # Grid + pagination
    │   │   ├── NoResultsComponent.tsx
    │   │   ├── utils.ts
    │   │   └── constants.ts
    │   ├── tireDetail/
    │   │   ├── index.tsx                   # TireDetailContainer (async RSC)
    │   │   ├── ImageGallery.tsx
    │   │   ├── PurchasePanel.tsx           # Price + Add to Cart
    │   │   ├── SpecSection.tsx
    │   │   ├── WarrantySection.tsx
    │   │   └── ReviewsSection.tsx
    │   └── brandLanding/
    │       └── index.tsx                   # BrandLandingContainer
    │
    ├── constants/
    │   ├── index.ts                        # Tire facet keys, cookie names
    │   ├── language/
    │   │   └── index.ts                    # i18nConfig
    │   ├── tires.ts                        # Tire size arrays, brand list, speed ratings
    │   ├── contentStack.ts                 # CMS content type UIDs for tires
    │   ├── styles.ts
    │   └── errorMessages/
    │
    ├── hooks/
    │   ├── useCheckScreen.ts               # Copy from search-catalog
    │   ├── useQueryString.ts               # Copy from search-catalog
    │   ├── useTireSearch.ts                # New: manages tire search state
    │   ├── useVehicleDropdowns.ts          # New: cascading Y/M/M/T dropdown state
    │   └── useWarehouseLocation.ts         # New: reads/writes selected warehouse
    │
    ├── mock/
    │   ├── handlers.ts                     # MSW handlers for tire APIs
    │   └── server.ts
    │
    ├── providers/
    │   └── mui/
    │       └── ThemeRegistry.tsx
    │
    ├── services/
    │   ├── logger.ts                       # Copy from search-catalog
    │   ├── content/                        # Copy entire directory from search-catalog
    │   │   ├── stackClient.ts
    │   │   ├── config.lib.ts
    │   │   ├── featureFlag.ts
    │   │   └── unstable_cache.ts
    │   └── tireSearch/
    │       ├── TireSearchProvider.tsx      # 'use client' — tire search context
    │       ├── TireSearchProvider.test.tsx
    │       ├── vehicleLookup.ts            # License plate → vehicle API call
    │       ├── vehicleLookup.test.ts
    │       ├── vehicleSizes.ts             # Y/M/M/T → tire sizes API call
    │       ├── vehicleSizes.test.ts
    │       ├── tireCatalogue.ts            # Tire search/filter API calls
    │       ├── tireCatalogue.test.ts
    │       ├── constants.ts               # Search URL params for tires
    │       └── types.ts                    # TireSearchResult, TireFilter, etc.
    │
    ├── styles/
    │   ├── global.scss
    │   ├── colors.scss
    │   └── fonts.scss
    │
    ├── types/
    │   ├── types.d.ts
    │   ├── cookies.ts
    │   ├── metrics.ts
    │   ├── contentStack/                   # Tire CMS entry types
    │   │   ├── TirePageConfig.ts
    │   │   ├── BrandLandingPage.ts
    │   │   └── InfoPage.ts
    │   ├── layouts/
    │   │   └── index.ts
    │   ├── pages/
    │   │   └── tire.ts                     # TirePageType, TireSearchPageProps
    │   └── tireSearch/
    │       ├── index.ts                    # TireSearchResult, TireSpec, TireFilter
    │       ├── vehicle.ts                  # Vehicle, VehicleYear, VehicleMake, etc.
    │       └── warehouse.ts               # Warehouse, AppointmentSlot, etc.
    │
    └── utils/
        ├── index.ts                        # Copy from search-catalog
        ├── lib.ts                          # Copy from search-catalog
        ├── cookieUtils.ts                  # Copy from search-catalog
        ├── middleware.ts                   # Adapt from search-catalog (tire-specific redirects)
        ├── metrics.ts                      # Copy from search-catalog
        ├── metricsHelper.ts               # Copy from search-catalog
        ├── metricsClassifier.ts           # Adapt — add tire route patterns
        ├── inboundMetricsClassifier.ts    # Copy from search-catalog
        ├── fetchContentStackPolicies.ts   # Copy from search-catalog
        ├── next.ts                         # Copy from search-catalog
        ├── tireSize.ts                     # NEW: parse "215/55R17" format helpers
        ├── tireSize.test.ts
        ├── vehicleUtils.ts                # NEW: format vehicle display strings
        ├── vehicleUtils.test.ts
        └── links.ts                        # Copy + add tire URL builders
```

---

### 12.4 Key Files — Code Templates

#### `src/services/tireSearch/TireSearchProvider.tsx`

```tsx
'use client'

import { createContext, useContext, useEffect, useState } from 'react'
import { useQueryParams } from '@nxt-costco-com/forge-digital-components'
import type { TireSearchResult, TireFilter, SearchMode } from '@/types/tireSearch'

interface TireSearchContextValue {
  results: TireSearchResult[]
  filters: TireFilter[]
  totalResults: number
  currentPage: number
  isLoading: boolean
  searchMode: SearchMode
  activeFilters: Record<string, string[]>
  setFilter: (key: string, values: string[]) => void
  clearFilters: () => void
}

const TireSearchContext = createContext<TireSearchContextValue | null>(null)

export function TireSearchProvider({ children, initialConfig }: Props) {
  const { queryParams, setQueryParams } = useQueryParams()
  const [results, setResults] = useState<TireSearchResult[]>([])
  const [isLoading, setIsLoading] = useState(false)

  // Determine search mode from URL params
  const searchMode = queryParams.get('vehicle')
    ? 'vehicle'
    : queryParams.get('size')
    ? 'size'
    : queryParams.get('plate')
    ? 'plate'
    : queryParams.get('item')
    ? 'item'
    : 'size'

  useEffect(() => {
    // Fetch tire results whenever search params change
    const fetchResults = async () => {
      setIsLoading(true)
      const params = Object.fromEntries(queryParams.entries())
      const response = await fetch(`/api/tires/search?${new URLSearchParams(params)}`)
      const data = await response.json()
      setResults(data.tires)
      setIsLoading(false)
    }
    fetchResults()
  }, [queryParams])

  return (
    <TireSearchContext.Provider value={{ results, isLoading, searchMode, ... }}>
      {children}
    </TireSearchContext.Provider>
  )
}

export function useTireSearch() {
  const ctx = useContext(TireSearchContext)
  if (!ctx) throw new Error('useTireSearch must be inside TireSearchProvider')
  return ctx
}
```

#### `src/constants/tires.ts`

```ts
// Tire size options for TireSizeTab dropdowns
export const TIRE_WIDTHS = [155, 165, 175, 185, 195, 205, 215, 225, 235, 245, 255, 265, 275, 285, 295, 305, 315, 325, 335, 345] as const

export const TIRE_RATIOS = [25, 30, 35, 40, 45, 50, 55, 60, 65, 70, 75, 80] as const

export const TIRE_RIMS = [14, 15, 16, 17, 18, 19, 20, 21, 22, 24] as const

// Speed ratings
export const SPEED_RATINGS = {
  T: { label: 'T (up to 118 mph)', maxSpeed: 118 },
  H: { label: 'H (up to 130 mph)', maxSpeed: 130 },
  V: { label: 'V (up to 149 mph)', maxSpeed: 149 },
  W: { label: 'W (up to 168 mph)', maxSpeed: 168 },
  Y: { label: 'Y (up to 186 mph)', maxSpeed: 186 },
} as const

// Tire types
export const TIRE_TYPES = {
  allSeason: 'All-Season',
  winter: 'Winter/Snow',
  summer: 'Summer',
  allTerrain: 'All-Terrain',
  highway: 'Highway',
  touring: 'Touring',
} as const

// Brands sold at Costco Tire Centers
export const TIRE_BRANDS = [
  'Bridgestone', 'BFGoodrich', 'Continental', 'Dunlop',
  'Firestone', 'General', 'Goodyear', 'Michelin',
  'Pirelli', 'Uniroyal', 'Yokohama', 'Cooper',
] as const

// Mileage warranty tiers
export const MILEAGE_WARRANTIES = [40000, 50000, 60000, 70000, 80000, 90000] as const

// URL query param keys
export const TIRE_VEHICLE_PARAM = 'vehicle'   // e.g. "2022-Honda-Civic-LX"
export const TIRE_SIZE_PARAM = 'size'         // e.g. "215-55R-17"
export const TIRE_PLATE_PARAM = 'plate'       // e.g. "ABC123"
export const TIRE_STATE_PARAM = 'state'       // e.g. "CA"
export const TIRE_ZIP_PARAM = 'zip'           // e.g. "90210"
export const TIRE_ITEM_PARAM = 'item'         // e.g. "1234567"
export const TIRE_BRAND_FILTER = 'brand'
export const TIRE_TYPE_FILTER = 'type'
export const TIRE_SORT_PARAM = 'sort'         // featured | price-asc | price-desc | rating
```

---

### 12.5 Tire-Specific Component Designs

#### `TireSearchWidget` — Detailed Props & Behaviour

```tsx
interface TireSearchWidgetProps {
  defaultTab?: 'plate' | 'vehicle' | 'size' | 'item'
  locale: string
  onSearch: (params: TireSearchParams) => void  // or navigate via router
}

// TireSizeTab — all three dropdowns must be selected before search enables
// VehicleTab — year→make→model→trim cascade: each dropdown only enables after prev is selected
// LicensePlateTab — all 3 fields required; shows warning for pre-2001 vehicles
// ItemNumberTab — simple text input, validates Costco item number format
```

#### `TireProductTile` — Data Shape

```ts
interface TireProductTile {
  itemNumber: string
  brand: string
  modelName: string         // e.g. "Defender T+H"
  size: string              // e.g. "215/55R17"
  price: number             // per tire
  setOfFourPrice: number    // price × 4 (may have bundle discount)
  imageUrl: string
  rating: number            // 0–5
  reviewCount: number
  mileageWarranty: number   // e.g. 80000
  tireType: string          // All-Season, Winter, etc.
  speedRating: string       // H, V, W, Y, T
  isRunFlat: boolean
  is3PMSF: boolean          // 3-Peak Mountain Snowflake (severe snow service)
  isMplusS: boolean         // M+S (mud and snow) rating
  isInStock: boolean
  deliveryDays: number      // estimated delivery to selected warehouse
}
```

---

### 12.6 API Routes Required

| Route | Method | Query Params | Response | External Service |
|---|---|---|---|---|
| `/api/tires/search` | GET | `size`, `vehicle`, `brand[]`, `type[]`, `minPrice`, `maxPrice`, `sort`, `page`, `pageSize` | `{ tires: TireProductTile[], total, facets }` | Tire catalogue service |
| `/api/tires/vehicle-lookup` | GET | `plate`, `state`, `zip` | `{ year, make, model, trim, sizes: string[] }` | DMV / vehicle database |
| `/api/tires/vehicle-sizes` | GET | `year`, `make`, `model`, `trim` | `{ sizes: TireSize[], vehicles: string[] }` | Tire fitment database |
| `/api/tires/vehicle-makes` | GET | `year` | `{ makes: string[] }` | Fitment database |
| `/api/tires/vehicle-models` | GET | `year`, `make` | `{ models: string[] }` | Fitment database |
| `/api/tires/vehicle-trims` | GET | `year`, `make`, `model` | `{ trims: string[] }` | Fitment database |
| `/api/tires/warehouses` | GET | `zip`, `radius?` | `{ warehouses: Warehouse[] }` | Costco warehouse API |
| `/api/tires/appointment` | POST | body: `{ warehouseId, date, time, tireItemNumbers[] }` | `{ confirmationNumber }` | Appointment system |
| `/api/health` | GET | — | `200 OK` | — |
| `/api/metrics` | GET | — | Prometheus text | — |
| `/api/revalidate` | POST | `secret`, `path` | `{ revalidated: true }` | — |

---

### 12.7 ContentStack Content Types Needed

Create these content types in ContentStack for the tires application:

| Content Type UID | Purpose | Key Fields |
|---|---|---|
| `tire_home_page` | Home page layout | hero_image, hero_title, carousel_slides, brand_carousel, seo_content |
| `tire_search_config` | Search results page config | results_per_page, sort_options, enabled_filters, facet_config |
| `tire_brand_page` | Brand landing page | brand_name, brand_logo, hero_image, promotional_banner, description |
| `tire_info_page` | FAQ, TireBasics, etc. | slug, title, sections[], content_blocks[] |
| `tire_advantage_content` | Costco Advantage page | advantage_items[], page_title, hero_image |
| `feature_toggle` | Feature flags | (same as search-catalog) |
| `configuration_setting` | App config including CSP | (same as search-catalog) |
| `key_value_pair` | UI labels and strings | (same as search-catalog) |

---

### 12.8 Feature-to-File Mapping

| `tires.costco.com` Feature | File(s) to Create/Adapt | Copy from search-catalog? |
|---|---|---|
| Home page hero | `containers/home/HeroSection.tsx` | No — New |
| Tire search widget (all 4 tabs) | `components/TireSearchWidget/` | No — New |
| License plate lookup | `services/tireSearch/vehicleLookup.ts` | No — New |
| Vehicle Y/M/M/T dropdowns | `components/TireSearchWidget/VehicleTab.tsx` + `hooks/useVehicleDropdowns.ts` | No — New |
| Tire size dropdowns | `components/TireSearchWidget/TireSizeTab.tsx` | No — New |
| "Why Costco?" slider | `containers/home/WhyCostcoSection.tsx` | No — New |
| Brand carousel | `components/BrandCarousel/` | Adapt from `DisplayGroup` |
| "How It Works" 4 steps | `components/HowItWorks/` | No — New |
| Tire results grid | `components/TireResultsGrid/` | Adapt from `SearchResultsGrid` |
| Tire product tile | `components/TireResultsGrid/TireProductTile.tsx` | Adapt from Forge `ProductTileUI` |
| Tire filters panel | `components/TireFilters/` | Adapt from `SearchResultsFacets` / `FilterGroup` |
| Sort selector | `components/SortSelector/` | Copy directly |
| Pagination | `components/TirePagination/` | Copy from `SearchResultsPagination` |
| Tire detail page | `containers/tireDetail/` | No — New |
| Tire spec table | `components/TireDetailSpec/` | No — New |
| Road hazard warranty badge | `components/TireDetailWarranty/` | No — New |
| Appointment scheduler | `components/AppointmentScheduler/` | No — New |
| Warehouse map/list | `components/WarehouseSelector/` | No — New |
| Brand landing page | `containers/brandLanding/` | No — New |
| FAQ page | `containers/infoPage/` + CMS content | No — New (uses CMS content) |
| Tire Basics page | Same `infoPage` container | No — New (CMS content) |
| Costco Advantage page | `components/CostcoAdvantage/` | No — New |
| Breadcrumb | `components/BreadCrumb/` | Copy directly |
| Header | `@nxt-costco-com/forge-digital-components` `<Header>` | Reuse exactly |
| Footer | `containers/common/footer-full.tsx` | Copy directly |
| i18n routing | `src/middleware.ts` + `constants/language/` | Copy + minimal adapt |
| ContentStack CMS | `services/content/` | Copy entire directory |
| Feature flags | `services/content/featureFlag.ts` | Copy directly |
| CSP dynamic loading | `instrumentation.ts` + `utils/fetchContentStackPolicies.ts` | Copy exactly |
| Prometheus metrics | `metrics.ts` + `metrics-server.ts` + `instrumentation.ts` | Copy exactly |
| OpenTelemetry traces | `instrumentation.ts` | Copy exactly |
| Auth (MSAL) | Layout providers | Copy layout pattern |
| Consent (Transcend) | Root layout `<head>` | Copy layout pattern |
| MSW test mocks | `src/mock/` | Copy + replace handlers |
| Vitest config | `vitest.config.mts` | Copy + update aliases |

---

---

## 13. Coding Rules & Conventions

All conventions from `.github/copilot-instructions.md` apply to the tires repo. Key rules:

### File Organisation

```
src/components/ComponentName/
  index.tsx          ← Main export (named export, not default)
  index.test.tsx     ← Co-located tests
  SubComponent.tsx   ← Sub-components in same directory
  types.ts           ← Interfaces used by this component only
  constants.ts       ← Constants used by this component only
```

### Naming Conventions

| Type | Convention | Example |
|---|---|---|
| Components | PascalCase | `TireSearchWidget` |
| Hooks | camelCase with `use` prefix | `useTireSearch` |
| Utilities | camelCase | `parseTireSize` |
| Constants | SCREAMING_SNAKE_CASE | `TIRE_WIDTHS` |
| Types/Interfaces | PascalCase | `TireSearchResult` |
| Files | kebab-case for dirs, PascalCase for components | `tire-search/TireSearchWidget.tsx` |
| CSS variables | kebab-case | `--forge-color-brand-primary` |

### Server vs Client Boundary

```
RSC (Server Component) — DEFAULT — NO 'use client'
  ├─ All async data fetching (ContentStack, tire catalogue)
  ├─ Initial page layout
  ├─ generateStaticParams / generateMetadata
  └─ Most container components

Client Component — ONLY when needed — requires 'use client'
  ├─ useState, useEffect, useRef
  ├─ Event handlers (onClick, onChange)
  ├─ Browser APIs (window, document, localStorage)
  ├─ Third-party libs requiring browser (LogRocket, Criteo)
  └─ Providers (context providers must be client)
```

### Zod Validation Pattern

```ts
// ✅ All API route inputs validated with Zod
import { z } from 'zod'
import { NextRequest, NextResponse } from 'next/server'

const SearchParamsSchema = z.object({
  size: z.string().regex(/^\d{3}\/\d{2}R\d{2}$/i).optional(),
  vehicle: z.string().max(100).optional(),
  page: z.coerce.number().int().min(1).max(100).default(1),
  pageSize: z.coerce.number().int().min(1).max(48).default(24),
})

export async function GET(request: NextRequest) {
  const parsed = SearchParamsSchema.safeParse(
    Object.fromEntries(request.nextUrl.searchParams)
  )
  if (!parsed.success) {
    return NextResponse.json({ error: parsed.error.flatten() }, { status: 400 })
  }
  // use parsed.data safely
}
```

### Accessibility Requirements

Following `.github/a11y.md`:
- Every interactive element must have an accessible name via `aria-label` or visible label
- All carousels must have play/pause controls (`aria-pressed`)
- Loading states must use `aria-busy="true"` on the container
- Filter checkboxes must be in a `<fieldset>` with `<legend>`
- Tab panels must use `role="tablist"` + `role="tab"` + `role="tabpanel"` + `aria-selected`
- Product tiles: price must not be visually ambiguous (include `/each` or `/set of 4`)
- Forms: error messages must be linked via `aria-describedby`

### Performance Rules

- Product images: use `next/image`, WebP format, `loading="lazy"` for below-fold tiles
- First tile: `priority` prop on first image (LCP optimization)
- Carousel auto-play: respect `prefers-reduced-motion` — pause if user prefers reduced motion
- Heavy components (TireSearchWidget, maps): use `dynamic()` with `ssr: false` if they use browser APIs
- Avoid `useEffect` for data fetching — use RSC or Suspense with `use()` hook

---

## 14. Quick Reference — Where to Find Things

### Search Catalog Repo (this repo)

| Question | Answer |
|---|---|
| How does routing work? | [src/middleware.ts](src/middleware.ts) + `src/app/[lang]/` |
| Where are search results fetched? | [src/services/search/SearchProvider.tsx](src/services/search/SearchProvider.tsx) |
| Where is CMS data fetched? | [src/services/content/stackClient.ts](src/services/content/stackClient.ts) + `config.lib.ts` |
| How are feature flags read? | [src/services/content/featureFlag.ts](src/services/content/featureFlag.ts) |
| Where are env vars declared? | [env.mjs](env.mjs) (root) |
| How is the page layout configured? | ContentStack `SearchPageConfig` → `getPageConfigs()` in containers |
| How do refinement filters work? | `SearchResultsFacets` + `useRefinement` hook + URL params |
| How do tests mock HTTP? | MSW in [src/mock/handlers.ts](src/mock/handlers.ts) |
| How is CSP loaded? | [src/instrumentation.ts](src/instrumentation.ts) + `utils/fetchContentStackPolicies.ts` |
| How does ISR / cache invalidation work? | Webhook → `/api/contentStack` → `revalidateTag(entryId)` |
| How do I add a new API route? | Create `src/app/api/[name]/route.ts` |
| How do I add a new page? | Create `src/app/[lang]/(customers)/[route]/page.tsx` |
| How do I add a new component? | Create `src/components/[Name]/index.tsx` + `index.test.tsx` |
| How are locale-specific strings loaded? | ContentStack `key_value_pair` + langId cookie |
| Which warehouse is the user in? | `getLocalWarehouseFromCookies()` in [src/utils/cookieUtils.ts](src/utils/cookieUtils.ts) |
| How does the A/B search backend work? | `SearchProvider` reads `provider` cookie/param → selects GRS or LW |
| How does the metrics server start? | `src/instrumentation.ts` → `src/metrics-server.ts` (port 9200) |
| How are Prometheus metrics defined? | [src/metrics.ts](src/metrics.ts) |

---

### Tires Repo (to be built)

| Question | Where to implement |
|---|---|
| Where does license plate search call? | `src/app/api/tires/vehicle-lookup/route.ts` |
| Where is tire search state managed? | `src/services/tireSearch/TireSearchProvider.tsx` |
| Where are tire size arrays defined? | `src/constants/tires.ts` |
| How does vehicle cascade dropdown work? | `src/hooks/useVehicleDropdowns.ts` |
| How does the appointment modal work? | `src/components/AppointmentScheduler/` + `/api/tires/appointment` |
| How is selected warehouse persisted? | Cookie in `src/hooks/useWarehouseLocation.ts` |
| How do CMS tire pages get configured? | ContentStack `tire_home_page` / `tire_search_config` content types |
| How do brand pages work? | `src/app/[lang]/(customers)/brands/[brandSlug]/page.tsx` |
| How is the free installation advantage shown? | `src/components/CostcoAdvantage/index.tsx` |
| How do info pages (FAQ, Basics) work? | `src/app/[lang]/(customers)/info/[slug]/page.tsx` + CMS `tire_info_page` |
| Where are Costco Advantage details? | `src/components/CostcoAdvantage/` + CMS `tire_advantage_content` |

---

## 15. Engineering Analysis — Repository as Template for tires.costco.com

> **Purpose of this section:** A practical, engineering-focused evaluation of this repo's reusability for building the `tires.costco.com` Next.js platform. Based on full source code analysis plus the declared npm dependency graph (including all `@nxt-costco-com/forge-*` packages).

---

### Executive Summary

The `gdx-ux-cnsw-search-catalog` repository is a **production-grade, enterprise Next.js 14 application** built by Costco's own engineering team using the exact same shared Forge Design System (`@nxt-costco-com/forge-components`, `forge-digital-components`, `forge-design-tokens`, `forge-icons`) that the `tires.costco.com` platform must use. The architecture is modern, the patterns are intentional, and the dependency graph is directly aligned with the target platform. The repo provides **very high reuse value** — not as a fork, but as a **reference implementation and source of copy-portable patterns**.

**Overall Usefulness Score: 8.5 / 10**
**React/Next.js Migration Relevance: 9 / 10**

---

### 15.1 Repository Overview

#### Tech Stack (full inventory)

| Layer | Technology | Version |
|---|---|---|
| Framework | Next.js (App Router) | ^14.2.35 |
| Language | TypeScript (strict) | ^5.6.3 |
| Runtime | Node.js | >=22.11.0 |
| UI — Core components | `@nxt-costco-com/forge-components` | ^3.8.1 |
| UI — Business components | `@nxt-costco-com/forge-digital-components` | ^3.14.4 |
| UI — Design tokens | `@nxt-costco-com/forge-design-tokens` | ^3.0.2 |
| UI — Icons | `@nxt-costco-com/forge-icons` | ^2.0.3 |
| Styling system | MUI v6 (Emotion) | ^6.1.6 |
| CSS | SCSS modules + MUI `sx` prop | — |
| State — server | RSC + ISR + Next.js cache | — |
| State — client | React Context (`SearchProvider`) | — |
| CMS | ContentStack (dual-stack: content + config) | SDK ^3.25.1 |
| Search backends | GRS (primary) + LucidWorks (A/B) | REST APIs |
| i18n | `next-i18n-router` | ^5.4.2 |
| Cache layer | Redis via `@neshca/cache-handler` | ^1.9.0 |
| Logging | Pino (GCP severity format) | ^9.6.0 |
| Observability | OpenTelemetry + `prom-client` | ^1.30.0 / ^15.1.3 |
| Auth | `useAuth` from `forge-digital-components` (MSAL + WCS) | — |
| Consent | Transcend + Queue-It | — |
| Analytics | Adobe Analytics (`window.digitalData`) + Criteo OneTag | — |
| Cookies | `js-cookie`, `react-cookie` | — |
| Env validation | `@t3-oss/env-nextjs` + Zod | — |
| HTTP client | `undici` (global fetch replacement) | ^6.21.3 |
| Process manager | PM2 (`ecosystem.config.js`) | — |
| Testing (unit) | Vitest + React Testing Library | ^3.2.4 / ^14.3.1 |
| Testing (API mock) | MSW v2 | ^2.6.5 |
| Testing (e2e) | Playwright | ^1.40.0 |
| Testing (a11y) | `vitest-axe` + `axe-playwright` | — |
| Linting | ESLint + oxlint (dual-linter) | — |
| Formatting | Prettier + `@trivago/prettier-plugin-sort-imports` | — |
| Bundle analysis | `@next/bundle-analyzer` | — |

#### Application Architecture Pattern

```
┌─────────────────────────────────────────────────────────────────┐
│  RSC (Server Components) — default — zero client JS bundle      │
│   ├─ CMS data fetching (ContentStack)                           │
│   ├─ Feature flag evaluation                                    │
│   ├─ ISR cache + revalidation                                   │
│   └─ Server-side Prometheus metrics                             │
├─────────────────────────────────────────────────────────────────┤
│  Edge Middleware (src/middleware.ts)                             │
│   ├─ i18n routing                                               │
│   ├─ CSP header injection                                       │
│   ├─ CDN cache-tag headers (Akamai)                             │
│   └─ Queue-It / metrics                                         │
├─────────────────────────────────────────────────────────────────┤
│  Client Components ('use client') — narrow boundary             │
│   ├─ SearchProvider (React Context)                             │
│   ├─ Filters, sort, pagination interactions                     │
│   └─ Analytics event firing                                     │
└─────────────────────────────────────────────────────────────────┘
```

#### State Management

No Redux, no Zustand, no external state library. State is managed through:
1. **URL search params** — the single source of truth for all filter/sort/page state (via `useQueryParams` from `forge-digital-components`)
2. **React Context** — `SearchProvider` wraps the results page subtree with typed context
3. **RSC cache** — server state is ISR-cached in Redis; `revalidateTag()` on webhook
4. **sessionStorage / localStorage** — GRS session ID and Adobe session ID via `useLocalForage`
5. **Cookies** — warehouse location, language preference, search provider A/B flag

---

### 15.2 Installed Forge Design System — Detailed Inventory

Since the `@nxt-costco-com/forge-*` packages are the primary UI dependency and `tires.costco.com` will use the same packages, here is a full inventory of what this repo uses from each package:

#### `@nxt-costco-com/forge-components` ^3.8.1

Core, low-level UI primitives (accessible, WCAG 2.1 AA compliant):

| Component / Export | Used In This Repo | Relevant for Tires? |
|---|---|---|
| `Grid` | `SearchResultsGrid` product grid | ✅ Tire results grid |
| `Pagination` | `SearchResultsPagination` | ✅ Tire results paging |
| `Select` | `SortSelector` | ✅ Sort, size dropdowns |
| `Skeleton` | Loading states throughout | ✅ All loading states |
| `BackToTopLink` | `SearchResultsPagination` | ✅ All long pages |
| `Divider` | `SearchResultsPagination` | ✅ Section separators |
| `MarkdownRenderer` | `DisplayGroup` CMS rich text | ✅ Brand/info pages |
| `MuiCostcoTheme` | `ThemeRegistry` | ✅ Required for all Forge MUI components |

**Key insight for tires.costco.com:** Import `MuiCostcoTheme` from `forge-components` and wrap the app in `ThemeProvider`. All `forge-components` primitives inherit this theme automatically. Do not create a custom MUI theme — use `MuiCostcoTheme` as-is or extend sparingly.

#### `@nxt-costco-com/forge-digital-components` ^3.14.4

Higher-level business/domain components and hooks. This is the richest package:

**UI Components:**

| Component | Used In This Repo | Relevant for Tires? |
|---|---|---|
| `ProductTileUI` | `SearchResultsGrid` product cards | ✅ Tire product card |
| `ProductTileSkeleton` | Loading state for grid | ✅ Loading tile |
| `CriteoProductTile` | Sponsored tile slots | ✅ Sponsored tire ads |
| `CriteoButterflyDisplay` | Criteo banner ads | ✅ Sponsored banners |
| `CriteoInGridBanner` | In-grid ad placement | ✅ In-grid ads |
| `AccordionSet` | FAQ, CMS content blocks | ✅ FAQ page, tire info |
| `AdBuilder` | Dynamic CMS ad content | ✅ Brand promos |
| `AdSet` | Multi-ad display groups | ✅ Brand promos |
| `BulletDetailCard` | Feature/spec listings | ✅ Tire specs, advantages |
| `ButtonSet` | CTA button groups | ✅ Search CTAs |
| `FeatureHighlightCardV2` | Highlight sections | ✅ "Why Costco?" cards |
| `MarketingTabs` | Tabbed content blocks | ✅ Tire info tabs |
| `ProgramCard` | Program promotion cards | ✅ Costco Advantage cards |
| `RichTextEntry` | CMS rich text content | ✅ All CMS content |
| `SegmentBasedEntry` | User-segment targeting | ✅ Member vs. non-member |
| `TextBuilder` | Dynamic text blocks | ✅ CMS-driven text |
| `TextSection` | Structured text sections | ✅ Info pages |
| `TieredOfferCard` | Tiered pricing display | ✅ Tire pricing tiers |
| `DeliveryLocationSelector` | Delivery zip selector | ✅ Store/delivery selector |
| `WarehouseSelector` | Store selection | ✅ Appointment warehouse |
| `Analytics` | Event wrapper component | ✅ All analytics tracking |

**Hooks:**

| Hook | Purpose | Relevant for Tires? |
|---|---|---|
| `useQueryParams` | Read/write URL search params | ✅ Filter state, search form |
| `useCheckScreen` | Responsive breakpoint detection | ✅ Mobile/desktop layout |
| `useCriteo` | Criteo ad placement logic | ✅ Sponsored results |
| `useBrowseContext` | Browse/CMS context | ✅ Page context |
| `useAuth` | MSAL/WCS authentication | ✅ Member pricing, cart |
| `useSearchProvider` | Search state hook | ✅ Adapt for tire search |
| `useLocalForage` | Async IndexedDB storage | ✅ Session persistence |

**Services / Utilities:**

| Export | Purpose | Relevant for Tires? |
|---|---|---|
| `logger` | Pino-based structured logger | ✅ Copy exact pattern |
| `createUUID` | UUID generation | ✅ Session IDs |
| `ContentStack` | Pre-configured CMS client | ✅ Page content stack |
| `ConfigStack` | Pre-configured config client | ✅ Config/feature flags |
| `FeatureToggle` (type) | Feature flag shape | ✅ All feature flags |
| `findCriteoPlacement` | Criteo slot injection | ✅ Sponsored ads |

#### `@nxt-costco-com/forge-design-tokens` ^3.0.2

CSS custom properties and JS token exports. Used for consistent spacing, color, typography:

```ts
// Example tokens used in this repo:
import { ColorGray80 } from '@nxt-costco-com/forge-design-tokens'
// Applied via MUI sx prop or styled-components
// Tires repo should use the same import pattern — never hard-code colors
```

**Rule for tires.costco.com:** Always use design token imports for color, spacing, and typography values. Never hard-code hex values or pixel spacing.

#### `@nxt-costco-com/forge-icons` ^2.0.3

SVG icon library in React component format. Used throughout for UI icons:

```ts
// Import pattern (same for tires.costco.com):
import { IconName } from '@nxt-costco-com/forge-icons'
```

---

### 15.3 Potential Usefulness for tires.costco.com

#### React Application Architecture ✅ HIGH VALUE

The RSC-first architecture is directly applicable. Copy the pattern:
- All CMS fetches in RSC (`async function Page()`)
- `SearchProvider`-style context for tire search state
- URL params as filter state source-of-truth
- `'use client'` boundary only at interaction leaves

#### Reusable UI Components ✅ HIGH VALUE

**Directly portable from Forge (no modification needed):**
- `ProductTileUI` — adapts to `TireProductTile` with tire-specific fields
- `Pagination` — identical pagination pattern
- `Skeleton` — identical loading states
- `AccordionSet` — FAQ page
- `FeatureHighlightCardV2` — "Why Costco?" / Costco Advantage cards
- `ProgramCard` — brand promotional cards
- `DeliveryLocationSelector` / `WarehouseSelector` — appointment flow
- `MarkdownRenderer` — all CMS rich-text content

**Directly portable from this repo (copy + minor rename):**
- `BreadCrumb/` — identical pattern
- `SortSelector/` — works for tire sort (by price, rating, size)
- `SearchResultsPagination/` — works as TirePagination
- `ErrorBoundary/` — copy exactly
- `ErrorServiceUnavailable/` — copy + update copy text
- `KMSI/` — copy exactly (auth refresh)
- `TranscendOverride/` — copy exactly (consent)
- `CookieBasedContentLoader/` — copy exactly (ad consent gate)

#### Design Patterns ✅ HIGH VALUE

- **RORO pattern** throughout — all functions use `{ param1, param2 }` input / return `{ result, error }` output
- **Discriminated unions** for search results (`isSearchResultGRS` / `isSearchResultLW` type guards)
- **Adapter pattern** for dual-backend normalization
- **Content-driven rendering** via `DisplayGroup` → type guards → specific component
- **Server-first data fetching** with client hydration boundary

#### API Service Layer Patterns ✅ HIGH VALUE

```
src/services/
  content/           ← CMS client (copy entire directory)
    stackClient.ts   ← Dual ContentStack + LoggingDispatcher pattern
    config.lib.ts    ← Typed config queries
    featureFlag.ts   ← Feature flag evaluation
    unstable_cache.ts← ISR cache wrapper
  search/            ← Adapt for tire search backend
    SearchProvider.tsx← Adapt as TireSearchProvider.tsx
    services/Grs.ts  ← Replace with tire catalogue API client
```

#### Authentication Flows ✅ DIRECT REUSE

`KMSI` component + `useAuth` from `forge-digital-components` handles both MSAL and WCS token paths. Copy `KMSI/index.tsx` verbatim. The auth logic lives in `forge-digital-components` — no reimplementation needed.

#### Performance Optimization ✅ HIGH VALUE

- ISR with `revalidate: 10800` — reduce CMS API calls by 99%
- Redis shared cache via `@neshca/cache-handler` — eliminates cold starts across PM2 processes
- `undici` `LoggingDispatcher` — per-request HTTP metrics without instrumentation overhead
- `unstable_cache` wrapping — fine-grained server-side caching
- `priority` image prop on first tile — LCP optimization
- Code splitting: dynamic imports with `ssr: false` for browser-only components

#### Responsive Design ✅ HIGH VALUE

`useCheckScreen` from `forge-digital-components` + Forge Grid system. No custom breakpoint logic needed.

#### Accessibility ✅ HIGH VALUE

Forge components are WCAG 2.1 AA compliant by design. Repo enforces `vitest-axe` in unit tests and `axe-playwright` in e2e. Copy the test setup verbatim.

#### Component Organization ✅ DIRECT COPY

```
src/components/ComponentName/
  index.tsx          ← Named export
  index.test.tsx     ← Co-located tests
  SubComponent.tsx   ← Sub-views
  types.ts           ← Local types
  constants.ts       ← Local constants
  styles.ts          ← MUI sx style objects
```

#### Frontend Scalability ✅ HIGH VALUE

The pattern of `containers/` (smart) + `components/` (dumb) scales cleanly. Containers own CMS data fetching and composition; components are pure/presentational.

#### Error Handling ✅ DIRECT COPY

`ErrorBoundary/` + `error.tsx` page + `ErrorServiceUnavailable/` covers all failure modes. The pattern of returning errors as values from server actions (via `useActionState`) is already encoded in `.github/copilot-instructions.md`.

#### Logging / Monitoring ✅ COPY EXACTLY

```
src/services/logger.ts        ← Pino logger (GCP severity format)
src/metrics.ts                ← prom-client metric definitions
src/metrics-server.ts         ← HTTP /metrics endpoint (port 9200)
src/instrumentation.ts        ← Next.js register() hook — startup
src/utils/metricsHelper.ts    ← Edge metrics helper
```
Copy all five files verbatim. Only change metric label names to `tires_*`.

---

### 15.4 Feature-Level Relevance for tires.costco.com

| tires.costco.com Feature | Relevance | Source in This Repo |
|---|---|---|
| **Hero banners** | `FeatureHighlightCardV2` from `forge-digital-components` + `AdBuilder` | `DisplayGroup` dispatch pattern |
| **Page Header** | `forge-digital-components` `<Header>` (used in root layout) | Root layout pattern |
| **Page Footer** | `containers/common/footer-full.tsx` (copy directly) | Direct copy |
| **Image carousels** | Adapt `DisplayGroup` + `BrandCarousel` (Forge `MarketingTabs`) | `DisplayGroup/index.tsx` |
| **Blog / info pages** | `RichTextEntry` + `AccordionSet` from Forge | `containers/main/utils.ts` dispatch |
| **Product listing pages** | `SearchResultsGrid` → adapt as `TireResultsGrid` | High direct reuse |
| **Search / filter** | `SearchResultsFacets` + `FilterGroup` + `useRefinement` | High direct reuse |
| **Product detail page** | New — use `ProductTileUI` data shape as reference | Low direct reuse |
| **Cart / checkout** | Links to `costco.com` — no cart in this app | Pattern reference only |
| **Appointment scheduling** | New component — `WarehouseSelector` from Forge reusable | `forge-digital-components` `WarehouseSelector` |
| **Store selection** | `DeliveryLocationSelector` + `WarehouseSelector` from Forge | Direct Forge reuse |
| **Inventory lookup** | Adapt search services pattern | `services/search/services/Grs.ts` pattern |
| **Pricing / promotions** | `TieredOfferCard` from Forge | `DisplayGroup` content type dispatch |
| **Vehicle / tire fitment** | New — 4-tab `TireSearchWidget` | Copy `QueryControls` pattern |
| **User profile / account** | `useAuth` from Forge + KMSI component | Direct copy of KMSI pattern |

**Key observation:** Even though the business domain differs (keyword search vs. tire fitment), the engineering infrastructure (CMS, feature flags, auth, analytics, logging, caching, i18n, CSP, testing) is **100% reusable**. Only the domain-specific search logic and UI components differ.

---

### 15.5 React Engineering Best Practices — Evaluation

#### ✅ Modern Patterns Used

| Practice | Status | Evidence |
|---|---|---|
| Functional components only | ✅ | All components except `ErrorBoundary` class |
| Custom hooks | ✅ | `useRefinement`, `useQueryString`, `useCheckScreen` |
| RSC-first, minimal `'use client'` | ✅ | Only 6 components marked `'use client'` |
| TypeScript strict mode | ✅ | `strict: true`, `noImplicitAny`, `noUncheckedIndexedAccess` |
| Lazy loading / code splitting | ✅ | `dynamic()` for non-critical components |
| URL as state source of truth | ✅ | All filters via URL params — bookmarkable, shareable |
| Discriminated unions + type guards | ✅ | `isSearchResultGRS()`, `isSearchResultLW()` |
| Zod input validation at boundaries | ✅ | API routes validate with Zod |
| Co-located tests | ✅ | `index.test.tsx` beside every component |
| MSW for HTTP mocking in tests | ✅ | `src/mock/handlers.ts` |
| Accessibility testing | ✅ | `vitest-axe`, `axe-playwright` |
| RORO pattern | ✅ | All service functions |
| Pino structured logging | ✅ | GCP-compatible severity format |
| Environment validation | ✅ | `@t3-oss/env-nextjs` + Zod in `env.mjs` |
| ISR caching | ✅ | `revalidate: 10800` + Redis |
| Emotion cache deduplication | ✅ | Explicit webpack alias in `next.config.mjs` |
| OTel traces | ✅ | `@vercel/otel` in `instrumentation.ts` |
| Prometheus metrics | ✅ | Custom counters, histograms in `metrics.ts` |
| Edge cache tags | ✅ | Akamai CDN purge via `Edge-Cache-Tag` header |

#### ⚠️ Outdated or Suboptimal Patterns

| Issue | Location | Severity | Recommendation |
|---|---|---|---|
| `ErrorBoundary` is a class component | `src/components/ErrorBoundary/index.tsx` | Low | React does not yet support a hook alternative; class is correct here |
| `ignoreBuildErrors: true` in CONTAINER_BUILD | `next.config.mjs` | Medium | Type errors silenced in CI builds — fix TypeScript errors properly |
| `ignoreDuringBuilds: true` for ESLint in production | `next.config.mjs` | Medium | Linting skipped in container builds — enable in CI pipeline |
| `NODE_TLS_REJECT_UNAUTHORIZED=0` in dev script | `package.json` | Low | Dev only, but should use cert pinning in local env |
| Some `useEffect` hooks for data sync | `SortSelector/index.tsx` | Low | Could be replaced with derived state |
| Legacy Jest config alongside Vitest | `jest.setup.js`, `@types/jest` in devDeps | Low | Remove Jest setup files — Vitest is the test runner |
| `any`-ish cast in `stackClient.ts` LoggingDispatcher | `stackClient.ts` line 22 | Low | Complex generic cast — acceptable but add explicit types |

---

### 15.6 Reusability Assessment

#### Directly Reusable (copy verbatim, zero modification)

| Item | Type |
|---|---|
| `src/services/content/` (entire dir) | CMS service layer |
| `src/services/logger.ts` | Structured logging |
| `src/metrics.ts` + `src/metrics-server.ts` | Prometheus metrics |
| `src/instrumentation.ts` | Startup hook (CSP + OTel + metrics) |
| `src/middleware.ts` | Edge middleware (i18n + CSP + cache tags) |
| `src/providers/mui/ThemeRegistry.tsx` | MUI + Forge theme setup |
| `src/components/ErrorBoundary/` | Error boundary |
| `src/components/KMSI/` | Session refresh |
| `src/components/TranscendOverride/` | Consent override |
| `src/components/CookieBasedContentLoader/` | Ad consent gate |
| `src/components/ErrorServiceUnavailable/` | 503 fallback UI |
| `src/mock/` | MSW test setup |
| `vitest.config.mts` | Vitest configuration |
| `src/utils/cookieUtils.ts` | Cookie parsing |
| `src/utils/links.ts` | URL normalisation |
| `src/utils/lib.ts` | Edge cache tag generation |
| `src/hooks/useQueryString.ts` | Query string builder |
| `tsconfig.json` | TypeScript config |
| `oxlintrc.json` | Linting config |
| `ecosystem.config.js` | PM2 config |
| `.github/copilot-instructions.md` | AI coding conventions |
| `next.config.mjs` (structure) | Next.js config pattern |
| `env.mjs` (structure) | Env validation pattern |

#### Reusable With Modification

| Item | Required Modification |
|---|---|
| `src/services/search/SearchProvider.tsx` | Replace GRS/LW calls with tire catalogue API; keep context shape |
| `src/components/SearchResultsGrid/` | Replace product tiles with `TireProductTile`; keep grid/loading logic |
| `src/components/SearchResultsFacets/` | Replace facet keys with tire facets (brand, size, load rating) |
| `src/components/FilterGroup/` | Replace facet switch cases with tire-specific facet views |
| `src/components/SortSelector/` | Replace sort options with tire sort options (price, rating, name) |
| `src/components/SearchResultsPagination/` | Keep exactly, rename to `TirePagination` |
| `src/components/BreadCrumb/` | Keep structure, update URL patterns |
| `src/components/DisplayGroup/` | Keep dispatch pattern, add tire-specific content types |
| `src/utils/grs.ts` | Remove GRS-specific logic; keep UUID/session ID helpers |
| `src/hooks/useRefinement.ts` | Adapt filter update logic for tire facet keys |
| `src/constants/index.ts` | Keep constants pattern, replace search-specific values |

#### Good Reference Implementation (study and adapt the pattern)

| Item | What to Learn |
|---|---|
| `src/services/search/services/Grs.ts` | Typed API response adapter pattern |
| `src/services/search/services/LucidWorks.ts` | Second adapter — how to normalize two backends |
| `src/services/content/config.lib.ts` | ConfigStack querying pattern |
| `src/services/content/featureFlag.ts` | Feature flag evaluation with per-app + per-site scoping |
| `src/utils/metricsHelper.ts` | Edge middleware metrics pattern |
| `src/utils/middleware.ts` | Middleware utility functions (slug validation, header injection) |
| `src/components/Analytics/` | Adobe Analytics `window.digitalData` push pattern |
| `src/components/OneTag/` | Server component with `unstable_cache` + fallback pattern |
| `src/containers/` | Smart container pattern separating data from presentation |

#### Not Recommended for Direct Reuse

| Item | Reason |
|---|---|
| `src/services/search/services/Grs.ts` (GRS-specific query logic) | GRS is a Costco Search API; tire search uses a different backend |
| `src/services/search/services/LucidWorks.ts` (LW-specific) | Not applicable for tire catalogue |
| `src/components/SearchResultsFacets/grs.constants.ts` | GRS-specific facet keys |
| `src/utils/grs.ts` (GRS session ID logic) | GRS A/B session management not needed |
| `src/components/DepartmentList/` | Search category navigation — not relevant for tires |
| `src/components/QueryControls/` | Search-specific query controls — replace with `TireSearchWidget` |

---

### 15.7 Risks & Concerns

#### Technical Debt

| Risk | Severity | Details |
|---|---|---|
| TypeScript errors suppressed in production builds | **High** | `ignoreBuildErrors: true` in `next.config.mjs` when `CONTAINER_BUILD=true`. Type safety is the first line of defence — this undermines it. Fix: enforce TS in CI, remove flag. |
| ESLint suppressed in production builds | **Medium** | `ignoreDuringBuilds: true` same condition. Fix: run lint as separate CI step before build. |
| Dual test runners (Jest config + Vitest) | **Low** | `jest.setup.js` and `@jest/globals`, `ts-jest` still in devDeps alongside Vitest. Creates confusion. Fix: remove all Jest dependencies. |
| `ignoreDuringBuilds` hides lint regressions | **Medium** | New developers adding code that passes lint locally may not notice CI failures until PR review. |

#### Scalability Concerns

| Concern | Details |
|---|---|
| Large `SearchProvider.tsx` | 400+ line client component handling GRS + LW state. For tires, design a smaller, focused `TireSearchProvider` from the start. |
| `DisplayGroup` type-guard dispatch | 18 content-type cases in one function. As content types grow this becomes hard to maintain. Consider a registry pattern (`Map<string, Component>`) instead. |
| Single middleware file | `src/middleware.ts` handles i18n + CSP + metrics + redirects + cache tags. For tires, split into composable middleware. |

#### Tight Coupling

| Area | Details |
|---|---|
| `SearchProvider` + GRS/LW types | The context type directly references `SearchResultGRS` and `SearchResultLW`. For tires, define a clean `TireSearchResult` interface first, then build the provider around it. |
| Analytics + search result shape | `SearchAnalytics` component is tightly coupled to GRS/LW-specific result fields. For tires, create a `TireAnalytics` component with tire-specific fields. |
| ContentStack entry IDs in env | Many components read specific entry IDs from `process.env.CONTENT_ENTRY_ID_*`. This is correct but must be documented for the tires project's ContentStack setup. |

#### Security Considerations (OWASP Top 10)

| Area | Status | Notes |
|---|---|---|
| SQL/NoSQL injection | ✅ N/A | No direct DB access; all data via ContentStack and GRS APIs |
| Input validation | ✅ Partial | API routes use Zod; some client-side query params not fully validated |
| CSP | ✅ Strong | Dynamic CSP from ContentStack config stack; fallback at build time |
| Sensitive data exposure | ✅ Strong | Logger strips sensitive fields; env vars via Zod validation |
| Auth | ✅ Strong | MSAL via `forge-digital-components` `useAuth`; no custom auth code |
| XSS | ✅ Strong | Next.js RSC auto-escapes; `MarkdownRenderer` from Forge is safe |
| CSRF | ✅ Strong | Server Actions use built-in Next.js CSRF protection |
| `NODE_TLS_REJECT_UNAUTHORIZED=0` | ⚠️ Dev Only | Only in `dev` script — confirm not set in production env |
| Criteo OneTag | ⚠️ Third-party | Loaded via `<Script>` after consent check — acceptable |

#### Performance Risks

| Risk | Details |
|---|---|
| Large client bundle from `forge-digital-components` | This package exports 40+ components. Ensure tree-shaking works (module: ESNext in tsconfig ✅). Verify with `npm run analyze`. |
| Redis unavailability | If Redis is down, ISR cache falls back to in-memory. This degrades multi-instance consistency but does not break the site. |
| ContentStack rate limits | If ContentStack is unreachable at build time (CSP fetch), `next.config.mjs` catches and uses `*` fallback. For tires, implement same fallback. |

---

### 15.8 Recommended Usage for tires.costco.com

#### Step 1 — Scaffold from this repo (Day 1)

```
1. Clone search-catalog and rename to gdx-ux-cnsw-tires-web (or similar)
2. Remove GRS/LW search services
3. Remove search-specific components (SearchResultsFacets, QueryControls, DepartmentList)
4. Keep: entire src/services/content/, middleware, providers, utils, mock, instrumentation, metrics
5. Keep: entire Forge design system dependency versions (exact same versions)
6. Install same npm dependencies (package.json base is identical)
```

#### Step 2 — Forge Components to Use for Each Tires Section

| UI Section | Forge Component(s) | Notes |
|---|---|---|
| Site Header | `forge-digital-components` `<Header>` | No implementation needed |
| Site Footer | `forge-digital-components` `<Footer>` (or copy `containers/common/footer-full.tsx`) | Direct copy |
| Hero Banner | `AdBuilder` + `FeatureHighlightCardV2` | Driven from ContentStack |
| Search Widget Tabs | `forge-components` `Select` + `forge-digital-components` form elements | Custom `TireSearchWidget` |
| "Why Costco?" section | `FeatureHighlightCardV2` | Data from ContentStack |
| Brand Carousel | `MarketingTabs` or custom `BrandCarousel` using `forge-components` `Grid` | Adapt `DisplayGroup` |
| "How It Works" steps | `BulletDetailCard` or custom `HowItWorks` | Simple layout |
| Tire Results Grid | `forge-components` `Grid` + `ProductTileUI` (adapted) | Adapt `SearchResultsGrid` |
| Filter Panel | `forge-components` `Select` + `Skeleton` + custom `TireFilterGroup` | Adapt `FilterGroup` |
| Sort Selector | Copy `SortSelector` directly | Direct copy |
| Pagination | Copy `SearchResultsPagination` | Direct copy |
| Tire Detail | `BulletDetailCard` + `MarkdownRenderer` | New component |
| FAQ / Info Pages | `AccordionSet` + `RichTextEntry` | Data from ContentStack |
| Costco Advantage | `ProgramCard` + `FeatureHighlightCardV2` | Data from ContentStack |
| Brand Pages | `AdSet` + `TextSection` | Data from ContentStack |
| Appointment Modal | `WarehouseSelector` from Forge | With custom scheduling API |
| Store Locator | `DeliveryLocationSelector` + `WarehouseSelector` | From Forge |
| Breadcrumb | Copy `BreadCrumb/` from this repo | Direct copy |
| Loading States | `Skeleton` + `ProductTileSkeleton` | Direct Forge usage |
| Analytics tracking | `Analytics` wrapper component + `useTracking` | Copy `AnalyticsObserver` pattern |
| Consent gate | `CookieBasedContentLoader` + `TranscendOverride` | Direct copy |

#### Step 3 — Architecture Decisions

```
Decision 1: Keep RSC-first architecture
  → All CMS data fetching in Server Components
  → TireSearchProvider is 'use client' (mirrors SearchProvider)
  
Decision 2: Keep URL as single filter state source
  → useQueryParams (forge-digital-components) manages all tire filter params
  → Size, brand, load, speed, price all as URL search params
  
Decision 3: Keep ContentStack dual-stack pattern
  → Content stack for page content (home, brand pages, info pages)
  → Config stack for CSP, feature flags, search rules (tire catalogue config)
  
Decision 4: Keep exact same observability stack
  → Same Prometheus metrics definitions (rename to tires_*)
  → Same OpenTelemetry traces
  → Same Pino logger (GCP format)
  
Decision 5: New tire-specific services only
  → src/services/tireSearch/ (new)
  → src/services/content/ (copy from this repo)
  → No GRS, no LucidWorks
```

#### Step 4 — Tire-Specific New Additions (no analogue in this repo)

```
src/services/tireSearch/
  TireSearchProvider.tsx      ← Adapted from SearchProvider
  vehicleLookup.ts            ← License plate + vehicle cascade API
  tireCatalogue.ts            ← Tire search API client
  
src/components/TireSearchWidget/
  index.tsx                   ← 4-tab: License Plate | Vehicle | Size | Item#
  LicensePlateTab.tsx
  VehicleTab.tsx              ← Year → Make → Model → Option cascades
  TireSizeTab.tsx             ← Width / Ratio / Diameter dropdowns
  ItemNumberTab.tsx
  
src/components/TireResultsGrid/
  index.tsx                   ← Adapted SearchResultsGrid
  TireProductTile.tsx         ← Adapted ProductTileUI (brand, size, load, speed, price, rating)
  TireProductTileSkeleton.tsx
  
src/components/TireFilters/
  index.tsx                   ← Brand | Price | Load Rating | Speed Rating | Season
  
src/components/AppointmentScheduler/
  index.tsx                   ← Modal: find warehouse → select date/time
  WarehouseStep.tsx
  DateTimeStep.tsx
  ConfirmStep.tsx
  
src/hooks/
  useVehicleDropdowns.ts      ← Cascading Year/Make/Model/Option selects
  useWarehouseLocation.ts     ← Warehouse cookie persistence
  useTireSearch.ts            ← Search state + URL param sync
```

---

### 15.9 Final Verdict

#### Scores

| Dimension | Score | Rationale |
|---|---|---|
| **Overall Repository Usefulness** | **8.5 / 10** | Enterprise-grade, same tech stack, same Forge packages, same Costco infrastructure |
| **React/Next.js Migration Relevance** | **9 / 10** | RSC patterns, Forge integration, i18n, auth, analytics all directly applicable |
| **Infrastructure Reusability** | **10 / 10** | CMS, logging, metrics, CSP, testing, env validation — copy verbatim |
| **UI Component Reusability** | **7 / 10** | Forge components are identical; search-specific UI needs adaptation |
| **Domain Logic Reusability** | **4 / 10** | GRS/LW search logic does not apply; tire fitment is new domain |
| **Pattern Quality** | **9 / 10** | RSC-first, typed, accessible, tested, observable — best-in-class for enterprise |

#### Repository Strengths

1. **Exact same Forge dependency versions** — tires.costco.com inherits all component + hook APIs without version conflicts
2. **Production-proven ISR + Redis caching** — copy-paste performance for ContentStack CMS
3. **Enterprise observability** — OTel + Prometheus ready in one file (`instrumentation.ts`)
4. **Strict TypeScript everywhere** — `noUncheckedIndexedAccess` catches null errors at compile time
5. **Dual-linting (ESLint + oxlint)** — fast oxlint for CI + Next.js ESLint for rules
6. **MSW-based API mocking** — test-first development with realistic HTTP mocks
7. **CSP dynamic loading** — ContentStack-driven CSP avoids security config drift
8. **URL-as-state** — filters are shareable, bookmarkable, server-renderable

#### Best Reusable Areas (high priority for tires.costco.com)

1. `src/services/content/` — ContentStack service layer (entire directory)
2. `src/instrumentation.ts` — startup hook (metrics + OTel + CSP)
3. `src/middleware.ts` — i18n + CSP + cache tags
4. `src/providers/mui/ThemeRegistry.tsx` — Forge/MUI theme
5. `src/components/ErrorBoundary/` — error handling
6. `src/components/KMSI/` — session management
7. `src/mock/` — MSW test setup
8. All Forge component usage patterns — especially `ProductTileUI`, `AccordionSet`, `FeatureHighlightCardV2`, `WarehouseSelector`

#### Areas to Avoid Copying

1. `src/services/search/services/Grs.ts` — GRS API is not the tire catalogue API
2. `src/services/search/services/LucidWorks.ts` — not applicable
3. `src/components/SearchResultsFacets/grs.constants.ts` — GRS facet keys
4. `src/components/QueryControls/` — replace with `TireSearchWidget`
5. `src/components/DepartmentList/` — search-specific navigation

#### Estimated Value to tires.costco.com Development

| Area | Without This Repo | With This Repo | Time Saved |
|---|---|---|---|
| CMS integration setup | 2–3 weeks | 1–2 days | ~2.5 weeks |
| Observability setup | 1–2 weeks | 1 day | ~1.5 weeks |
| Auth integration | 1–2 weeks | 1 day | ~1.5 weeks |
| i18n setup | 3–5 days | 1 day | ~3 days |
| Forge component integration | 1 week | 2–3 days | ~4 days |
| Test infrastructure setup | 3–5 days | 1 day | ~3 days |
| Middleware (CSP, cache tags) | 1–2 weeks | 1 day | ~1.5 weeks |
| **Total estimated savings** | | | **~7–9 weeks** |

> This repository accelerates `tires.costco.com` development by approximately **7–9 engineer-weeks** of foundational infrastructure work, allowing the team to focus exclusively on tire-domain business logic from week 1.

---

*End of ARCHITECTURE.md*
