---
stepsCompleted: ["step-01-init", "step-02-discovery", "step-02b-vision", "step-02c-executive-summary", "step-03-success", "step-04-journeys", "step-05-domain", "step-06-innovation", "step-07-project-type", "step-08-scoping", "step-09-functional", "step-10-nonfunctional", "step-11-polish"]
inputDocuments:
  - "docs/tires-webapp/vision-requirements.md"
  - "docs/tires-webapp/readme.md"
  - "docs/vision.txt"
  - "docs/sow.txt"
  - "Goals-Personas-UserJourneys.txt"
  - "docs/BacklogStructure-AI-Input.txt"
  - "_bmad-output/planning-artifacts/product-brief-costco-tires.md"
  - "docs/bff-layer/document-project/project-context.md"
  - "docs/bff-layer/document-project/project-overview.md"
workflowType: "prd"
author: "Rupesh.Jain"
date: "2026-05-12"
project: "Costco Tires Web App"
outputFile: "docs/tires-webapp/prd.md"
classification:
  projectType: "web_app"
  domain: "ecommerce"
  complexity: "medium"
  projectContext: "brownfield"
---

# Product Requirements Document — Costco Tires Web App

**Author:** Rupesh.Jain  
**Date:** 2026-05-12  
**Status:** Draft  
**Scope:** MVP 1 — Next.js Project Setup + Homepage

---

## Executive Summary

Costco's tire shopping experience fragments at a critical point: members select a tire, then get ejected into a disconnected scheduling system — re-authenticating, re-entering their vehicle, losing cart context. This drives abandonment at the highest-intent moment in the funnel.

The Costco Tires Web App modernizes tires.costco.com and tires.costco.ca as a **unified, context-preserving tire-buying experience**. Member context — vehicle selection, fitment, cart, and appointment availability — persists across all touchpoints and channels. No re-entry. No redirects.

**Five delivery phases (SOW-aligned):**
- **Phase 1:** Web journey reimagine — Next.js website, fitment-first navigation, optimized entry points
- **Phase 2:** Real-time warehouse inventory, integrated appointment scheduling, tire education content
- **Phase 3:** Member journey analytics + US Battery Sales
- **Phase 4:** Vehicle profiles + order history for frictionless repurchase

**Differentiating insight:** Costco is the only retailer that can show a member which tires fit their vehicle, confirm stock at their nearest warehouse, and present an open installation slot — all before "add to cart." These assets exist today. This initiative connects them.

**Technical foundation:** Next.js (SSR) front-end consuming a Spring Boot 3.2 BFF, Redux Toolkit for cross-session state, Bing Maps for warehouse selection.

---

## Project Classification

| Dimension | Value |
|---|---|
| **Project Type** | Web App — Next.js SSR/MPA, browser-based, responsive |
| **Domain** | E-commerce — retail tire buying with integrated scheduling |
| **Complexity** | Medium — multi-market, multi-API integration surface |
| **Context** | Greenfield — Next.js tires.costco.com/.ca; Spring Boot BFF |
| **Markets** | en-US, en-CA, fr-CA |

---

## Success Criteria

### User Success

- A member can initiate a tire search from the homepage using any of the 4 search methods (License Plate, Vehicle Year/Make/Model/Trim, Tire Size, Item/Part No.) and be routed to the correct downstream page without error
- All 4 search types handle their defined error and edge-case scenarios (partial match, no match, invalid part number, trim required) with custom inline error messaging — no browser-native HTML5 error popups
- A member without a pre-selected Tire Center sees the ZIP Code field and is prompted through the Tire Center Locator modal before search proceeds
- Search inputs are preserved across session navigation (Redux + localStorage persistence)
- Homepage renders correctly across mobile and desktop breakpoints per design references

### Business Success

- Homepage is deployed to a working environment and demo-ready for stakeholder review
- All 5 homepage sections render: Tire Search Form, Deals Carousel, Why Buy at Costco, Tire Information Slider, Blog Section
- Deals Carousel and Tire Information Slider function with play/pause controls and correct slide content

### Technical Success

- Next.js project scaffolded with Webpack (not Turbopack), SCSS + Bootstrap 5, Redux Toolkit, ESLint + Prettier
- Tire Search Form implemented as a reusable component with `isModalContent` prop for modal/page rendering modes
- Tire Center Locator implemented as a reusable component with `isModalContent` prop for modal/page rendering modes
- BFF API client baseline established (vehicle catalog lookups for License Plate and Vehicle search)
- Unit tests written for Tire Search Form component and Redux slices
- Page meets ≤2s load time target on desktop and mobile

