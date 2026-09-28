---
stepsCompleted: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14]
status: complete
completedAt: 2026-05-01
inputDocuments:
  - _bmad-output/planning-artifacts/prd.md
  - _bmad-output/planning-artifacts/architecture.md
  - docs/bff-layer/document-project/project-context.md
  - docs/images/web-app/ (Figma screenshots - all sections)
  - docs/images/mobile-app/ (Figma screenshots - all sections)
---

# UX Design Specification — ct-planning-artifacts

**Author:** Rupesh.Jain
**Date:** 2026-05-01

---

<!-- UX design content will be appended sequentially through collaborative workflow steps -->

## Executive Summary

### Project Vision

The Costco Tires Digital Experience Modernization delivers a unified, context-preserving omnichannel tire-buying and service-scheduling platform across costco.com/.ca (web) and the Costco native mobile app. The UX goal is a seamless member journey where vehicle selection, fitment validation, real-time warehouse inventory, cart, and appointment booking work identically across web and mobile — eliminating re-entry friction and driving measurable conversion improvement.

The experience replaces a fragmented multi-tool, multi-redirect journey with a single intelligent flow that remembers the member, their vehicle, and their context from first touch to appointment confirmation.

### Target Users

Seven distinct personas have been defined, each with a unique entry point and behavioral profile:

| Persona | Entry Point | Core UX Need |
|---|---|---|
| Task/Mission-Driven Buyer | Tire Search | Shortest path to fitment → results → checkout (<5 taps mobile) |
| Searcher / Problem Solver | Global Search | High-accuracy intent detection, autocomplete, fit confirmation in results |
| Explorer / Category Browser | Shop Navigation | Rich filters, educational content, comparison tools, guided sizing |
| Deal Hunter / Promo-Driven Buyer | Deals / Promotions | Auto-route promo into fitment; clear savings; no dead-end promo pages |
| Brand Loyalist | Brand Gallery | Brand entry → immediate vehicle selector → fitment-only brand results |
| Maintenance Seeker / Car-Care Planner | Services / Scheduling | Unified single scheduling flow; warehouse slot availability during product selection |
| Repeat Buyer / Owner | Account / Reorder | Vehicle persistence; 1-2 tap reorder; purchase history recall |

### Key Design Challenges

1. **Context Continuity Across Touchpoints** — Vehicle selection made once must persist across all screens, sessions, and channels. Every screen must reflect the member's vehicle state without requiring re-entry.

2. **Seven Entry Points, One Unified Journey** — Homepage, Search, Brand Gallery, Deals, Brand Landing, Services, and Account all converge into a shared purchase funnel. Navigation and fitment routing must be intelligent and error-free at each entry.

3. **Fitment Validation at the Right Moment per Persona** — Task-Driven buyers need fitment first; Explorers need it after browsing; Repeat Buyers need it pre-filled. The fitment gate must be persona-adaptive, not a single fixed step.

4. **Real-Time Inventory Confidence** — "In stock at your warehouse today" must be visible on search results and PLP, not just at checkout, so members commit to purchase with full confidence.

5. **No-Redirect Scheduling Integration** — Appointment booking must live as a native in-flow component (time slot picker, warehouse confirmation) at the cart/checkout layer — never a separate tool redirect.

6. **Responsive + Native Parity** — Web (375px, 768px, 1024px, 1280px) and Native Mobile App must each feel platform-native while maintaining complete functional and contextual equivalence.

### Design Opportunities

1. **Persistent "Selected Vehicle" Chip** — A glanceable vehicle identity element (make/model, tire size) displayed across the entire journey header creates trust, reduces cognitive load, and eliminates the need to re-enter fitment context.

2. **Appointment Slot Visibility on PLP / PDP** — Surfacing "Available today at Warehouse X" directly on product tiles and product pages mirrors the decision confidence that booking platforms provide — a direct conversion driver.

3. **Saved Vehicle Experience as Loyalty Anchor** — "My Vehicles" with purchase history recall and 1-tap reorder is a differentiated CX moment unavailable from online-only competitors (TireRack, SimpleTire). It should be a prominently featured capability in the account layer.

4. **Design System Formalization** — Figma assets already demonstrate a mature multi-breakpoint design language across all key flows. Formalizing this into a component library and pattern spec accelerates engineering confidence and design-dev handoff quality.

---

## Core User Experience

### Defining Experience

The core loop driving every persona is: **Fitment Selection → Fitment-Validated Results with Live Warehouse Stock → One-Tap Add to Cart + Appointment Booking**.

All seven personas converge on this loop at different entry velocities. The Task-Driven Buyer arrives at the start of it; the Repeat Buyer skips directly to the end. The UX must make this loop fast, trustworthy, and frictionless on both web and mobile.

### Platform Strategy

- **Web (costco.com/.ca):** Responsive at 375px, 768px, 1024px, 1280px. SSR mandatory for performance (LCP <2.5s). Mouse/keyboard desktop; touch mobile web.
- **Native Mobile (iOS 16+ / Android 10+):** Touch-primary. Bottom sheets, native gestures, haptic feedback, camera LP scan. Separate native build sharing BFF API.
- **No offline mode** — real-time inventory confidence is a core UX contract.
- **Platform leverage:** LP scan via camera; haptic confirmation on add-to-cart; push notifications for appointment reminders.

### Effortless Interactions

