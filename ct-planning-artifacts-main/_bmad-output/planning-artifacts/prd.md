# PRODUCT REQUIREMENTS DOCUMENT
## Costco Tires Digital Experience Modernization

```yaml
---
title: "Product Requirements Document - Costco Tires Digital Experience Modernization"
projectName: "ct-planning-artifacts"
author: "Rupesh.Jain"
date: "2026-04-16"
status: "Complete PRD"
stepsCompleted: 
  - "step-01-init"
  - "step-02-discovery"
  - "step-03-synthesis"
  - "step-04-goals"
  - "step-05-personas"
  - "step-06-journeys"
  - "step-07-requirements"
  - "step-08-success"
  - "step-09-scope"
  - "step-10-risks"
  - "step-11-review"
  - "step-12-complete"
inputDocuments:
  - "Goals-Personas-UserJourneys.txt"
  - "product-brief-costco-tires.md"
  - "docs/vision.txt"
  - "docs/sow.txt"
  - "docs/bff-layer/document-project/project-context.md"
workflowType: "prd"
infoRating: "high-density"
traceabilityLevel: "persona-journey-requirement"
classification:
  domain: "ecommerce"
  projectType: "web_app+mobile_app"
  complexity: "medium"
lastEdited: "2026-05-01"
editHistory:
  - date: "2026-05-01"
    changes: "Applied validation report fixes: added classification, User Journey 5, Platform & Distribution NFR, measurability fixes for FR5/FR9/FR10/FR13, fixed NR-Scale-2/3 Redis leakage, NR-Avail-2 threshold, NR-Comp-1 tax assumption, reformatted all FR requirement statements to BMAD actor-capability pattern"
---
```

---

## EXECUTIVE SUMMARY

**Purpose:** Enable members to find and purchase tires that fit their vehicle with confirmed availability for installation at a selected warehouse at their preferred time, across a seamless omnichannel experience (web and native mobile app).

**Problem Statement:** Costco's current tire shopping journey fragments across multiple disconnected systems. Members navigate separately to select tires, schedule installation, and check inventory. Each transition requires re-authentication and context re-entry (vehicle info, tire preferences). This fragmentation drives abandonment at predictable inflection points and creates a competitive disadvantage against integrated competitors (TireRack, Discount Tire, SimpleTire).

**Impact of Current State:**
- Abandoned carts when redirect to separate scheduling system occurs
- Lost repeat purchase velocity due to missing vehicle/order history
- Underutilized tire center appointment capacity from booking friction
- Lower mobile app engagement vs. native competitors
- Missed market share in growing EV tire segment

**Strategic Approach:** Build a unified, context-preserving tire-buying ecosystem across costco.com/.ca and the Costco mobile app using a native-first mobile strategy and integrated inventory/scheduling visibility.

**Transformation Scope:** Five phased delivery (Phase 1: Web reimagine + cloud migration → Phase 5: Vehicle profiles + order history) delivering omnichannel capability, real-time inventory transparency, and integrated appointment scheduling without context loss.

**Success Outcome:** Seamless member journey where vehicle selection, fitment validation, pricing, real-time availability, cart persistence, and appointment booking work identically across web and mobile channels—eliminating re-entry friction and driving conversion rate improvement.

---

## GOALS & BUSINESS DRIVERS

### Experience Goals & Business Impact

| Goal # | Experience Goal | Business Impact | Personas Served |
|--------|-----------------|-----------------|-----------------|
| **1** | **Native Mobile App Experience** for tire selection and appointment booking | Engagement rate increase, Click-through rate improvement, Conversion rate lift | Task-Driven, Searcher, Services Seeker, Repeat Buyer |
| **2** | **Consistent Experience Across Channels** (costco.com/.ca web, Costco mobile app, other touchpoints) | Lower exit/drop-off rate, Reduced bounce rate, Improved user retention | All personas (foundation) |
| **3** | **Preserve Shopper Context Across Touchpoints** (vehicle selection, fitment, cart, preferences) | Increased Customer Lifetime Value, Improved repeat purchase rate, Higher loyalty signal | Brand Loyalist, Repeat Buyer, Services Seeker |
| **4** | **Eliminate Disconnected Flows** that redirect shoppers away from scheduling/service | Improved add-to-cart completion, Improved checkout completion, Higher appointment booking rate | All personas (required) |

### Derived Strategic Imperatives

**Speed & Friction Reduction:** Members entering with high purchase intent must reach checkout in minimal steps (<5 taps on mobile, <3 clicks on web).

**Context Continuity:** Vehicle selection made once persists across session/channel boundaries; members never re-enter fitment information.

**Trustworthy Inventory:** Real-time availability messaging builds confidence in purchase decision; no post-sale surprises due to false availability claims.

**Unified Scheduling:** Appointment booking integrates into the purchase flow with no redirects to separate tools or warehouse systems.

**Channel Flexibility:** Complete any journey started on one channel (e.g., mobile browse) on any other channel (web checkout) with full context and history preserved.

---

## WHO THIS SERVES: USER PERSONAS

### Persona 1: Task/Mission-Driven Buyer
**Archetype:** High-intent professional shopper who arrives with a clear goal

**Description:** Shopper with explicit mission: buy the correct tires fast, without distractions. Arrival carries high purchase probability signal.

**Entry Point:** Start Tire Search (highest-intent flow) — arrives directly to tires landing page or via CTA

**Behavioral Profile:**
- Wants shortest path to fitment → results → checkout
- No interest in browsing, learning, or comparison
- Extremely time-sensitive; measures success by speed and certainty
- High sensitivity to friction (extra steps, load times, interruptions)

**Product Priorities:**
1. Fast fitment validation (≤3 taps to vehicle selection on mobile)
2. Instant results filtering (no loading delays)
3. Direct-to-cart path (minimize product detail pages if decision is clear)
4. Certainty of availability before committing to purchase

**Primary KPI:** Add-to-Cart Conversion Rate (%, indexed to baseline)

**Use Case Example:** "I need winter tires for my Model 3, I know the size, show me what's in stock at my warehouse NOW. I want to checkout in under 2 minutes."

---

### Persona 2: Searcher / Problem Solver
**Archetype:** Intent-driven searcher who trusts search interface over navigation menus

**Description:** User who begins with a direct keyword, size, brand, or need into search rather than navigating category menus. Knows something (tire size, brand, seasonal need) but requires confirmation/guidance.

**Entry Point:** Global Search (fastest path to targeted results) — uses site search box, app search, or voice search

**Behavioral Profile:**
- Trusts search box more than menu hierarchies
- Wants intelligent suggestions and quick shortcuts (autocomplete, recent searches, brand suggestions)
- Needs high search accuracy and intent detection (e.g., search "winter" routes to seasonal tire filters)
- Quick to abandon if search results are irrelevant or require refinement
- May need guidance on tire sizing if search is by need rather than size

**Product Priorities:**
1. High-accuracy search indexing (tire size formats, brand synonyms, seasonal keywords)
2. Intent detection and pre-filtered results (search "EV" → shows only compatible EV tires)
3. Quick links to narrowing options (recent searches, "you searched for X, did you mean Y?")
4. Fitment validation in search results (show "fits your vehicle" callout if context preserved)

**Primary KPI:** Search Success Rate (%, defined as: user completed purchase or got to product detail after search query)

**Use Case Example:** "I search 'Michelin Defender', I get Michelin results filtered to my vehicle size, with clear 'fits your vehicle' confirmation. I don't need to use the vehicle selector again."

---

### Persona 3: Explorer / Category Browser
**Archetype:** Discovery-oriented shopper who browses, compares, and educates before deciding

**Description:** Shopper who prefers to explore categories and filters, view comparison tables, and read educational content before committing. May not know vehicle fitment size initially.

**Entry Point:** Shop Navigation (browse intent) — arrives via Shop → Tires & Auto, category tiles, or brand galleries

**Behavioral Profile:**
- Moves through category filters, product comparison pages, and educational modules
- Wants to understand options (tire types, brands, performance levels, price ranges) before committing
- May not know tire size and requires guidance (size guide, vehicle lookup help)
- High engagement (reads reviews, compares specs, opens multiple tabs)
- Success measured by confidence in decision, not speed

**Product Priorities:**
1. Strong information architecture (clear category hierarchy: Winter → Brands → Performance Levels)
2. Educational modules (tire sizing guide, seasonal tire benefits, performance tier explanations)
3. Comparison tools (side-by-side product specs, performance ratings, price transparency)
4. Guidance overlays (tooltips explaining tire metrics: traction grade, temperature grade, UTQG)
5. Filter strength (price range, brand, tire class, seasonal, customer ratings)

**Primary KPI:** Engagement Depth (pages per session, average session duration) + CTR from educational content

**Use Case Example:** "I click on 'Tires & Auto', I see category cards for Winter, All-Season, Performance. I click Winter, I see a size guide overlay and brands filtered to what fits my vehicle. I read the winter tire buying guide, compare three options, add one to cart."

---

### Persona 4: Deal Hunter / Promo-Driven Buyer
**Archetype:** Price-sensitive shopper motivated by promotions and limited-time offers

**Description:** Member arriving from promotional banners, email campaigns, seasonal sales pages, or deals section. Primary motivation is capturing value through promotion/coupon/discount.

**Entry Point:** Deals / Promotions (deal-first journey) — arrives from banners, email link, Deals section tile, or seasonal promo page

**Behavioral Profile:**
- Expects immediate visibility into promotional value (e.g., "$50 off" or "free installation")
- Needs confirmation that promotion applies to their fitment selection before cart
- Abandons quickly if promotion disappears or doesn't apply
- Sensitive to clear savings messaging (show savings amount, not just final price)
- Often price-comparison shops; needs transparent pricing

**Product Priorities:**
1. Promo auto-routing: Deal link goes directly to fitment selector (not generic sale page)
2. Clear savings display: Show promotion details (e.g., "$50 rebate on Michelin tires") before add-to-cart
3. No dead-end promo pages: Never land on a promotion page that doesn't lead to purchase flow
4. Promotion confirmation: Show "Your promotion is applied" after add-to-cart
5. Expired promotion handling: Clear messaging if user's selections don't qualify ("This promotion applies to Michelin winter tires; this tire is 3-season")

**Primary KPI:** Fitment Start Rate (%, users who start fitment selection from promo link) + Promo-to-purchase conversion

**Use Case Example:** "I click 'Winter Tire Sale: $50 Off Michelin' in email, I land on fitment selection for my vehicle, I see Michelin tires with '$50 off this tire' badges, savings total is clearly shown, I add to cart with confidence the promotion is applied."

---

### Persona 5: Brand Loyalist / Preference-Anchored Buyer
**Archetype:** Decision anchored in brand trust; starts shopping from brand preference, not price or need

**Description:** Shopper who trusts a specific tire brand (Michelin, Bridgestone, Pirelli, etc.) and enters journey anchored in that brand. Decision tree is "I want Michelin → What Michelin tire fits my vehicle? → Buy."

**Entry Point:** Brand Entry (brand galleries, brand shelves, brand search) — arrives via brand links, brand carousel, or search for brand name