### Measurable Outcomes

- 100% of homepage sections rendered and visually verified against design references (mobile web + desktop web)
- 0 browser-native HTML5 validation popups — all errors use custom inline style
- Search state for License Plate round-trips correctly: enter license plate number → navigate away → return → state restored
- Search state for Vehicle round-trips correctly: select vehicle → navigate away → return → state restored
- Search state for Tire Size round-trips correctly: select tire size → navigate away → return → state restored
- Search state for Item Number or Part Number round-trips correctly: enter item number or part number → navigate away → return → state restored
- Lighthouse performance score ≥ 90 on desktop, ≥ 75 on mobile for homepage

---

## MVP 1 Feature Scope

### Project Setup

- Next.js app scaffolded (Webpack — not Turbopack, SCSS, Bootstrap 5, Redux Toolkit)
- Folder structure: `components/`, `pages/`, `redux/`, `styles/`, `utils/`, `services/`
- ESLint + Prettier configured; CI pipeline baseline
- BFF API integration layer established

### Homepage — 5 Sections

| Section | Key Behaviour |
|---|---|
| **Hero: Tire Search Form** | 4 search tabs: License Plate (US only), Vehicle, Tire Size, Item/Part No. All scenarios, error states, ZIP/Tire Center conditional, Redux persistence, `isModalContent` reusability |
| **Deals Carousel** | 3 brand slides (Michelin, Firestone, Bridgestone), play/pause controls |
| **Why Buy Tires at Costco** | Static benefits content + Schedule Appointment CTA (link only; scheduling out of scope for MVP 1) |
| **Tire Information Slider** | 8 educational slides, each linking to an external URL on click |
| **Blog Section** | Full-width "Best Place to Buy Tires" editorial article |

- License Plate search: en-US only; tab suppressed for en-CA and fr-CA

### Growth Features (Phase 2)

- Change Vehicle Modal (reuses Tire Search Form via `isModalContent` prop)
- PLP, Size Selection Page, Brand landing pages, Brands Ribbon
- Returning member vehicle pre-population from saved profile

### Expansion (Phase 3+)

- PDP, Cart, Checkout, Appointment Scheduling, real-time inventory
- Member login, Vehicle Profile, Order History
- US Battery Sales microsite; CMS-driven promotional content

---

## User Personas

Three personas are the primary drivers of the MVP 1 homepage. They represent the members who land on `tires.costco.com/.ca`, encounter the Tire Search Form, and whose success depends on the homepage working correctly at launch.

### Persona 1 — Task-Driven Buyer

**Archetype:** High-intent professional shopper who arrives with a clear goal  
**Represented by:** Marcus (Journey 1)

**Description:** Shopper with an explicit mission — buy the correct tires fast, without distractions. Arrives with high purchase probability and zero tolerance for friction. Measures success by speed and certainty.

**Behavioral Profile:**
- Wants the shortest path from homepage → fitment → results → checkout
- No interest in browsing, promotions, or educational content
- Extremely time-sensitive; a single unnecessary step causes abandonment
- Trusts Vehicle Year/Make/Model/Trim search over license plate if plate lookup adds uncertainty

**Homepage Priorities:**
1. Tire Search Form visible immediately on load — above the fold
2. Vehicle tab pre-selected or clearly default; no hunting for entry point
3. Tire Center pre-check works silently — does not interrupt a returning member
4. "Find Tires" CTA routes directly to Size Selection Page with no intermediate steps

**Primary KPI:** Add-to-Cart Conversion Rate (indexed to homepage search initiation rate)  
**Phase Activation:** Phase 1A

---

### Persona 2 — Searcher / Problem Solver

**Archetype:** Intent-driven member who enters through License Plate or a direct size — trusts lookup over navigation  
**Represented by:** Priya (Journey 2)

**Description:** Member who knows something (their plate, size, or a brand name) and uses that as their entry point. May be a new member, a first-time tire buyer, or someone in a new location. Requires the system to guide them from what they know to what they need.

**Behavioral Profile:**
- Enters via License Plate search (fastest fitment confirmation) or Tire Size direct entry
- New to the specific Costco market — no Tire Center pre-selected; ZIP Code flow is their reality
- Comfortable with digital lookup flows; abandons quickly if the system returns unhelpful errors
- Expects the form to do the hard work (resolve plate → vehicle → compatible sizes automatically)