1. **Enter Vehicle Once, Persist Always** — LP scan or Year/Make/Model selection persists across the full session, channel switches, and return visits.
2. **Automatic Fitment Signals** — "Fits your vehicle ✓" appears on every product tile without user action when vehicle context is active.
3. **Stock Confidence on PLP** — "In stock at [Warehouse] · Next available: Today 2pm" surfaced on product tiles — not revealed for the first time at checkout.
4. **Return Member Recognition** — Returning members land with pre-loaded vehicle, surfaced last purchase, and one-tap reorder path.
5. **In-Cart Appointment Booking** — Time slot selection lives inside the cart experience; tire + installation confirmed in a single checkout session.

### Critical Success Moments

| Moment | UX Impact |
|---|---|
| First Fitment Confirmation | Trust established: only compatible tires shown |
| PLP with Live Stock Signal | Purchase confidence before PDP click |
| One-Checkout Cart + Appointment | Eliminates conversion-killing redirect |
| Return Member Recognition | Creates "this knows me" loyalty moment |
| Order + Appointment Confirmation | Eliminates post-purchase anxiety |

### Experience Principles

1. **Fitment First, Always** — Vehicle selection is the UX foundation; never ask twice; never show results before fitment is established (or explicitly skipped).
2. **Inventory Confidence Upfront** — Stock visibility travels with the product on PLP and PDP; no checkout-time surprises.
3. **Zero Redirect Guarantee** — Scheduling, warehouse selection, and checkout are entirely in-flow; no external tool redirects.
4. **Context Travels with the Member** — Vehicle, warehouse, promotion, and cart state persist across screens, sessions, and channel switches.
5. **Platform-Appropriate Native Feel** — Web is keyboard-navigable and scroll-optimized; mobile uses native sheets, gestures, and camera input.
6. **Speed as a Feature** — <5 taps to add-to-cart on mobile; <3 clicks on web. Performance is a design constraint, not an engineering afterthought.

---

## Desired Emotional Response

### Primary Emotional Goals

The dominant emotion this experience must produce is **Confidence** — "I know this is the right tire, it fits my car, it's at my warehouse, and I'm getting a good deal."

Secondary feelings:
- **Efficiency / Control** — "I accomplished this in minutes, without help."
- **Recognition / Loyalty** — "Costco already knows my vehicle. They remember me."
- **Relief** — "No redirects, no re-entry, no surprises."

### Emotional Journey Mapping

| Journey Stage | Desired Emotion | Negative Emotion to Prevent |
|---|---|---|
| Arrival / Discovery | Orientation, calm | Overwhelm ("where do I start?") |
| Vehicle / Fitment Selection | Certainty | Anxiety ("wrong size?") |
| PLP / Search Results | Confidence | Doubt ("are these compatible?") |
| Product Detail Page | Informed excitement | Decision paralysis |
| Add to Cart | Accomplishment + momentum | Hesitation ("will it be available?") |
| Appointment Booking (in Cart) | Control, convenience | Dread ("another redirect") |
| Order + Appointment Confirmation | Relief, satisfaction | Post-purchase anxiety |
| Return Visit | Belonging, loyalty | Frustration ("start over again?") |

### Micro-Emotions

| Micro-Emotion | Target State | Design Lever |
|---|---|---|
| Confidence vs. Confusion | Confidence | Fitment badge; clear visual hierarchy |
| Trust vs. Skepticism | Trust | Specific warehouse + time stock signals |
| Efficiency vs. Anxiety | Efficiency | Step minimization; no dead ends |
| Accomplishment vs. Frustration | Accomplishment | Post-add-to-cart confirmation micro-animation |
| Belonging vs. Isolation | Belonging | Returning member vehicle recognition |
| Delight vs. Satisfaction | Satisfaction (primary); Delight (reorder) | Smooth 1-tap reorder recognition moment |

### Design Implications

| Emotion | UX Design Approach |
|---|---|
| **Confidence** | "Fits your vehicle ✓" badge on every tile; fitment confirmed before SKUs shown |
| **Efficiency** | Progress indicators in fitment flow; tap count minimization; no unnecessary modals |
| **Recognition / Loyalty** | Persistent "Selected Vehicle" chip; returning member vehicle + last purchase surfaced immediately |
| **Relief (No-Redirect)** | Appointment slot picker in cart; single-screen order + appointment confirmation |
| **Trust in Inventory** | Real-time badge: warehouse name + date/time; never "usually available" |
| **Post-Purchase Calm** | Confirmation screen: order #, tire model, warehouse, appointment date/time, SMS option |

### Emotional Design Principles

1. **Resolve Anxiety Before It Forms** — Show fitment and stock before the member has to ask.
2. **Recognition Creates Loyalty** — Returning members must feel known within 2 seconds of arrival.
3. **Completeness = Calm** — Every terminal screen leaves zero open questions. No "check your email."
4. **Trust Is Earned Through Specificity** — Named warehouse, specific time slot; never vague availability.
5. **Speed Feels Respectful** — A fast experience respects the member's time. Friction communicates the opposite.

---

## UX Pattern Analysis & Inspiration

### Inspiring Products Analysis

| Product | UX Strength | Pattern to Adopt | Pattern to Avoid |
|---|---|---|---|
| TireRack | Fitment-first gate; strong filter | Vehicle selector architecture | External scheduling redirect |
| Instacart | Inline inventory + slot surfacing | Availability badge on product tiles | — |
| iOS Native Patterns | Bottom sheets; haptic; swipe-dismiss | Mobile native modal patterns | — |
| Amazon | Repeat purchase recognition | Vehicle + last purchase reorder card | PDP information overload |
| OpenTable | In-flow slot picker; no redirect | Appointment component in cart | — |

