---
stepsCompleted: ["step-01-validate-prerequisites", "step-02-design-epics", "step-03-create-stories"]
inputDocuments:
  - "docs/tires-webapp/prd.md"
  - "docs/tires-webapp/01-architecture/frontend-architecture.md"
  - "docs/tires-webapp/03-user-flows/user-flows-home-page.md"
  - "docs/tires-webapp/03-user-flows/user-flows-common.md"
project: "Costco Tires Web App"
author: "Rupesh.Jain"
date: "2026-05-12"
scope: "MVP 1 — Homepage"
---

# Costco Tires Web App — Home Page Epic & Story Breakdown

## Overview

This document provides the epic and story breakdown for the **Costco Tires Web App — MVP 1 Homepage**, decomposing requirements from the PRD, Frontend Architecture, and UX User Flow specifications into implementable stories with acceptance criteria.

**Scope:** MVP 1 — Next.js Project Setup + Full Homepage (all 5 sections + common shell)

---

## Requirements Inventory

### Functional Requirements

FR1: A member can search for compatible tires by selecting their vehicle's Year, Make, Model, and Trim — with cascading dropdown population from the BFF vehicle catalog API; Trim required before submission; vehicle context persists in Redux for the full session

FR2: A member can search for compatible tires by entering their License Plate number and State (US markets only) — plate + state resolves to vehicle via Mitchell1 VIN decode; partial match, no match, and service unavailable states each return distinct inline errors without crashing the form or disabling other tabs

FR3: A member can search for tires by selecting Tire Size (Width, Aspect Ratio, Diameter) with a toggle for same or different front and rear sizes — invalid formats trigger inline errors with corrective examples; staggered mode accepts two independent size inputs

FR4: A member can search for a specific tire by entering an Item Number or Part Number — valid SKU routes to PDP on same page load; unrecognised part number shows inline error without full page reload

FR5: The system routes members to the correct downstream page based on search type (Vehicle/License Plate → Size Selection; Tire Size → PLP; Item No. → PDP) — routing on submission with no additional action required; vehicle context pre-populated on destination page

FR6: The system displays inline field-level validation errors using custom error styling (not browser-native HTML5 validation) — zero HTML5 popups across all browsers; red border + inline message rendered within 200ms of submission attempt; error clears immediately on field correction

FR7: The system handles and communicates four specific error states: partial license plate match (multiple vehicles found), vehicle found but trim required, no vehicle found, invalid part number — all without full page reload, blank page, or unhandled JS exception

FR8: The system persists member search criteria (vehicle selection, search type, inputs) in session state and restores them on return navigation — search criteria restores in Redux within 500ms; guest context persists 24 hours via localStorage; no sensitive data (auth tokens, PII) in localStorage

FR9: The system detects whether a member has a pre-selected Tire Center and conditionally shows a ZIP Code field when no Tire Center is stored — ZIP field absent from DOM (not hidden) when Tire Center pre-selected; ZIP renders as required when no Tire Center; re-evaluates if state changes mid-session

FR10: The Tire Center Locator renders as a standalone page (/TireCenterLocator) on first tire center selection, and as a modal (isModalContent prop) for all subsequent changes — single reusable implementation; standalone page not directly accessible without navigation context; modal applies minor design adjustments without duplicating logic; selected Tire Center written to Redux + localStorage; header link updates to show chosen name; no routing to Size Selection or PLP until Tire Center is selected

FR11: The Tire Center Locator allows members to search for nearby tire centers by city, state, or ZIP, presenting results as a list alongside a Bing Maps view with interactive markers — found: list + markers + Set as My Tire Center button; not found: "unable to find warehouse" message, prior markers cleared; on selection: Redux + localStorage updated, header updates, member returned to previous flow

FR12: The homepage displays a Deals Carousel with promotional brand slides (Michelin, Firestone, Bridgestone), play/pause controls, and navigation between slides

FR13: The homepage displays a "Why Buy Tires at Costco" section with key benefit content and a Schedule Appointment call-to-action

FR14: The homepage displays a Tire Information Slider with 8 educational content slides that link to external resources on click (via External Link Modal)

FR15: The homepage displays a Blog Section featuring a full-width editorial article ("Best Place to Buy Tires")

FR16: Deals Carousel and Tire Information Slider slides link to external URLs when clicked — External Link Modal intercepts all external navigation with Cancel + Visit External Link options

FR17: The system suppresses the License Plate search tab for Canadian market users (en-CA and fr-CA locales)

FR18: The system renders all user-facing content, labels, and error messages in French for fr-CA locale users — all UI strings externalized into locale resource files; no hardcoded English strings in component code

FR19: The system serves locale-appropriate content and UI variants based on the member's market (en-US, en-CA, fr-CA) — hreflang tags, canonical URLs, locale prefix routing (/[lang]/...)

FR20: The Tire Search Form renders correctly as a standalone page component and as modal content, controlled by an isModalContent configuration prop

FR21: The system reuses the same Tire Search Form component instance when a member initiates a vehicle change from subsequent pages (Change Vehicle Modal)

FR22: The system persists member fitment context (vehicle selection, Tire Center selection, search criteria) across browser sessions using localStorage (whitelist: vehicle, tireCenter, searchCriteria slices; UI slice not persisted)

FR23: The system does not store sensitive member data (authentication tokens, PII) in client-side storage

FR24: The system retrieves vehicle data (Year/Make/Model/Trim options) from the BFF vehicle catalog API to populate search form dropdowns — cascading: each level populates only after the previous is selected

FR25: The system resolves a license plate and state to a vehicle record via the BFF, handling partial match and no-match responses with distinct inline messages

FR26: The system degrades gracefully when the license plate lookup service is unavailable — inline error shown; other search tabs unaffected

FR27: The system retrieves and displays the member's pre-selected Tire Center from Redux/localStorage on page load

FR28: All interactive form elements (inputs, dropdowns, tabs, buttons) are operable by keyboard without requiring a mouse

FR29: All form inputs have programmatically associated labels or ARIA labels, and error messages are linked via aria-describedby

FR30: Carousel and slider components announce slide changes and provide keyboard controls (play/pause, arrow navigation) for screen reader users — aria-live regions required

### Non-Functional Requirements

NFR1: Page load time — LCP ≤ 2 seconds; TTI ≤ 3.5s desktop / ≤ 5s mobile (3G); CLS ≤ 0.1; Lighthouse ≥ 90 desktop / ≥ 75 mobile; Initial JS bundle ≤ 200KB gzipped

NFR2: BFF API response time ≤ 500ms (p95) under normal load

NFR3: Application uptime 99.99% (≤ 52 min downtime/year); BFF API uptime 99.9%

NFR4: All client-server communication over HTTPS; secure headers required (HSTS, CSP, X-Frame-Options: DENY, X-Content-Type-Options: nosniff)

NFR5: Rate limiting applied at BFF on License Plate lookup and vehicle catalog endpoints; all form inputs validated client-side and sanitized server-side before processing

NFR6: WCAG 2.1 Level AA conformance — keyboard operation, ARIA attributes, color contrast ≥ 4.5:1 (normal text) / ≥ 3:1 (large text), visible focus indicators, semantic HTML, focus-trapped modals, automated axe-core CI scanning

NFR7: Three locales fully supported: en-US, en-CA, fr-CA — fr-CA delivered simultaneously with en-CA; no partial-French launch acceptable

NFR8: Responsive design across Bootstrap 5 breakpoints (xs <576px through xxl 1400px+); mobile-first; touch targets ≥ 44px; no horizontal scroll

NFR9: Next.js SSR pages support horizontal scaling; no server-side session state; CDN caching for static assets; ISR eligible for homepage

NFR10: Webpack (not Turbopack) required; SCSS + Bootstrap 5; ESLint + Prettier; CI pipeline baseline; environment variables in .env files (never committed)

### Additional Technical Requirements (Architecture)