**Homepage Priorities:**
1. License Plate tab visible and clearly labelled (en-US only)
2. State selector paired with plate field — required for VIN resolution
3. Conditional ZIP Code field renders when no Tire Center is stored (not before)
4. Tire Center Locator opens on search submit — first-time selection renders as a standalone page (`/TireCenterLocator`); subsequent changes open as a modal using the same component via `isModalContent` prop
5. Graceful inline error if license plate service is unavailable; other search tabs remain usable

**Primary KPI:** Search Success Rate (% of license plate queries that resolve to a vehicle and proceed to Size Selection)  
**Phase Activation:** Phase 1A

---

### Persona 4 — Deal Hunter / Promo-Driven Buyer

**Archetype:** Price-sensitive member motivated by promotions and seasonal offers  
**Represented by:** Derek (Journey 3)

**Description:** Member who arrives from a promotional mailer, email campaign, or seasonal offer. Their first stop on the homepage is the Deals Carousel — they evaluate the offer before engaging with fitment. If the deal is visible and compelling, they proceed. If the deal is hidden, broken, or leads to a dead-end, they leave.

**Behavioral Profile:**
- Scans the Deals Carousel before any other homepage element
- Uses Tire Size direct entry (knows their size; skips vehicle selection)
- Uses same/staggered front-rear toggle — staggered fitment is common in their vehicle segment
- May not have a pre-selected Tire Center — ZIP Code flow activates

**Homepage Priorities:**
1. Deals Carousel renders immediately with promotional brand slides (Michelin, Firestone, Bridgestone)
2. Play/pause controls functional — members pause to read rebate details
3. Carousel slides link correctly to external promo pages (open in new tab, no dead-ends)
4. Tire Size tab supports same front/rear toggle and stagger input for different sizes
5. ZIP Code field renders correctly when no Tire Center stored

**Primary KPI:** Fitment Start Rate (% of members who open a promotional slide and then initiate a tire search)  
**Phase Activation:** Phase 1A

---

### Persona-to-Homepage Mapping (MVP 1)

| Persona | Enters via Homepage | Primary Homepage Action | Critical Homepage Dependency | Success Signal |
|---|---|---|---|---|
| **1. Task-Driven (Marcus)** | Yes — direct URL or search | Vehicle tab → Year/Make/Model/Trim → Find Tires | Tire Center pre-check in Redux; route to Size Selection | Search initiates in <2 min with no re-entry |
| **2. Searcher (Priya)** | Yes — new to market, no Tire Center | License Plate + State → ZIP Code → Tire Center Locator (standalone first-use) | Conditional ZIP field; Tire Center Locator renders as standalone page on first selection; Bing Maps list + results | License Plate resolves to vehicle; Tire Center selected; ZIP flow proceeds |
| **4. Deal Hunter (Derek)** | Yes — promo mailer or email link | Deals Carousel pause → external promo link → Tire Size search | Carousel play/pause + external links; same/staggered tire size toggle | Carousel CTR; Tire Size search initiates correctly |

> **Personas 3, 5, 6, 7, 8** (Explorer, Brand Loyalist, Services Seeker, Repeat Buyer, Battery Buyer) are **out of scope for MVP 1**. Their homepage entry hooks (Brands Ribbon, category navigation, account shortcuts) activate in Growth and subsequent phases.

---

## User Journeys

### Journey 1 — The Task-Driven Buyer (Primary Success Path)

**Persona: Marcus, 38, Seattle.**
His wife noticed the front tires are worn down — needs handling before their road trip in two weeks. On his lunch break, he pulls up the site on his laptop. He lands on the homepage, sees the search form immediately, selects the **Vehicle** tab, picks 2021 Toyota RAV4 XLE. No ZIP Code field — a Tire Center is already stored from a previous visit. He hits **Find Tires** and is routed directly to the Size Selection Page. Eight minutes of his lunch break used. Problem solved.

**Key moments:** Clean Vehicle search → Tire Center pre-check → no friction → Size Selection.

---

### Journey 2 — The First-Timer with No Tire Center (Edge Case / ZIP Flow)

**Persona: Priya, 29, Austin, TX.**
Just moved to Austin. First time on Costco Tires. Enters her license plate and state (TX). System finds her 2022 Honda Civic. No Tire Center in her session — the **ZIP Code field** appears. She enters her ZIP. On submit, the **Tire Center Locator modal** opens. She selects the nearest warehouse. Search proceeds — Size Selection page loads with vehicle and location context preserved.

**Key moments:** License Plate search → ZIP conditional rendering → Tire Center Locator modal handoff → Size Selection.

---

### Journey 3 — The Deal Hunter (Promotional Entry)