### Transferable UX Patterns

**Navigation:**
- Persistent "Selected Vehicle" chip in header throughout journey
- Fitment-first gate: vehicle/size selection before results display; adapts per persona (pre-filled for Repeat Buyer, prompted for Task-Driven, soft-prompted for Explorer)

**Interaction:**
- Inline slot picker in cart (appointment embedded, not redirected)
- Availability badge on PLP tiles: "In stock today at [Warehouse] · Next slot: Today 2pm"
- Bottom sheet vehicle selector on mobile (swipeable, keyboard-optimized)
- 1-tap reorder card: vehicle + last tire + "Buy Again" on account landing

**Visual:**
- Fitment confidence badge: ✓ "Fits your vehicle" on every compatible tile
- Savings callout on tile for Deal Hunter persona ("$50 off · Member Price")
- Progressive disclosure on PDP: confidence signals first (fits/stock/price/rating), specs collapsed, educational content expandable

### Anti-Patterns to Avoid

1. **Scheduling Redirect** — Any URL transition mid-flow to an external scheduling tool; zero tolerance
2. **Generic Availability Language** — "Usually available" or "Check at store"; specificity is non-negotiable
3. **Post-Browse Fitment Gate** — Fitment must precede results, not appear at add-to-cart
4. **PDP Information Overload** — Hierarchy: fits? → in stock? → price? → rating? → specs (collapsed)
5. **Return Member Re-entry** — Vehicle and warehouse must persist across sessions; never ask twice
6. **Dead-End Promo Pages** — Every promo link must deep-link into the fitment selector, not a static banner

### Design Inspiration Strategy

**Adopt directly:**
- TireRack fitment-first gate architecture
- OpenTable in-flow appointment slot picker
- Instacart availability badge on product tiles
- iOS native bottom sheet patterns for mobile

**Adapt for Costco context:**
- Amazon "Buy Again" → "Your Vehicle + Last Tire + Book Installation" card (richer, warehouse-aware, appointment-integrated)
- Instacart delivery slot → warehouse appointment slot (same surfacing pattern, different domain)