- Next.js 14 App Router + TypeScript 5 strict mode (noImplicitAny, noUncheckedIndexedAccess)
- Redux Toolkit ^2.x with redux-persist (localStorage whitelist: vehicle, tireCenter, searchCriteria; UI slice session-only)
- Forge design system: @nxt-costco-com/forge-components ^3.8.1, forge-design-tokens ^3.0.2, forge-digital-components ^3.14.4, forge-icons ^2.0.3
- MUI v6 + Emotion for layout primitives and component-level styling; global styles via SCSS modules
- next-i18n-router for URL-prefix locale routing (/[lang]/...); locales: en-US, en-CA, fr-CA
- Spring Boot 3.2 BFF as sole backend; all BFF calls routed through Next.js API routes (not called directly from client components); MSW v2 handlers for development stubs
- Bing Maps JavaScript API (NEXT_PUBLIC_BING_MAPS_KEY env var); never hardcoded
- Vitest + React Testing Library + MSW for unit/component/integration tests; Playwright for E2E
- oxlint (Rust-based) as primary linter
- All components use named exports only (no default exports); RORO pattern; early returns; RSC-first (add 'use client' only when browser APIs or hooks required)
- Redux provider in root layout (client boundary); ThemeRegistry (MUI) wraps app
- Zod validation on all Next.js API route inputs before BFF calls
- Path aliases: @/components/*, @/redux/*, @/services/*, @/hooks/*, @/types/*, @/utils/*, @/constants/*
- All dropdowns except Sort By implement combobox pattern (searchable, keyboard nav, ARIA attributes)

### UX Design Requirements

UX-HP1: Full-page responsive homepage with Hero Banner containing TireSearchForm visible above the fold across mobile and desktop breakpoints (reference: docs/tires-webapp/02-design/mobile|desktop/images/Home Page/Home Page.png)

UX-HP2: TireSearchForm — License Plate tab: plate + state input fields; designs for No Zip Code and With Zip Code variants; scenarios for Partial License (multiple vehicles found), License Plate Found (trim required), and all error states per design references

UX-HP3: TireSearchForm — Vehicle tab: cascading Year → Make → Model → Trim combobox dropdowns; each level enables only after previous is selected; No Zip Code and With Zip Code design variants

UX-HP4: TireSearchForm — Tire Size tab: Width/Aspect Ratio/Diameter; same front & rear toggle (Toggle 1) and different front & rear toggle (Toggle 2); No Zip Code and With Zip Code design variants; both routed to PLP

UX-HP5: TireSearchForm — Item No./Part No. tab: text input; No Zip Code and With Zip Code variants; error state for invalid part number shows custom inline error per design reference

UX-HP6: Custom form error state design: red border on invalid field + error message directly below field; field background unchanged; no HTML5 browser popup; design reference: docs/.../Error States/Form Field - Error State.png

UX-HP7: Conditional ZIP Code field rendering — absent from DOM when Tire Center pre-selected; required field when no Tire Center; re-evaluates on state change

UX-HP8: Deals Carousel — 3 brand slides (Michelin, Firestone, Bridgestone); play and pause button controls; mobile and desktop layout variants; slide images per design references

UX-HP9: "Why Buy Tires at Costco" section — benefit content + Schedule Appointment CTA; mobile and desktop layout

UX-HP10: Tire Information Slider — 8 educational slides; play/pause controls; click each slide → External Link Modal → external URL; mobile and desktop layout

UX-HP11: Blog Section — full-width "Best Place to Buy Tires" article with warehouse image; mobile and desktop layout

UX-HP12: Header — Signed Out (Sign In/Register link) and Signed In (My Account link) states; Tire Center link (standalone page when no TC selected; modal when TC stored); Cart link; mobile and desktop designs

UX-HP13: Main Site Navigation — desktop: all links visible; mobile: collapsed into "Shop" hamburger menu (off-canvas, slides from left) with Schedule an Appointment link remaining outside the menu

UX-HP14: Footer — standard site footer; mobile and desktop designs

UX-HP15: External Link Modal — intercepts all external navigation; "Cancel" and "Visit External Link" buttons; mobile and desktop designs; required for all external links sitewide (carousel slides, info slider slides, nav links)

UX-HP16: Selected Vehicle / Tire Size banner — displayed below Header + Main Nav; hidden until a search is performed; shows vehicle or size attributes from Redux; "Change" link opens Change Vehicle Modal

UX-HP17: Tire Center Locator — standalone page (/TireCenterLocator) and modal variant (isModalContent prop); Bing Maps + results list; Set as My Tire Center button; Change Tire Center modal and Schedule an Appointment flow modal variants per design references

UX-HP18: TireSearchForm reusability — isModalContent prop controls layout for standalone (homepage) vs modal (Change Vehicle Modal) rendering; same component, no duplication; Change Vehicle Modal design references

### FR Coverage Map

FR1: Epic 3 — Vehicle search tab (cascading dropdowns, BFF catalog API, Redux persistence)
FR2: Epic 3 — License Plate tab (plate + state, Mitchell1 VIN decode, error scenarios, US-only)
FR3: Epic 3 — Tire Size tab (Width/Ratio/Diameter, same/staggered toggle, PLP routing)
FR4: Epic 3 — Item/Part Number tab (SKU lookup, PDP routing, invalid part error)
FR5: Epic 3 — Search routing logic (all 4 search types → correct destination pages)
FR6: Epic 3 — Custom inline form validation (red border + inline message, zero HTML5 popups)
FR7: Epic 3 — Four defined error state scenarios (partial match, trim required, no match, invalid part)
FR8: Epic 3 — Redux state persistence (search criteria, localStorage, 24hr guest session)
FR9: Epic 4 — Conditional ZIP Code field (DOM absent when TC stored; required when no TC)
FR10: Epic 4 — Tire Center Locator (standalone page + isModalContent modal; Redux + localStorage)
FR11: Epic 4 — Tire Center search (Bing Maps + results list; Set as My Tire Center; no-results state)
FR12: Epic 5 — Deals Carousel (3 slides, play/pause, mobile + desktop)
FR13: Epic 5 — Why Buy at Costco section (content + Schedule Appointment CTA)
FR14: Epic 5 — Tire Information Slider (8 slides, external link on click)
FR15: Epic 5 — Blog Section (full-width editorial article)
FR16: Epic 2 — External Link Modal (intercepts all external navigation sitewide)
FR17: Epic 3 — Market-conditional License Plate tab (suppressed for en-CA/fr-CA)
FR18: Epic 2 — fr-CA locale content rendering (all strings externalized, no hardcoded English)
FR19: Epic 1 — Locale routing and multi-market rendering (next-i18n-router, hreflang)
FR20: Epic 3 — TireSearchForm isModalContent prop (standalone + modal rendering modes)
FR21: Epic 3 — Change Vehicle Modal (reuses TireSearchForm with isModalContent)
FR22: Epic 3 — Redux + localStorage persistence (fitment context across sessions)
FR23: Epic 1 — State security (no auth tokens/PII in localStorage)
FR24: Epic 3 — BFF vehicle catalog API integration (Year/Make/Model/Trim dropdown data)
FR25: Epic 3 — BFF license plate resolution (plate + state → vehicle; partial match; no match)
FR26: Epic 3 — License plate service graceful degradation (inline error; other tabs unaffected)
FR27: Epic 4 — Tire Center pre-load from Redux/localStorage on page load
FR28: Epic 2 — Keyboard operability for all interactive elements
FR29: Epic 2 — ARIA labels + aria-describedby for form inputs and errors
FR30: Epic 5 — Carousel aria-live + keyboard controls for accessibility

## Epic List

### Epic 1: Project Foundation & Application Shell
Members can access a working Next.js application at the correct locale-prefixed URL, with all infrastructure (Redux store, i18n routing, API client, environment config, testing) in place for all subsequent epics to build upon.
**FRs covered:** FR19, FR22, FR23 (partial — state security foundations)

### Epic 2: Site Chrome — Header, Navigation & Footer
Members experience a fully functional page shell with responsive header, main navigation, footer, External Link Modal, and the Selected Vehicle banner — all common components that appear on every page.
**FRs covered:** FR16, FR18 (locale string infrastructure), FR28, FR29

### Epic 3: Tire Search Form (Hero Section)
Members can initiate a tire search using any of the 4 search methods (Vehicle, License Plate, Tire Size, Item/Part No.), receive inline validation feedback, and be routed to the correct downstream page with their search context preserved across sessions.
**FRs covered:** FR1, FR2, FR3, FR4, FR5, FR6, FR7, FR8, FR17, FR20, FR21, FR22, FR24, FR25, FR26

### Epic 4: Tire Center Locator (Standalone Page + Modal)
Members can find and select a nearby Costco Tire Center via a Bing Maps view and results list — either on the standalone /TireCenterLocator page (first-time use) or in a modal (subsequent changes) — with their selection persisted to Redux and shown in the header.
**FRs covered:** FR9, FR10, FR11, FR27

### Epic 5: Homepage Content Sections (Carousels, Blog, Why Buy)
Members see a complete, engaging homepage below the hero with the Deals Carousel, Why Buy Tires at Costco section, Tire Information Slider, and Blog Section — all with correct content, play/pause controls, external link handling, and responsive layouts.
**FRs covered:** FR12, FR13, FR14, FR15, FR16 (External Link Modal use), FR30

## Epics & Stories

---

## Epic 1: Project Foundation & Application Shell

Members can access a working Next.js application at the correct locale-prefixed URL, with all infrastructure in place — Redux store, i18n routing, BFF API client, environment configuration, and testing pipeline — so that all subsequent epics can build upon a stable, production-ready foundation.

**FRs covered:** FR19, FR22 (partial — store + persistence config), FR23

---

### Story 1.1: Next.js Application Scaffold

As a developer,
I want the Next.js 14 App Router project bootstrapped with all required tooling and configuration,
So that all subsequent stories have a stable, correctly structured foundation to build upon.

**Acceptance Criteria:**

**Given** the project has been initialized
**When** a developer runs `npm run dev`
**Then** the Next.js development server starts without errors on localhost:3000
**And** Webpack (not Turbopack) is the active bundler (confirmed via next.config.js `webpack` key present; no `--turbo` flag in dev script)

**Given** the project structure
**When** a developer opens the repository
**Then** the following top-level folders exist under `src/`: `app/`, `components/`, `redux/`, `services/`, `hooks/`, `styles/`, `types/`, `utils/`, `constants/`, `mock/`, `providers/`
**And** path aliases are configured in tsconfig.json: `@/components/*`, `@/redux/*`, `@/services/*`, `@/hooks/*`, `@/types/*`, `@/utils/*`, `@/constants/*`
**And** aliases are mirrored in vitest.config.mts

**Given** the TypeScript configuration
**When** `npm run tsc` is executed
**Then** compilation succeeds with zero errors
**And** `strict: true`, `noImplicitAny: true`, and `noUncheckedIndexedAccess: true` are set in tsconfig.json

**Given** code quality tooling
**When** `npm run lint` is executed
**Then** oxlint runs with zero violations on the scaffold code
**And** ESLint config is present as a secondary linter for rules not covered by oxlint
**And** Prettier is configured and `npm run format:check` exits with code 0

**Given** the SCSS and Forge design system
**When** a developer imports `@nxt-costco-com/forge-components`, `forge-design-tokens`, `forge-digital-components`, and `forge-icons`
**Then** all four packages resolve without errors
**And** `src/styles/global.scss` exists with CSS reset and Forge design token custom properties
**And** `src/providers/mui/ThemeRegistry.tsx` is present, wrapping the app with MUI + Emotion theme

**Given** the environment configuration
**When** required env vars are absent at build time
**Then** the build fails with a descriptive Zod validation error from `env.mjs`
**And** `.env.example` contains all required variables with placeholder values
**And** `.env` and `.env.local` are listed in `.gitignore`

**Given** a CI baseline
**When** a PR is opened
**Then** the CI pipeline runs `tsc`, `lint`, `test`, and `build` steps in sequence
**And** the pipeline fails fast on the first failing step

---

### Story 1.2: Redux Toolkit Store & Persistence Setup

As a developer,
I want the Redux Toolkit store configured with redux-persist and all four slices initialized,
So that fitment context (vehicle, Tire Center, search criteria) persists across browser sessions while UI state remains session-only.

**Acceptance Criteria:**

**Given** the Redux store
**When** the app renders
**Then** `src/redux/store.ts` exports a configured store with four slices: `vehicleSlice`, `tireCenterSlice`, `searchCriteriaSlice`, `uiSlice`
**And** `src/redux/ReduxProvider.tsx` is a `'use client'` component wrapping `<Provider>` and `<PersistGate loading={null}>`
**And** `ReduxProvider` is placed inside `ThemeRegistry` in the root layout

**Given** the persist configuration
**When** the store is configured
**Then** `redux-persist` whitelist includes `vehicle`, `tireCenter`, and `searchCriteria` only
**And** `ui` slice is NOT in the persist whitelist — UI state is session-only
**And** the storage key is `tires-root`

**Given** the `vehicleSlice`
**When** it is initialized
**Then** its initial state contains: `year: null`, `make: null`, `model: null`, `trim: null`, `activeSearchTab: 'vehicle'`
**And** it exports actions: `setVehicle`, `clearVehicle`, `setActiveTab`
**And** all state properties are typed (no `any`)

**Given** the `tireCenterSlice`
**When** it is initialized
**Then** its initial state contains: `id: null`, `name: null`, `address: null`
**And** it exports actions: `setTireCenter`, `clearTireCenter`

**Given** the `searchCriteriaSlice`
**When** it is initialized
**Then** its initial state contains: `tireWidth: null`, `tireRatio: null`, `tireDiameter: null`, `isSameFrontRear: true`, `rearTireWidth: null`, `rearTireRatio: null`, `rearTireDiameter: null`, `itemNumber: null`
**And** it exports actions: `setTireSize`, `toggleSameFrontRear`, `setItemNumber`, `clearSearchCriteria`

**Given** typed Redux hooks
**When** a component needs store access
**Then** `useAppDispatch` and `useAppSelector` are exported from `src/redux/hooks.ts`
**And** no component imports plain `useDispatch` or `useSelector` directly — only the typed variants

**Given** unit tests for Redux slices
**When** `npm run test` is executed
**Then** all four slices have co-located test files (`*.test.ts`)
**And** tests cover: initial state shape, each action's effect on state, and persistence whitelist behaviour for vehicle/tireCenter/searchCriteria

---

### Story 1.3: i18n Routing & Locale Infrastructure

As a member visiting tires.costco.com or tires.costco.ca,
I want the application to serve locale-appropriate content at the correct URL prefix,
So that English US, English Canada, and French Canada members each receive the right language and regional experience.

**Acceptance Criteria:**

**Given** a member visiting `/`
**When** `next-i18n-router` middleware processes the request
**Then** the member is redirected to the correct locale-prefixed route (e.g., `/en-US/`, `/en-CA/`, `/fr-CA/`) based on `Accept-Language` header
**And** subsequent navigation preserves the locale prefix in all internal links

**Given** the i18n config at `src/constants/language/index.ts`
**When** it is loaded
**Then** it defines three locales: `en-US`, `en-CA`, `fr-CA`
**And** `en-US` is the default locale
**And** the config is used by both the middleware and `next-i18n-router`

**Given** `hreflang` alternate link tags
**When** a page is rendered
**Then** the `<head>` contains `<link rel="alternate" hreflang="...">` tags for all three locale variants of the current page
**And** canonical URLs are set per locale to prevent duplicate content indexing

**Given** the locale resource file structure
**When** a developer adds a new UI string
**Then** it is added to locale JSON/TS files under `src/constants/` or equivalent locale resource location — not hardcoded in any component
**And** at minimum a placeholder entry exists for `en-US`, `en-CA`, and `fr-CA` for every string key

**Given** a `fr-CA` member
**When** they access any page
**Then** all UI labels, button text, error messages, and navigation items render in French
**And** locale string files for `fr-CA` are non-empty and have parity with `en-US` string count (no missing keys)

---

### Story 1.4: BFF API Client Layer & MSW Mock Setup

As a developer,
I want a centralized BFF API client and MSW mock handlers configured for all required tires endpoints,
So that the development team can build and test all search features without requiring live BFF APIs.

**Acceptance Criteria:**

**Given** the BFF client
**When** any server-side service calls the BFF
**Then** all calls go through `src/services/bff/client.ts` using the `BFF_BASE_URL` environment variable — never hardcoded
**And** `BFF_BASE_URL` is server-only (not `NEXT_PUBLIC_`) and absent from the client bundle
**And** the client throws a typed `BFFError` with HTTP status and path on non-2xx responses

**Given** the Next.js API routes for tires
**When** they are registered
**Then** the following routes exist: `/api/tires/vehicle-catalog/makes`, `/api/tires/vehicle-catalog/models`, `/api/tires/vehicle-catalog/trims`, `/api/tires/vehicle-catalog/sizes`, `/api/tires/vehicle-lookup`
**And** each route validates its query parameters using Zod before calling the BFF
**And** a Zod validation failure returns `400` with the flattened error object

**Given** MSW v2 mock handlers in `src/mock/handlers.ts`
**When** `npm run dev` is started
**Then** MSW intercepts all `/api/tires/*` requests and returns realistic stub data matching the agreed BFF contract schemas
**And** mock data covers: vehicle makes list, models list, trims list, sizes list, license plate found (unique vehicle), license plate partial match (multiple vehicles), license plate not found, and service unavailable (500)

**Given** a standard health check
**When** `/api/health` is called
**Then** it returns `200 OK` with `{ status: 'ok' }`

---

### Story 1.5: Testing Infrastructure

As a developer,
I want Vitest, React Testing Library, MSW, and Playwright configured and verified end-to-end,
So that every subsequent story's tests can be written and run without configuration friction.

**Acceptance Criteria:**

**Given** the unit/component test setup
**When** `npm run test` is executed
**Then** Vitest runs and the scaffold test suite passes (≥ 1 smoke test per slice)
**And** `vitest.config.mts` includes the correct path aliases matching tsconfig.json
**And** `vitest.setup.ts` configures RTL's `@testing-library/jest-dom` matchers

**Given** the MSW integration with Vitest
**When** a component test imports from `src/mock/server.ts`
**Then** MSW intercepts fetch calls in the test environment using `msw/node`
**And** a sample test for a BFF API route handler passes using MSW stub data

**Given** Playwright E2E configuration
**When** `npm run e2e:headless` is executed
**Then** Playwright runs against `localhost:3000` and a smoke test verifying the homepage loads (status 200) passes
**And** `playwright.config.ts` has `baseURL` set from an environment variable

**Given** code coverage
**When** `npm run test:coverage` is executed
**Then** Vitest generates a coverage report
**And** coverage thresholds are configured (branches ≥ 70%, functions ≥ 70%)

---

## Epic 2: Site Chrome — Header, Navigation & Footer

Members experience a complete, responsive page shell with a functional header (signed in/out states, dynamic Tire Center link), main site navigation (desktop + mobile hamburger), footer, External Link Modal, and Selected Vehicle/Tire Size banner — all common components that appear consistently across every page.

**FRs covered:** FR16, FR18 (locale string infrastructure applied), FR28, FR29

---

### Story 2.1: Responsive Header

As a member visiting the tires site,
I want a responsive header showing the correct sign-in state, Tire Center status, and cart link,
So that I can navigate to account management, find or change my Tire Center, and access my cart from any page.

**Acceptance Criteria:**

**Given** an anonymous (signed-out) member
**When** the header renders
**Then** a "Sign In / Register" link is visible
**And** clicking it redirects the member to the external sign-in/register page (via External Link Modal)
**And** "My Account" is not visible

**Given** a signed-in member
**When** the header renders
**Then** "My Account" link is visible and "Sign In / Register" is hidden
**And** clicking "My Account" redirects via External Link Modal to the external account/profile page

**Given** no Tire Center is stored in Redux state
**When** the header renders
**Then** the header displays the text "Tire Center"
**And** clicking the link navigates to the standalone `/TireCenterLocator` page (not a modal)

**Given** a Tire Center is stored in Redux state (`tireCenterSlice.name` is non-null)
**When** the header renders
**Then** the header displays the selected tire center name instead of "Tire Center"
**And** clicking the link opens the Tire Center Locator in a modal (`isModalContent={true}`)

**Given** the Cart link
**When** a member clicks it
**Then** they are navigated to the external Costco cart page via the External Link Modal

**Given** the mobile breakpoint (xs/sm)
**When** the header renders
**Then** it matches the mobile design references: `docs/tires-webapp/02-design/mobile/images/Common Functionality/Header/`
**And** all links are tappable with minimum 44×44px touch targets

**Given** the desktop breakpoint (lg+)
**When** the header renders
**Then** it matches the desktop design references: `docs/tires-webapp/02-design/desktop/images/Common Functionality/Header/`

**Given** keyboard accessibility
**When** a keyboard user tabs through the header
**Then** all interactive elements (Sign In, My Account, Tire Center, Cart) receive visible focus indicators
**And** each link has an `aria-label` or visible text label identifying its destination

---

### Story 2.2: Main Site Navigation

As a member,
I want a responsive main navigation with all site section links and a Schedule an Appointment entry point,
So that I can move between major site areas from any page on both desktop and mobile.

**Acceptance Criteria:**

**Given** the desktop breakpoint (lg+)
**When** the navigation renders
**Then** the following links are visible: "Tires Home", "Batteries" (external), "Care & Maintenance" (external), "Warranty" (external), "Costco.com" (external), "Schedule an Appointment", "Locations" (external)
**And** the layout matches `docs/tires-webapp/02-design/desktop/images/Common Functionality/Main Site Navigation/Main Site Navigation.png`

**Given** the mobile breakpoint (xs/sm)
**When** the navigation renders
**Then** site section links are collapsed into a "Shop" hamburger menu
**And** "Schedule an Appointment" remains visible outside the hamburger menu
**And** tapping the hamburger opens an off-canvas panel sliding in from the left with all site links
**And** the collapsed and expanded states match the mobile design references

**Given** the "Tires Home" link
**When** a member clicks it
**Then** they are navigated to the homepage (`/[lang]/`) with no External Link Modal

**Given** any external navigation link (Batteries, Care & Maintenance, Warranty, Costco.com, Locations)
**When** a member clicks it
**Then** the External Link Modal opens with "Cancel" and "Visit External Link" buttons
**And** clicking "Visit External Link" navigates to the external URL in a new tab
**And** clicking "Cancel" closes the modal and keeps the member on the current page

**Given** the "Schedule an Appointment" link
**When** a member clicks it
**Then** the Schedule an Appointment Modal opens (link only for MVP 1; scheduling flow out of scope)

**Given** the off-canvas mobile menu
**When** it is open
**Then** focus is trapped within the menu
**And** pressing Escape closes the panel
**And** focus returns to the hamburger button on close

---

### Story 2.3: Footer

As a member,
I want a standard site footer with branding and links,
So that I can access supplementary site navigation and legal information from the bottom of any page.

**Acceptance Criteria:**

**Given** the footer component
**When** it renders at mobile breakpoint
**Then** the layout matches `docs/tires-webapp/02-design/mobile/images/Common Functionality/Footer/Footer.png`

**Given** the footer component
**When** it renders at desktop breakpoint
**Then** the layout matches `docs/tires-webapp/02-design/desktop/images/Common Functionality/Footer/Footer.png`

**Given** any external link in the footer
**When** a member clicks it
**Then** the External Link Modal is triggered before navigating away

**Given** the footer as a Server Component
**When** the page renders
**Then** the footer HTML is present in the initial server-rendered HTML (not client-only)
**And** no `'use client'` directive is added to the footer unless an interactive element requires it

---

### Story 2.4: External Link Modal

As a member,
I want to see a warning modal before being sent to an external website,
So that I am aware I am leaving the Costco Tires site and can cancel if the click was accidental.

**Acceptance Criteria:**

**Given** any external URL link anywhere on the site
**When** a member clicks the link
**Then** the External Link Modal opens instead of immediately navigating
**And** the modal displays a clear message that the member is leaving the site
**And** two buttons are shown: "Cancel" and "Visit External Link"

**Given** the External Link Modal is open
**When** the member clicks "Visit External Link"
**Then** the external URL opens in a new browser tab
**And** the modal closes
**And** the member remains on the current page in their original tab

**Given** the External Link Modal is open
**When** the member clicks "Cancel" or presses Escape
**Then** the modal closes
**And** no navigation occurs
**And** focus returns to the element that triggered the modal

**Given** the modal layout at mobile breakpoint
**When** it renders
**Then** it matches `docs/tires-webapp/02-design/mobile/images/Common Functionality/External Link - Modal/External Link Modal.png`

**Given** the modal layout at desktop breakpoint
**When** it renders
**Then** it matches `docs/tires-webapp/02-design/desktop/images/Common Functionality/External Link - Modal/External Link Modal.png`

**Given** accessibility requirements
**When** the modal opens
**Then** focus moves to the modal container
**And** focus is trapped within the modal while it is open
**And** the modal has `role="dialog"`, `aria-modal="true"`, and an `aria-labelledby` pointing to the modal title
**And** both buttons are keyboard-operable (Tab to focus, Enter/Space to activate)

**Given** the uiSlice Redux state
**When** an external link is clicked
**Then** `uiSlice.isExternalLinkModalOpen` is set to `true` and `uiSlice.externalLinkTarget` is set to the target URL
**And** on modal close (Cancel or navigation), `isExternalLinkModalOpen` is set to `false` and `externalLinkTarget` is cleared

---

### Story 2.5: Selected Vehicle / Tire Size Banner

As a member who has performed a search,
I want to see my currently selected vehicle or tire size displayed below the navigation on all pages,
So that I always know what fitment context is active and can easily change it if needed.

**Acceptance Criteria:**

**Given** no search has been performed (Redux `vehicleSlice` and `searchCriteriaSlice` are at initial null state)
**When** any page renders
**Then** the Selected Vehicle / Tire Size banner is absent from the DOM (not hidden with CSS — not rendered at all)

**Given** a member has selected a vehicle (year, make, model, trim are non-null in `vehicleSlice`)
**When** any page renders after the search
**Then** the banner is visible below the Header and Main Navigation
**And** it displays the selected vehicle attributes: Year, Make, Model, Trim
**And** a "Change" link is displayed

**Given** a member has searched by tire size (width, ratio, diameter are non-null in `searchCriteriaSlice`)
**When** any page renders after the search
**Then** the banner displays the selected tire size in the format "Width/Ratio R Diameter" (e.g., "225/55 R17")
**And** a "Change" link is displayed

**Given** the "Change" link in the banner
**When** a member clicks it
**Then** the Change Vehicle Modal opens
**And** the modal renders the `TireSearchForm` component with `isModalContent={true}`
**And** form fields are pre-populated with the current Redux state values

**Given** a member updates their vehicle in the Change Vehicle Modal and submits
**When** the modal closes
**Then** the banner immediately updates to reflect the new vehicle/size selection from Redux state
**And** no page reload is required

**Given** keyboard accessibility
**When** the banner is visible
**Then** the "Change" link is keyboard-focusable with a visible focus indicator
**And** it has an `aria-label` such as "Change selected vehicle"

---

## Epic 3: Tire Search Form (Hero Section)

Members can initiate a tire search using any of the 4 search methods — Vehicle, License Plate, Tire Size, Item/Part Number — receive inline validation feedback per the custom error design, and are routed to the correct downstream page with their search context persisted across browser sessions.

**FRs covered:** FR1, FR2, FR3, FR4, FR5, FR6, FR7, FR8, FR17, FR20, FR21, FR22, FR24, FR25, FR26

---

### Story 3.1: TireSearchForm Container, Tab Shell & Hero Layout

As a member landing on the homepage,
I want to see the Tire Search Form prominently above the fold with clearly labelled search tabs,
So that I can immediately understand my search options and select the most appropriate method.

**Acceptance Criteria:**

**Given** the homepage at any viewport width
**When** the page loads
**Then** the Hero Banner with the TireSearchForm is visible above the fold without scrolling
**And** four tabs are present: "License Plate", "Vehicle", "Tire Size", "Item No. / Part No."
**And** the "Vehicle" tab is active/selected by default on first load

**Given** the `en-CA` or `fr-CA` locale
**When** the homepage renders
**Then** the "License Plate" tab is absent from the DOM entirely (not hidden with CSS)
**And** the "Vehicle" tab is the first visible tab

**Given** the `en-US` locale
**When** the homepage renders
**Then** all four tabs are visible including "License Plate"

**Given** the active tab state
**When** a member clicks a different tab
**Then** the active tab is updated in `vehicleSlice.activeSearchTab` Redux state
**And** the tab content area transitions to show the selected tab's form
**And** the active tab persists in Redux so returning members see their last-used tab

**Given** the `isModalContent` prop
**When** `TireSearchForm` is rendered with `isModalContent={false}` (homepage)
**Then** it renders with the full hero banner layout and design
**And** when rendered with `isModalContent={true}` (Change Vehicle Modal)
**Then** it renders in a compact modal-appropriate layout without duplicating component logic

**Given** the design references
**When** the hero section renders on mobile
**Then** it matches `docs/tires-webapp/02-design/mobile/images/Home Page/Home Page.png` (hero area)

**Given** the design references
**When** the hero section renders on desktop
**Then** it matches `docs/tires-webapp/02-design/desktop/images/Home Page/Home Page.png` (hero area)

**Given** accessibility
**When** the tab list renders
**Then** it uses `role="tablist"` with child elements having `role="tab"` and `aria-selected` attributes
**And** the active tab panel has `role="tabpanel"` and is linked via `aria-controls`
**And** arrow key navigation moves focus between tabs

---

### Story 3.2: Vehicle Search Tab (Cascading Dropdowns + BFF Integration)

As a member who knows their vehicle,
I want to select my Year, Make, Model, and Trim from cascading combobox dropdowns populated from the vehicle catalog,
So that I am routed to the Size Selection Page with my vehicle context pre-loaded and saved for my session.

**Acceptance Criteria:**

**Given** the Vehicle tab is active
**When** it first renders
**Then** only the "Year" dropdown is enabled; Make, Model, and Trim are disabled
**And** the Year dropdown is populated by calling `GET /api/tires/vehicle-catalog/makes` (or appropriate years endpoint)

**Given** a member selects a Year
**When** the Year value changes
**Then** `GET /api/tires/vehicle-catalog/makes?year={year}` is called
**And** the Makes dropdown becomes enabled and populated with the results
**And** Model and Trim remain disabled

**Given** a member selects a Make
**When** the Make value changes
**Then** `GET /api/tires/vehicle-catalog/models?year={year}&make={make}` is called
**And** the Models dropdown becomes enabled and populated
**And** Trim remains disabled

**Given** a member selects a Model
**When** the Model value changes
**Then** `GET /api/tires/vehicle-catalog/trims?year={year}&make={make}&model={model}` is called
**And** the Trims dropdown becomes enabled and populated

**Given** all four dropdowns are filled and the member submits
**When** "Find Tires" is clicked
**Then** the vehicle is written to `vehicleSlice` in Redux (`year`, `make`, `model`, `trim`)
**And** the member is navigated to `/[lang]/sizeSelection?year=...&make=...&model=...&trim=...`

**Given** a member submits without selecting Trim
**When** "Find Tires" is clicked
**Then** the Trim dropdown shows a red border and an inline error message: "Please select a trim"
**And** no navigation occurs
**And** no HTML5 browser validation popup appears

**Given** all dropdowns implement the combobox pattern
**When** a member types in a dropdown
**Then** the option list filters to matching entries
**And** the member can navigate options with Up/Down arrow keys and confirm with Enter
**And** ARIA attributes (`role="combobox"`, `aria-expanded`, `aria-autocomplete`, `aria-activedescendant`) are applied correctly

**Given** the member navigates away and returns to the homepage within the same session
**When** the Vehicle tab renders
**Then** the previously selected Year, Make, Model, and Trim are restored from Redux state
**And** restoration happens within 500ms of page load

---

### Story 3.3: License Plate Tab (US Only — All Scenarios)

As a US-based member who knows their license plate,
I want to enter my plate number and state to automatically look up my vehicle,
So that I reach tire results without manually selecting my vehicle year, make, model, and trim.

**Acceptance Criteria:**

**Given** the License Plate tab is active and no Tire Center is stored
**When** it renders
**Then** three fields are visible: "License Plate Number" (text), "State" (combobox dropdown), and "ZIP Code" (text)
**And** the layout matches `docs/tires-webapp/02-design/mobile|desktop/images/.../Search by License Plate (With Zip Code).png`

**Given** a Tire Center is already stored in Redux
**When** the License Plate tab renders
**Then** only "License Plate Number" and "State" fields are visible — ZIP Code field is absent from the DOM
**And** the layout matches the "No Zip Code" design reference

**Given** Scenario 4 — unique vehicle found
**When** a member enters a valid plate + state and submits
**Then** the BFF resolves to a single vehicle record
**And** the vehicle is written to `vehicleSlice` in Redux
**And** the member is navigated to the Size Selection Page

**Given** Scenario 1 — partial match (multiple vehicles found)
**When** the BFF returns multiple vehicle matches
**Then** the form does not navigate away
**And** the message "We found multiple vehicles for this license plate" is displayed inline
**And** a list of matched vehicles is rendered for the member to select from
**And** after selecting the correct vehicle and confirming, the member is navigated to Size Selection Page
**And** the layout matches the "Partial License" design reference

**Given** Scenario 2 — vehicle found but Trim required
**When** the BFF returns a vehicle with a missing trim
**Then** a Trim input field appears inline below the plate fields
**And** the member must enter/select the trim before confirming
**And** on confirmation with trim provided, the member is navigated to Size Selection Page
**And** the layout matches the "License Plate Found" design reference

**Given** Scenario 3 — no vehicle found
**When** the BFF returns no match
**Then** the inline message "We were unable to match your license plate to a vehicle. Please try again or search by Vehicle, Size, or Item #." is displayed
**And** no navigation occurs
**And** other search tabs remain fully functional

**Given** the license plate lookup service is unavailable (BFF returns 500 or times out)
**When** the member submits
**Then** an inline error message is displayed (e.g., "License plate lookup is temporarily unavailable. Please try another search method.")
**And** no unhandled JavaScript exception is thrown
**And** the form remains usable; all other tabs are unaffected

**Given** form submission with an empty plate or state field
**When** "Find Tires" is clicked
**Then** the empty field(s) show red borders with inline required-field error messages
**And** no API call is made
**And** no HTML5 browser popup appears

---

### Story 3.4: Tire Size Tab (Same / Staggered Toggle + PLP Routing)

As a member who knows their tire size,
I want to select Width, Aspect Ratio, and Diameter from dropdowns with a same/staggered front-rear toggle,
So that I am routed to the Products Listing Page filtered by my selected size(s).

**Acceptance Criteria:**

**Given** the Tire Size tab is active
**When** it first renders
**Then** Toggle 1 ("Same Front & Rear Tires Size") is active by default
**And** three dropdowns are shown: Width, Aspect Ratio, Diameter
**And** the layout matches the "Toggle 1 - No Zip Code" or "Toggle 1 - With Zip Code" design references as appropriate

**Given** Toggle 1 is active and a member selects Width, Aspect Ratio, and Diameter then submits
**When** "Find Tires" is clicked
**Then** the selected size is written to `searchCriteriaSlice` (tireWidth, tireRatio, tireDiameter)
**And** the member is navigated to `/[lang]/SearchResultsBySize?width=...&ratio=...&diameter=...`

**Given** the member clicks Toggle 2 ("Different Front & Rear Tires Sizes")
**When** the toggle switches to staggered mode
**Then** six dropdowns are shown: Front Width, Front Aspect Ratio, Front Diameter, Rear Width, Rear Aspect Ratio, Rear Diameter
**And** the layout matches the "Toggle 2" design references
**And** `searchCriteriaSlice.isSameFrontRear` is set to `false`

**Given** Toggle 2 is active and a member completes both front and rear size selections and submits
**When** "Find Tires" is clicked
**Then** both front and rear sizes are written to `searchCriteriaSlice`
**And** the member is navigated to the PLP with both front and rear size parameters in the URL

**Given** the member submits without selecting all required size values
**When** "Find Tires" is clicked
**Then** each empty required dropdown shows a red border with an inline error message
**And** no navigation occurs and no HTML5 popup appears

**Given** the Width dropdown
**When** a member interacts with it
**Then** it implements the combobox pattern (searchable, keyboard navigable, ARIA attributed)
**And** the same applies to Aspect Ratio and Diameter dropdowns

---

### Story 3.5: Item / Part Number Tab (PDP Routing + Invalid Error)

As a member who knows the exact Item Number or Part Number of a tire,
I want to enter it directly into a search field,
So that I am taken immediately to that tire's Product Details Page.

**Acceptance Criteria:**

**Given** the Item/Part Number tab is active
**When** it renders
**Then** a text input labelled "Item No. / Part No." is shown
**And** the layout matches the "Search by Item Number - No Zip Code" or "With Zip Code" design references as appropriate

**Given** Scenario 1 — member enters a valid Item Number and submits
**When** "Find Tires" is clicked
**Then** the member is navigated to `/[lang]/Product?id={itemNumber}` on the same page load
**And** the item number is written to `searchCriteriaSlice.itemNumber`

**Given** Scenario 1 — member enters a valid Part Number and submits
**When** "Find Tires" is clicked
**Then** the member is navigated to the Product Details Page for the matching SKU

**Given** Scenario 2 — member enters an invalid or unrecognised Part Number and submits
**When** the BFF returns a not-found response
**Then** the input field shows a red border
**And** the inline error message "Part number does not exist" is displayed directly below the field
**And** no full page reload occurs
**And** no HTML5 browser popup appears
**And** the field background colour remains unchanged (matching design reference for error state)

**Given** the member submits with an empty field
**When** "Find Tires" is clicked
**Then** the input shows a red border with an inline "This field is required" error
**And** no API call is made

---

### Story 3.6: Conditional ZIP Code Field & Tire Center Locator Trigger

As a member without a pre-selected Tire Center,
I want a ZIP Code field to appear in the search form and the Tire Center Locator to open on submit,
So that I can select a nearby warehouse before being routed to tire results.

**Acceptance Criteria:**

**Given** a Tire Center is stored in Redux (`tireCenterSlice.id` is non-null)
**When** any search tab renders
**Then** the ZIP Code field is absent from the DOM (not rendered — not hidden with CSS)
**And** submitting the form routes directly to the appropriate page based on search type

**Given** no Tire Center is stored (`tireCenterSlice.id` is null)
**When** any search tab renders
**Then** the ZIP Code field is present as a required field
**And** the label reads "ZIP Code"

**Given** the member submits a valid search with a ZIP code and no Tire Center stored
**When** "Find Tires" is clicked with a valid ZIP
**Then** the Tire Center Locator opens
**And** on the member's first-ever visit, it opens as the standalone `/TireCenterLocator` page
**And** on subsequent visits (Tire Center was previously selected and has since been cleared or changed), it opens as a modal

**Given** the ZIP Code field is present and the member submits with an empty ZIP
**When** "Find Tires" is clicked
**Then** the ZIP field shows a red border with an inline error: "Please enter your ZIP code"
**And** no navigation or modal opens

**Given** the member enters a non-5-digit value in the ZIP field
**When** "Find Tires" is clicked
**Then** the ZIP field shows a red border with an inline error: "Please enter a valid 5-digit ZIP code"
**And** no navigation or modal opens

**Given** a Tire Center becomes selected (tireCenterSlice.id changes from null to a value) mid-session
**When** the search form re-renders
**Then** the ZIP Code field is removed from the DOM immediately — without a page reload

---

### Story 3.7: Custom Inline Form Validation (All Search Tabs)

As a member who makes an input error in the Tire Search Form,
I want clear, styled inline error messages to appear directly on the invalid field,
So that I understand exactly what to correct without seeing browser-native error popups.

**Acceptance Criteria:**

**Given** any required field in any search tab is left empty on submission
**When** "Find Tires" is clicked
**Then** every empty required field receives a red border
**And** an error message appears directly below the field
**And** zero browser-native HTML5 validation dialogs appear in Chrome, Safari (iOS + macOS), Edge, or Firefox

**Given** the custom error state
**When** an error is shown on a field
**Then** the field's background colour is unchanged (only the border changes to red)
**And** the error message text is styled per the design reference: `docs/tires-webapp/02-design/desktop/images/Home Page/Tire Search - Search options/Error States/Form Field - Error State.png`

**Given** a field is in an error state
**When** the member types a valid value into the field
**Then** the red border and error message disappear immediately (on change, not on next submit)

**Given** each error message
**When** it renders
**Then** the input element has `aria-invalid="true"` and `aria-describedby` pointing to the error message element's ID
**And** the error message element has `role="alert"` so screen readers announce it immediately

**Given** all form inputs across all four tabs
**When** the HTML source is inspected
**Then** no input has the `required` attribute set as a native HTML attribute (preventing HTML5 popup)
**And** validation is entirely controlled by the React component logic and Zod/react-hook-form

---

### Story 3.8: Search State Persistence (Redux + localStorage)

As a returning member,
I want my search inputs to be automatically restored when I come back to the homepage,
So that I don't have to re-enter my vehicle, size, or location information each visit.

**Acceptance Criteria:**

**Given** a member selects a vehicle (year, make, model, trim) and navigates away
**When** they return to the homepage within the same session
**Then** the Vehicle tab is pre-populated with the previously selected values from Redux
**And** restoration happens within 500ms of page render without a visible loading flash

**Given** a member returns after closing and reopening the browser
**When** the homepage loads
**Then** redux-persist rehydrates `vehicleSlice`, `tireCenterSlice`, and `searchCriteriaSlice` from localStorage
**And** the active search tab is restored to the member's last-used tab
**And** the UI reflects the restored state before the first user interaction

**Given** the persistence whitelist
**When** redux-persist writes to localStorage
**Then** the `vehicle`, `tireCenter`, and `searchCriteria` slices are present in localStorage under the key `tires-root`
**And** the `ui` slice is NOT present in localStorage
**And** no authentication tokens, member PII, or payment data are present in localStorage

**Given** a guest session context
**When** a member has been inactive for more than 24 hours
**Then** the persisted localStorage data expires and is cleared on next page load
**And** the form renders in its default empty state

---

## Epic 4: Tire Center Locator (Standalone Page + Modal)

Members can find and select a nearby Costco Tire Center by searching with a city, state, or ZIP code — viewing results as an interactive list alongside a Bing Maps view with markers — with their selection persisted to Redux and reflected in the header and search form.

**FRs covered:** FR9, FR10, FR11, FR27

---

### Story 4.1: Tire Center Locator — Standalone Page & Search

As a member selecting a Tire Center for the first time,
I want to search for nearby Costco Tire Centers on a dedicated page,
So that I can review warehouse details, see them on a map, and set one as my preferred location before searching for tires.

**Acceptance Criteria:**

**Given** a member navigating to `/[lang]/TireCenterLocator` for the first time
**When** the page loads
**Then** the Tire Center Locator renders as a full standalone page
**And** a search input is visible with a label "Search by city, state, or ZIP code"
**And** a search button is present

**Given** a member enters a valid city, state, or ZIP and submits the search
**When** the BFF `/api/tires/tire-centers` endpoint returns results
**Then** a list of matching Tire Centers is displayed, each showing: name, address, and a "Set as My Tire Center" button

**Given** Scenario 2 — no Tire Centers found for the entered location
**When** the BFF returns an empty results array
**Then** the message "We were unable to find a Costco warehouse near your location. Try searching again." is displayed
**And** the map (if rendered) shows no markers
**And** any markers from a prior search on the same page are cleared

**Given** the member clicks "Set as My Tire Center" on a result
**When** the selection is confirmed
**Then** `tireCenterSlice` is updated with the selected tire center's `id`, `name`, and `address`
**And** the selection is persisted to localStorage via redux-persist
**And** the header "Tire Center" link updates immediately to show the selected tire center name
**And** the member is returned to their previous navigation flow (e.g., the pending search proceeds to Size Selection or PLP)

**Given** the page is accessed directly via URL without a navigation context
**When** the member selects a Tire Center
**Then** they are redirected to the homepage after selection

---

### Story 4.2: Bing Maps Integration for Tire Center Locator

As a member using the Tire Center Locator,
I want to see search results plotted on an interactive Bing Map alongside the list,
So that I can visually understand the locations of nearby warehouses and select the most convenient one.

**Acceptance Criteria:**

**Given** the Tire Center Locator page or modal is open
**When** it renders
**Then** the Bing Maps JavaScript API is loaded lazily (only when the locator component renders — not on every page)
**And** a map container div renders and initialises a `Microsoft.Maps.Map` instance
**And** the Bing Maps API key is read from `NEXT_PUBLIC_BING_MAPS_KEY` — never hardcoded in source code

**Given** a search returns Tire Center results
**When** the results are displayed in the list
**Then** each result is simultaneously plotted as a pushpin marker on the Bing Map
**And** the map viewport adjusts to fit all markers

**Given** a member clicks a map marker
**When** the marker is clicked
**Then** the corresponding Tire Center card in the list is highlighted (visual distinction per design)
**And** the list scrolls to bring the highlighted card into view

**Given** a member clicks a Tire Center card in the list
**When** the card is clicked or focused
**Then** the corresponding map marker is visually highlighted

**Given** the Bing Maps API fails to load (network error, invalid key)
**When** the Tire Center Locator renders
**Then** the map area shows a fallback message ("Map unavailable")
**And** the results list renders and is fully functional without the map
**And** "Set as My Tire Center" functionality works without the map

**Given** the `NEXT_PUBLIC_BING_MAPS_KEY` environment variable
**When** it is absent or empty at build time
**Then** env.mjs Zod validation fails the build with a descriptive error

---

### Story 4.3: Tire Center Locator — Modal Variant (isModalContent)

As a member who wants to change their Tire Center from any page,
I want the Tire Center Locator to open in a modal with minor layout adjustments,
So that I can change my warehouse without leaving my current shopping context.

**Acceptance Criteria:**

**Given** the `TireCenterLocator` component is rendered with `isModalContent={true}`
**When** the modal opens
**Then** it renders the same search, results list, and map functionality as the standalone page
**And** the layout applies modal-specific design adjustments (reduced header, modal chrome)
**And** no component logic is duplicated — it is the same component with a different layout mode

**Given** the "Change Tire Center" flow (from header when TC already selected)
**When** the header Tire Center link is clicked
**Then** `uiSlice.isTireCenterLocatorOpen` is set to `true`
**And** the Tire Center Locator modal opens
**And** the modal matches `docs/tires-webapp/02-design/mobile|desktop/images/Common Functionality/Change Tire Center - Modal/Change Tire Center - Modal.png`

**Given** the "Schedule an Appointment" flow modal variant
**When** the Tire Center Locator opens in this context
**Then** the "Set as My Tire Center" button is replaced with a "Schedule an Appointment" button
**And** the modal matches the "Select or Change Tire Center (Schedule an Appointment Flow)" design references

**Given** the modal is open
**When** accessibility requirements are applied
**Then** focus is trapped within the modal
**And** pressing Escape closes the modal and returns focus to the trigger element
**And** the modal has `role="dialog"`, `aria-modal="true"`, and `aria-labelledby`

---

### Story 4.4: Tire Center Pre-load & Header Sync on Page Load

As a returning member with a previously selected Tire Center,
I want my stored Tire Center to be reflected in the header immediately on page load,
So that I can see my location context without re-selecting and proceed directly to search.

**Acceptance Criteria:**

**Given** a returning member has a Tire Center stored in localStorage (via redux-persist)
**When** the homepage loads
**Then** redux-persist rehydrates `tireCenterSlice` before the first render
**And** the header displays the stored tire center name (not the default "Tire Center" text)
**And** the ZIP Code field is absent from the Tire Search Form

**Given** the Tire Center is stored
**When** `GET /api/tires/tire-centers` is called on page load to validate the stored selection
**Then** if the tire center is still valid, the stored Redux state is unchanged
**And** if the tire center is no longer valid (e.g. closed), `tireCenterSlice` is cleared and the header reverts to "Tire Center"

**Given** `tireCenterSlice` changes during a session (new selection made)
**When** the change is dispatched
**Then** the header updates synchronously with the new tire center name
**And** the ZIP Code field disappears from the search form if it was previously shown
**And** no page reload is required for the UI to reflect the new state

---

## Epic 5: Homepage Content Sections (Carousels, Blog, Why Buy)

Members see a complete and engaging homepage with the Deals Carousel, Why Buy Tires at Costco section, Tire Information Slider, and Blog Section — all rendering with correct content, functional play/pause controls, accessible keyboard support, and correct external link handling via the External Link Modal.

**FRs covered:** FR12, FR13, FR14, FR15, FR16, FR30

---

### Story 5.1: Deals Carousel (3 Brand Slides + Play/Pause)

As a deal-conscious member,
I want to see the Deals Carousel with promotional brand slides and play/pause controls below the hero,
So that I can browse active promotions at my own pace and click through to external deal pages.

**Acceptance Criteria:**

**Given** the Deals Carousel section
**When** the page loads
**Then** the carousel renders below the hero section with three slides in sequence:
Slide 1 — Michelin (`docs/tires-webapp/02-design/desktop/images/Home Page/Tires Deals - Slider/Slides/Slide 1 - Michelin.png`),
Slide 2 — Firestone (`Slide 2 - Firestone.png`),
Slide 3 — Bridgestone (`Slide 3 - Bridgestone.png`)

**Given** the carousel is auto-playing
**When** the play/pause button is visible
**Then** it displays a "Pause" state and the carousel advances slides automatically on an interval
**And** the layout matches `docs/tires-webapp/02-design/mobile|desktop/images/Home Page/Tires Deals - Slider/Tires Deals - Slider with Play and Pause buttons.png`

**Given** the member clicks the Pause button
**When** auto-play is paused
**Then** the carousel stops advancing
**And** the button changes to a "Play" state
**And** clicking Play resumes auto-advance

**Given** a member hovers over the carousel (desktop) or focuses a slide control (keyboard)
**When** the hover or focus occurs
**Then** auto-play is paused while the hover/focus is maintained
**And** auto-play resumes when the member moves focus away

**Given** a member clicks a carousel slide image
**When** the click is registered
**Then** the External Link Modal opens with the slide's promotional URL as the target
**And** confirming navigation opens the external promo page in a new tab

**Given** keyboard navigation
**When** a keyboard user focuses the carousel
**Then** the play/pause button is reachable by Tab and activatable by Enter/Space
**And** Left/Right arrow keys navigate between slides
**And** the current slide position is announced to screen readers via an `aria-live="polite"` region

---

### Story 5.2: "Why Buy Tires at Costco" Section

As a member evaluating whether to buy at Costco,
I want to see key benefits of purchasing tires at Costco and an option to schedule an appointment,
So that I can understand Costco's value proposition and take immediate action.

**Acceptance Criteria:**

**Given** the "Why Buy Tires at Costco" section
**When** the page loads
**Then** it renders below the Deals Carousel with benefit content
**And** the layout matches `docs/tires-webapp/02-design/mobile/images/Home Page/Why Buy Tires at Costco/Why Buy Tires at Costco.png` (mobile)
**And** the layout matches `docs/tires-webapp/02-design/desktop/images/Home Page/Why Buy Tires at Costco/Why Buy Tires at Costco.png` (desktop)

**Given** the "Schedule an Appointment" CTA button in this section
**When** a member clicks it
**Then** the Schedule an Appointment Modal opens
**And** no external navigation occurs without the modal appearing first

**Given** the section content
**When** it renders as a Server Component
**Then** all benefit text is present in the initial server-rendered HTML (SEO indexable)
**And** no `'use client'` directive is needed unless the CTA button requires client interaction only

---

### Story 5.3: Tire Information Slider (8 Educational Slides + External Links)

As a member who wants to learn about tires,
I want to see an educational content slider with topic slides that link to detailed external resources,
So that I can explore tire education content and access more information on topics relevant to my purchase.

**Acceptance Criteria:**

**Given** the Tire Information Slider section
**When** the page loads
**Then** it renders below the "Why Buy" section with 8 educational slides in sequence:
Slide 1 — How To Buy Tires, Slide 2 — When To Buy Tires, Slide 3 — Tire Care, Slide 4 — FAQ, Slide 5 — Tire Glossary, Slide 6 — Warranty Information, Slide 7 — The Costco Advantage, Slide 8 — Legal Disclaimers
(slide images from `docs/tires-webapp/02-design/desktop/images/Home Page/Tire Information - Slider/Slides/`)

**Given** the slider is rendering
**When** the layout is assessed
**Then** it matches `docs/tires-webapp/02-design/mobile|desktop/images/Home Page/Tire Information - Slider/Tire Information - Slider with Play and Pause buttons.png`
**And** play/pause controls are functional with the same behavior as the Deals Carousel

**Given** a member clicks any slide
**When** the click is registered
**Then** the External Link Modal opens with the slide's associated external URL
**And** confirming opens the external resource in a new tab
**And** dismissing returns the member to the homepage with the slider in its current state

**Given** keyboard accessibility
**When** a keyboard user navigates the slider
**Then** each slide is focusable via Tab
**And** pressing Enter on a focused slide triggers the External Link Modal (same as click)
**And** slide transitions are announced via `aria-live="polite"`
**And** the play/pause control is keyboard-operable (Tab to reach, Enter/Space to activate)

---

### Story 5.4: Blog Section ("Best Place to Buy Tires")

As a member browsing the homepage,
I want to see a full-width editorial article about why Costco is the best place to buy tires,
So that I receive engaging content that reinforces purchase confidence.

**Acceptance Criteria:**

**Given** the Blog Section
**When** the page loads
**Then** it renders below the Tire Information Slider as a full-width section
**And** it displays the "Best Place to Buy Tires" article with a full-width warehouse image
**And** the layout matches `docs/tires-webapp/02-design/mobile/images/Home Page/The Best Place to Buy Tires/The Best Place to Buy Tires.png` (mobile)
**And** the layout matches `docs/tires-webapp/02-design/desktop/images/Home Page/The Best Place to Buy Tires/The Best Place to Buy Tires.png` (desktop)

**Given** the Blog Section is a Server Component
**When** the page is rendered server-side
**Then** all article text and the warehouse image are present in the initial server-rendered HTML
**And** the image is served via Next.js `<Image>` component with lazy loading, WebP format, and correct `alt` text

**Given** any external link within the article body
**When** a member clicks it
**Then** the External Link Modal intercepts the navigation with Cancel and Visit External Link options