**Persona: Derek, 44, Phoenix.**
Got a mailer about Michelin rebates. Lands on homepage. **Deals Carousel** is front and center — Michelin slide is first. He pauses the slider, reads the rebate details, clicks through to the external promo page. Returns to the homepage. Uses the **Tire Size** tab — knows his size (225/55R17). Toggles same front/rear. No Tire Center stored — ZIP field appears, Tire Center Locator flow would fire.

**Key moments:** Deals Carousel engagement (play/pause, external link) → Tire Size search with stagger toggle → ZIP conditional flow.

---

### Journey 4 — The Confused Shopper / Error Recovery (Canadian Market)

**Persona: Linda, 61, Ontario, Canada.**
Navigates to the Canadian site. Looks for the License Plate tab — it is not there (suppressed for en-CA/fr-CA). Uses **Vehicle** tab instead. Selects 2019 Subaru Forester but forgets to select Trim. Hits search. The Trim dropdown shows a **red border** with an inline error message: "Please select a trim" — no browser popup. She selects "2.5i Premium" and continues successfully.

**Key moments:** Market-conditional tab suppression → custom inline validation → Trim required error → successful recovery.

---

### Journey Requirements Summary

| Journey | Persona | Key Capabilities Required |
|---|---|---|
| Task-Driven Buyer | Marcus | Vehicle search, Tire Center pre-check (Redux), suppress ZIP, route to Size Selection |
| First-Timer / No Location | Priya | License Plate search, conditional ZIP field, Tire Center Locator (standalone page on first use; modal for subsequent changes), session context preservation |
| Deal Hunter | Derek | Deals Carousel (play/pause, external links), Tire Size search (same/staggered toggle), route to PLP |
| Error Recovery / CA Market | Linda | Market-conditional tab rendering (no License Plate on CA), custom inline form validation for all field types |

**Cross-cutting requirements revealed by all journeys:**
- Search state (vehicle, Tire Center, size) persisted in Redux + localStorage for session continuity
- Tire Search Form must be a reusable component with `isModalContent` prop (Change Vehicle Modal reuse in later phases)
- All form validation must use custom error styles (red border + inline message) — zero browser-native HTML5 error popups
- License Plate search tab conditionally rendered based on market (en-US only)

---

## Domain-Specific Requirements

### Compliance & Regulatory

- **Multi-market localization:** en-CA and fr-CA markets are required. French language support for Canada is a legal requirement under Quebec's Bill 96 — all user-facing content, error messages, and UI labels must be translated for the fr-CA locale. English-only delivery is not acceptable for Quebec users.
- **Accessibility:** WCAG 2.1 AA conformance required across all pages. Specific requirements: ARIA attributes on interactive elements, semantic HTML structure, full keyboard navigation support, and sufficient color contrast ratios. All modals, carousels, and interactive elements must be accessible.
- **Member data / PII handling:** Member profile data (vehicle history, order history) is PII. Input validation and sanitization required on all form submissions. HTTPS enforced across all environments. Secure HTTP headers (HSTS, CSP, X-Frame-Options) required.

### Technical Constraints

- **BFF API dependency:** Homepage search functionality (License Plate lookup, Vehicle catalog) depends on Spring Boot 3.2 BFF APIs being available. APIs not yet migrated from the .NET 4.8 monolith require mock/stub responses during development. BFF API contract stability is a delivery risk for MVP 1.
- **Rate limiting:** License Plate lookup and vehicle catalog endpoints are external-facing and must have rate limiting applied at the BFF layer to prevent abuse and protect third-party API quotas.
- **Input validation & sanitization:** All Tire Search Form inputs (text fields, dropdowns, ZIP code) must be validated and sanitized before submission — server-side validation at BFF, client-side inline validation in the Next.js form component.
- **Redux state security:** No sensitive member data (auth tokens, full PII) stored in localStorage. Only fitment context (vehicle selection, Tire Center ID, search criteria) persisted client-side.

### Integration Requirements

- **Vehicle catalog API (BFF):** Required for License Plate search (vehicle lookup by plate + state) and Vehicle search (Year/Make/Model/Trim dropdowns). Must handle partial matches, no-match, and trim-required response states.
- **Tire Center data API (BFF):** Required to read/write the selected Tire Center to Redux state. Homepage must read pre-existing Tire Center selection on load.
- **Bing Maps API:** Required for the Tire Center Locator — renders nearby tire centers as interactive markers on a Bing Map alongside the search results list. Clicking a map marker highlights the corresponding tire center in the list. API key managed via environment config; must not be hardcoded or committed to source control.
- **Third-party vehicle lookup service:** Powers License Plate search. Must degrade gracefully if unavailable — show inline "service unavailable" message, do not crash the form or block other search tabs.