**Avoid entirely:**
- Current Costco scheduling redirect (the #1 conversion killer to eliminate)
- Vague availability messaging
- Post-browse fitment prompting

---

## Design System Foundation

### Design System Choice

**Web:** Tailwind CSS-based custom component library built from existing Figma design language
**iOS Native:** SwiftUI components with Costco brand tokens aligned to Apple Human Interface Guidelines
**Android Native:** Jetpack Compose with Material Design 3 customized to Costco brand tokens
**Shared:** Design token set (colors, spacing, type scale, elevation) as single source of truth — synced from Figma variables to web CSS custom properties and mobile theme files

### Rationale for Selection

- **Brand continuity:** Existing Figma assets demonstrate a mature, distinctive Costco design language — a generic established system (Material, Ant) would conflict with this existing visual identity
- **SSR compatibility:** Tailwind generates static CSS at build time; no runtime CSS-in-JS performance penalty, critical for LCP <2.5s requirement
- **Accessibility built-in:** Component-level WCAG 2.1 AA compliance enforced at design system layer, not retrofitted per feature
- **Tokened cross-platform:** Shared design tokens enable visual consistency across web and native without forcing identical component implementations
- **Brownfield pragmatism:** Existing Figma assets are the design truth; the design system formalizes what already exists rather than starting from scratch

### Implementation Approach

1. Extract design tokens from existing Figma assets: color palette, spacing scale, type scale, elevation, border radius
2. Implement tokens as: CSS custom properties (web) / Swift theme (iOS) / Compose theme (Android)
3. Build core component library in order of journey criticality: Vehicle Selector → Fitment Badge → Product Tile → Availability Badge → Appointment Slot Picker → Cart Summary
4. Document each component in Storybook with accessibility notes, usage guidance, and do/don't examples
5. Establish CI gates: Axe accessibility scan, Lighthouse performance scores, visual regression tests

### Customization Strategy

- **Color:** Costco brand palette (Costco red, navy, white, success green for fitment confirmation) as primary tokens
- **Typography:** Existing Figma type scale formalized; mobile type scale adjusted for density constraints
- **Spacing:** 4px base grid consistent across web and mobile, aligned to existing Figma spacing
- **Component variants:** Each component supports light/dark context, loading/skeleton state, and error state by default
- **Motion:** Subtle and purposeful — micro-animations on add-to-cart confirmation, fitment badge appearance, appointment slot selection; no decorative motion that increases cognitive load

---

## Core Interaction Definition

### Defining Experience

"Scan your plate (or pick your vehicle) → See tires that fit, in stock at your warehouse today → Add to cart with an appointment slot — done."

This is the core loop every persona converges on. Getting it fast, trustworthy, and frictionless on both web and mobile is the entire product's UX proposition.

### User Mental Model

Costco members arrive with three inherited anxieties from the current fragmented experience:
1. "I hope I get the right size" → resolved by fitment-first gate + "Fits your vehicle ✓"
2. "I'll have to call to check stock" → resolved by live warehouse stock on PLP tiles
3. "I'll have to go somewhere else to book" → resolved by appointment slot picker in cart

The defining experience actively disconfirms all three within 30 seconds of arrival.

### Success Criteria

- Vehicle selected + fitment-validated results visible in ≤3 taps / ≤2 clicks
- Every product tile shows "Fits your vehicle ✓" without any user-initiated check
- Add-to-cart is 1 tap after seeing fit + price — no intermediate confirmation dialogs
- Cart shows: tire, price, warehouse, and appointment slots in one view
- From add-to-cart through confirmation: zero re-entry of any previously given input

### Novel vs. Established Patterns

| Interaction | Pattern Type | Innovation |
|---|---|---|
| Fitment-first gate | Established | Adapts per persona: pre-filled / prompted / skippable |
| "Fits your vehicle" badge | Established | Applied to every tile, persistent across browse |
| Appointment slot in cart | Novel for tire retail | Familiar from OpenTable / booking apps |
| LP camera scan | Established | First-class entry equal to manual selection |
| Persistent vehicle chip in header | Novel for tire retail | Familiar from delivery address chips (DoorDash/Uber Eats) |

### Experience Mechanics

**Initiation (Vehicle Selection):**
- Bottom sheet (mobile) / modal (web): LP Scan | Year/Make/Model | Enter Size
- LP scan resolves vehicle spec via Mitchell1 in <2s
- Returning member: vehicle chip pre-loaded; single tap to confirm or change

**Interaction (Browse + Fitment Validation):**
- PLP pre-filtered to compatible tires only; all tiles show fitment badge
- Tile displays: Fitment ✓ | Warehouse stock | Price (member + install) | Rating
- Filters refine within the compatible set; fitment is never broken by filter actions

**Feedback (Real-Time Signals):**
- Live stock: "In stock today at [Warehouse] · Next slot: 2pm Thu"
- Stock change: "Low stock — 2 left" or "Check other warehouses" with selector
- Promo applied: "$50 rebate applied · Total: $XXX" badge on tile and PDP
- Error: clear message + recovery action ("Doesn't fit your vehicle — here's what does")

**Completion (Cart + Appointment + Confirmation):**
- Add to Cart → cart drawer (mobile) / sidebar (web): tire + price + "Choose Installation Slot" component
- Slot picker: calendar + time grid for selected warehouse; 1 tap to confirm and lock slot
- Checkout: pre-filled for members; no new inputs required
- Confirmation: order #, tire model, warehouse address, appointment date/time, Add to Calendar, SMS opt-in

---

## Visual Design Foundation

### Color System

**Brand Palette:**

| Token | Value | Usage |
|---|---|---|
| `--color-primary` | `#E31837` | Costco red — primary CTAs, badges, brand accents |
| `--color-primary-dark` | `#B5122C` | Hover/pressed state for primary actions |
| `--color-navy` | `#003087` | Header, navigation, secondary emphasis |
| `--color-navy-dark` | `#001F5C` | Active nav state, deep emphasis |
| `--color-white` | `#FFFFFF` | Page backgrounds, card surfaces |
| `--color-gray-50` | `#F8F9FA` | Page background, section dividers |
| `--color-gray-100` | `#F0F1F3` | Card backgrounds, input fields |
| `--color-gray-300` | `#C4C6CC` | Borders, dividers, inactive states |
| `--color-gray-600` | `#5A5E6B` | Secondary text, labels |
| `--color-gray-900` | `#1A1B1E` | Primary text, headings |

**Semantic Palette:**

| Token | Value | Usage |
|---|---|---|
| `--color-success` | `#1A7F37` | Fitment badge ✓, in-stock signal |
| `--color-success-bg` | `#E8F5E9` | Fitment badge background |
| `--color-warning` | `#D97706` | Low stock warning, "2 left" indicator |
| `--color-warning-bg` | `#FFF8E1` | Warning badge background |
| `--color-error` | `#C0392B` | Error states, incompatible fitment message |
| `--color-error-bg` | `#FDECEA` | Error state background |
| `--color-member-price` | `#003087` | Member pricing callout in navy |

All foreground/background pairs target WCAG 2.1 AA minimum 4.5:1 contrast ratio (large text: 3:1). Success green on white = 5.2:1 ✓. Primary red on white = 4.6:1 ✓.

### Typography System

| Role | Size / Weight | Usage |
|---|---|---|
| H1 | 28px / 700 | Page-level headings |
| H2 | 22px / 600 | Section headings |
| H3 | 18px / 600 | Card titles, product names |
| H4 | 16px / 600 | Sub-labels, grouped headings |
| Body | 16px / 400 | Default text, descriptions |
| Body SM | 14px / 400 | Secondary text, metadata, ratings |
| Caption | 12px / 400 | Badges, labels, timestamps |
| CTA / Button | 16px / 600 | All interactive button labels |

Mobile adjustments: Body 15px; H1 24px; H2 18px. Line-height: 1.5 (body), 1.3 (headings).

### Spacing & Layout Foundation

**Base unit:** 4px. All spacing is multiples of 4.

| Token | Value | Common Usage |
|---|---|---|
| `--space-1` | 4px | Inline icon gaps, micro-spacing |
| `--space-2` | 8px | Badge padding, tight label spacing |
| `--space-3` | 12px | Input padding, compact list items |
| `--space-4` | 16px | Card padding (mobile), section gaps |
| `--space-6` | 24px | Card padding (desktop), section dividers |
| `--space-8` | 32px | Major section gaps, page padding (desktop) |
| `--space-12` | 48px | Page-level vertical rhythm |
| `--space-16` | 64px | Hero sections, major layout gaps |

**Grid system:**
- Mobile (375px): 1 column, 16px gutters, 16px page margins
- Tablet (768px): 2 columns, 24px gutters, 24px page margins
- Desktop (1024px): 3–4 columns, 24px gutters, 32px page margins
- Wide (1280px): 4 columns, 24px gutters, max content width 1200px, 40px margins

**Component rules:** Product tile gap 16px (mobile) / 24px (desktop). Min touch target 48×48px. Modal internal padding 24px.

### Accessibility Considerations

- **Contrast:** All text/background pairs meet WCAG 2.1 AA (4.5:1 body; 3:1 large text/UI components)
- **Touch targets:** Minimum 48×48px for all interactive elements on mobile
- **Focus indicators:** 2px solid `--color-navy`, 2px offset — on all focusable elements
- **ARIA:** Fitment badge, stock signal, and promo badge use `aria-label` to convey meaning beyond color
- **Motion:** All animations respect `prefers-reduced-motion`; non-animated fallbacks provided
- **i18n:** No hardcoded strings; all UI text via i18n keys; number/currency formatting locale-aware (en-US, en-CA, fr-CA)

---

## Design Direction Decision

### Design Directions Explored

Three directions were evaluated based on existing Figma assets and persona needs:

- **Direction A: Clean Transactional** — High contrast, white backgrounds, product-forward. Costco red for primary CTAs and critical badges only. Current Figma foundation direction. Best for Task-Driven Buyers and Repeat Buyers in the core purchase funnel.
- **Direction B: Rich Merchandising** — Elevated promotional surfaces, brand color accent bands, lifestyle imagery zones. Best for Deal Hunter and Brand Loyalist persona entry points (Homepage, Deals pages, Brand Landing pages).
- **Direction C: Mobile-Native First** — Direction A foundation adapted with native iOS/Android conventions — bottom sheets, tab bar, swipe gestures, 1-column mobile PLP, native time-wheel slot picker.

### Chosen Direction

**Hybrid A + B + C:** Direction A as the core purchase journey foundation, Direction B applied surgically to promotional and brand entry surfaces, Direction C as the native mobile platform adaptation.

- Core journey screens (Size Selection, PLP, PDP, Cart, Checkout, Confirmation): **Direction A**
- Homepage hero, Deals/Promotions pages, Brand Landing pages: **Direction A + B blended**
- Native iOS app: **Direction A tokens + Direction C interaction patterns** (SwiftUI)
- Native Android app: **Direction A tokens + Direction C interaction patterns** (Jetpack Compose)
- Web mobile breakpoint (375px): **Direction A layout with Direction C spacing density**

### Design Rationale

1. **No visual reinvention:** Figma assets embody Direction A — rebuilding from a different visual direction would create brand inconsistency and increase design-dev cost with no conversion benefit.
2. **Surgical merchandising elevation:** Direction B elements apply only at persona-specific entry points (Deal Hunter, Brand Loyalist) — not throughout the core purchase funnel, which must remain fast and transactional.
3. **Platform-native mobile:** Direction C is a platform adaptation on shared design tokens — required by the iOS 16+ / Android 10+ native app mandate in the architecture specification.

### Implementation Approach

- Shared design token foundation (colors, spacing, typography) applies across all directions and platforms
- Core component library built in Direction A style; promotional surface components follow Direction B enrichment rules
- Mobile-specific component variants (bottom sheets, tab bar, native list patterns) defined separately under Direction C adaptation layer
- Design-to-dev handoff via Storybook showcasing all three direction contexts

---

## User Journey Flows

### Journey 1: Mission-Driven / Search-First Purchase (Personas 1 & 2)

The primary purchase funnel covering the highest-volume path across all entry points.

```mermaid
flowchart TD
    A([Entry: Tire Landing / Search / Homepage CTA]) --> B{Vehicle context\nalready saved?}
    B -- Yes --> C[Show 'Selected Vehicle' chip\n+ Confirm or Change]
    B -- No --> D[Vehicle Selector\nLP Scan | Year/Make/Model | Size]
    C --> E[PLP: Fitment-validated results\nwith live stock badges]
    D --> D1{LP scan or\nmanual?}
    D1 -- LP Scan --> D2[Camera → Mitchell1 VIN lookup\n<2s resolution]
    D1 -- Manual --> D3[Year → Make → Model typeahead\nor direct size entry]
    D2 --> E
    D3 --> E
    E --> F{Filter / Sort\nwithin compatible set}
    F --> E
    E --> G[Tap product tile → PDP\nFitment ✓ | Stock | Price | Rating]
    G --> H{Promotion\napplicable?}
    H -- Yes --> H1[Show '$X rebate applied' badge on PDP]
    H -- No --> G2[Standard PDP view]
    H1 --> I[Add to Cart CTA]
    G2 --> I
    I --> J[Cart Drawer / Sidebar\nTire + Price + Choose Installation Slot]
    J --> K[Slot Picker: Calendar + Time Grid\nfor selected warehouse]
    K --> L{Slot selected?}
    L -- Yes --> M[Checkout: Pre-filled member details]
    L -- Skip / Later --> M
    M --> N[Order + Appointment Confirmation\nOrder # | Tire | Warehouse | Date/Time | SMS opt-in]
```

### Journey 2: Brand Loyalist Purchase (Persona 5)

```mermaid
flowchart TD
    A([Entry: Brand Gallery / Brand Carousel / Brand Search]) --> B[Brand Landing Page\nBrand imagery + tier hierarchy]
    B --> C[Vehicle Selector triggered immediately\n'Which vehicle are we fitting for?']
    C --> D{Vehicle saved?}
    D -- Yes --> E[Confirm vehicle + show brand-filtered fitment results]
    D -- No --> F[LP Scan | Year/Make/Model]
    F --> E
    E --> G[PLP: Only brand tires fitting vehicle\nOrganized by tier: All-Season | Winter | Performance | EV]
    G --> H[Tap tile → Brand PDP\nBrand differentiators | Warranty | Specs]
    H --> I[Add to Cart]
    I --> J[Cart + Slot Picker]
    J --> K[Order + Appointment Confirmation]
```

### Journey 3: Services-First / Maintenance Seeker (Persona 6)

```mermaid
flowchart TD
    A([Entry: Services Tab / Tire Center / Schedule CTA]) --> B[Services Landing\n'What do you need?']
    B --> C{Service type}
    C -- Installation --> D[Vehicle Selector + Tire confirmation required]
    C -- Rotation or Repair --> E[Vehicle Selector\nNo tire purchase required]
    D --> F[Quick tire selection:\nLast purchased OR size-matched options]
    F --> G[Select tier: 3 options by price + rating]
    G --> H[Appointment Slot Picker\nWarehouse + Calendar + Time]
    E --> H
    H --> I{Slot available today?}
    I -- Yes --> J[Confirm Same-Day\nHighlight 'Available Today']
    I -- No --> K[Show next 3 available slots across ±2 days]
    J --> L[Booking Confirmation]
    K --> L
    L --> M[Confirmation: Service | Vehicle | Warehouse | Date/Time | SMS]
```

### Journey 4: Repeat Buyer / Reorder (Persona 7)

```mermaid
flowchart TD
    A([Entry: My Account / Buy Again / Order History]) --> B[Account Landing\n'My Vehicles' card surfaced immediately]
    B --> C[Vehicle card: Make/Model + Last tire purchased\n'Buy Again' CTA prominent]
    C --> D{Tap 'Buy Again'}
    D --> E[Previous tire shown in cart\n'Same tire as last purchase' confirmation]
    E --> F{Fitment still correct?}
    F -- Yes --> G[Add to Cart + Slot Picker for installation]
    F -- Changed vehicle --> H[Update vehicle → re-validate fitment\nShow compatible options]
    H --> G
    G --> I[Checkout: Fully pre-filled\nPayment + address from account]
    I --> J[Confirmation: Order # | Same tire | New appointment]
```

### Journey Patterns

| Pattern | Description | Applies To |
|---|---|---|
| **Fitment Gate** | Vehicle selector before results; adapts (pre-filled / prompted / skippable) by persona | All journeys |
| **Persistent Vehicle Chip** | Selected vehicle in header throughout; tap to change triggers re-validation | All journeys |
| **Live Stock Signal on Tile** | "In stock today at [Warehouse] · Next slot: Xpm" on PLP tiles | Journeys 1, 2, 3 |
| **In-Cart Slot Picker** | Appointment time selection embedded in cart — never a redirect | Journeys 1, 2, 3, 4 |
| **Reorder Card** | Vehicle + last tire + "Buy Again" on account landing | Journey 4 |
| **Brand-First Fitment** | Brand selected before fitment gate; results filtered to brand + vehicle | Journey 2 |
| **Service-First Routing** | Service type selected first; tire selection optional or quick-select | Journey 3 |

### Flow Optimization Principles

1. **Shortest path wins:** Mission-Driven buyer: 5 steps to checkout. Repeat Buyer: 3 steps. Every journey has been measured for minimum step count.
2. **No dead ends:** Every error state (out of stock, fitment mismatch, expired promo) has a labelled recovery action with a path forward.
3. **Progressive commitment:** Slot picker offered but not mandatory at cart — members can purchase and schedule later, reducing checkout abandonment from scheduling hesitation.
4. **Context always visible:** "Selected Vehicle" chip and warehouse selection accessible from any screen — one tap to change without losing cart state.

---

## Component Strategy

### Design System Components (Tailwind + Headless UI)

The following standard components are covered by Tailwind CSS + Headless UI with Costco brand token theming — require theming only, no custom design:

Buttons (primary/secondary/ghost), Text Inputs, Dropdowns/Selects, Checkboxes/Radios, Modals/Dialogs, Tooltips, Badges/Pills, Skeletons, Toasts/Notifications, Accordions, Tabs, Breadcrumbs.

### Custom Components

| Component | Purpose | Priority |
|---|---|---|
| **Vehicle Selector** | Fitment gate — LP scan / Year/Make/Model / manual size entry | Phase 1 |
| **Fitment Confirmation Badge** | "✓ Fits your vehicle" on every tile and PDP | Phase 1 |
| **Warehouse Availability Badge** | Live stock + next slot on PLP tiles and PDP | Phase 1 |
| **Product Tile (Tire Card)** | Full purchase-confidence tile for PLP | Phase 1 |
| **Appointment Slot Picker** | In-cart slot booking component, no redirect | Phase 1 |
| **Selected Vehicle Chip** | Persistent header vehicle context element | Phase 1 |
| **Reorder Card** | Account landing: last purchase + "Buy Again" CTA | Phase 2 |
| **Promo Badge** | Promotion overlay on tile and PDP | Phase 2 |
| **Brand Spotlight Card** | Brand Loyalist landing entry surface | Phase 2 |
| **Service Selection Card** | Journey 3 service-first entry card | Phase 2 |
| **Educational Content Card** | Inline tire buying guide module | Phase 3 |
| **Comparison Tray** | Side-by-side tire spec comparison | Phase 3 |
| **Staggered Fitment Selector** | Front/rear different size handling | Phase 3 |

**Key component specs:**

- **Vehicle Selector:** 3-tab switcher (LP Scan | Vehicle | Size); states: empty, active, VIN-resolving, resolved, error; bottom sheet on mobile / modal on web; `role="dialog"`, focus trap, escape-to-dismiss
- **Fitment Badge:** Green checkmark + "Fits your [Make Model]"; states: confirmed, warning, no-vehicle; `aria-label` carries meaning beyond color
- **Warehouse Availability Badge:** Stock dot + warehouse name + slot preview; `aria-live="polite"` for real-time updates
- **Product Tile:** Image | brand/model | fitment badge | stock badge | price | rating | Add-to-cart; states: default, hover, out-of-stock (dimmed), promo-applied
- **Appointment Slot Picker:** Warehouse chip + date scroll calendar + time grid + Confirm; keyboard-navigable grid; `role="grid"` with per-slot `aria-label`
- **Selected Vehicle Chip:** Car icon + vehicle text + edit icon; `role="button"`, `aria-label="Change vehicle: [vehicle]"`

### Component Implementation Strategy

- All custom components built on Tailwind design tokens for cross-context consistency
- Every component ships with: default, hover, focus, loading/skeleton, error, and empty states
- WCAG 2.1 AA enforced at component level: contrast, ARIA labels, keyboard navigation, visible focus
- Storybook documentation: anatomy, all states, variants, do/don’t usage examples
- CI gates: Axe accessibility scan + visual regression test per component

### Implementation Roadmap

- **Phase 1:** Vehicle Selector, Fitment Badge, Availability Badge, Product Tile, Slot Picker, Vehicle Chip
- **Phase 2:** Reorder Card, Promo Badge, Brand Spotlight Card, Service Selection Card
- **Phase 3:** Educational Content Card, Comparison Tray, Staggered Fitment Selector

---

## UX Consistency Patterns

### Button Hierarchy

| Level | Style | Usage |
|---|---|---|
| **Primary** | Solid Costco red, white text, 16px/600, 48px min height | One per screen; most important action (Add to Cart, Confirm Slot, Checkout) |
| **Secondary** | White background, navy border + text | Supporting action alongside primary |
| **Ghost / Tertiary** | No background, navy text, underline on hover | Low-emphasis actions (Skip, Cancel, View Details) |
| **Destructive** | Solid error red, white text | Irreversible actions only (Remove from Cart) |
| **Disabled** | 40% opacity, `cursor: not-allowed` | Action unavailable; `aria-disabled="true"` + tooltip why |

Labels: always verb + noun ("Add to Cart", "Choose Slot"). Loading state shows spinner + present-tense label ("Adding…"). Min touch target 48×48px on all platforms.

### Feedback Patterns

| Type | Trigger | Placement | Duration |
|---|---|---|---|
| **Success Toast** | Add to cart, slot confirmed, order placed | Top-right (web) / top (mobile) | 4s auto-dismiss |
| **Error Toast** | API failure, out-of-stock at checkout | Same as success | Persistent until dismissed |
| **Inline Validation Error** | Form field blur failure | Below field | Until corrected |
| **Warning Badge** | Low stock, promo expiring | On relevant tile/PDP element | Persistent |
| **Confirmation Dialog** | Destructive or irreversible action | Center overlay | User-dismissed |
| **Skeleton Loading** | Any async content | In-place, shape-matched | Until content loads |

Error messages state what went wrong + what to do. Never "An error occurred" without context. Add-to-cart success plays cart icon badge-count micro-animation.

### Form Patterns

- Labels above inputs; placeholder text as hint only, never substitutes label
- Validation on blur (not on keypress); error shown after field is left
- Mark optional fields "(optional)"; required fields have no asterisk
- Year/Make/Model: typeahead autocomplete with keyboard navigation (`aria-autocomplete`, `role="listbox"`)
- On submit: focus moved to first error field; error summary at top for screen readers
- Success state: green checkmark on field after valid entry (fitment size especially)

### Navigation Patterns

**Web:** Persistent global header — Costco logo | Search | Selected Vehicle Chip | Warehouse | Cart | Account. Breadcrumb on PLP and PDP. All elements keyboard-focusable in DOM order. Skip-to-main-content link.

**Mobile Native:** Bottom tab bar — Home | Shop | Services | Account | Cart (5 tabs). Native back gesture. Progress dots or "Step X of Y" in multi-step flows. Bottom sheet for contextual actions; swipe-down to dismiss.

### Modal & Overlay Patterns

| Modal | Trigger | Behavior |
|---|---|---|
| Vehicle Selector | Any "Select Vehicle" CTA | Centered modal (web) / bottom sheet (mobile); focus trapped; Escape/swipe dismisses |
| Warehouse Selector | "Change Warehouse" tap | Searchable list; close returns to previous state |
| Slot Picker Panel | "Choose Installation Slot" in cart | Inline panel, not full-screen; remains open while user views cart |
| External Link Warning | Any link leaving costco.com | Advisory: "You're leaving Costco.com" + Continue + Cancel |
| Tire Size Guide | "?" next to size selector | Right drawer (web) / bottom sheet (mobile); doesn't block flow |

All modals: visible close (X) + Escape support. Return focus to triggering element on close. No modal-on-modal stacking.

### Empty States

Every empty state has: illustration or icon + specific message + recovery CTA.

- No search results: "No tires found for [term]. Try a different size or browse all." + CTA
- No vehicles saved: "Add your vehicle to see tires that fit." + Add Vehicle CTA
- No slots today: "Next available: [date] at [Warehouse]." + Find other warehouses CTA
- Out of stock: greyed tile + "Check other warehouses" CTA

### Search & Filter Patterns

- Search autocomplete: recent searches + brand suggestions + size shortcuts
- Active filter pills shown above results; each dismissible; "Clear all" when >1 active
- Live filter application — no "Apply" button needed; results update instantly
- Result count always reflects current fitment + filter state: "Showing 24 tires that fit your 2022 Tesla Model 3"
- Default sort: "Best match" (fitment + rating + stock composite); options: Price ↑↓, Rating, Brand A–Z

---

## Responsive Design & Accessibility

### Responsive Strategy

**Mobile-first design** — all layouts designed at 375px and progressively enhanced upward.

| Platform | Strategy |
|---|---|
| **Mobile Web (375px)** | 1-column; thumb-zone CTAs in bottom 60%; sticky bottom Add-to-Cart; filter as bottom tray; compact product tiles |
| **Tablet (768px)** | 2-column PLP; collapsible filter sidebar; sticky header with full vehicle chip; horizontal slot calendar |
| **Desktop (1024px)** | 3-column PLP; always-visible filter sidebar; persistent cart sidebar on PDP; full slot picker inline in cart |
| **Wide (1280px)** | 4-column PLP; max-width 1200px container; increased whitespace; expanded brand imagery zones |
| **Native Mobile App** | SwiftUI (iOS) / Compose (Android); bottom tab bar; bottom sheets; no hamburger; system font scaling respected |

**Key layout transitions:**
- PLP: 1 → 2 → 3 → 4 columns across breakpoints
- Filters: bottom tray (mobile) → collapsible sidebar (tablet) → persistent sidebar (desktop)
- Slot picker: stacked vertical (mobile) → horizontal calendar row (tablet+)
- Vehicle Selector: full-screen (mobile web) → centred 480px modal (tablet+)

### Breakpoint Strategy

| Breakpoint | px | Key Layout Change |
|---|---|---|
| `sm` | 375px | Mobile base; single column, thumb-zone optimised |
| `md` | 768px | 2-column PLP; filter sidebar; tablet navigation |
| `lg` | 1024px | 3-column PLP; persistent filter + cart sidebar |
| `xl` | 1280px | 4-column PLP; max-width 1200px; wider margins |

Implementation: Tailwind CSS mobile-first prefix classes. Base styles = mobile; prefixed classes override at breakpoints.

### Accessibility Strategy (WCAG 2.1 AA)

| Category | Requirement | Implementation |
|---|---|---|
| **Colour contrast** | ≥4.5:1 body text; ≥3:1 large text/UI | All token pairs validated; CI Axe `color-contrast` rule |
| **Keyboard navigation** | All interactive elements reachable by Tab | Focus management on modals, slot picker, vehicle selector |
| **Focus indicators** | Visible on all focusable elements | 2px solid navy, 2px offset; never `outline: none` |
| **Screen reader** | All state changes announced | `aria-live` on stock/fitment badges; `aria-label` on icon-only buttons |
| **Touch targets** | Min 48×48px | Enforced in component specs; tested with device inspection |
| **Skip links** | Skip to main content on all web pages | Visually hidden, appears on focus |
| **Images** | Meaningful `alt` text | Product image: brand + model + size; decorative images `alt=""` |
| **Motion** | Respect `prefers-reduced-motion` | All CSS animations behind `@media (prefers-reduced-motion: no-preference)` |
| **Zoom** | Functional at 200% browser zoom | Relative units throughout; tested at 200% |
| **i18n** | en-US, en-CA, fr-CA | No hardcoded strings; locale-aware currency/measurement formatting |
| **Forms** | All inputs labelled; errors programmatically linked | `aria-labelledby`; `aria-describedby` for error messages; `aria-invalid="true"` |

**Assistive technology targets:** VoiceOver (iOS + macOS) · TalkBack (Android) · NVDA + Chrome (Windows)

### Testing Strategy

**Automated CI gates:**
- Axe-core scan on every component and every route (zero AA violations to merge)
- Lighthouse accessibility score ≥90 on all pages
- Visual regression per component state (Storybook + Chromatic)
- Colour contrast check on every design token pair

**Manual per sprint:**
- Keyboard-only navigation of all 4 user journeys
- VoiceOver (iOS) end-to-end purchase journey
- TalkBack (Android) end-to-end purchase journey
- NVDA + Chrome critical path
- 200% zoom on all key screens
- Colour blindness simulation (Protanopia, Deuteranopia) on stock/fitment badge states

**Device matrix:** iPhone 15, iPhone SE · Samsung Galaxy S24, Pixel 7 · iPad 10th gen, iPad Pro 12.9" · Desktop 1280px+ (macOS + Windows)

### Implementation Guidelines

**Responsive:**
- Tailwind breakpoint prefixes; never fixed `px` widths on containers
- Images: `srcset` + `sizes` + WebP with JPEG fallback
- `touch-action: manipulation` on tap targets to eliminate 300ms delay

**Accessibility:**
- Semantic HTML first: `<nav>`, `<main>`, `<button>` — never `<div>` for interactive elements
- Modal focus trap via `focus-trap-react` or equivalent; restore focus to trigger on close
- `aria-live="polite"` on stock badge, fitment badge, search result count
- Slot picker: `role="grid"`, `role="gridcell"`, arrow-key navigation
- Skip link: `<a href="#main-content" class="sr-only focus:not-sr-only">Skip to main content</a>`