**Behavioral Profile:**
- Decision is anchored in brand trust, not price comparison or broad exploration
- Needs fitment validation BEFORE seeing SKUs (trust is lost if they're shown an incompatible tire from their preferred brand)
- Responds strongly to brand-specific value messaging (e.g., "Michelin EV tire with 20% better range")
- May want brand-specific content (brand history, warranty, performance tiers)
- High confidence in purchase once fitment validated with preferred brand

**Product Priorities:**
1. Brand entry without fitment friction: Click "Michelin" → vehicle selector appears immediately (not SKU list first)
2. Fitment-first filtering: Show ONLY Michelin tires compatible with selected vehicle
3. Brand-specific content: Michelin warranty info, brand differentiators, available tire classes from that brand
4. Performance tier clarity: Show Michelin's tire hierarchy (Defender, Crossclimate, Pilot Sport, etc.)
5. Exclusive offers: Brand-specific promotions or member discounts

**Primary KPI:** Validated Fitment Rate (%, users who validate fitment for brand before adding to cart) + Brand-to-purchase conversion

**Use Case Example:** "I click on Michelin brand gallery, I'm immediately asked 'What vehicle?', I select my vehicle, I see 8 Michelin tires that fit—organized by tire class (all-season, winter, performance, EV). I click Defender (familiar product), I see specific features, warranty, price, I add to cart."

---

### Persona 6: Maintenance Seeker / Car-Care Planner  
**Archetype:** Service-first entry with tire shopping as secondary need; arrives to solve service problem (rotation, flat repair, installation scheduling)

**Description:** User arriving with primary intent to schedule a service (tire rotation, flat repair) or book installation at a tire center. Tire shopping may be secondary upsell opportunity or prerequisite for service (e.g., install appointment requires tires first).

**Entry Point:** Services-First (service menu → scheduling) — arrives via warehouse/tire center page, Services tab, or appointment booking flow

**Behavioral Profile:**
- Primary intent is service convenience (nearest warehouse, fastest available appointment)
- Requires seamless single scheduling flow (not multiple tools or redirects)
- May need quick tire replacement if there's an urgent need (flat tire, wear concern)
- Values clarity on what's included (rotation free? installation included? appointment cancellation policies?)
- Success measured by appointment confirmation, not product selection breadth

**Product Priorities:**
1. Unified scheduling: One flow for "Schedule Appointment" covering tire selection, time slot, warehouse
2. Warehouse availability prominence: Show available appointment times DURING tire selection (not after)
3. Service bundling: Clear options for tire purchase + installation + rotation as packages
4. Quick-path for established customers: "Schedule rotation for vehicle X" (pre-filled from history)
5. Appointment confirmation clarity: Show exactly what's included (tire, labor, balance, alignment availability?)

**Primary KPI:** Appointment Completion Rate (%, users who book appointment successfully) + Average appointment lead time

**Use Case Example:** "I go to Schedule Appointment, my vehicle is remembered from history, I see available time slots at my warehouse TODAY and TOMORROW with prices (tire + installation), I click 2 PM tomorrow, confirm, I get appointment number and SMS reminder."

---

### Persona 7: Repeat Buyer / Owner
**Archetype:** Returning customer with previous Costco tire purchase; highest lifetime value if friction is minimized

**Description:** Member who previously purchased tires from Costco and is returning for replacement/rotation/service. May not remember previous tire size or order details. Expects frictionless repurchase path.

**Entry Point:** Ownership / Reorder (Buy Again, past purchase history) — arrives via Account tab, "Reorder" link, "My Vehicles" section, or "Buy Again" from order history

**Behavioral Profile:**
- Expects 1-2 step path to reorder (not start from scratch like new customer)
- Fitment is pre-validated from saved vehicle history (expects "Same tire as last purchase?" suggestion)
- Highest-value target: Low tolerance for re-entry friction (will switch to competitor if forced to start over)
- Likely to upsell on service (rotation, installation) or add-on services
- Strong loyalty signal if experience is smooth

**Product Priorities:**
1. Vehicle persistence: "My Vehicles" section shows all vehicles member has purchased tires for
2. Purchase history recall: Click vehicle → see "Last tire purchased: Michelin Defender Winter 225/45R18 on October 15, 2023"
3. Quick reorder path: "Buy Again" button → shows previous tire → add to cart OR "Browse similar"
4. Fitment reminder: "Is [previous size] still correct for your [Vehicle]?" (allow override)
5. Appointment suggestion: "Schedule installation? Your last appointment was on [date]."

**Primary KPI:** Reorder Success Rate (%, repeat customers who complete purchase) + Time to Checkout (avg minutes)

**Use Case Example:** "I login, click 'My Account', I see 'Model 3 - Last purchased: Michelin Defender Winter 225/45R18 on 2023-10-15', I click 'Buy Again', previous tire shows in cart pre-selected, I confirm fitment is still correct, I add to cart and checkout in under 1 minute. Installation reminder appears: 'Schedule appointment?'"

---

### Persona 8: Battery Buyer
**Archetype:** Member with an automotive battery replacement need, following the same digital purchase journey as tire shoppers on costco.ca

**Description:** Member who needs to purchase an automotive battery (lead-acid, AGM, or lithium) for their vehicle. Arrives with varying urgency — from emergency dead-battery scenarios to planned seasonal replacements. The battery buying journey mirrors the tires experience end-to-end: vehicle-first fitment, product browsing, warehouse inventory confirmation, and optional installation scheduling — all integrated into the tires.costco.ca platform infrastructure.

**Entry Point:** Direct Battery Search / Vehicle Selection for battery compatibility — arrives via the "Batteries" section on tires.costco.ca, site search query ("car battery"), promotional link, or cross-sell prompt during tire shopping

**Behavioral Profile:**
- Often arrives with urgency (dead battery, imminent trip, or seasonal prep) — time-to-resolution is critical
- Typically does NOT know battery specifications (group size, CCA, reserve capacity) — requires guided vehicle lookup
- Values speed and certainty of warehouse availability (emergency scenario requires in-stock confirmation today)
- Installation need varies — some members prefer DIY; others need Costco warehouse battery replacement service
- Price-aware but warranty and brand confidence matter more than cheapest option
- High cross-sell affinity: likely to notice tire promotions while purchasing battery, and vice versa

**Product Priorities:**
1. Fast vehicle-to-battery compatibility lookup (Year/Make/Model → instantly see compatible batteries)
2. Clear battery specification display (group size, CCA rating, reserve capacity, warranty years)
3. Warehouse availability confirmation with today's stock status (urgency scenario)
4. Optional battery installation bundling (Costco tire center battery replacement appointment)
5. Cross-sell awareness: display tire suggestions alongside batteries for combined visit efficiency

**Primary KPI:** Battery Purchase Completion Rate (%, members who complete battery purchase from fitment start) + Average Time to Add-to-Cart

**Use Case Example:** "My car battery died. I open costco.ca on my phone, search 'car battery', enter my 2020 Toyota Camry, see 3 compatible batteries with CCA ratings, one is in stock today at my nearest Costco. I add to cart, schedule battery installation for 2 PM, checkout in under 4 minutes. I arrive at Costco, battery is waiting, installed in 20 minutes."

**Phase Activation:** Phase 4B — Battery Sales Expansion

---

### Persona Priority Matrix

| Persona | Primary Goal | Product Priority | Entry Point | Primary KPI | Phase Activation |
|---------|--------------|------------------|-------------|------------|------------------|
| **1. Task/Mission-Driven** | Buy correct tires quickly without confusion | Fastest path to fitment and checkout, eliminate diversions; prefill vehicle info; strongest performance budget | Start Tire Search | Add-to-Cart Conversion Rate | Phase 1A, Phase 2 |
| **2. Searcher/Problem Solver** | Jump to desired product using keywords | High-accuracy search; intent detection; synonyms support; pre-fitment routing | Global Search | Search Success Rate | Phase 1A, Phase 2 |
| **3. Explorer/Category Browser** | Browse, compare, learn before deciding | Strong navigation; educational modules; filters; comparison tools; guidance | Shop Navigation | Engagement Depth, CTR | Phase 1A, Phase 2 |
| **4. Deal Hunter/Promo-Driven** | Find best deal that fits their car | Promo auto-routing + fitment; no dead-end pages; clear savings | Deals/Promotions | Fitment Start Rate | Phase 1A, Phase 2 |
| **5. Brand Loyalist** | Buy from trusted brand for their vehicle | Brand entry without fitment friction; fitment-first filtering; brand content; exclusive offers | Brand Entry | Validated Fitment Rate | Phase 1A, Phase 2 |
| **6. Services Seeker** | Schedule service easily without multiple tools | Unified scheduling; warehouse availability; service bundling; confirmation clarity | Services-First | Appointment Completion Rate | Phase 5A, Phase 5B |
| **7. Repeat Buyer** | Reorder fast with minimal effort | Buy Again shortcuts; prefilled vehicle; tire history; appointment suggestion | Ownership/Reorder | Reorder Success Rate | Phase 5B |
| **8. Battery Buyer** | Purchase a compatible automotive battery quickly and confidently | Vehicle-first battery fitment; in-stock confirmation; optional installation bundling; cross-sell with tires | Battery Search / Vehicle Selection | Battery Purchase Completion Rate | Phase 4B |

> **Note — Cross-Cutting Phases:** Phases 3A (Real-Time Inventory), 3B (Installation Scheduler), 3C (Tire Education), 3D (New Content Pages), and 4A (Analytics & Personalization) are applicable to **all personas (1–8)** but activate progressively per the delivery timeline (Q1–Q2 2027). These phases enhance the experience for every persona as they come online.

---

### Persona-to-Phase Delivery Mapping

The table below shows which personas are the primary intended beneficiaries at each delivery phase. Phases 3A–4A are cross-cutting and serve all personas once activated.

| Phase | Description | Primary Personas Served | Activation |
|-------|-------------|------------------------|------------|
| **Phase 1A** | Web Foundation — Tires Journey Reimagine | Personas 1, 2, 3, 4, 5 | Q2–Q3 2026 |
| **Phase 1B** | Azure to GCP Cloud Migration | All (infrastructure, no persona-specific features) | Q3 2026 |
| **Phase 2** | Native Mobile App Integration | Personas 1, 2, 3, 4, 5 | Q3–Q4 2026 |
| **Phase 3A** | Real-Time Inventory Availability | All Personas (1–8) — activates per delivery timeline | Q1 2027 |
| **Phase 3B** | Installation Scheduler Integration | All Personas (1–8) — activates per delivery timeline | Q1 2027 |
| **Phase 3C** | Tire Education Guide (3rd Party Content) | All Personas (1–8) — activates per delivery timeline | Q1 2027 |
| **Phase 3D** | New Content Pages (Season, EV, Driving Needs) | All Personas (1–8) — activates per delivery timeline | Q1 2027 |
| **Phase 4A** | Analytics & User Journey Tracking | All Personas (1–8) — activates per delivery timeline | Q2 2027 |
| **Phase 4B** | Battery Sales Expansion (US Market) | Persona 8 (Battery Buyer) | Q2 2027 |
| **Phase 5A** | Vehicle Profile Storage | Persona 6 (Services Seeker) — all personas benefit from vehicle persistence | Q3 2027 |
| **Phase 5B** | Order History & Rebate Hub | Personas 6 (Services Seeker), 7 (Repeat Buyer) — all personas benefit from order history | Q3 2027 |

---

## USER JOURNEYS

### User Journey 1: Direct Purchase (Task-Driven & Searcher Entry Points)
*Applies to: Persona 1 (Task/Mission-Driven), Persona 2 (Searcher/Problem Solver), Persona 4 (Deal Hunter via promo link)*

**Entry Triggers:**
- Direct link to tires landing page
- Search query for tire size, brand, or seasonal need
- Promo link leading to fitment flow

**Journey Steps:**

1. **Homepage / Entry Point**
   - Member arrives at tires.costco.com/.ca landing page OR mobile app Tires section
   - Displays: vehicle selector CTA, search box, promotional banner (if applicable)
   - State: Fresh start; no vehicle context

2. **Vehicle Selection / Fitment Entry**
   - Three options presented: License Plate Lookup, Vehicle Manual Selection, Tire Size Direct Entry
   - Member selects vehicle via preferred method
   - System validates fitment and stores vehicle context in session/device
   - State: Vehicle context established; member knows compatible tire sizes

3. **Search / Browse (Optional Branch)**
   - Member may refine search by brand, price, seasonal type
   - Applies filters or searches for specific product
   - System suggests compatible tires for selected vehicle
   - State: Intent clarified; results filtered

4. **Product Results Page / Product List Page**
   - Displays filtered tire results (4-8 options initially)
   - Shows: tire image, brand, model, price, availability at warehouse, fitment confirmation badge ("Fits your vehicle")
   - Warehouse/location selector visible (default: member's preferred warehouse or nearest)
   - Inline appointment slot indicator ("2 slots available today", "Book now")
   - Change Vehicle option available to adjust fitment

5. **Product Detail Page (Optional)**
   - Member clicks a tire result to see full specifications
   - Shows: detailed specs (tread rating, UTQG, reviews), warranty info, in-stock confirmation, installation pricing
   - Add-to-Cart prominent CTA; quantity selector; appointment option
   - Related products/upsell suggestions (batteries, accessories, services)

6. **Add to Cart**
   - Member selects tire and quantity
   - Tire, warehouse, and fitment context are persisted in cart
   - System confirms: tire, quantity, warehouse availability, price, estimated delivery date
   - Member can review cart or continue shopping

7. **Cart Page**
   - Displays items: tire(s) selected, quantity, warranty options, price, warehouse
   - Options: Change vehicle, change warehouse, edit quantity, apply membership discounts
   - Installation option visible: "Add installation? ($XX). Select time slot."
   - Checkout CTA (web: "Proceed to Checkout"; mobile: "Go to Payment")
   - State: Ready to finalize purchase

8. **Checkout / Payment**
   - Standard Costco payment flow (member login, shipping/warehouse selection, payment method)
   - Warehouse and fitment context pre-populated (no re-entry)
   - Order total includes tires, installation (if selected), applicable promotions
   - Completion: Order confirmation number generated

9. **Order Confirmation**
   - Displays: Order number, items, price, warehouse, estimated fulfillment date
   - Installation appointment details (if booked): Date, time, warehouse, what's included
   - Options: Schedule appointment (if not done), view order, manage account
   - Communication: Confirmation email sent

10. **Schedule Appointment (Optional — May Occur Earlier in Journey)**
    - If installation not selected at cart, member can schedule now
    - Warehouse/location pre-filled (from order)
    - Time slots displayed for next 14 days
    - Member selects preferred time
    - Appointment confirmation sent (email + SMS if opted)
    - State: Purchase complete, appointment confirmed

11. **Appointment Confirmation**
    - Displays: Appointment date/time, warehouse address, what to bring (vehicle keys, membership card)
    - Reminders: SMS/email sent 24h and 1h before appointment
    - Cancellation/modification options available

**Exit Points / Variations:**
- Member abandons at Search Results (if no suitable options found)
- Member abandons at Product Detail (if price or specs unsatisfactory)
- Member abandons at Checkout (payment friction, unexpected fees)
- Member completes tire purchase but skips appointment booking (handles scheduling later)
- Member changes warehouse mid-journey (system updates cart, availability, delivery timeline)

---

### User Journey 2: Brand-First Purchase (Brand Loyalist Entry Point)
*Applies to: Persona 5 (Brand Loyalist/Preference-Anchored Buyer)*

**Entry Triggers:**
- Click on brand link (Michelin, Bridgestone, Pirelli, etc.) in gallery
- Brand-specific promotional link
- Search for brand name (e.g., "Michelin tires")

**Journey Steps:**

1. **Homepage / Brand Gallery**
   - Member arrives at brand gallery page or searches for brand
   - Displays: Brand image, brand messaging, multiple tire options from that brand

2. **Select Warehouse or Tire Center**
   - Member selects preferred warehouse/location (if not already selected)
   - Context: Warehouse location established for inventory and appointment availability

3. **Click on Brand Image**
   - Member clicks brand card/link to enter brand-specific entry flow
   - System recognizes brand intent

4. **Brand Landing Page**
   - Displays: Brand overview, warranty/value proposition, available tire classes from that brand
   - Vehicle selector CTA: "What vehicle do you have?" (fitment question presented FIRST, not SKU list)
   - State: Member is committed to brand; now confirming fitment compatibility

5. **Select Vehicle / Confirm Fitment**
   - Three options: License Plate, Vehicle Manual Entry, Tire Size Entry
   - Member selects vehicle
   - System validates: "We have [N] Michelin tires that fit your [Vehicle]"
   - State: Fitment confirmed; brand + vehicle context established

6. **Click "Shop Our Tires"**
   - Member clicks CTA to see all compatible tires from selected brand
   - Transition to product browse

7. **Search Modal (Optional)**
   - Member may refine by tire class (all-season, winter, performance, EV, etc.)
   - Or continue to view all options

8. **Size Selection (if not pre-selected)**
   - If vehicle has multiple compatible sizes, member selects from compatible options
   - System pre-filters: "These Michelin tires fit your [Vehicle]"

9. **View Tires (Brand Results)**
   - Displays: All Michelin tires compatible with selected vehicle and size
   - Shows: Tire model, performance tier, price, warranty, in-stock badge ("Available at [Warehouse]")
   - Organized by performance level (e.g., Defender, Crossclimate, Pilot Sport)

10. **Search Results Page / Product List Page**
    - Filters available: Price, tire class, performance rating, customer reviews
    - Each result shows: Michelin model, specs, price, availability, "Add to Cart"
    - Warehouse indicator: "In stock at [preferred warehouse]", option to check nearby
    - Appointment availability inline: "2 slots available today"
    - Change Warehouse option to compare availability across locations

11. **Product Detail Page**
    - Member clicks a Michelin tire for detailed specs
    - Shows: Tire specifications, brand positioning, warranty details, reviews from Michelin purchasers
    - Installation pricing specific to this tire
    - Add-to-Cart CTA; quantity selector; warranty options

12. **Add to Cart**
    - Member adds selected Michelin tire to cart
    - Brand, model, size, warehouse, and fitment context persisted

13. **Cart Page**
    - Displays: Michelin tire selected, quantity, warranty, price, warehouse
    - Installation option: "Add installation?" with time slot selector for preferred warehouse
    - Change Warehouse, Change Vehicle options available
    - Checkout CTA

14. **Checkout / Payment**
    - Standard Costco checkout
    - All context (Michelin product, fitment, warehouse, installation time) pre-populated

15. **Order Confirmation & Appointment Booking**
    - Order confirmed with Michelin tire details
    - Option to schedule or modify appointment time
    - Appointment confirmation sent

**Exit Points / Variations:**
- Member switches brands mid-journey (system updates results to new brand)
- Member decides fitment incompatibility makes selected brand unsuitable (can backtrack to vehicle selector)
- Member abandons if preferred Michelin model is out of stock at selected warehouse (can change warehouse or select alternate Michelin)

---

### User Journey 3: Services-First / Quick Service (Services Seeker Entry Point)
*Applies to: Persona 6 (Maintenance Seeker/Car-Care Planner) | **Phase Activation: Phase 5A, Phase 5B***

**Entry Triggers:**
- Click "Schedule Appointment" in Tire Center services menu
- Click "Schedule Tire Service" from warehouse page
- Click "Services" tab in mobile app and select tire service

**Journey Steps:**

1. **Homepage**
   - Member arrives at homepage or warehouse services page
   - Services section visible with tire-related options (tire installation, rotation, flat repair, balance)

2. **Select Warehouse or Tire Center**
   - Member selects preferred warehouse location
   - System filters available appointment times and services for that location
   - State: Warehouse context established

3. **Proceed to Appointment Scheduling**
   - Member selects "Schedule Appointment" or "Tire Installation" service
   - System presents appointment scheduling interface

4. **Confirm Tire Selection or Vehicle Details**
   - Member's vehicle and tire history auto-populated from account (if available)
   - System shows: "Last service on [date]: Tire rotation. Vehicle: [Make/Model]. Tire: [Size/Brand]"
   - Member confirms current vehicle and tire details or updates if changed
   - If no history exists: Member selects vehicle manually

5. **Determine Service Need / Tire Selection**
   - System presents options:
     - **Tire Installation:** Member needs to purchase/bring tires (leads to product selection flow)
     - **Tire Rotation:** Already owns tires; just scheduling rotation
     - **Flat Repair:** Quick flat tire repair appointment
     - **Balance/Alignment:** Additional services
   - Member selects primary service need

6. **Select Service Type** (if Tire Installation selected)
   - System asks: "Do you have tires ready, or need to purchase?" 
   - **Option A:** "I need to purchase tires" → Brief fitment flow (vehicle pre-selected) → Product selection → Continue to step 7
   - **Option B:** "I have tires ready" → Skip to step 7

7. **Schedule Appointment**
   - System displays available time slots for next 14 days at selected warehouse
   - Shows: Date, time, estimated duration, total service cost (if applicable)
   - For tire installation: Cost includes tire (if purchased) + installation labor + balance
   - Member selects preferred time slot
   - State: Service scheduled

8. **Appointment Confirmation**
   - Displays: Appointment date/time, warehouse address, service type, vehicle details, cost
   - What to bring: Vehicle keys, membership card
   - Confirmation actions:
     - **SMS notification:** Instant SMS confirmation with appointment details and warehouse address
     - **Email confirmation:** Detailed email with all details, cancellation/modification link
     - **Calendar integration:** Option to add to calendar (Google, Outlook, Apple)

9. **Appointment Confirmation Page / Thank You**
    - Displays: "Your appointment is confirmed!"
    - Shows: Appointment number, date/time, warehouse address, phone number, cancellation policy
    - Options: Modify appointment, view account, manage preferences
    - Reminders: "You'll get SMS reminder 24h before and 1h before appointment"

**Exit Points / Variations:**
- Member selects service, realizes no appointment slots available at preferred warehouse today/tomorrow → Can choose different warehouse or defer appointment
- Member needs new tires but hasn't decided on product → Leads into User Journey 1 (product selection) before confirming appointment
- Member cancels during scheduling (changes mind, realizes inconvenient time)
- Member books appointment successfully but doesn't proceed to tire purchase (if needed later)

---

### User Journey 4: Repeat Purchase / Reorder (Repeat Buyer Entry Point)
*Applies to: Persona 7 (Repeat Buyer/Owner) | **Phase Activation: Phase 5B***

**Entry Triggers:**
- Click "My Account" → "My Vehicles" or "My Tires"
- Click "Reorder" link from past order
- Email reminder: "Time for a tire check-up on your [Vehicle]?"
- Mobile app "Reorder" tile based on purchase history

**Journey Steps:**

1. **Homepage**
   - Member arrives at homepage or warehouse services page
   - Homepage displays personalized section: "Reorder Your Tires" or "Time for a Check-Up?"

2. **Navigate to My Account**
   - Member clicks "My Account" in header (web) or Account tab (mobile)
   - Account landing page displays options: My Vehicles, My Orders, My Tire History

3. **Select/View Order**
   - Member clicks "My Vehicles" or "My Tires"
   - System displays: Vehicle list with associated tire purchase history
   - Example: "2023 Toyota Camry - Last purchase: Michelin Defender Winter 225/45R18 on October 15, 2023"
   - Member selects vehicle they want to reorder for

4. **Buy Again**
   - Member clicks "Buy Again" or "Reorder This Tire"
   - System pre-loads previous purchase details (tire model, size, quantity, warehouse from last purchase)
   - Displays: "You purchased [Tire Name] [Size] on [Date]. Ready to reorder?"

5. **Confirm Tire Selection or Vehicle Details**
   - System asks: "Is this tire still correct for your [Vehicle]?"
   - Member confirms or can:
     - **Change tire:** "Browse similar tires" or "Start new search"
     - **Change quantity:** Adjust number of tires (4 tires, 2 tires, 1 tire)
     - **Update vehicle:** "This vehicle has been modified" → update tire size
   - Default action: Proceed with same tire (highest conversion path)

6. **Review Cart**
   - Previous tire model, size, quantity auto-populated
   - Warehouse: Default to member's previous warehouse, with option to change
   - Availability confirmation: "Available at [Warehouse]. [N] in stock."
   - Price: Current price displayed (may differ from last purchase)
   - Discount eligibility: Any available member discounts applied automatically
   - Installation option visible: "Add tire installation? ($XX). Available times: [Show slots]"

7. **Optional: Schedule Installation**
   - Member can opt to schedule installation concurrently
   - Warehouse pre-selected (from last order)
   - Available appointment times displayed
   - Time slot selection leads to appointment booking

8. **Checkout**
   - Warehouse, vehicle, tire selection, quantity all pre-populated (no re-entry)
   - Member reviews order summary
   - Payment method auto-filled (if using saved payment)
   - "Complete Purchase" CTA

9. **Order Confirmation**
   - Displays: Order confirmation number, items, price, warehouse, estimated fulfillment date
   - Vehicle and tire details confirmed
   - If installation booked: Appointment date/time displayed

10. **Appointment Confirmation (if scheduled)**
    - If appointment booked in step 7: Confirmation page or email shows appointment details
    - If not booked: "Schedule installation? [Schedule Now]" link available
    - Reminders sent (SMS/email) 24h and 1h before appointment

**Exit Points / Variations:**
- Member finds previous tire is out of stock → System suggests similar alternatives
- Member wants different tire than last purchase → "Browse similar" leads into search/browse flow
- Member completes purchase but delays appointment scheduling (links available in order confirmation)
- Member updates vehicle details (tire size changed) → System recalculates fitment, validates size
- Member abandons if price increase seems too high (no safety mechanism; member can add to cart as saved item)

---

### User Journey 5: Discovery & Compare (Explorer Entry Point)
*Applies to: Persona 3 (Explorer/Category Browser) | **Phase Activation: Phase 1A, Phase 2***

**Entry Triggers:**
- Click "Shop" → "Tires & Auto" in site navigation
- Click on category tile (e.g., "All-Season Tires", "Winter Tires")
- Landing on tires.costco.com/.ca with no prior search intent

**Journey Steps:**

1. **Homepage / Category Entry**
   - Member arrives at Shop → Tires & Auto category hub or tires landing page
   - Displays: Category tiles (All-Season, Winter, Summer, Performance, EV, Truck/SUV), featured brands ribbon, promotion slider
   - No fitment context yet; member is in browse/explore mode
   - State: Fresh start; discovery intent

2. **Browse Categories**
   - Member clicks into a tire category (e.g., "Winter Tires")
   - Displays: Category description, sub-categories (Studded, Studless, Performance Winter), featured tires, educational module ("Why winter tires?")
   - Filters visible on left/top: Brand, price range, performance rating, customer reviews, rim size
   - State: Category context established; member starting to narrow options

3. **Educational Content Engagement (Optional)**
   - Member clicks educational module ("How to choose winter tires", "What is UTQG?", "Tire size guide")
   - Displays: Educational overlay or content hub page with guides, diagrams, video clips (15-30 seconds)
   - Content links back into product browse with informed filter selections
   - State: Member gaining confidence in purchase decision

4. **Apply Filters & Browse Results**
   - Member applies filters (price range, brand, performance tier, seasonal type)
   - Displays: Filtered tire grid with images, brand, model, rating, price, availability indicator
   - Each result shows: "Fits most vehicles — confirm yours for exact sizing"
   - Comparison checkbox visible on each product card (select up to 3)
   - State: Narrowing field; building consideration set

5. **Comparison Table (Optional)**
   - Member selects 2-3 tires for side-by-side comparison
   - Displays: Comparison table with: price, tread rating, UTQG grade, warranty, performance tier, reviews, availability
   - "Add to Cart" available from comparison table (proceeds to fitment validation)
   - State: Decision narrowed to 1-2 finalists

6. **Fitment Validation**
   - Member selects a tire to add to cart or clicks "Check if this fits my vehicle"
   - System presents vehicle selection: License Plate, Manual Entry, or Tire Size
   - Member selects vehicle
   - System validates: "This tire is compatible with your [Vehicle]" or offers compatible alternatives
   - State: Fitment confirmed; member ready to purchase

7. **Product Detail Page**
   - Member views full product details for selected tire
   - Shows: Full specs (tread rating, UTQG, warranty, reviews, EV suitability if applicable), installation pricing
   - Educational tooltips available inline on spec labels
   - Warehouse availability + appointment slots displayed
   - Add-to-Cart CTA prominent

8. **Add to Cart / Cart / Checkout / Appointment**
   - Standard checkout flow (same as Journey 1 steps 6-11)

**Exit Points / Variations:**
- Member engages with educational content and does not purchase in session (high value for future return)
- Member narrows via comparison but returns to browse more options
- Member applies fitment validation and finds selected tire incompatible → System offers compatible alternatives within same category/brand
- Member abandons at comparison (price or specs don't meet expectations)

**Traceability:** Goals 1, 2 | Persona 3 (Explorer/Category Browser) | Supports FR7, FR9, FR13

---

### User Journey 6: Battery Purchase (Battery Buyer Entry Point)
*Applies to: Persona 8 (Battery Buyer) | **Phase Activation: Phase 4B***

**Entry Triggers:**
- Search query for "car battery", "battery replacement", "automotive battery"
- Navigate to "Batteries" section on tires.costco.ca
- Promotional link (seasonal battery deal, email campaign)
- Cross-sell prompt during tire shopping ("Also need a battery?")

**Journey Steps:**

1. **Homepage / Entry Point**
   - Member arrives at the Batteries section on tires.costco.ca or navigates from tires homepage
   - Displays: Vehicle selection CTA, featured battery promotions, brand options (Kirkland Signature, Interstate, etc.)
   - State: Fresh start; existing vehicle context loaded if member is already logged in from tire session

2. **Vehicle Selection / Fitment Entry**
   - Three options presented: License Plate Lookup, Vehicle Manual Selection (Year/Make/Model/Trim), or Battery Size Direct Entry (Group Size, e.g., 35, 24F)
   - System validates vehicle and determines compatible battery group size(s)
   - State: Vehicle context established; compatible battery group sizes identified

3. **Product Results Page / Product List Page**
   - Displays compatible battery options (typically 2–4 per vehicle)
   - Shows: Battery image, brand, group size, CCA (Cold Cranking Amps) rating, reserve capacity, warranty years, price
   - Warehouse inventory badge: "In Stock at [Warehouse] — Available today" or "Out of Stock"
   - Optional filters: Battery type (lead-acid, AGM, lithium), brand, warranty period
   - Compatibility badge on each tile: "Fits your [Year Make Model]"

4. **Product Detail Page (Optional)**
   - Shows: Full battery specifications (group size, CCA, reserve capacity, dimensions, terminal type)
   - Warranty details and Costco coverage
   - Installation pricing: "Battery replacement service: $XX at Costco Tire Center"
   - Compatibility confirmation: "Confirmed fit for your [Year Make Model Trim]"
   - Add-to-Cart CTA; quantity selector

5. **Add to Cart**
   - Battery, warehouse, and vehicle context persisted in cart
   - System confirms: battery model, quantity, warehouse availability, price, estimated availability date
   - Cross-sell prompt: "While you're here, your tires are [X] years old. Need a tire check?"

6. **Cart Page**
   - Displays: Battery selected, quantity, warranty, price, warehouse
   - Installation option: "Add battery installation? ($XX). Select time slot."
   - Combined appointment option (if tires also in cart): "Book one appointment for both battery and tire service"
   - Checkout CTA

7. **Checkout / Payment**
   - Standard Costco payment flow (SSO pre-population; saved payment methods)
   - Warehouse and fitment context pre-populated from earlier in journey
   - Order confirmation number generated

8. **Order Confirmation**
   - Displays: Order number, battery product, price, warehouse, availability date
   - Installation appointment details (if booked)
   - Communication: Confirmation email sent with product details and service instructions

9. **Schedule Appointment (Optional)**
   - Integrated appointment booking for battery installation at Costco tire center
   - Warehouse and vehicle details pre-filled; no re-entry required
   - Time slots for battery installation displayed (typically 30–60 min service)
   - Appointment confirmation sent (email + SMS if opted)

**Exit Points / Variations:**
- Member finds battery out of stock at preferred warehouse → system suggests nearest warehouse with available stock or estimated restock date
- Member opts for DIY installation (skips appointment scheduling; product purchased for self-installation)
- Member started tire shopping and is cross-sold to battery (combined appointment for both services at single warehouse visit)
- Member abandons if price is higher than expected (return path via saved cart or email reminder)

**Traceability:** Goals 1, 2, 4 | Persona 8 (Battery Buyer) | Supports FR7 (entry point 8), FR21

---

## FUNCTIONAL REQUIREMENTS

### Core Functional Capabilities

#### **FR1: Vehicle Selection & Fitment Validation**
- **Requirement:** Members can select their vehicle by license plate, manual make/model/year/trim, or direct tire size entry, and receive only compatible tire options validated against manufacturer fitment specifications
- **Entry Methods:** 
  - License Plate lookup (integration with Mitchell1 VIN decode service)
  - Manual vehicle selection (Year/Make/Model/Trim hierarchical selector)
  - Direct tire size entry (format: ###/##R##, e.g., 225/45R18)
- **Validation Logic:** Accepted tire sizes for selected vehicle must be pre-validated against manufacturer specifications
- **Output:** Confirmed vehicle context (make, model, year, compatible sizes) persisted in session/device for entire journey
- **Error Handling:** Invalid tire sizes rejected with guidance ("This size is not compatible with your vehicle; valid sizes are: 225/45R18, 235/40R18")
- **Applicable Personas:** All 8 personas (Persona 8 uses same vehicle lookup mechanism for battery fitment)

**Traceability:** Goals 2, 3 | Personas 1, 2, 3, 4, 5, 6, 7, 8

---

#### **FR2: Tire Size Format Validation**
- **Requirement:** Members can enter a tire size in standard format (###/##R##) and receive validation feedback if the format is invalid or the size is incompatible with their selected vehicle
- **Accepted Format:** Standard tire size format (###/##R##, e.g., 225/45R18, P225/45R18)
- **Validation Triggers:** 
  - Direct size entry (tires.costco.com/search?size=225/45R18)
  - Tire selection confirmation (before add-to-cart)
- **Error Messages:** Clear guidance for invalid formats ("Enter size as: Width/AspectRatio R Diameter, e.g., 225/45R18")
- **Correction Options:** Size guide overlay with visual tire size diagram and manual entry helper
- **Applicable Personas:** Personas 1, 2, 3

**Traceability:** Goals 1, 4 | Personas 1, 2, 3

---

#### **FR3: Warehouse-Specific Inventory Display**
- **Requirement:** Members can see real-time warehouse availability status and delivery timeline for each tire at their selected warehouse; availability data refreshed every 15 minutes minimum
- **Display Mechanism:** 
  - **Product Results Page:** "Available at [Warehouse Name] ([#] in stock). Delivery by [Date]."
  - **Product Detail Page:** Stock status by warehouse (member's selected warehouse prominent, nearby warehouses listed)
  - **Cart:** "In stock at [Selected Warehouse]. Confirm warehouse?"
- **Inventory Status Options:**
  - "In Stock" ([#] available) with delivery/pickup date
  - "Limited Stock" ([#] available) — visual warning (yellow)
  - "Out of Stock" — disabled option or grey-out
  - "Expected [Date]" — for backorder items
- **Warehouse Switching:** Member can change warehouse mid-journey; system updates inventory status, delivery date, installation availability
- **Real-Time Accuracy:** Inventory data refreshed every 15 minutes (tolerance for high-velocity sales)
- **Applicable Personas:** All 8 personas (Persona 8 — Battery Buyer — uses same warehouse inventory display for battery availability)

**Traceability:** Goals 1, 2, 3 | Personas 1, 2, 3, 4, 5, 6, 7, 8

---

#### **FR4: Appointment Availability Display During Tire Selection**
- **Requirement:** Members can view available installation appointment slots at their selected warehouse during the tire selection process without navigating to a separate scheduling tool
- **Display Mechanism:**
  - **Product Results Page:** "Appointments available: 2 slots today, 5 slots this week" (expandable to show actual times)
  - **Product Detail Page:** Calendar view showing available appointment times for next 14 days at selected warehouse
  - **Cart Page:** Appointment selector showing available times when member indicates interest in installation
- **Availability Data:** Integrated with warehouse tire center scheduling system; real-time slot availability
- **Installation Service Bundling:** System bundles tire + installation + balance into single appointment; member selects time during tire selection flow
- **Warehouse-Specific:** Availability shown for member's selected warehouse; option to check nearby locations
- **Applicable Personas:** Personas 1, 4, 6, 7 (particularly services-focused)

**Traceability:** Goals 1, 4 | Personas 1, 4, 6, 7

---

#### **FR5: Context Preservation Across Channels & Sessions**
- **Requirement:** Members can switch between web and mobile channels mid-journey without re-entering vehicle selection, fitment preferences, cart contents, or warehouse selection; context persists across sessions for logged-in members up to 30 days and guest sessions up to 24 hours
- **Acceptance Criterion:** Vehicle context, fitment preferences, and cart items are available on the alternate channel within 10 seconds of a channel switch for logged-in members; zero context re-entry required
- **Context Elements:**
  - Vehicle selection (make, model, year, size)
  - Fitment preferences (tire size, seasonal type, performance level)
  - Cart contents (selected tires, quantity, warehouse)
  - Warehouse selection
  - Appointment preferences
- **Persistence Mechanism:**
  - **Logged-in members:** Context stored in Costco member account (cloud-synced)
  - **Guest members:** Context stored in device-local session storage + anonymous session ID
  - **Cross-device:** When member logs in on different device, account context loads (vehicle history, past purchases, preferences)
- **Session Timeout:** Context persists for 30 days for logged-in members; 24 hours for guest sessions
- **Cart Sync:** Switching between web and mobile app shows same cart contents (if logged in)
- **Applicable Personas:** All 7 personas (foundation)

**Traceability:** Goals 2, 3 | Personas 1, 2, 3, 4, 5, 6, 7

---

#### **FR6: Integrated Appointment Scheduling Without Redirects**
- **Requirement:** Members can book tire installation appointments within the tire selection and checkout flow without being redirected to any external system, third-party booking service, or separate application
- **Integration Model:**
  - User selects tire(s) and reaches cart/checkout
  - Within same flow, user can immediately select appointment time slot (no redirect, no new page load)
  - Appointment confirmed as part of order confirmation
- **No External Redirects:** System must never redirect member to separate warehouse scheduling tool, third-party booking service, or disconnected app
- **One-Flow Booking:** Appointment booking visible/available at multiple points in journey (product page, cart, checkout, order confirmation) but never fragmenting into separate flows
- **Appointment Confirmation:** Integrated into order confirmation email/page; single confirmation covers both tire purchase and installation appointment
- **Applicable Personas:** Personas 1, 4, 6, 7, 8 (Persona 8 — Battery Buyer — requires integrated battery installation scheduling)

**Traceability:** Goals 1, 4 | Personas 1, 4, 6, 7, 8

---

#### **FR7: Multi-Entry Point Support (8 Entry Patterns)**
- **Requirement:** System must support and optimize all eight entry point patterns without context loss
- **Entry Points:**
  1. **Start Tire Search:** Dedicated landing page with fitment CTA
  2. **Global Search:** Costco site search with tire intent detection + autocomplete
  3. **Shop Navigation:** Category browsing (Tires & Auto section)
  4. **Deals/Promotions:** Promo banners/links leading to fitment flow (no dead-end promo pages)
  5. **Brand Entry:** Brand gallery links routing through fitment validation
  6. **Services-First:** Warehouse service pages with inline appointment booking
  7. **Ownership/Reorder:** "Buy Again" links from purchase history
  8. **Battery Search:** Dedicated battery section on tires.costco.ca with vehicle-first fitment and cross-sell integration (Phase 4B)
- **Implementation:** Each entry point has optimized landing page or flow start; all converge on same core experience (fitment → search → product → cart → checkout) with entry-point-specific customizations
- **No Entry Point Dead-Ends:** Every entry point leads to complete purchase flow; no entry point leaves member stranded or confused
- **Applicable Personas:** All 8 personas (one per primary entry point)

**Traceability:** Goals 1, 2 | Personas 1, 2, 3, 4, 5, 6, 7, 8

---

#### **FR8: Real-Time Inventory Accuracy**
- **Requirement:** Members can complete a tire purchase and book an installation appointment with confidence that the tire is in stock at their selected warehouse; no appointment cancellations due to post-purchase inventory inaccuracies
- **Accuracy Requirements:**
  - Inventory displayed at product browsing stage must match inventory at checkout
  - If tire runs out of stock between browsing and cart, system must alert member before completing purchase
  - If tire runs out of stock after purchase, warehouse must confirm availability before appointment date or notify member immediately
- **Inventory Sync:** Warehouse inventory management system must feed real-time stock data to web/mobile interfaces (refresh: every 15 minutes minimum, ideally real-time)
- **Appointment Guarantees:** Appointment can only be booked if tire is confirmed in stock at that warehouse for that day
- **Cancellation Prevention:** No post-purchase inventory inaccuracies leading to appointment cancellations or tire unavailability
- **Fallback Mechanism:** If inventory discrepancy detected post-purchase, system alerts member and offers: reschedule appointment, substitute tire, full refund
- **Applicable Personas:** All 8 personas (Persona 8 — Battery Buyer — same inventory accuracy requirement applies to battery stock)

**Traceability:** Goals 1, 2, 3 | Personas 1, 2, 3, 4, 5, 6, 7, 8

---

#### **FR9: Smart Search & Intent Detection**
- **Requirement:** Members can search using tire-specific vocabulary (sizes, brands, seasonal terms, vehicle types, performance levels) and receive fitment-relevant results without navigating menus
- **Acceptance Criterion:** ≥95% of valid tire size queries return fitment-compatible results; brand search accuracy ≥90%; misspelling correction applies when edit distance ≤2 characters; search results return in <500ms (p95)
- **Search Capabilities:**
  - **Tire size search:** Accept all common tire size formats and variations
  - **Brand search:** Support brand name + model searches (e.g., "Michelin Defender", "Bridgestone Ecopia")
  - **Seasonal search:** Keyword routing ("winter", "summer", "all-season") to appropriate tire categories
  - **Need-based search:** Keywords like "performance", "comfort", "towing", "off-road" route to tire classes
  - **Vehicle-based search:** "EV tires", "Model 3 tires", "truck tires" route to vehicle-specific tire selection
- **Autocomplete & Suggestions:**
  - As member types, show suggestions (brands, tire models, size formats, seasonal types)
  - Recent searches displayed after first 3 keystrokes
  - Brand suggestions if fitment context available ("You're looking for [Vehicle], here are popular brands: Michelin, Bridgestone...")
- **Search Result Ranking:** Relevance-based ranking; sponsored products (promotions) flagged but not overweighting organic relevance
- **Search Error Handling:** Suggested corrections for misspellings ("Did you mean Bridgestone?"); fallback to browse if no results
- **Applicable Personas:** Personas 1, 2, 4

**Traceability:** Goals 1, 2 | Personas 1, 2, 4

---

#### **FR10: Multi-Channel Cart Sync**
- **Requirement:** Members can switch between web and mobile channels and find their cart contents, vehicle selection, and fitment preferences unchanged without re-entry
- **Acceptance Criterion:** Cart sync completes within 5 seconds of channel switch for logged-in members; guest cart converts to account cart within 10 seconds of login on an alternate device; cart state persists across connection interruptions (read-only cache restore on reconnect)
- **Implementation:**
  - Cart stored in cloud (member's account) for logged-in users
  - When member switches from web to mobile (or vice versa), cart loads automatically with all contents
  - Updates to cart on one channel reflect immediately on all channels
  - Example: Member adds tire on desktop, opens mobile app, tire appears in cart
- **Guest Cart Behavior:** Guest carts stored locally and converted to account cart if member logs in
- **Device Persistence:** Cache cart locally on device for offline access (read-only); sync on connection restore
- **Applicable Personas:** All 7 personas

**Traceability:** Goals 2, 3 | Personas 1, 2, 3, 4, 5, 6, 7

---

#### **FR11: Vehicle Profile & Order History**
- **Requirement (Phase 5):** Returning members can save multiple vehicles to their account and reorder previous tires in ≤2 minutes using one-click "Buy Again" from their tire purchase history
- **Vehicle Profile Storage:**
  - Save multiple vehicles per member account
  - Store: Make, model, year, tire size, purchase history for each vehicle
  - Auto-populate fitment for returning purchases (Persona 7 — Repeat Buyer)
- **Order History Retrieval:**
  - "My Tires" section shows past tire purchases with vehicle associations
  - "Buy Again" links pull previous purchase details for one-click reorder
  - Maintenance reminders: "Time to replace tires on your [Vehicle]? Last purchased [Date]"
- **Service History:** Track rotation, repair, installation appointments per vehicle
- **Applicable Personas:** Persona 7 (Repeat Buyer), secondary benefit to Personas 6 (Service Seeker)

**Traceability:** Goals 3 | Personas 6, 7 | **Phase 5 Feature**

---

### Customer Journey Functional Support

#### **FR12: Mobile-Native Performance**
- **Requirement (Phase 2):** Members can browse, search, select, and purchase tires through a native iOS/Android app experience with app-grade performance — not a web-view wrapper — delivering cold start in <1.5 seconds and page transitions in <1 second
- **Performance Targets:**
  - App launch time: <1.5 seconds (cold start)
  - Page load time: <1 second (typical page transitions)
  - Interaction response: <100ms (touch → visual feedback)
- **Native Features:**
  - Native navigation patterns (iOS tab bar, Android bottom nav)
  - Native search with voice input support
  - Local device storage for vehicle/fitment context
  - Offline mode support (browse cached products, view past orders)
  - Integration with device camera (license plate scanning for VIN lookup)
- **Applicable Personas:** All 7 personas (Phase 2 launch)

**Traceability:** Goals 1, 2 | Personas 1, 2, 3, 4, 5, 6, 7 | **Phase 2 Feature**

---

#### **FR13: Educational Content Integration**
- **Requirement (Phase 3C):** Members can access expert tire guides, sizing help, and seasonal buying guidance within the tire shopping experience without leaving the purchase flow
- **Content Types:**
  - Tire sizing guide (interactive with visual diagrams)
  - Seasonal tire buying guide (when to buy winter/summer/all-season)
  - Performance tier explanations (comfort vs. performance vs. all-season trade-offs)
  - EV-specific tire content (range impact, weight handling, specialized tire features)
  - Brand positioning and warranty explanations
  - Tire maintenance tips (rotation, pressure, wear indicators)
  - Installation service explanations (what's included, why it matters)
- **Content Placement:**
  - Overlays on product pages (e.g., "What does UTQG mean?" tooltip)
  - Separate "Tire Buying Guide" content hub
  - Contextual tooltips during product selection (e.g., "All-season? Here's why it might not be ideal for winter...")
  - Video content (15-30 second clips on tire installation, sizing, seasonal needs)
- **Content Provider:** Third-party content partnership (supplier TBD in Phase 3 planning)
- **Acceptance Criterion:** Educational content CTR from product pages ≥15% of product page views; educational hub average session duration >2 minutes; tooltips displayed on-demand within 200ms of trigger
- **Applicable Personas:** Personas 2 (Problem Solver), 3 (Explorer), 5 (Brand Loyalist)

**Traceability:** Goals 1, 2 | Personas 2, 3, 5 | **Phase 3C Feature**

---

#### **FR14: Promotional Auto-Routing**
- **Requirement:** Members arriving from any promotional link land directly in a fitment-based tire search flow showing only promotion-eligible results; no promotional link dead-ends on a standalone promotional page
- **Promo Link Behavior:**
  - Promo banner ("$50 Off Winter Tires") → Member lands on fitment selector (not generic sale page)
  - After fitment selection → Filtered results show only promotion-eligible tires
  - Price display shows promotion applied ("Regular $150 → You pay $100 with promotion")
  - Cart shows promotion applied before checkout
- **Promotion Tracking:** System tracks promo code/campaign source through entire journey
- **Promotion Failure Handling:** If member's selections don't qualify for promotion, clear messaging ("This promotion applies to Michelin winter tires; this tire is 3-season")
- **Applicable Personas:** Persona 4 (Deal Hunter)

**Traceability:** Goals 1, 4 | Persona 4 | **Phase 1+ Feature**

---

#### **FR21: Battery Product Catalog & Fitment (Phase 4B)**
- **Requirement:** System must provide a battery product catalog with vehicle-specific fitment validation, enabling Persona 8 (Battery Buyer) to find compatible automotive batteries using the same vehicle-lookup infrastructure as the tires journey
- **Vehicle Lookup:** License Plate, Manual Year/Make/Model/Trim, or Battery Group Size direct entry
- **Compatibility Data:** Battery fitment database mapping vehicles to compatible group sizes (e.g., Group 35, 24F, 48)
- **Product Data Required:**
  - Battery group size, CCA (Cold Cranking Amps), reserve capacity (minutes), dimensions, terminal type
  - Battery type (lead-acid, AGM, EFB, lithium)
  - Warranty (months): manufacturer warranty + Costco extended coverage
  - Brand and model (Kirkland Signature, Interstate, etc.)
  - Price, warehouse-specific inventory, installation pricing
- **Display Placement:** Dedicated "Batteries" section on tires.costco.ca; cross-sell tile on tire product pages
- **Cross-Sell Integration:** During tire shopping, system can surface a battery suggestion: "Also consider a battery check — your vehicle's battery is [age-range]"
- **Installation Bundling:** Battery installation appointment bookable in the same flow as tire installation (Phase 3B infrastructure reused)
- **Applicable Personas:** Persona 8 (Battery Buyer); cross-sell benefit to Personas 1, 4, 6, 7

**Traceability:** Goals 1, 2, 4 | Persona 8 | **Phase 4B Feature**

---

## NON-FUNCTIONAL REQUIREMENTS

### Performance Requirements

**NR-Perf-1: Page Load Time**
- **Requirement:** Load times of ≤2 seconds for web; <1.5s for mobile native app (cold start); <1s for typical page transitions
- **Measurement:** 95th percentile load time as measured from member's device
- **Includes:** HTML parsing, CSS/JS execution, asset loading, API response time
- **Breakdown Target:** (Phase 1A/2) Initial page load <2s; subsequent pages <1s
- **Monitoring:** Continuous Real-time User Monitoring (RUM) via Application Insights

**NR-Perf-2: Search Query Response**
- **Requirement:** Search query results returned in <500ms (p95)
- **Applies to:** Product search, size suggestions, brand autocomplete, appointment slot queries
- **Baseline:** Measured against current system (establish baseline in Phase 1 discovery)

**NR-Perf-3: Inventory Query Response**
- **Requirement:** Real-time warehouse inventory queries return in <2 seconds (p95)
- **Applies to:** Warehouse availability check, appointment slot availability
- **Constraint:** System must refresh inventory data every 15 minutes minimum; <10 minute latency acceptable

---

### Availability & Reliability Requirements

**NR-Avail-1: System Uptime**
- **Requirement:** 99.99% uptime on the app and website (tier 1 SLA)
- **Calculation:** Measured monthly; no more than 4.3 minutes downtime/month
- **Excludes:** Planned maintenance windows (must be scheduled outside peak hours; max 2 hours/month)
- **Applicable to:** tires.costco.com, tires.costco.ca, Costco mobile app Tires section

**NR-Avail-2: Graceful Degradation**
- **Requirement:** If any backend service fails (inventory, scheduling, payments), core browse and checkout functions remain available at ≥80% of normal capacity; no complete service outage due to a single-service failure
- **Behavior:**
  - If inventory service down: Display "Checking availability..." and allow purchase with manual fulfillment confirmation later
  - If scheduling service down: Allow tire purchase; offer appointment booking via email/call
  - If search service down: Fall back to category browsing
- **Measurement:** Degraded mode is confirmed when core checkout flow succeeds for ≥80% of attempted transactions during backend service failure
- **Member Communication:** Transparent messaging ("We're currently experiencing delays checking warehouse availability; you can still purchase and we'll confirm availability within 1 hour")

**NR-Avail-3: Disaster Recovery**
- **Requirement:** Backup systems and failover mechanism to restore service within 30 minutes of failure
- **RTO (Recovery Time Objective):** <30 minutes to restore full functionality
- **RPO (Recovery Point Objective):** <5 minutes data loss tolerance

---

### Security & Data Protection Requirements

**NR-Sec-1: Data Loss Prevention**
- **Requirement:** No instance of data loss or any such security issues
- **Implementation:**
  - Encrypted data transmission (TLS 1.3 for all data in transit)
  - Encrypted data at rest (AES-256 for databases)
  - Regular backups (hourly snapshots, 30-day retention)
  - Automated failover with no manual intervention required
- **Member Data:** Never logged or exposed; PII protected under GDPR/CCPA/PIPEDA

**NR-Sec-2: Authentication & Authorization**
- **Requirement:** Secure member authentication integrated with Costco member account (SSO)
- **Implementation:**
  - OAuth 2.0 / OpenID Connect integration with Costco identity platform
  - Two-factor authentication (2FA) optional for high-value transactions (purchases >$500)
  - Session timeout: 30 minutes for tier-2 transactions (tire purchase), 15 minutes for tier-3 (payment entry)
- **Role-Based Access Control:** Separate permissions for warehouse staff (appointment management), admins (promo management), members (purchasing)

**NR-Sec-3: Payment Security**
- **Requirement:** PCI DSS Level 1 compliance for payment processing
- **Implementation:**
  - Tokenized payment (no raw card data stored on Costco systems)
  - Integration with PCI-compliant payment processor (e.g., Stripe, Adyen)
  - Fraud detection (real-time transaction monitoring, 3D Secure for high-risk transactions)

**NR-Sec-4: API Security**
- **Requirement:** All API endpoints authenticated and rate-limited
- **Implementation:**
  - API authentication via OAuth 2.0 bearer tokens or API keys
  - Rate limiting: Max 100 requests/minute per member, max 1000 requests/minute per API key
  - CORS policy: Only authorized Costco domains allowed
  - Input validation: All API inputs sanitized to prevent SQL injection, XSS

---

### Scalability Requirements

**NR-Scale-1: Traffic Scaling**
- **Requirement:** Ability to scale real-time to support high volumes during peak periods or campaign-driven spikes (10x normal traffic)
- **Normal Traffic:** ~1,000 concurrent users, ~5,000 requests/minute
- **Peak Traffic:** ~10,000 concurrent users, ~50,000 requests/minute (holiday weekends, major promotions)
- **Auto-scaling:** System must auto-scale compute resources (containers, database connections) within 2 minutes of traffic spike
- **No Degradation:** Response time must remain <2 seconds at 10x normal load

**NR-Scale-2: Database Scalability**
- **Requirement:** Database must support 500 concurrent connections with API response time remaining <2 seconds at 10x normal load for all product and inventory query types
- **Implementation:**
  - Read replicas for high-volume queries (inventory, product catalog)
  - Distributed caching layer for frequently accessed data (warehouse lists, popular products)
  - Partitioning: Product catalog partitioned by category/brand for query parallelization
- **Baseline Capacity:** Support 500 concurrent database connections

**NR-Scale-3: Cache Strategy**
- **Requirement:** Multi-tier caching (browser, CDN, application, database) reduces origin load by ≥60% under normal traffic conditions
- **Cache Layers:**
  - Browser cache (static assets: CSS, JS, images; 7-day TTL)
  - CDN cache (product images, design assets; 1-day TTL)
  - Application cache (product catalog, warehouse info, promotions; 1-hour TTL)
  - Database query cache (frequently accessed queries; 15-minute TTL)
- **Cache Invalidation:** When data changes (inventory update, price change, promotion end), cache invalidated immediately

---

### Accessibility Requirements

**NR-Acc-1: WCAG 2.1 AA Compliance**
- **Requirement:** Inclusive experience on app and website for all shoppers through accessibility features
- **Implementation:**
  - Keyboard navigation support (all functionality accessible via keyboard)
  - Screen reader compatibility (ARIA labels on all interactive elements)
  - Color contrast ratio ≥4.5:1 for normal text
  - Focus indicators visible and understandable
  - No flashing/blinking content (risk of photosensitive seizures)
- **Testing:** Automated accessibility scanning (Axe, Lighthouse) + manual testing with screen reader software

**NR-Acc-2: Mobile Accessibility**
- **Requirement:** Accessible experience on iOS/Android including voice over, switch control, magnification
- **Implementation:**
  - iOS VoiceOver compatible labels
  - Android TalkBack compatible descriptions
  - Minimum touch target size: 44x44 points (iOS), 48x48 dp (Android)
  - Support for system text scaling (up to 200%)

**NR-Acc-3: Localization**
- **Requirement:** Full support for en-US, en-CA, fr-CA with localized content
- **Implementation:**
  - All UI text translatable (strings externalized, not hardcoded)
  - Date/time formatting locale-appropriate (MM/DD/YYYY vs. DD/MM/YYYY)
  - Currency display appropriate (USD vs. CAD; symbols vs. codes)
  - Phone number formatting locale-specific
  - No English-only assumptions in code or design

---

### Compliance Requirements

**NR-Comp-1: Legal & Regulatory Compliance**
- **Requirement:** Product meets all legal and regulatory obligations for e-commerce operations in the United States (US) and Canada (CA)
- **Tax Calculation:** Tax calculation and remittance is handled by Costco's existing checkout platform; this product does not introduce new tax logic. If scope changes to include a new checkout flow, tax-by-jurisdiction requirements must be re-scoped.
- **Standards:**
  - GDPR (EU member data privacy)
  - CCPA (California consumer privacy)
  - PIPEDA (Canada personal information privacy)
  - SOC 2 Type II (security, availability, processing integrity)
  - CAN-SPAM (for email communications)
- **Implementation:**
  - Privacy policy clearly disclosing data usage
  - Member opt-in/opt-out controls for marketing communications
  - Data retention limits (purchase history deleted after 7 years)
  - Right-to-deletion (member can request account + data deletion)

**NR-Comp-2: Warranty & Returns Policy**
- **Requirement:** Clear display of tire warranty and return policy before purchase
- **Implementation:**
  - Warranty details visible on product detail page (manufacturer warranty, Costco coverage)
  - Return policy clearly linked in checkout (60-day return, free rotation/balance)
  - No hidden fees; all costs transparent at checkout

---

### Analytics & Monitoring Requirements

**NR-Analytics-1: User Journey Tracking**
- **Requirement (Phase 4):** Capture member behavior insights across all seven entry points
- **Data Tracked:**
  - Entry point used (Start Search, Global Search, Shop Nav, Deals, Brand, Services, Ownership)
  - Fitment selection path (license plate vs. manual vs. direct size)
  - Product browsing behavior (filters applied, time spent, products viewed, comparison)
  - Purchase decision (tire selected, add-to-cart time, cart abandonment)
  - Appointment booking behavior (if scheduled, time to completion, confirmation)

**NR-Analytics-2: Real-Time Alerting**
- **Requirement:** System alerts for critical events (errors, performance degradation, suspicious activity)
- **Alert Triggers:**
  - Page error rate >1%
  - API response time >3 seconds
  - Search success rate <95%
  - Inventory sync failures
  - Payment transaction failures >2%
- **Alert Channel:** Slack, PagerDuty, email to ops team

---

### Technical Architecture Requirements

**NR-Tech-1: Cloud Hosting**
- **Requirement (Phase 1B):** Modernized hosting infrastructure for scalability and reliability
- **Provider:** Google Cloud Platform (GCP) — Costco's preferred provider
- **Services:**
  - Compute: Cloud Run (containerized app) or App Engine (managed PaaS)
  - Database: Cloud SQL (managed SQL Server) with read replicas
  - Cache: Cloud Memorystore (managed Redis)
  - Storage: Cloud Storage (images, assets)
  - CDN: Cloud CDN (edge caching)
  - Monitoring: Cloud Monitoring, Cloud Logging
- **Backup:** Automated backups to Cloud Storage, cross-region redundancy

**NR-Tech-2: API Architecture**
- **Requirement:** RESTful API design for partner integrations and internal services
- **Standards:**
  - RESTful endpoints (GET, POST, PUT, DELETE)
  - Versioning: URL-based (/api/v1/, /api/v2/)
  - Response format: JSON with consistent schema
  - Error handling: HTTP status codes + error detail payloads
  - Documentation: OpenAPI 3.0 specification (Swagger)

**NR-Tech-3: Build & Deployment**
- **Requirement:** CI/CD pipeline deploys approved code changes to production within 1 hour of approval with automated rollback capability
- **Tools:**
  - Source control: Git (GitHub)
  - CI/CD: Cloud Build (GCP native)
  - Container registry: Artifact Registry (GCP)
  - Deployment: Continuous deployment to staging; manual promotion to production
  - Rollback: One-click rollback to previous stable version
- **Deployment Frequency:** Ability to deploy code changes within 1 hour of approval

---

### Platform & Distribution Requirements

**NR-Plat-1: Browser Compatibility**
- **Requirement:** Web experience fully functional across all major browsers used by Costco members on both US and CA sites
- **Supported Browsers:**
  - Chrome ≥ 108 (desktop + Android)
  - Safari ≥ 16 (desktop + iOS — highest usage for mobile Costco shoppers)
  - Firefox ≥ 108 (desktop)
  - Edge ≥ 108 (desktop)
- **Exclusions:** Internet Explorer not supported
- **Testing:** Automated cross-browser testing on every release candidate; Safari iOS receives dedicated QA given Costco mobile demographics

**NR-Plat-2: SEO Requirements**
- **Requirement:** Tire product listing pages and product detail pages must be indexable and rank for relevant search queries on Google and Bing
- **Technical Implementation:**
  - Server-side rendering (SSR) or static generation for product pages (not client-side-only rendering)
  - Structured data markup (Schema.org Product type) on all tire product detail pages
  - Canonical URL management for size-filtered and warehouse-filtered URLs
  - Sitemap.xml automatically updated when new products or pages are published
  - Core Web Vitals targets: LCP <2.5s, CLS <0.1, FID <100ms (as measured by Google Search Console)
- **Target Queries:** "[brand] tires [city]", "winter tires [vehicle]", "buy tires online Costco", "tire installation near me"

**NR-Plat-3: Push Notification Requirements (Phase 2 — Mobile)**
- **Requirement (Phase 2):** Members can opt into push notifications for order updates, appointment reminders, and promotional offers on iOS and Android
- **Compliance:**
  - iOS: Push notification permission requested only after member has demonstrated engagement (post-first-purchase or post-appointment); complies with Apple's explicit opt-in requirement
  - Android: Opt-in prompt presented at first relevant trigger (e.g., after appointment booking confirmation)
  - Members can manage notification preferences in-app; opt-out honored within 24 hours
- **Notification Types:**
  - Order confirmation and status updates
  - Appointment reminder (24h and 1h before appointment)
  - Promotional offers (opt-in only; max 2 per week)
- **Measurement:** Push opt-in rate target ≥40% of members who complete purchase

**NR-Plat-4: App Store Compliance (Phase 2 — Mobile)**
- **Requirement (Phase 2):** Mobile app Tires section meets Apple App Store and Google Play Store guidelines for retail/e-commerce apps
- **Apple App Store:**
  - In-app purchases (tire transactions) processed via existing Costco checkout (web checkout integration within app); no Apple IAP fees apply as tire purchase is a physical good
  - App Privacy Nutrition Label completed and accurate (data types collected: purchase history, vehicle profile, location, contact info)
  - App Review Guidelines 3.1.1 compliance confirmed: physical goods transactions exempt from Apple's 30% commission
- **Google Play:**
  - Data safety form completed and accurate
  - Target API level meets current Play Store minimum (≥API 34 at Phase 2 launch)
- **App Version Management:** Minimum supported iOS version: iOS 16; minimum Android version: Android 10 (API 29)

---

## SUCCESS CRITERIA & METRICS


### Goals-to-Metrics Traceability

#### **Goal 1: Native Mobile App Experience**
- **Metric:** App Engagement Rate (daily active users, session frequency, time-in-app)
- **Baseline:** Current web-view link engagement (establish in Phase 1 discovery)
- **Target (Phase 2 +3 months):** 3x engagement vs. web-view baseline
- **Measurement:** App analytics dashboard; segment by Tires section

**Associated KPIs:**
- **Click-Through Rate (CTR):** From marketing campaigns to native app > web campaigns
- **Session Duration:** Avg session length in Tires section >3 minutes (vs. <1 min for web-view)
- **Feature Adoption:** % of users accessing native features (search, filters, comparison) >60%

---

#### **Goal 2: Consistent Experience Across Channels**
- **Metric:** Exit Rate Reduction (lower abandonment when channel-switching)
- **Baseline:** Current exit rate when redirecting between web and mobile (establish Phase 1)
- **Target (Phase 2 +3 months):** 40% reduction in exit rate for cross-channel flows
- **Measurement:** Track drop-off rate before/after context persistence feature launch

**Associated KPIs:**
- **Channel Consistency Satisfaction:** Post-purchase survey: "I could start on mobile and finish on web without re-entering info" (target: >85% agree)
- **Bounce Rate:** Reduce bounce rate on tires pages by 25%
- **Returning Customer Volume:** % of members visiting tires section multiple times per quarter increases 20%

---

#### **Goal 3: Preserve Shopper Context**
- **Metric:** Customer Lifetime Value (CLV) Increase for repeat tire buyers
- **Baseline:** Current avg CLV for repeat tire purchasers (4-5 year replacement cycles)
- **Target (Phase 5 +6 months):** 15% increase in CLV from context preservation features
- **Measurement:** Comparison cohort analysis (members with profiles vs. without)

**Associated KPIs:**
- **Repeat Purchase Rate:** % of prior tire buyers who repurchase within 5 years increases from 35% → 50%
- **Repeat Purchase Cycle Acceleration:** Avg time between tire purchases decreases (5.2 years → 4.8 years)
- **Reorder Completion Time:** Members using "Buy Again" feature complete purchase in <2 minutes (vs. 8-10 min for new customer flow)

---

#### **Goal 4: Eliminate Disconnected Flows**
- **Metric:** Add-to-Cart Completion Rate (% of users who add tires to cart)
- **Baseline:** Current completion rate (establish Phase 1 discovery)
- **Target (Phase 2 +3 months):** 25% improvement
- **Measurement:** Analytics: (# users completing add-to-cart) / (# users entering fitment flow)

**Associated KPIs:**
- **Checkout Completion Rate:** % of users who complete checkout (add-to-cart → payment confirmation)
- **Baseline (establish Phase 1):** Target 35% improvement after integrated scheduling launch (Phase 3)
- **Appointment Booking Rate:** % of tire purchasers who book installation appointment >80% (vs. current <40%)
- **No-Show Rate Reduction:** Reduce tire center appointment no-shows by 30% (from inventory accuracy + integration)

---

### Persona-Specific Success Criteria

| Persona | Primary KPI | Baseline | Phase 2 Target | Phase 3 Target | Measurement Method |
|---------|-------------|----------|----------------|----------------|-------------------|
| **1. Task/Mission-Driven** | Add-to-Cart Conv. | [Establish Phase 1] | +20% | +30% | Analytics |
| **2. Searcher/Problem Solver** | Search Success Rate | [Establish Phase 1] | >90% | >95% | Query analysis, user tracking |
| **3. Explorer/Category Browser** | Engagement Depth | [Establish Phase 1] | +40% | +50% | Pages/session, duration |
| **4. Deal Hunter/Promo-Driven** | Fitment Start Rate | [Establish Phase 1] | +25% | +35% | Promo link tracking |
| **5. Brand Loyalist** | Validated Fitment Rate | [Establish Phase 1] | >85% | >90% | Brand entry flow tracking |
| **6. Services Seeker** | Appointment Completion | [Establish Phase 1] | +30% | +40% | Appointment system |
| **7. Repeat Buyer** | Reorder Success Rate | [Establish Phase 1] | +50% | +60% | Order history tracking |
| **8. Battery Buyer** | Battery Purchase Completion Rate | [Establish Phase 4B launch] | — | >70% | Order analytics (Phase 4B) |

---

### Business Metrics

| Metric | Business Owner | Phase | Baseline | Target | Notes |
|--------|----------------|-------|----------|--------|-------|
| **Tire Sales Revenue** | Finance | Phase 2 | [TBD] | +10% YoY | Requires market conditions validation |
| **Tire Center Utilization** | Ops | Phase 3 | [Establish] | +15% install bays in-use | Integrated scheduling impact |
| **Customer Support Volume** | Customer Svc | Phase 1+ | [Establish] | -20% inbound calls | Self-service features reduction |
| **Repeat Customer Acquisition** | Marketing | Phase 5 | 35% | 50% | Vehicle profile + history features |
| **Mobile App Adoption** | Product | Phase 2 | 0% (Tires native) | 40% of tire shoppers | 12 months post-launch |
| **Member NPS (Tires)** | Product | Phase 2 | [Establish Phase 1] | +15 points | Post-purchase survey |

---

### Technical Success Criteria

| Metric | Target | SLA | Monitoring |
|--------|--------|-----|-----------|
| **Page Load Time (p95)** | <2 sec (web), <1.5 sec (mobile cold) | 99% of pages meet target | Real User Monitoring |
| **API Response Time (p95)** | <500 ms | 99% of requests meet target | Application Insights |
| **System Uptime** | 99.99% | Monthly SLA | Automated alerts |
| **Inventory Sync Accuracy** | <10 min latency | 100% of inventory updates within SLA | Automated validation |
| **Search Accuracy** | >95% relevant results | Precision measured per query type | Search analytics |

---

## SCOPE & PHASED DELIVERY

### Phased Transformation Approach

The modernization occurs across five strategic phases, each delivering cumulative value while building foundation for subsequent phases. Sequence is intentionally ordered to maximize early revenue impact and competitive advantage.

---

### **PHASE 1: WEB FOUNDATION & CLOUD MIGRATION**
**Duration:** 4-5 months | **Target Launch:** Q3 2026

#### **Phase 1A: Tires Journey Reimagine (Web)**
**Goal:** Modernize tires.costco.com/.ca to reduce purchase friction and establish navigation patterns for omnichannel integration

> **Supported Personas:** Persona 1 (Task/Mission-Driven), Persona 2 (Searcher), Persona 3 (Explorer), Persona 4 (Deal Hunter), Persona 5 (Brand Loyalist)
> Personas 6, 7, 8 have foundational touchpoints here but their primary features activate in later phases.

1. **Reimagined Tire Selection Flow**
   - Clean information architecture supporting 7 entry points
   - Vehicle selection by License Plate, Vehicle Specs, or Tire Size
   - Fitment validation and compatible size determination
   - Session/account context persistence

2. **Optimized Entry Points (7 Patterns)**
   - Start Tire Search: Dedicated landing page with fitment CTA
   - Global Search: Enhanced site search with tire intent detection + autocomplete
   - Shop Navigation: Tires & Auto hub with category-based browsing
   - Deals/Promotions: Promo links routing directly to fitment (no dead-end promo pages)
   - Brand Entry: Brand galleries routing through fitment before SKU display
   - Services-First: Warehouse service pages with inline appointment booking
   - Ownership/Reorder: "Buy Again" flows from order history

3. **Responsive Design (4 Breakpoints)**
   - 375px (mobile portrait)
   - 768px (tablet portrait)
   - 1024px (tablet landscape/small desktop)
   - 1280px+ (desktop)
   - Touch-optimized for mobile; keyboard navigation support

4. **Core Product Pages**
   - **Product Listing Page:** Filtered results (tire models, brands, prices); advanced filters; 3-4 product comparison; warehouse selector
   - **Product Detail Page:** Full specifications, warranty, installation pricing, reviews, related products, add-to-cart, appointment selector
   - **Cart Page:** Item review, cart persistence, installation option, warehouse change, checkout CTA

5. **Customer Support Components**
   - **Tire Size Guide:** Interactive overlay explaining tire sizing format, vehicle fitment lookup, size conversion calculator
   - **Tire Center Locator:** Warehouse search, location details, hours, appointment availability
   - **Schedule Appointment Modal:** Calendar picker, time slot display, appointment confirmation, SMS/email confirmation
   - **Tire Compatibility Check:** Validation messaging, error prevention, size alternative suggestions
   - **Change Vehicle Modal:** Update vehicle selection mid-session, refetch compatible sizes, cart update

6. **Technical Foundation**
   - Responsive design patterns and component library (reusable across web + mobile)
   - Client-side session storage for context persistence
   - API layer design (REST endpoints for product, inventory, scheduling)
   - Performance optimization (image lazy-loading, critical CSS, asset minification)

**Key Metrics (Phase 1A Target):**
- Page load time: ≤2 seconds (p95)
- Mobile bounce rate: <40%
- Add-to-cart conversion (new baseline): [Establish Phase 1]
- User engagement (pages per session): >3.5 pages

**Success Gate:** Phase 1A performance metrics meet targets; ready for Phase 1B cloud migration

---

#### **Phase 1B: Azure to GCP Migration**
**Goal:** Modernize hosting infrastructure to Google Cloud Platform (Costco's preferred provider)

> **Infrastructure Phase:** No persona-specific feature changes. Enables all personas across all phases through improved reliability, scalability, and performance.

**Deliverables:**

1. **Infrastructure Modernization**
   - Assess current Azure infrastructure (compute, database, storage, networking)
   - Design GCP equivalent architecture (Cloud Run, Cloud SQL, Cloud Storage, Cloud CDN, Cloud Monitoring)
   - Plan migration strategy (lift-and-shift vs. rearchitecture; phased cutover to minimize risk)

2. **Database Migration**
   - Migrate SQL Server database from Azure to Cloud SQL (managed SQL Server on GCP)
   - Configure read replicas for high-volume queries
   - Validate data integrity post-migration
   - Performance testing (confirm <2 sec response times maintained)

3. **Application Deployment**
   - Containerize tires web application (Docker containers, stored in Artifact Registry)
   - Deploy to Cloud Run (serverless, auto-scaling compute)
   - Configure auto-scaling policies (scale 1-1000 instances based on traffic)

4. **Cache & CDN**
   - Implement Cloud Memorystore (managed Redis) for application cache
   - Configure Cloud CDN for edge caching of static assets
   - Cache invalidation strategy for content updates

5. **Monitoring & Logging**
   - Setup Cloud Monitoring dashboards (latency, error rates, resource utilization)
   - Configure Cloud Logging for centralized log aggregation
   - Alerting policies for SLA violations

6. **Backup & Disaster Recovery**
   - Implement automated Cloud SQL backups (hourly snapshots, 30-day retention)
   - Cross-region backup replication
   - Disaster recovery runbook and testing

**Infrastructure Targets:**
- Uptime: 99.99% SLA capability
- Auto-scale time: <2 minutes to handle 10x traffic spike
- RTO (Recovery): <30 minutes
- RPO (Data Loss): <5 minutes

**Dependencies:**
- Reference: "Infogain - Costco-Tires-Azure-To-GCP Migration-V5.pdf" for complete technical scope

**Success Gate:** Production traffic successfully migrated to GCP; Phase 2 mobile development can begin in parallel

---

### **PHASE 2: NATIVE MOBILE APP INTEGRATION** ⭐ **HIGHEST PRIORITY**
**Duration:** 4-6 months | **Target Launch:** Q4 2026

**Goal:** Build fully native iOS/Android tire shopping experience within the existing Costco mobile app, replacing current web-view link with integrated native capability

> **Supported Personas:** Persona 1 (Task/Mission-Driven), Persona 2 (Searcher), Persona 3 (Explorer), Persona 4 (Deal Hunter), Persona 5 (Brand Loyalist)
> All five primary purchase personas receive native app experience. Personas 6, 7, 8 benefit from app infrastructure but their dedicated features activate in Phase 4B (Persona 8) and Phase 5A/5B (Personas 6, 7).

**Why Phase 2 is Priority:**
- Competitive differentiation: Delivers app-grade experience vs. competitors' web-based solutions
- Engagement multiplier: Native performance drives 3x higher engagement (Phase 1 baseline established)
- Market segment: EV owners and digitally-savvy members (high LTV segment) expect native experiences
- Revenue opportunity: Mobile traffic often >40% of total for retail experiences; unlocking native conversion lifts revenue immediately

**Deliverables:**

1. **Native App Architecture (iOS & Android)**
   - Native iOS app (Swift) with iOS SDK integration
   - Native Android app (Kotlin) with Android SDK integration
   - Shared business logic and API layer (minimizing platform-specific code)
   - Offline-first architecture (cache products locally, work offline for browsing)

2. **Native Feature Implementations**
   - **Home Screen:** Vehicle selection, deals carousel, value proposition, recent searches, quick actions
   - **Native Search:** Search box with voice input, brand/product autocomplete, recent searches, intent detection
   - **Product Navigation:** Native tab-based navigation pattern
   - **Product Listing:** Native list view optimized for mobile scrolling, advanced filter drawer, comparison tray
   - **Product Detail:** Native carousel for product images, expandable specs, reviews, add-to-cart
   - **Size Guide:** Interactive overlay explaining tire sizing with visual diagrams
   - **Tire Center Locator:** Native map view, location search, appointment availability per location
   - **Schedule Appointment:** Native date picker, time slot selector, SMS confirmation

3. **Mobile Performance Optimization**
   - App launch time: <1.5 seconds (cold start)
   - Page transitions: <1 second
   - Interaction response: <100ms
   - Local image caching (reduce network requests)
   - Progressive image loading (show placeholder → low-res → high-res)

4. **Integration with Costco App**
   - Single sign-on (SSO) with existing Costco member account
   - Unified cart (tires cart syncs with Costco app cart)
   - Costco checkout integration (member profile, payment methods pre-filled)
   - Deep linking (URLs opening directly to products from web links or marketing)
   - Push notifications (order updates, appointment reminders, promotional offers)

5. **Cross-Device Context Persistence**
   - Vehicle selection saved to member account (sync across devices)
   - Cart synced between web and mobile app
   - Shopping history and saved items
   - Recent searches and browsing history

6. **Advanced Mobile Features**
   - Camera-based license plate scanning (VIN lookup on device camera)
   - Device geolocation (nearest warehouse detection)
   - Biometric authentication (fingerprint, face recognition for checkout)
   - Voice search ("Show me winter tires for my Model 3")
   - Augmented reality (AR) tire visualization on vehicle (future expansion)

7. **Tablet & Landscape Orientation Support**
   - Adaptive layouts for tablet screen sizes (iPad Pro, Galaxy Tab)
   - Landscape orientation support with optimized layout
   - Split-screen multitasking support (iOS 14+, Android 10+)

8. **Cross-Sell & Upsell Opportunities**
   - Related products: batteries, tire accessories, services
   - Bundle recommendations ("Tires + Installation" vs. tires only)
   - Services prompts: "Schedule tire rotation?"

**Designs Reference:**
- All `/docs/images/mobile-app/` designs serve as UI specifications for Phase 2 implementation

**Technical Specifications:**
- Reference: "Costco Tires & Batteries-Transforming to Native App.pdf" for complete technical architecture
- Minimum OS versions: iOS 13+, Android 9+
- Network requirements: Supports 3G (fallback slow network handling)

**Key Phase 2 Metrics (3 months post-launch):**
- Native app adoption: >40% of tire shoppers using native app
- Engagement (native vs. web): 3x higher session time, 4x higher CTR
- Mobile conversion: Mobile tire purchase rate >35% of total (vs. current <20%)
- NPS score (Tires native): >50 (vs. <30 for current web-view)

**Success Gate:** Native app launches with >40% feature parity to web; engagement metrics meet targets; ready for Phase 3

---

### **PHASE 3: DIGITAL EXPERIENCE MODERNIZATION**
**Duration:** 3-4 months | **Target Launch:** Q1 2027

#### **Phase 3A: Real-Time Inventory Availability**
**Goal:** Provide members with accurate, real-time visibility into warehouse tire inventory and fulfillment timelines

> **Applicable Personas:** All Personas (1–8) — activates per delivery timeline (Q1 2027). This cross-cutting capability enhances every persona's confidence in purchase decisions once live.

**Deliverables:**

1. **Warehouse-Specific Inventory Integration**
   - API integration with Costco warehouse inventory management system
   - Real-time stock status for each product at each warehouse
   - Automatic inventory sync (every 15 minutes minimum; real-time preferred)
   - Inventory accuracy validation (compare displayed vs. actual; alert on discrepancies >5%)

2. **Inventory Display & Accuracy**
   - **Product Results Page:** "Available at [Warehouse Name] ([#] in stock). Delivery by [Date]."
   - **Product Detail Page:** Warehouse-specific stock status with nearby warehouse options
   - **Cart:** "In stock at [Selected Warehouse]. Confirm?"
   - **Change Warehouse:** System updates inventory, delivery date, installation availability for new location

3. **Inventory Status Options**
   - "In Stock": Show quantity, delivery date
   - "Limited Stock": Show quantity with yellow warning; auto-notify if stock drops below threshold
   - "Out of Stock": Disabled option or grey-out
   - "Expected [Date]": For backorder items; allow waitlist signup

4. **Fulfillment Timeline Accuracy**
   - Delivery date = warehouse fulfillment date (when tire available for pickup/installation at warehouse)
   - Calculations based on warehouse processing time (same-day vs. next-day vs. 2-3 day)
   - Real-time adjustment (if inventory low, add buffer to fulfillment date)

5. **Inventory Confidence Scoring** (Advanced)
   - Display confidence level for availability claim: "Likely available" vs. "Confirmed in stock"
   - Conservative scoring approach: Only show high-confidence availability; prevent disappointment

6. **No Cancellation Guarantee**
   - Promise: No tire center appointment cancellations due to inventory discrepancy
   - Implementation: Verify tire in stock 24 hours before appointment; alert member immediately if issue
   - Fallback: Offer substitute tire or reschedule appointment immediately if unavailable

**Technical Implementation:**
- Real-time sync with warehouse inventory APIs
- Caching strategy (inventory cached 15 minutes; invalidate immediately on stock changes)
- Confidence algorithm (based on sales velocity, turnover rate, supplier lead time)
- Monitoring: Alert if inventory sync fails or latency exceeds SLA

---

#### **Phase 3B: Installation Scheduler Integration**
**Goal:** Integrate appointment scheduling directly into the tire selection flow; display available appointment times during product browsing

> **Applicable Personas:** All Personas (1–8) — activates per delivery timeline (Q1 2027). Particularly high-impact for Personas 1 (fast checkout), 4 (deal hunters wanting bundled installation), 6 (service-first scheduling), 7 (repeat buyers), and 8 (battery installation booking).

**Deliverables:**

1. **Warehouse Tire Center Scheduler Integration**
   - API integration with tire center scheduling system (extract available appointment slots)
   - Real-time sync of appointment availability (every 5 minutes minimum)
   - Warehouse-specific scheduling (no 2-minute appointment buffer; allow contiguous bookings)

2. **Appointment Availability Display**
   - **Product Results Page:** "Appointments available: 2 slots today, 5 slots this week" (expandable to show times)
   - **Product Detail Page:** Calendar view with available times for next 14 days at selected warehouse
   - **Cart Page:** Time slot selector (before checkout)
   - **Checkout:** Appointment confirmation as part of order checkout

3. **Integrated Appointment Booking**
   - No redirect to separate scheduling tool
   - Appointment booking available at multiple points in journey (product page, cart, checkout, order confirmation)
   - Single confirmation covers both tire purchase and installation appointment
   - One-flow booking: Member sees tire + appointment + total cost together

4. **Service Bundling**
   - Tire purchase + installation bundled into single appointment
   - Installation cost calculated and added to tire cost (displayed before checkout)
   - Optional: balance/rotation add-ons available during booking
   - Package pricing displayed clearly ("Complete Tire Installation: $50 labor + balance + rotation included")

5. **Appointment Confirmation & Reminders**
   - SMS confirmation immediately upon booking ("Your appointment: 2 PM tomorrow at Costco Warehouse #1234. Confirm? Reply Y/N")
   - Email confirmation with all details
   - Reminders: SMS/email 24 hours before, 1 hour before appointment
   - Appointment modification/cancellation: Self-service in account (up to 4 hours before appointment)

6. **Warehouse Capacity Management**
   - No overbooking: Appointment slots exactly match tire center bay availability
   - Demand forecasting: Adjust slot availability based on tire purchase trends (more slots during season changes)
   - Waitlist: If no slots available, option to join waitlist or select alternate warehouse

**Technical Implementation:**
- Scheduler API design (query available slots, book appointment, update confirmation)
- Conflict prevention: Real-time slot locking during booking to prevent double-booking
- Monitoring: Alert if scheduler is down; fallback to offline booking (email request to tire center)

**Phase 3B Success Metrics:**
- Appointment booking rate: >80% of tire purchasers book installation
- No-show rate: Reduce by 30% (from better confirmation + integration)
- Average appointment lead time: <2 days for new customers, same-day for repeat customers

---

#### **Phase 3C: Tire Education Guide (3rd Party Content)**
**Goal:** Integrate expert educational content supporting informed purchase decisions without friction

> **Applicable Personas:** All Personas (1–8) — activates per delivery timeline (Q1 2027). Primarily benefits Persona 2 (Problem Solver), Persona 3 (Explorer), and Persona 5 (Brand Loyalist). Also supports Persona 8 (Battery Buyer) through battery selection guidance content.

**Deliverables:**

1. **Content Partnerships**
   - Select 3rd-party content provider (e.g., tire manufacturer, industry education organization, content specialist)
   - Content licensing agreement covering: usage rights, refresh frequency, exclusivity terms
   - OPEN ITEM: Content provider selection + licensing model + maintenance strategy

2. **Educational Content Hub**
   - Dedicated "Tire Buying Guide" section with expert resources
   - Content types:
     - Tire sizing guide (interactive with visual diagrams)
     - Seasonal tire benefits (winter, summer, all-season trade-offs)
     - Performance tier explanations (comfort vs. performance vs. durability)
     - EV-specific tire content (range impact, weight handling, specialized tire features)
     - Brand positioning and warranty explanations
     - Tire maintenance tips (rotation schedules, pressure monitoring, wear indicators)
     - Installation service explanations (what's included, why it matters)

3. **Contextual Education**
   - Tooltips on product pages ("What does UTQG mean?", "Why is this tire better for winter?")
   - Size guide overlay (triggered when member selects tire size)
   - Video content (15-30 second clips on tire installation, sizing, seasonal needs)
   - Smart content recommendations based on member's journey (search "winter" → winter tire guide)

4. **Content Freshness & Maintenance**
   - Content updates: Quarterly minimum (seasonal updates, new product info)
   - OPEN ITEM: Maintenance responsibility (Costco vs. 3rd-party provider)
   - Quality assurance: Editorial review before publishing

**Content Strategy:**
- Content supports member confidence in decision-making (not sales-driven bias)
- Brand-agnostic approach (educate on tire types, not "Brand X is better")
- Accessibility-first: Video captions, screen-reader friendly descriptions

**Applicable Personas:** Personas 2 (Problem Solver), 3 (Explorer), 5 (Brand Loyalist)

**Phase 3C Success Metrics:**
- Content engagement: >60% of shoppers view at least one educational piece
- Purchase confidence: Post-purchase survey: "I felt confident in my tire choice" >80%
- Support ticket reduction: Tire-related support tickets decrease 15%

---

#### **Phase 3D: New Content Pages**
**Goal:** Expand content navigation to support browsing by driving needs, season, category, family, and EV specialization

> **Applicable Personas:** All Personas (1–8) — activates per delivery timeline (Q1 2027). New browsing surfaces benefit all personas; EV content particularly relevant for Persona 1, 2, 3; seasonal content benefits Persona 3 and 4; battery content pages support Persona 8.

**Deliverables:**

1. **Browse by Driving Needs**
   - Performance (sporty handling, responsive steering)
   - Comfort (quiet ride, fuel efficiency)
   - Towing (high load capacity, durability)
   - Off-road (traction in mud/snow, ruggedness)
   - Each category filters tire results to matching products

2. **Browse by Season**
   - Winter (ice/snow traction)
   - Summer (dry/wet handling, heat resistance)
   - All-Season (balanced performance year-round)
   - Each season shows tires optimized for that condition

3. **Browse by Category & Tire Family**
   - By tire type (all-season, winter, summer, performance, eco, off-road)
   - By tire family/series (Michelin Defender, Bridgestone Turanza, etc.)
   - By manufacturer/brand galleries

4. **Browse by EV Cars** ⭐ **Competitive Differentiation**
   - Dedicated EV section with specialized tire options
   - EV-specific content: range impact, weight handling, tire features
   - Filter: "All EV tires", "EV optimized" badge on compatible products
   - EV makes: Tesla, Chevrolet Bolt, Hyundai Ioniq, Kia EV6, BMW i4, Mercedes EQE, Audi E-Tron, Volkswagen ID., etc.
   - Positioning advantage: Costco as automotive partner for EV segment

5. **Navigation Integration**
   - Homepage hero section: "Shop by Season", "Shop by Driving Needs", "EV Tire Specialist"
   - Main navigation: New menu items for these browse categories
   - Mobile app: Tab-based navigation + category tiles
   - Cross-linking: Product detail pages link to related categories

**Reference:** "Screenshots of Tires Design.pdf" for additional screen examples and navigation patterns

**Phase 3 Success Metrics (1 month post-launch):**
- Inventory accuracy: >98% (mismatch between displayed vs. actual <2%)
- Appointment completion rate: >80% of tire buyers book installation
- Content engagement: >60% of shoppers view at least one educational piece
- EV tire segment: 15% of tire sales from EV vehicle segment

---

### **PHASE 4: ANALYTICS & BATTERY SALES EXPANSION**
**Duration:** 3 months | **Target Launch:** Q2 2027

#### **Phase 4A: User Journey Tracking & Analytics**
**Goal:** Capture member behavior insights across all seven entry points to refine user experience and optimize conversion

> **Applicable Personas:** All Personas (1–8) — activates per delivery timeline (Q2 2027). Analytics tracks behavior for all personas including the new Battery Buyer (Persona 8) entry point. Enables data-driven optimization across all experience layers.

**Deliverables:**

1. **Journey Tracking Implementation**
   - Track entry point used (Start Search, Global Search, Shop Nav, Deals, Brand, Services, Ownership)
   - Track fitment selection path (license plate vs. manual vs. direct size)
   - Track product browsing behavior (filters applied, time spent, products viewed, comparison)
   - Track purchase decision (tire selected, add-to-cart time, cart abandonment points)
   - Track appointment booking behavior (if scheduled, time to completion, confirmation)

2. **Behavioral Analytics Dashboard**
   - Funnel analysis: Entry point → Fitment → Browse → Cart → Checkout → Confirmation
   - Conversion rates by entry point (which entry points convert best?)
   - Product affinity: Which tires are most popular by persona/entry point?
   - Abandonment points: Where do members drop off most frequently?
   - Channel behavior: Web vs. mobile differences; cross-channel switching patterns

3. **Persona Performance Tracking**
   - Segment analytics by inferred persona (entry point proxy)
   - Task-Driven journey efficiency: How fast do they check out?
   - Explorer journey depth: How many filters/comparisons before purchase?
   - Deal Hunter: Impact of promotions on conversion?
   - Brand Loyalist: Brand entry point success rates?
   - Services Seeker: Appointment booking behavior?
   - Repeat Buyer: Reorder flow efficiency?

4. **Optimization Recommendations**
   - A/B testing framework: Test design variants, copy, UI changes
   - Experimentation governance: What changes are tested, how results are validated
   - Iteration velocity: Ability to run test, measure results, iterate weekly

5. **Privacy & Compliance**
   - All tracking non-PII (no personal data logged)
   - Member opt-out available for behavioral tracking (while maintaining core site functionality)
   - Data retention: 24-month retention for analysis

**Analytics Tools:** Application Insights, custom event tracking

**Phase 4A Metrics:**
- Dashboard adoption: 100% of product/ops team using daily
- A/B testing cadence: Minimum 4 tests per month launched
- Insights action rate: 80% of recommended optimizations prioritized in roadmap

---

#### **Phase 4B: Battery Sales Expansion (US Market)**
**Goal:** Extend unified experience to automotive battery sales; standardize buying experience across all regions

> **Supported Personas:** Persona 8 (Battery Buyer) — primary beneficiary. All other personas benefit from the cross-sell between tires and batteries.

**Deliverables:**

1. **Battery Product Catalog**
   - Inventory of compatible batteries for vehicles
   - Vehicle fitment validation (similar to tires)
   - Battery specs, warranty, pricing, availability by warehouse

2. **Unified Shopping Experience**
   - Vehicle selection (same flow as tires)
   - Product browsing (battery options by type: lead-acid, AGM, lithium)
   - Installation bundling (battery + installation service)
   - Appointment scheduling (for battery replacement/installation)

3. **Battery + Tire Bundling**
   - Cross-sell opportunities: "Tires + Battery" package pricing
   - Joint installation appointment (if both services needed)
   - Cumulative discount on bundles

4. **Regional Standardization (US)**
   - Consistent experience across all US Costco warehouses
   - OPEN ITEM: Canada battery sales availability / phasing

5. **Content & Education**
   - Battery buying guide (when to replace, types, warranty)
   - Installation service explanations
   - Maintenance tips (battery care, lifespan, performance)

**Scope Note:** Battery sales are additive to tire experience; shares infrastructure, scheduling, and checkout flows

**Phase 4B Success Metrics:**
- Battery sales volume: Establish baseline, target 10% growth Y1
- Cross-sell rate: 15% of tire customers also purchase/consider battery
- Bundle adoption: 5% of tire+battery sales purchased as bundle

---

### **PHASE 5: MEMBER RETENTION & PERSONALIZATION**
**Duration:** 2 months | **Target Launch:** Q3 2027

#### **Phase 5A: Vehicle Profile Storage**
**Goal:** Enable members to save multiple vehicles and associated tire history for frictionless repurchase

> **Supported Personas:** Persona 6 (Services Seeker/Maintenance Planner) — primary beneficiary, enabling pre-populated service history and quick appointment re-booking. All other personas (1–8) benefit from vehicle profile persistence reducing fitment re-entry on return visits.

**Deliverables:**

1. **Vehicle Profile Management**
   - "My Vehicles" section in member account
   - Add multiple vehicles (auto, name, year, make, model, trim)
   - Vehicle tire history: Past tire purchases, tire specifications, service dates
   - Vehicle maintenance notes (rotation history, repairs, tire wear)

2. **Vehicle Auto-Population**
   - Member enters vehicle once; system remembers for future purchases
   - Auto-suggested vehicle selection on next visit ("Welcome back! Shopping for your Model 3?")
   - Fitment context automatically loaded (no re-entry of tire size)

3. **Maintenance Reminders**
   - "Time for tire check-up on your [Vehicle]?" reminder emails based on tire age/usage
   - Rotation reminder: "Tire rotation recommended every 5,000 miles"
   - Seasonal reminders: "Consider switching to winter tires"

4. **Multi-Vehicle Household**
   - Family members with shared Costco account can manage multiple vehicles
   - Permission controls: Primary account holder + authorized family members
   - Each vehicle has its own tire purchase history

**Profile Features:**
- Edit vehicle details (change trim, add nickname "Daily Driver" vs. "Weekend Car")
- Vehicle image/avatar (custom photo or generic icon)
- Service history tracking (tire rotations, repairs, installations)

---

#### **Phase 5B: Order History & Rebate Hub**
**Goal:** Simplify repeat purchase interactions and rebate management through centralized order history

> **Supported Personas:** Persona 6 (Services Seeker) and Persona 7 (Repeat Buyer) — primary beneficiaries. Persona 7 receives the full Buy Again / Reorder path. Persona 6 benefits from service history and appointment re-booking. All personas (1–8) benefit from rebate tracking and order history access.

**Deliverables:**

1. **Order History Retrieval**
   - "My Tires" section showing all past tire purchases
   - Order details: Tire brand/model, size, quantity, price, purchase date, warehouse
   - Associated vehicle: Which vehicle was this tire for?
   - "Buy Again" link: One-click reorder using previous purchase details

2. **Quick Reorder Path**
   - Click vehicle → See past tire purchase → Click "Buy Again" → Pre-populated cart
   - Fitment validation: "Is this size still correct?" (allow override)
   - Option to browse similar tires (if member wants alternative)
   - Streamlined checkout: 1-2 minutes to complete repurchase

3. **Rebate Tracking & Management**
   - "My Rebates" section showing available/earned tire rebates
   - Rebate eligibility: Which rebates apply to past purchases?
   - Redemption: One-click rebate application (auto-applied at checkout)
   - Rebate status: Tracking rebates in fulfillment process

4. **Service History**
   - Tire center appointment history (past appointments, services completed)
   - Rotation records, service notes, technician feedback
   - Integration: When booking new appointment, system suggests "You're due for rotation"

5. **Integrated Reorder Workflow**
   - Member views past purchase → Confirms vehicle still correct → Confirms tire size still correct → Adds to cart
   - **Total time to checkout:** <2 minutes
   - Appointment suggestion: "Schedule installation?"

**Success Metrics (Phase 5):**
- Repeat purchase rate: 50% of prior tire buyers repurchase within 5 years (vs. current 35%)
- Repeat purchase cycle: Accelerate from 5.2 years → 4.8 years average
- Reorder completion time: <2 minutes (vs. 8-10 min for new customer)
- Rebate redemption rate: 80% of eligible rebates redeemed

---

### Phase Delivery Timeline

```
Phase 1A: Web Reimagine    ████████ Q2-Q3 2026 (4-5 months)
Phase 1B: GCP Migration    ████████ Q3 2026 (parallel with 1A end)
Phase 2: Native Mobile     ████████████ Q3-Q4 2026 (4-6 months, starts mid-Phase 1)
Phase 3: Digital Exp       ████████ Q1 2027 (3-4 months)
Phase 4: Analytics & Bat.  ██████ Q2 2027 (3 months)
Phase 5: Retention & Prof. ████ Q3 2027 (2 months)
```

**Parallel Execution:** Phases 1B and 2 can run in parallel (1A completes, migration happens while mobile development begins)

**Dependencies:**
- Phase 1A must complete before Phase 1B launch (foundation locked)
- Phase 1B (GCP infrastructure) enables Phase 3 (real-time inventory/scheduling)
- Phase 2 uses Phase 1A patterns and components; can start mid-Phase 1A

---

## WHAT'S OUT OF SCOPE

**Explicitly Not Included in This Transformation:**

- **Used/Refurbished Tire Marketplace:** Secondary market for used tires not addressed
- **Peer-to-Peer Recommendations:** User-generated reviews/recommendations not in MVP; rely on manufacturer data + 3rd-party content
- **International Expansion:** Initially US/Canada only (tires.costco.com, tires.costco.ca); expansion considered post-Phase 5
- **Non-Costco Tire Center Integration:** Partnership with 3rd-party installers out of scope; Costco warehouses only
- **Alternative Fuel Vehicles:** Phase 3D focuses on EV; hybrid integration deferred
- **Automotive Services Beyond Tires/Batteries:** Oil changes, brake service, etc. not included in initial scope

---

## RISKS, DEPENDENCIES & MITIGATION

### Risk 1: Inventory Accuracy & Trust Damage
**Risk:** Real-time inventory promises create high expectations. Inaccurate data → member frustration + trust damage worse than no visibility.

**Impact:** High | **Probability:** Medium

**Mitigation:**
- Phased rollout with confidence scoring algorithms
- Launch Phase 3A with conservative "likely available" indicators; escalate to guarantees only after 2-week validation period
- Warehouse-by-warehouse validation before full deployment
- Automated accuracy monitoring with alert thresholds (<95% accuracy triggers rollback)
- Fallback: If inventory accuracy can't be maintained, degrade to "estimated availability" with manual verification option

---

### Risk 2: Scheduling System Integration Failures
**Risk:** Tire center scheduling system incompatibility or real-time sync failures → Double-bookings, no-shows, member frustration

**Impact:** High | **Probability:** Medium

**Mitigation:**
- Phased scheduling integration (pilot warehouses first: 5 pilot locations before full rollout)
- Dual-system approach during transition (digital + manual paper scheduling backup)
- Sync monitoring with alerts (if sync fails >5 min, revert to manual booking mode)
- Member communication: "Appointments subject to warehouse confirmation" until Phase 3B fully validated
- Scheduling OPEN ITEM: Change management plan + pilot location selection

---

### Risk 3: Competitive Response (TireRack Mobilizes)
**Risk:** TireRack or Discount Tire fast-follows with native mobile app, reducing competitive advantage.

**Impact:** Medium | **Probability:** High

**Mitigation:**
- Speed to market: Phase 2 native mobile as top priority (Q4 2026 target)
- Unique advantages: Integrated scheduling + inventory transparency + warehouse proximity = multi-year moat competitors can't quickly replicate
- Marketing: Aggressive positioning of "native-first" advantage before competitors launch
- Continuous innovation: Plan Phase 4 analytics for ongoing UX optimization (keep ahead of competitor offerings)

---

### Risk 4: Organizational Change (Warehouse Adoption Friction)
**Risk:** Integrated scheduling requires tire center staff to adopt new workflows; resistance to digital-first processes.

**Impact:** Medium | **Probability:** Medium

**Mitigation:**
- Change management plan: Phased rollout to pilot warehouses with enthusiastic early adopters
- Training: Clear SOPs for tire center staff on digital scheduling, inventory accuracy, appointment management
- Incentive alignment: Staff performance metrics tied to no-show reduction + accurate appointment completion
- Feedback loops: Regular check-ins with warehouse managers to surface friction and adjust processes
- OPEN ITEM: Detailed change management plan + pilot location selection

---

### Risk 5: Scope Creep & Feature Sprawl
**Risk:** Five phases invite mission creep; teams add "nice-to-haves" that delay core value delivery.

**Impact:** Medium | **Probability:** High

**Mitigation:**
- Ruthless phase gate discipline: Each phase has clear exit criteria; features not meeting criteria move to next phase
- Phase 2 (native mobile) is non-negotiable priority — must launch Q4 2026; Phase 3+ features deferred if needed
- Steering committee: Executive review of scope changes; "must be-haves" only per phase
- Scope baseline documentation: This PRD serves as single source of truth; feature requests require executive approval + reprioritization

---

### Risk 6: User Expectation Mismatch (Bar Set Too High)
**Risk:** Marketing promises "seamless native-first experience" inviting comparison to Apple-grade polish. Any friction amplifies disappointment.

**Impact:** Medium | **Probability:** Medium

**Mitigation:**
- Under-promise marketing: Focus on functional benefits (fast fitment, integrated scheduling) vs. "seamless" messaging
- Beta testing: Pilot with loyalist members (Phase 2) to gather honest feedback before public launch
- Quality gate: NPS >50 required for Phase 2 launch (vs. NPS 30-40 currently)
- Continuous improvement: Post-launch roadmap focused on friction reduction based on user feedback
- OPEN ITEM: Beta testing strategy + user acceptance criteria

---

### Risk 7: Mobile App Performance Regression
**Risk:** Phase 2 native app performs poorly in production (crashes, slow loads); member reviews tank.

**Impact:** High | **Probability:** Low

**Mitigation:**
- Performance testing: Load testing (simulate 10x traffic), stress testing (edge cases), battery drain testing
- Staged rollout: Phase 2 launch to 10% of users first; expand to 50%, then 100% based on crash/performance metrics
- Monitoring: Real-time crash reporting, performance dashboards, automated alerts for regressions
- Rollback plan: Ability to rollback to previous app version within 1 hour of critical issue detection

---

### Dependency: Costco Identity Platform
- **Dependency:** Phase 1+ requires SSO integration with Costco member identity platform
- **Risk:** If identity platform changes or deprecates APIs, authentication breaks
- **Mitigation:** Establish API contract with identity platform team; ensure backward compatibility; plan API versioning

### Dependency: Warehouse Inventory System
- **Dependency:** Phase 3A requires real-time API access to warehouse inventory management system
- **Risk:** Inventory API doesn't support real-time queries; latency prevents accurate display
- **Mitigation:** Requirements gathering with warehouse operations; API design review; performance SLA contract

### Dependency: Tire Center Scheduling System
- **Dependency:** Phase 3B requires integration with tire center scheduling/appointment system
- **Risk:** Legacy scheduling system may not support API integration; requires custom bridge
- **Mitigation:** System assessment; if legacy system incompatible, plan for new scheduling platform deployment

---

## OPEN ITEMS REQUIRING RESOLUTION

### Go-to-Market Strategy

1. **Launch Timeline**
   - OPEN: Phase 1A/1B target launch dates (Q3 2026 assumed; confirm with exec sponsors)
   - OPEN: Phase 2 public launch timing (Q4 2026 assumed; tied to Phase 1A completion)
   - OPEN: Phase 3 feature sequencing (features 3A-3D prioritization)

2. **Rollout Approach**
   - OPEN: Phased by region? (e.g., West Coast first, then Mountain, then Central, then East?)
   - OPEN: Beta pilot locations for Phase 3 (5 pilot warehouses for scheduling integration)
   - OPEN: If phased, what's the expansion cadence? (1 month between regions? 2 weeks?)

3. **Marketing Activation**
   - OPEN: Phase 1 launch campaign strategy (web redesign announcement)
   - OPEN: Phase 2 launch campaign (native mobile app marketing)
   - OPEN: Member communication plan (when/how to announce new features)
   - OPEN: Competitive messaging (vs. TireRack, Discount Tire, SimpleTire)

4. **Success Definition**
   - OPEN: What metrics/milestones trigger go/no-go for Phase 2? (e.g., Phase 1A adoption >25%?)
   - OPEN: What success criteria must be met before Phase 3 commitment?
   - OPEN: Revenue targets per phase (if any)

---

### Success Metrics & Targets

1. **Conversion Rate Targets**
   - Baseline (Phase 1 discovery): TBD
   - Phase 2 target: +20-30% improvement
   - Phase 3 target: +35-50% improvement
   - OPEN: Confirm these targets with Finance + Product leadership

2. **Revenue Impact**
   - Tire sales revenue lift (Phase 2 +3 months): TBD
   - Tire sales revenue lift (Phase 3 +6 months): TBD
   - OPEN: Establish baseline; project revenue impact

3. **CLV & Retention Metrics**
   - Repeat purchase rate: Current 35% → Target 50% (Phase 5)
   - Repeat purchase cycle: Current 5.2 years → Target 4.8 years (Phase 5)
   - OPEN: Validate these targets; establish measurement methodology

4. **Operational Metrics**
   - Tire center no-show rate reduction: Target 30% reduction (Phase 3)
   - Tire center bay utilization: Target +15% increase (Phase 3)
   - OPEN: Establish current baselines; define measurement methods

---

### Budget & Resources

1. **Program Budget**
   - OPEN: Total investment across Phases 1-5 (assume $3-5M; confirm with Finance)
   - OPEN: Phase-specific budgets and resource allocation
   - OPEN: Ongoing operational costs (hosting, analytics, content licensing)

2. **Team Structure**
   - OPEN: Internal team (headcount, roles, reporting structure)
   - OPEN: Partner/vendor requirements (UX agency, development vendor, QA partner)
   - OPEN: Ongoing support team (Phase 5+: analytics, optimization, maintenance)

3. **ROI Expectations**
   - OPEN: Payback period (investment vs. revenue lift)
   - OPEN: NPV/IRR targets for project approval
   - OPEN: 3-year financial projection

---

### Stakeholder & Change Management

1. **Executive Sponsors**
   - OPEN: Who owns success? (likely Chief Digital Officer + VP Merchandising Tires)
   - OPEN: Decision authority for scope changes, budget approvals, phase gates

2. **Warehouse Operations Impact**
   - OPEN: How will tire center operations change? (staffing impact, training needs)
   - OPEN: Who leads change management with warehouse managers? (Operations or Digital team?)
   - OPEN: Incentive alignment: How do tire center staff benefit from digital scheduling?

3. **Pilot Warehouse Selection**
   - OPEN: Which 5 warehouses selected for Phase 3 scheduling pilot? (Recommend: diverse geographies, high-volume locations, early adopter cultures)

4. **Beta Testing Strategy**
   - OPEN: User acceptance criteria for Phase 2 native app launch
   - OPEN: Beta testing approach (opt-in member group vs. A/B testing vs. staged rollout)
   - OPEN: Feedback collection methodology (surveys, user testing, analytics)

---

### Content & Partnerships

1. **3rd Party Education Content (Phase 3C)**
   - OPEN: Content provider selection (which organization partners for tire guides?)
   - OPEN: Licensing model (per-content-piece fee? annual subscription? revenue share?)
   - OPEN: Content refresh frequency and update responsibilities
   - OPEN: Brand positioning in educational content (Costco vs. tire manufacturers)

2. **Content Maintenance Strategy**
   - OPEN: Maintenance responsibility (Costco vs. 3rd-party provider)
   - OPEN: Quality assurance process for content updates

---

## APPENDIX: SUPPORTING ARTIFACTS

### Design Assets Mapping

**Web App Designs (`/docs/images/web-app/`):**
- Homepage: Responsive entry points, vehicle selection, deals carousel
- Size Selection Page: Fitment validation, tire size guide overlay
- Products Listing Page: Filtered results, advanced filters, product comparison
- Product Details Page: Full specs, warranty, installation pricing, add-to-cart
- Common Functionality: Schedule appointment modal, tire center locator, change vehicle modal

**Mobile App Designs (`/docs/images/mobile-app/`):**
- Home Page: Vehicle selection, deals slider, value proposition
- Brands Ribbon: Manufacturer navigation
- Tires PLP: Product listing with filters and comparison
- Tires PDP: Product detail with specs and purchase actions

### Technical References

- "Infogain - Costco-Tires-Azure-To-GCP Migration-V5.pdf" (Phase 1B infrastructure)
- "Costco Tires & Batteries-Transforming to Native App.pdf" (Phase 2 mobile architecture)
- "Screenshots of Tires Design.pdf" (Phase 3D content navigation)

### Input Document Summary

- **Goals-Personas-UserJourneys.txt:** 7 detailed personas, 4 user journeys, 4 experience goals, persona-KPI matrix
- **product-brief-costco-tires.md:** Existing strategic brief with vision, problem, solution, scope, risks
- **docs/vision.txt:** Functional/non-functional requirements, primary purpose statement
- **docs/sow.txt:** 5-phase delivery plan, technical integration points
- **docs/bff-layer/project-context.md:** Technical architecture rules, implementation patterns

---

**PRD Status:** Complete BMAD-standard document ready for stakeholder review and approval.

**Next Steps:**
1. Review and approve PRD with executive sponsors
2. Resolve OPEN ITEMS (budget, timelines, stakeholder roles)
3. Begin Phase 1A development planning
4. Establish baseline metrics for success criteria

---

**Document Version:** 1.0 | **Last Updated:** 2026-04-16 | **Author:** Generated by BMAD PRD Workflow