### Risk Mitigations

| Risk | Impact | Mitigation |
|---|---|---|
| BFF APIs not ready at MVP 1 start | Blocks Vehicle/License Plate search development | Use MSW (Mock Service Worker) or stub API layer for development; agree BFF contract early |
| License Plate third-party service outage | License Plate tab non-functional | Graceful degradation: show inline error, tab remains visible but displays service message |
| French content not available at launch | fr-CA market non-compliant | fr-CA locale strings must be delivered alongside en-CA; no partial-French launch acceptable |
| Bing Maps API key not provisioned before sprint start | Tire Center Locator map renders blank; results list still functional | Confirm API key provisioning in sprint 0; list-only fallback if map fails to load |

---

## Web App Specific Requirements

### Project-Type Overview

The Costco Tires Web App is a **server-side rendered (SSR) Next.js application** with client-side navigation, delivering a hybrid multi-page application (MPA) architecture. Pages are rendered server-side for SEO and initial performance, with client-side React hydration for interactive elements (search form, carousels, Redux state management). The app is browser-based, responsive across mobile and desktop, and consumes the Spring Boot 3.2 BFF REST API layer.

### Browser Matrix

| Browser | Minimum Version | Priority |
|---|---|---|
| Chrome | Latest - 1 | P1 |
| Safari (iOS + macOS) | Latest - 1 | P1 |
| Edge | Latest - 1 | P1 |
| Firefox | Latest - 1 | P2 |
| Samsung Internet | Latest - 1 | P2 |
| IE11 | Not supported | — |

Mobile browser parity with desktop is required. iOS Safari receives P1 priority given Costco mobile app target audience.

### Responsive Design

- **Breakpoints:** Mobile-first design; breakpoints per Bootstrap 5 grid (xs: <576px, sm: 576px, md: 768px, lg: 992px, xl: 1200px, xxl: 1400px)
- **Design references:** Mobile and desktop comps provided in `docs/tires-webapp/02-design/` for all homepage sections
- All 5 homepage sections must render correctly at both mobile and desktop reference breakpoints
- Touch interactions (carousel swipe, tap targets ≥44px) required for mobile
- No horizontal scroll on any viewport width

### Performance Targets

| Metric | Target | Measurement |
|---|---|---|
| Page load time | ≤ 2 seconds | Largest Contentful Paint (LCP) |
| Lighthouse Performance | ≥ 90 desktop / ≥ 75 mobile | Chrome Lighthouse audit |
| Time to Interactive | ≤ 3.5 seconds | Lighthouse TTI |
| Cumulative Layout Shift | ≤ 0.1 | Core Web Vitals |
| Bundle size (initial JS) | ≤ 200KB gzipped | Webpack bundle analyzer |

Images must use Next.js `<Image>` component for automatic optimization, lazy loading, and WebP conversion. Routes and components lazy-loaded via Next.js dynamic imports.

### SEO Strategy

- All homepage content rendered server-side (SSR) for full crawler visibility
- `<title>` and `<meta description>` tags set per page via Next.js `<Head>`
- Canonical URLs configured to prevent duplicate content across en-US / en-CA / fr-CA locales
- `hreflang` tags required for multi-market locale alternates
- No client-side-only rendering of primary content on indexable pages
- Structured data (JSON-LD) not required for MVP 1 homepage; flagged for PLP/PDP phases

### Accessibility Level

- **Target conformance:** WCAG 2.1 Level AA
- All form inputs (Tire Search Form, search tabs, dropdowns) must have associated `<label>` elements or `aria-label` attributes
- Error messages linked to inputs via `aria-describedby`
- Deals Carousel and Tire Information Slider must have keyboard controls (arrow keys, play/pause via keyboard) and `aria-live` regions for screen reader announcements
- Color contrast ratio ≥ 4.5:1 for normal text, ≥ 3:1 for large text
- Focus indicators visible on all interactive elements
- Modals (including Tire Center Locator modal) must trap focus when open and return focus on close

### Implementation Considerations

- **Webpack** (not Turbopack) — explicitly required; `next.config.js` customized if needed
- **SCSS + Bootstrap 5** — modular SCSS, Bootstrap 5 utility classes; no inline styles
- **Redux Toolkit** — centralized store; slices for: vehicle selection, Tire Center selection, search criteria, UI state
- **localStorage persistence** — search criteria and Tire Center selection persisted; no auth tokens or PII in localStorage
- **API client layer** — centralized in `services/` folder; error boundary handling; BFF base URL from environment config (not hardcoded)
- **Environment configuration** — API base URLs and feature flags (e.g. License Plate tab enabled for en-US only) managed via `.env` files; never committed to source control

---

## Project Scoping & Risk

### MVP Strategy

**Approach:** Functional foundation MVP — a deployable Next.js app with a production-quality homepage that validates the architecture, establishes the component system, and enables all 4 tire search entry points. The homepage is the highest-leverage page: it serves all 7 personas, contains the most complex component (Tire Search Form, 4 modes), and establishes reusable patterns every subsequent page depends on.

### Must-Have Capabilities for MVP 1

| Capability | Justification |
|---|---|
| Next.js scaffold (Webpack, Redux, SCSS/Bootstrap 5) | Foundation for all subsequent pages |
| Tire Search Form — all 4 types with all error/edge cases | Primary conversion entry point |
| Market-conditional rendering (License Plate US-only) | en-CA/fr-CA legal and UX requirement |
| ZIP Code conditional + Tire Center Locator (standalone page + `isModalContent` modal, Bing Maps) | Required for all users without a pre-selected Tire Center |
| Redux state + localStorage persistence | Cross-session context preservation |
| Deals Carousel, Why Buy section, Info Slider, Blog | Homepage completeness; all 5 sections |
| BFF API client baseline | Vehicle catalog + Tire Center lookups |
| Unit tests: Tire Search Form + Redux slices | Code quality gate |

### Risk Mitigation

| Risk | Mitigation |
|---|---|
| BFF APIs not ready at sprint 1 | MSW (Mock Service Worker) stubs; BFF contract signed off before development starts |
| License Plate third-party service outage | Graceful degradation — inline error shown; other search tabs unaffected |
| fr-CA French content not ready | fr-CA strings required simultaneously with en-CA; no partial-French launch |
| Bing Maps API key not provisioned before sprint start | Confirm key in sprint 0; Tire Center list renders without map as fallback |
| BFF migration pace | Next.js decoupled from monolith via BFF; BFF stability is the only hard dependency |

---

## Functional Requirements

### Search & Vehicle Fitment

- **FR1:** A member can search for compatible tires by selecting their vehicle's Year, Make, Model, and Trim  
  **AC:** Year/Make/Model/Trim dropdowns populate sequentially from the BFF vehicle catalog API; Trim is required before submission (not selectable until Make and Model are chosen); confirmed vehicle context (make, model, year, compatible sizes) persists in Redux for the full session with no re-entry required on return navigation

- **FR2:** A member can search for compatible tires by entering their License Plate number and State (US markets only)  
  **AC:** License plate + state resolves to a unique vehicle record within 3 seconds (p95) via the BFF Mitchell1 VIN decode integration; partial match (multiple vehicles found), no vehicle found, and service unavailable states each return a distinct inline error message; a service outage on the license plate lookup does not crash the form or disable other search tabs

- **FR3:** A member can search for tires by selecting Tire Size (Width, Aspect Ratio, Diameter) with options for same or different front and rear sizes  
  **AC:** Tire size input accepts standard formats (###/##R##, P###/##R##, e.g., 225/45R18, P225/45R18); an invalid format triggers an inline error with a corrective example ("Enter size as: Width/Aspect Ratio R Diameter, e.g., 225/45R18"); staggered size mode accepts two independent size inputs, each validated independently before submission

- **FR4:** A member can search for a specific tire by entering an Item Number or Part Number  
  **AC:** A valid SKU routes directly to the PDP on the same page load; an unrecognised or malformed part number displays an inline error ("We couldn't find that item number — try searching by vehicle or tire size") without a full page reload or browser error

- **FR5:** The system routes members to the correct downstream page based on search type (Vehicle/License Plate → Size Selection; Tire Size → PLP; Item No. → PDP)  
  **AC:** Each search type routes to its designated page with no intermediate redirect page; vehicle/license plate searches land on Size Selection Page with vehicle context pre-populated; tire size searches land on PLP pre-filtered by the entered size; item number searches land on the matching PDP; routing occurs on submission with no additional member action required

- **FR6:** The system displays inline field-level validation errors using custom error styling (not browser-native HTML5 validation) for all search form inputs  
  **AC:** Zero browser-native HTML5 validation popups appear on any search form input across all browsers in the browser matrix; all validation errors render as a red border on the invalid field plus an inline message adjacent to that field within 200ms of submission attempt; error styling clears immediately when the member corrects the field

- **FR7:** The system handles and communicates specific error states: partial license plate match (multiple vehicles found), vehicle found but trim required, no vehicle found, invalid part number  
  **AC:** All four defined error states render their specific inline messages without a full page reload; no error state results in a blank page, unhandled JavaScript exception, or browser-native error dialog; the member can correct their input and resubmit without refreshing the page

- **FR8:** The system persists member search criteria (vehicle selection, search type, inputs) in session state and restores them on return navigation  
  **AC:** Search criteria (selected vehicle, active search tab, form inputs) restore in Redux within 500ms of the member returning to the homepage within the same session; guest session context persists for 24 hours via localStorage; no sensitive member data (auth tokens, PII beyond vehicle fitment context) is written to localStorage

### Tire Center & Location

- **FR9:** The system detects whether a member has a pre-selected Tire Center and conditionally shows a ZIP Code field when no Tire Center is stored  
  **AC:** The ZIP Code field is absent from the DOM (not hidden) when a Tire Center is pre-selected in Redux state; it renders as a required field when no Tire Center is stored; ZIP validation rejects non-5-digit US ZIP inputs with an inline error before submission; the field re-evaluates its visibility if Tire Center state changes mid-session

- **FR10:** The Tire Center Locator renders as a standalone page (`/TireCenterLocator`) on a member's first tire center selection, and as a modal (using the `isModalContent` prop) for all subsequent changes or when accessed via the Schedule an Appointment flow  
  **AC:** The component is a single reusable implementation controlled by the `isModalContent` configuration prop; the standalone page renders at route `/TireCenterLocator` and is not directly accessible without navigation context; the modal variant applies minor design adjustments without duplicating component logic; on Tire Center selection in either rendering mode, the selected warehouse ID is written to Redux state, persisted to localStorage, and the header link updates to show the chosen tire center name; a member cannot be routed to Size Selection or PLP from a search submission until a Tire Center is selected

- **FR11:** The Tire Center Locator allows members to search for nearby tire centers by city, state, or ZIP code, presenting results as a list alongside a Bing Maps view with interactive markers  
  **AC (Scenario 1 — Tire Centers Found):** Matching tire centers display as a list with details and a "Set as My Tire Center" button; each result is simultaneously plotted as a marker on the Bing Map; clicking a map marker highlights the corresponding list item and shows its details  
  **AC (Scenario 2 — No Tire Centers Found):** The results list is replaced with the message "We were unable to find a Costco warehouse near your location. Try searching again."; any markers from a prior search are cleared from the map  
  **AC (Selection):** On tire center selection, the member's chosen warehouse is written to Redux, persisted to localStorage, and the header "Tire Center" link updates to show the selected tire center name; the member is returned to their previous flow (search routing proceeds)

### Homepage Content & Merchandising

- **FR12:** The homepage displays a Deals Carousel with promotional brand slides, play/pause controls, and navigation between slides
- **FR13:** The homepage displays a "Why Buy Tires at Costco" section with key benefit content and a Schedule Appointment call-to-action
- **FR14:** The homepage displays a Tire Information Slider with educational content slides that link to external resources on click
- **FR15:** The homepage displays a Blog Section featuring a full-width editorial article
- **FR16:** Deals Carousel and Tire Information Slider slides link to external URLs when clicked

### Market & Localization

- **FR17:** The system suppresses the License Plate search tab for Canadian market users (en-CA and fr-CA locales)
- **FR18:** The system renders all user-facing content, labels, and error messages in French for fr-CA locale users
- **FR19:** The system serves locale-appropriate content and UI variants based on the member's market (en-US, en-CA, fr-CA)

### Component Reusability

- **FR20:** The Tire Search Form renders correctly as a standalone page component and as modal content, controlled by a configuration prop
- **FR21:** The system reuses the same Tire Search Form component instance when a member initiates a vehicle change from subsequent pages (Change Vehicle Modal)

### State & Session

- **FR22:** The system persists member fitment context (vehicle selection, Tire Center selection, search criteria) across browser sessions using local storage
- **FR23:** The system does not store sensitive member data (authentication tokens, PII) in client-side storage

### API & Integration

- **FR24:** The system retrieves vehicle data (Year/Make/Model/Trim options) from the BFF vehicle catalog API to populate search form dropdowns
- **FR25:** The system resolves a license plate and state to a vehicle record via the BFF, handling partial match and no-match responses
- **FR26:** The system degrades gracefully when the license plate lookup service is unavailable, displaying an inline error without disrupting other search tabs
- **FR27:** The system retrieves and displays the member's pre-selected Tire Center from the BFF on page load

### Accessibility

- **FR28:** All interactive form elements (inputs, dropdowns, tabs, buttons) are operable by keyboard without requiring a mouse
- **FR29:** All form inputs have programmatically associated labels or ARIA labels, and error messages are linked to their respective inputs
- **FR30:** Carousel and slider components announce slide changes and provide keyboard controls for screen reader users

---

## Non-Functional Requirements

### Performance

| Metric | Requirement | Measurement Method |
|---|---|---|
| Page load time | Largest Contentful Paint (LCP) ≤ 2 seconds | Chrome Lighthouse / Core Web Vitals |
| Time to Interactive | TTI ≤ 3.5 seconds on desktop; ≤ 5 seconds on mobile (3G) | Lighthouse audit |
| Cumulative Layout Shift | CLS ≤ 0.1 | Core Web Vitals |
| Initial JS bundle | ≤ 200KB gzipped | Webpack bundle analyzer |
| Lighthouse score | ≥ 90 desktop / ≥ 75 mobile | Chrome Lighthouse |
| API response time | BFF responses ≤ 500ms at p95 under normal load | API monitoring |

All images served via Next.js `<Image>` with lazy loading and WebP format. Routes and heavy components code-split via Next.js dynamic imports.

### Security

- All client-server communication over HTTPS; HTTP requests redirected to HTTPS
- Secure HTTP headers required: `Strict-Transport-Security`, `Content-Security-Policy`, `X-Frame-Options: DENY`, `X-Content-Type-Options: nosniff`
- All form inputs validated client-side (inline) and sanitized server-side at the BFF layer before processing
- No authentication tokens, session tokens, or full PII persisted in `localStorage` or `sessionStorage` — only fitment context (vehicle selection, Tire Center ID, search criteria)
- Rate limiting applied at the BFF layer on License Plate lookup and vehicle catalog endpoints
- Environment variables (API keys, BFF base URLs) never committed to source control; managed via `.env` files excluded by `.gitignore`

### Scalability

- Application must handle real-time traffic spikes during promotional campaigns and peak retail periods (e.g. Black Friday, seasonal tire changeover) without degradation
- Next.js SSR pages must support horizontal scaling; no server-side session state that prevents scaling out
- Static and semi-static homepage content (carousels, blog section) eligible for SSG or ISR (Incremental Static Regeneration) to reduce BFF load under peak traffic
- CDN caching required for static assets (images, fonts, CSS, JS bundles)

### Reliability

| Metric | Requirement |
|---|---|
| Application uptime | 99.99% (≤ 52 minutes downtime/year) |
| BFF API uptime | 99.9% (≤ 8.7 hours downtime/year) |
| Graceful degradation | License Plate lookup service failure must not prevent other search tabs from functioning |
| Error handling | Unhandled API errors display user-friendly inline messages; no raw error codes or stack traces exposed to members |

### Accessibility

- **Conformance target:** WCAG 2.1 Level AA across all pages
- All interactive elements keyboard-operable (Tab, Enter, Space, Arrow keys as appropriate)
- All form inputs have associated `<label>` elements or `aria-label`; errors linked via `aria-describedby`
- Carousel and slider components include `aria-live` announcements and keyboard play/pause/navigation controls
- Minimum color contrast ratio: 4.5:1 (normal text), 3:1 (large text / UI components)
- Visible focus indicators on all interactive elements
- Semantic HTML structure (`<main>`, `<nav>`, `<header>`, `<section>`, landmark roles) throughout
- Modals trap focus when open; return focus to trigger element on close
- Automated accessibility scanning (axe-core or equivalent) integrated into CI pipeline

### Localization

- Three locales fully supported: `en-US`, `en-CA`, `fr-CA`
- All UI strings, labels, error messages, and content must be externalized into locale resource files — no hardcoded English strings in component code
- `fr-CA` locale content must be delivered simultaneously with `en-CA` — no phased or partial French launch
- `hreflang` tags configured for all locale alternates to prevent duplicate content indexing
- Locale determined by domain/URL path; must not rely solely on browser language detection

### Integration

- **BFF API:** All BFF endpoints consumed via a centralized API client in `services/`; base URL configured via environment variable; request/response error handling standardized across all calls
- **License Plate lookup service:** Third-party dependency; must degrade gracefully on timeout or error — inline message shown, other search tabs unaffected
- **Bing Maps API:** Required for the Tire Center Locator — renders nearby tire centers as interactive markers on a Bing Map alongside list results; API key managed via environment config
- **Mock/stub layer:** MSW (Mock Service Worker) used in development to simulate BFF responses when live APIs are unavailable; stubs must match agreed BFF contract schemas
