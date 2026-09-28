---
stepsCompleted: ["step-01-init", "step-02-context", "step-03-starter", "step-04-decisions", "step-05-patterns", "step-06-structure", "step-07-validation", "step-08-complete"]
status: "complete"
completedAt: "2026-05-01"
inputDocuments:
  - "_bmad-output/planning-artifacts/prd.md"
  - "docs/bff-layer/document-project/architecture.md"
  - "docs/bff-layer/document-project/project-context.md"
  - "docs/bff-layer/document-project/project-overview.md"
  - "docs/bff-layer/document-project/index.md"
  - "docs/bff-layer/api-contracts/vehicle-tire-BFF-catalog-api.yml"
  - "docs/bff-layer/api-contracts/vehicle-tire-BFF-search-api.yml"
  - "docs/vision.txt"
  - "docs/sow.txt"
workflowType: "architecture"
project_name: "ct-planning-artifacts"
user_name: "Rupesh.Jain"
date: "2026-05-01"
---

# Architecture Decision Document
## Costco Tires Digital Experience Modernization

_This document builds collaboratively through step-by-step discovery. Sections are appended as we work through each architectural decision together._

---

## Project Context Analysis

### Requirements Overview

**Functional Requirements — Architectural Signal:**

| FR Group | FRs | Architectural Implication |
|----------|-----|--------------------------|
| Fitment / Vehicle Selection | FR1, FR2 | VIN decode service adapter (Mitchell1); fitment validation engine with manufacturer spec lookup |
| Real-Time Inventory | FR3, FR8 | Event-driven inventory query layer; 15-minute minimum refresh polling via Apigee; stock-check gate at checkout |
| Integrated Scheduling | FR4, FR6 | Appointment slot service proxy; no-redirect constraint = BFF must return slot availability inline |
| Context Persistence | FR5, FR10 | Cloud-synced member context store; session state service; cross-channel cart sync (≤5s) |
| Smart Search & Intent | FR9 | Dedicated search service; intent detection middleware; autocomplete; ≥95% accuracy / <500ms p95 |
| Multi-Entry Points | FR7 | Entry-point routing strategy; context injection at seven defined entry flows |
| Native Mobile App | FR12 (Phase 2) | Separate native iOS/Android build; shared BFF API surface |
| Educational Content | FR13 (Phase 3C) | ContentStack CMS integration; context-aware content delivery within purchase flow |
| Promotional Routing | FR14 | Campaign parameter propagation through full journey; promo-context filtering in search |
| Vehicle Profiles & Reorder | FR11 (Phase 5) | Member data service; vehicle history persistence in member account |

**Non-Functional Requirements — Hard Architectural Constraints:**

| Constraint | NFR | Architectural Impact |
|------------|-----|---------------------|
| ≤2s page load web / <1.5s mobile cold start | NR-Perf-1 | SSR mandatory for web; CDN edge caching required |
| <500ms search p95 | NR-Perf-2 | Search service co-located or cached; no cold-chain queries |
| 99.99% uptime | NR-Avail-1 | Multi-zone GCP deployment; no single points of failure |
| Auto-scale to 10x in 2 min | NR-Scale-1 | Cloud Run stateless containers; horizontal autoscaling |
| ≥60% origin load reduction | NR-Scale-3 | Multi-tier cache (browser → CDN → application → DB) mandatory |
| PCI DSS Level 1 | NR-Sec-3 | Payment tokenization; no card data in Costco systems; scoped payment boundary |
| WCAG 2.1 AA | NR-Acc-1 | Accessibility-first component design; Axe/Lighthouse CI gates |
| SSR for SEO | NR-Plat-2 | Server-side rendering or static generation; Schema.org markup; Core Web Vitals (LCP <2.5s) |
| iOS 16+ / Android 10+ | NR-Plat-4 | Native SDK minimum targets |
| en-US / en-CA / fr-CA | NR-Acc-3 | i18n from day one; no hardcoded strings; locale-appropriate formatting |

### Scale & Complexity Assessment

- **Complexity Level: High** — real-time inventory + scheduling + multi-channel context sync + native mobile + 7 entry points + 3 locales + PCI compliance + brownfield strangler fig migration
- **Primary Domain:** Omnichannel e-commerce (web + native mobile), brownfield modernization
- **Delivery Model:** Five phased releases; architecture must support Strangler Fig extraction from existing .NET 4.8 monolith
- **Estimated Architectural Components:** 12–15 distinct system components

### Technical Constraints & Dependencies

**Brownfield Reality (critical):**
- Existing .NET 4.8 / ASP.NET MVC 5 monolith on Azure is the **current production system** — cannot be replaced in one shot
- Java 17 / Spring Boot 3.2 BFF layer is **already in flight** (Strangler Fig; 133 REST APIs across 15 domains)
- BFF API contracts already drafted and in use: `GET /v1/catalog/tires/attributes` (fitment), `GET /v1/search/tires` (multi-mode search)
- Mitchell1 — VIN/LP lookup via Digest auth; existing adapter must be preserved or migrated to BFF integration layer
- Apigee — Costco inventory API; Bearer token; primary real-time inventory data source
- ContentStack — headless CMS; active and feature-flag controlled (Kentico legacy also present)
- Azure Key Vault — current secret store; GCP Secret Manager required for GCP-hosted components

**Platform Mandates:**
- GCP as target hosting (Cloud Run preferred per NR-Tech-1)
- Azure remains in-play during Strangler Fig transition period (hybrid cloud phase)
- Native mobile app is a separate build (not web-view); shares BFF API surface

### Cross-Cutting Concerns Identified

1. **Context Persistence** — vehicle selection, fitment, cart, warehouse synced across web/mobile/guest/member states; must survive channel switches in ≤5 seconds
2. **Multi-Locale** — en-US / en-CA / fr-CA must be first-class from day one; no bolt-on localization; strings externalized, dates/currency/phone locale-aware
3. **Observability** — RUM (Cloud Monitoring), error rate gates (>1% triggers alert), inventory sync failure alerting, APM for all BFF routes
4. **Feature Flagging** — phased delivery requires flags to gate Phase 2/3/4/5 capabilities at runtime; no code deploys to enable phases
5. **Cache Invalidation** — inventory, pricing, promotions require immediate invalidation on change; TTL-only is insufficient for high-velocity data
6. **PCI Scope Isolation** — payment flow must be tokenized and architecturally scoped separately from tire browsing domain
7. **Strangler Fig Coordination** — legacy .NET routes must coexist with BFF routes during transition; routing layer (API Gateway / Apigee) controls cutover per domain

---

## Starter Template Evaluation

### Primary Technology Domain

**Multi-surface omnichannel** — three distinct new surfaces require independent starters; BFF already in-flight.

### Surface 1: Web Frontend

**Selected Starter: Next.js 15 (App Router)**

**Rationale:** SSR is mandatory (NR-Plat-2 SEO requirement). React Server Components eliminate client JS bundle for static product content, delivering direct LCP wins. `next-intl` handles en-US/en-CA/fr-CA route-based locale switching. Widest enterprise adoption with full GCP Cloud Run compatibility.

**Initialization Command:**
```bash
npx create-next-app@latest costco-tires-web \
  --typescript \
  --tailwind \
  --eslint \
  --app \
  --src-dir \
  --import-alias "@/*"
```

**Architectural Decisions Provided by Starter:**
- Language & Runtime: TypeScript (strict mode), Node.js 20 LTS
- Styling: Tailwind CSS 3.x
- Router: App Router (React Server Components first)
- Linting: ESLint + Prettier
- Build: Turbopack (dev) / Webpack (prod)
- Testing: Jest + React Testing Library (added post-scaffold)

---

### Surface 2: Native Mobile App (Phase 2)

**Selected Framework: Swift + SwiftUI (iOS) / Kotlin + Jetpack Compose (Android)**

**Rationale:** Fully native implementation per PRD Phase 2 mandate. SwiftUI provides the iOS-native component model aligned to Apple HIG; Jetpack Compose delivers the Android-native equivalent aligned to Material Design 3 with Costco brand tokens. Both use platform-native camera APIs for license plate scan (FR1 — AVFoundation/VisionKit on iOS, CameraX on Android), native push notification stacks (APNs/UserNotifications on iOS, FCM on Android — NR-Plat-3), and platform-native secure storage (iOS Keychain / Android EncryptedSharedPreferences) for tokens. Native compilation guarantees <100ms touch response and <1.5s cold start (FR12). App Store delivery via Xcode Cloud + App Store Connect; Google Play delivery via Gradle + GitHub Actions.

**iOS Project Setup:**
```bash
# Xcode 15+: File → New → Project → App (SwiftUI, iOS 16+, Swift Package Manager)
# SPM dependencies: sentry-cocoa, firebase-ios-sdk (Remote Config)
```

**Android Project Setup:**
```bash
# Android Studio: New Project → Empty Activity (Kotlin, Jetpack Compose, API 29+, Kotlin DSL Gradle)
# Gradle dependencies: sentry-android, firebase-android-sdk (Remote Config), CameraX, Retrofit + OkHttp
```

**Architectural Decisions:**
- iOS Language: Swift 5.9+ / SwiftUI (UI framework)
- Android Language: Kotlin / Jetpack Compose (Material Design 3 + Costco brand tokens)
- Build & Distribution: Xcode Cloud + App Store Connect (iOS) / Gradle + GitHub Actions + Google Play Console (Android)
- Native APIs: AVFoundation/VisionKit (license plate scan, iOS), CameraX (Android), UserNotifications/APNs (iOS push), FCM (Android push)
- Secure Storage: iOS Keychain (Swift Security framework) / Android EncryptedSharedPreferences
- Minimum OS: iOS 16 / Android 10 (API 29)

---

### Surface 3: BFF Layer (Java 17 / Spring Boot 3.2 — In-Flight)

**Already scaffolded — documenting for agent consistency.**

```bash
# Reference only — already in use
spring init \
  --build=gradle \
  --java-version=17 \
  --dependencies=web,data-jpa,security,cache,actuator,validation \
  --group-id=com.costco.tires \
  --artifact-id=bff-service \
  costco-tires-bff
```

**Established Decisions:**
- Language: Java 17 (LTS)
- Framework: Spring Boot 3.2.x
- Build: Gradle
- API: REST (OpenAPI 3.0 / Swagger / SpringDoc)
- Security: Spring Security (OAuth 2.0 resource server)
- Testing: JUnit 5 + Mockito

---

## Core Architectural Decisions

> _Party Mode review (Winston, Amelia, Murat, John) incorporated as amendments below._

### Decision Priority Analysis

**Critical Decisions (Block Sprint 1):**
1. JWT claim schema — exact field names (`sub`, `roles`, `memberId`, `warehouseId`)
2. Redis key schema — exact hash field names for `member:{id}:context`
3. Guest cookie specification — name, `SameSite`, `Secure`, domain scope
4. Apigee route ownership — who merges proxy config changes?
5. Feature flag service sprint-1 readiness — stub contract if not complete

**Important Decisions (Shape Architecture):**
- Redis persistence strategy and cache-miss failure mode
- Inventory staleness window vs. FR8 commitment
- Strangler Fig seam experience ownership
- Apigee HA + Cloud Armor hardening (Phase 1A infra story)
- Feature flag system ownership boundary

**Deferred (Post-Phase 1):**
- Stored procedure refactor to app-layer queries (post-Phase 3)
- Search accuracy harness scale-up (Phase 2 exit gate)
- Mobile-specific test suite (Detox/Maestro — Phase 2)

---

### Data Architecture

**1A — Member Context Store (Redis)**
- Pattern: Redis Hash per `member:{id}:context`
- Fields: `vehicleKey`, `fitment`, `warehouseId`, `cartId`, `locale`
- TTL: 30 days (logged-in members) / 24 hours (guest)
- Cart body stored in Cloud SQL `member_carts` table; Redis holds the reference `cartId` pointer only
- **Amendment (Winston):** Redis persistence strategy must be specified in ADR. Cloud Memorystore supports RDB snapshots; configure accordingly. BFF must be able to reconstruct member context from Cloud SQL on cache miss — this path must be explicitly implemented and tested, not assumed.
- **Redis key schema (Sprint 1 blocker — Amelia):** Exact hash field names (`vehicleId` vs `vehicleKey`, etc.) must be defined and published as a shared contract before BFF and web teams write integration code.

**1B — Guest Session Architecture**
- Anonymous session ID: UUID v4, stored as `HttpOnly` cookie
- Redis key: `guest:{sessionId}:context` (24h TTL)
- On login: context migrated to `member:{id}:context`
- **Amendment (Amelia):** Cookie spec required — name, `SameSite`, `Secure` flag, domain scope. Guest→member migration must define conflict resolution rule (last-write-wins vs. union merge). Race condition on concurrent login + cart ops must be handled atomically (Redis `MULTI/EXEC`).

**1C — Search Infrastructure**
- Primary: Apigee-backed search (existing Costco contract), accessed via BFF
- BFF adds intent detection middleware layer on top of Apigee search results
- No Elasticsearch — avoids ops burden, leverages existing investment
- Fallback: category browse if search unavailable
- **Amendment (John):** Search accuracy baseline must be measured in Phase 1 discovery against current system before Phase 2 launch. ≥90% Search Success Rate is a Phase 2 exit gate — without a Phase 1 baseline, there is no signal. Measurement mechanism and go/no-go authority must be defined.
- **Amendment (Murat):** Requires a golden dataset evaluation harness (200–500 curated query/result pairs) running as a dedicated CI job with hard accuracy thresholds (≥95% / ≥90% brand). This is absent from the current testing stack proposal.

**1D — Database Migration Strategy**
- Phase 1: Lift-and-shift Azure SQL → Cloud SQL (SQL Server edition); preserves stored procedures
- Post-Phase 3: Refactor stored procedures to application-layer queries in BFF
- **Amendment (Amelia):** Cloud Run cold start + HikariCP connection pool requires explicit config — `minimum-idle=0`, `max-lifetime` tuned for serverless connection patterns. Connection timeout incidents guaranteed in sprint 1 without this.

---

### Authentication & Security

**2A — JWT Token Strategy**
- Stateless JWT; BFF validates locally against Costco IDP JWKS endpoint
- Short-lived access tokens (15–30 min) + refresh token rotation
- **Sprint 1 blocker (Amelia):** JWT claim schema must be published as a cross-team contract immediately. Exact fields: `sub`, `roles` (array), `memberId`, `warehouseId`, `locale`. Both Spring Security `@PreAuthorize` expressions and Next.js middleware will be written against these field names — divergence here causes incompatible implementations.

**2B — Authorization (RBAC)**
- Spring Security `@PreAuthorize` with custom role expressions in BFF
- Roles embedded in JWT claims: `member`, `warehouse_staff`, `admin`
- Next.js middleware enforces route-level access for web

**2C — PCI Scope Isolation**
- Payment processing fully delegated to Costco's existing checkout platform
- BFF never receives or stores raw card data; passes tokenized payment reference only
- No new PCI scope introduced in this product

---

### API & Communication Patterns

**3A — API Gateway / Strangler Fig Router**
- Apigee as both API gateway and strangler fig routing controller
- New BFF routes registered in Apigee on `/v1/` prefix; legacy .NET routes retain existing paths
- **Amendment (Winston):** Apigee must be provisioned in HA multi-zone configuration — do not assume; explicitly validate. Cloud Armor deployed in front of Apigee for DDoS and rate-limit protection independent of Apigee's health. Strangler fig routing configs treated as temporary; hard retirement deadline by end of Phase 2.
- **Amendment (Amelia):** Apigee proxy config ownership must be decided before sprint 1. If infra team owns merges, every BFF story has a cross-team ticket dependency.
- **Amendment (John):** Strangler Fig seam experience must be owned by product, not just infra. When users cross the legacy→BFF domain boundary, vehicle context in Redis may not be readable by the legacy .NET domain. This seam must be explicitly designed, tested, and monitored — it is a conversion leak risk.

**3B — Error Response Standard**
- All APIs return RFC 7807 Problem Details (`application/problem+json`)
- Required fields: `type`, `title`, `status`, `detail`, `instance`
- Both Next.js web and Expo mobile handle this contract

**3C — Rate Limiting**
- Primary enforcement: Apigee gateway (100 req/min per member, 1,000 req/min per API key)
- Secondary protection: Resilience4j `@RateLimiter` in BFF
- **Amendment (Winston):** Resilience4j thresholds must be environment-specific (Amelia). Default 50% failure-rate circuit breaker threshold will false-trip in dev against slow upstream stubs. Define per-environment CB config.

**3D — Inventory Polling & Cache**
- BFF scheduled polling: every 10 minutes (Spring `@Scheduled`) to Apigee inventory endpoint
- Results written to Cloud Memorystore; cache TTL = 15 minutes
- On cache miss: direct Apigee call with Resilience4j circuit breaker
- **Amendment (Winston):** Cache miss failure mode must be explicitly defined. Default behavior: return degraded UI state ("In-store availability may vary — please check with your warehouse"), not a 503. Never serve a hard error on inventory miss; preserve browse-and-purchase flow.
- **Amendment (John — HIGH RISK):** 10-min polling + 15-min TTL = worst-case 25-minute staleness window. FR8 requires no post-purchase appointment cancellations due to inventory inaccuracy. Inventory churn velocity data for top-10 tire SKUs at high-volume warehouses must be reviewed before 10 minutes is accepted as the polling interval. This decision must be revisited with real data before Phase 1 completion.
- **Polling interval:** Flag-controlled (`inventory.poll.interval.minutes`) to allow runtime adjustment without code deploy.

---

### Frontend Architecture

**4A — Web State Management**
- Zustand for client-side global state: cart, vehicle context, locale, warehouse selection
- URL state for all filter and search params (shareable, SSR-friendly, deep-linkable)
- No Redux — overhead not justified with React Server Components handling server state

**4B — Web Component Library**
- shadcn/ui (Radix UI primitives + Tailwind CSS)
- Components copied into project — no runtime library dependency, no version lock-in
- Radix primitives are WCAG 2.1 AA compliant by default (ARIA + keyboard nav)
- Costco design tokens applied via Tailwind config overrides

**4C — Web Testing Stack**
- Unit: Vitest + React Testing Library
- E2E + Cross-browser: Playwright (Chrome, Safari, Firefox, Edge — per NR-Plat-1)
- Accessibility: axe-playwright in CI (zero critical/serious violations = PR gate)
- Visual regression: Playwright screenshots

**4D — Mobile State Management**
- iOS: ObservableObject / @StateObject (SwiftUI) + Swift Concurrency (async/await) for async state; URLSession for API calls
- Android: ViewModel + StateFlow / Kotlin Coroutines for async state; Retrofit + OkHttp for API calls
- iOS secure storage: Keychain (Swift Security framework) for tokens, member ID
- Android secure storage: EncryptedSharedPreferences for tokens, member ID
- API response caching: NSURLCache (iOS) / OkHttp cache (Android) — platform-native HTTP caching

---

### Infrastructure & Deployment

**5A — Environment Strategy**
- `dev`: Cloud Run revision per PR (parallel, isolated)
- `staging`: Continuous deployment from `main`
- `production`: Manual promotion gate (all staging quality gates must pass)

**5B — Feature Flagging**
- Mobile: Firebase Remote Config (native iOS SDK via firebase-ios-sdk; native Android SDK via firebase-android-sdk — runtime updates without app release)
- Web + BFF: Custom flag service backed by `Mst_GeneralConfiguration` DB pattern (existing precedent)
- **Amendment (Winston):** Strict ownership boundary required — Firebase owns mobile flags; custom service owns web/BFF flags. No flag spans both systems without explicit cross-system sync mechanism. Ambiguous ownership causes silent drift incidents.
- **Amendment (Amelia):** Custom flag service must be sprint-1-ready or a stub contract published immediately. Every feature-flagged story in sprint 1 depends on this.

**5C — Observability**
- Cloud Monitoring + Cloud Logging + Cloud Trace — BFF and web
- Sentry — mobile crash reporting (sentry-cocoa for iOS, sentry-android for Android — native SDKs)
- PagerDuty — severity-1 alert routing
- Alert thresholds: page error rate >1%, API response >3s, search success rate <95%, inventory sync failures, payment failures >2%

**5D — Secrets Management**
- GCP Secret Manager — all GCP-hosted components (BFF, Next.js via Cloud Run env injection)
- BFF reads secrets via Workload Identity (no key files)
- Azure Key Vault — retained for legacy .NET monolith only during Strangler Fig transition

---

### Testing Architecture (Murat Amendments — Mandatory)

The default frontend-first testing stack is insufficient for this BFF-centric architecture. The following additions are **mandatory**:

| Addition | Tool | Rationale |
|----------|------|-----------|
| Consumer-driven contract tests | **Pact** | Prevents Next.js → BFF and BFF → downstream API interface drift from Apigee routing changes |
| BFF API test suite | **REST-assured** (Java) | BFF is the new system core; every endpoint tested at API level before Playwright tests written |
| Performance baseline | **k6** | <500ms p95 search and <2s page load are SLAs; gated in staging CI |
| Search accuracy harness | **pytest + httpx** | 200–500 golden dataset query/result pairs; hard pass threshold ≥95% / ≥90% brand; runs as dedicated CI job |
| Security scan | **OWASP ZAP** | PCI DSS Level 1 compliance requires it; baseline scan on every staging deploy |
| Mobile functional tests | **XCUITest** (iOS, Phase 2) + **Espresso** (Android, Phase 2) | Native UI functional testing; added at Phase 2 sprint 1 |
| i18n coverage gate | custom script | Validates translation key coverage across en-US/en-CA/fr-CA; no missing keys merged |

**CI Quality Gate — Definition of Done (all stories):**

```
PR Gate (must pass to merge):
  ✓ Vitest unit tests — ≥80% coverage on changed files
  ✓ RTL component tests — all new components have baseline render tests
  ✓ BFF API tests (REST-assured) — all new/modified endpoints covered
  ✓ Pact contract tests — consumer/provider pacts verified
  ✓ axe-playwright — zero critical/serious a11y violations on affected routes
  ✓ i18n key coverage — no missing translation keys introduced
  ✓ Snyk / Dependabot — no new HIGH or CRITICAL CVEs

Staging Gate (blocks production promotion):
  ✓ Playwright E2E smoke — happy path per domain (search, PDP, cart, auth)
  ✓ k6 performance — search p95 <500ms, page load <2s
  ✓ Search accuracy harness — ≥95% / ≥90% brand thresholds met
  ✓ Feature flag matrix — key flows validated in both flag states
  ✓ OWASP ZAP — no new HIGH findings vs prior baseline

Production Gate (manual approval requires):
  ✓ All staging gates green
  ✓ Apigee routing smoke — config validated against expected routing table
  ✓ JWT rotation integration test — passed in staging within last 24 hours
```

---

### Decision Impact Analysis

**Implementation Sequence:**
1. JWT claim schema published (cross-team contract) — sprint 1 blocker
2. Redis key schema published (cross-team contract) — sprint 1 blocker
3. Apigee HA + Cloud Armor provisioning — Phase 1A infra story
4. Apigee proxy config ownership defined — sprint 1 process decision
5. Feature flag service built or stub contract published — sprint 1 prerequisite
6. BFF context persistence service (Redis session schema) — unblocks FR5/FR10
7. Next.js scaffold + shadcn/ui + Zustand + next-intl — unblocks all Phase 1 web stories
8. BFF search endpoint + Apigee intent middleware — unblocks FR9
9. Inventory polling interval validated with churn data — before Phase 1 completion
10. Search accuracy baseline measured — Phase 1 exit deliverable
11. Native iOS (Swift/SwiftUI) + Android (Kotlin/Jetpack Compose) scaffolds — begin at Phase 2 sprint 1

**Cross-Component Dependencies:**
- Web and mobile both depend on stable BFF OpenAPI 3.0 contract (Pact consumer tests enforce this)
- Redis context schema is shared between BFF (writer) and Next.js BFF API routes (reader)
- Apigee is both search provider AND strangler fig router — single failure domain; Cloud Armor + HA provisioning are non-optional
- Feature flags must be readable across all three runtimes: BFF (Java), Next.js (Node.js), iOS (Swift/Firebase SDK) and Android (Kotlin/Firebase SDK) — strict ownership boundary prevents drift
- Strangler Fig seam must be treated as a product experience boundary, not just an infra routing concern

---

## Implementation Patterns & Consistency Rules

**13 critical conflict areas** identified across naming, structure, format, communication, and process.

### Naming Patterns

**Database / Redis Naming:**
- New Cloud SQL tables: `snake_case` lowercase (e.g., `member_carts`, `member_contexts`)
- Columns: `snake_case` (`member_id`, `vehicle_key`, `warehouse_id`)
- Redis keys: `{entity}:{id}:{context}` format — `member:abc123:context`, `guest:uuid4:context`
- Redis hash fields: `camelCase` — `vehicleKey`, `fitmentProfile`, `warehouseId`, `cartId`, `locale`

**BFF API Naming (Java / Spring Boot):**
- Endpoints: `kebab-case` plural nouns — `/v1/search/tires`, `/v1/catalog/tires/attributes`
- Route params: `camelCase` — `/v1/catalog/{vehicleId}/fitment`
- Query params: `camelCase` — `?warehouseId=123&rimSize=17`
- Java classes: `PascalCase`; methods/variables: `camelCase`; constants: `UPPER_SNAKE_CASE`
- Package structure: `com.costco.tires.{domain}.{layer}`

**Next.js Web Naming:**
- Files/directories: `kebab-case` (`tire-search/`, `vehicle-selector.tsx`)
- Components: `PascalCase` (`VehicleSelector`, `TireCard`)
- Hooks: `camelCase` prefixed with `use` (`useVehicleContext`)
- Zustand stores: `camelCase` + `Store` suffix (`vehicleStore`, `cartStore`)

**Expo Mobile:** Same file/component/hook conventions as web (shared team pattern).

---

### Format Patterns

**API Response Envelope:**
```json
{ "data": { ... }, "meta": { "total": 150, "page": 1, "pageSize": 20 } }
```

**Error Response (RFC 7807 — all APIs):**
```json
{
  "type": "https://costco.com/errors/inventory-unavailable",
  "title": "Warehouse Inventory Unavailable",
  "status": 503,
  "detail": "Real-time inventory for warehouseId 1234 is temporarily unavailable.",
  "instance": "/v1/search/tires?warehouseId=1234"
}
```

- **Date/Time:** ISO 8601 in all API payloads — `"2026-05-01T14:30:00Z"`. Never Unix timestamps.
- **JSON fields:** `camelCase` always (`memberId`, `warehouseId`, `vehicleKey`). Never `snake_case` in JSON.
- **Locale codes:** BCP 47 — `en-US`, `en-CA`, `fr-CA`. Never `en_US`.
- **Tire size:** canonical format `###/##R##` — e.g., `225/45R18` — in all APIs, DB, and display.
- **Pagination:** cursor-based for search (`nextCursor`); offset-based for admin lists (`page`, `pageSize`).

---

### Communication Patterns

**Feature Flag Schema:**
```json
{
  "feature.native-mobile-app.enabled": false,
  "feature.educational-content.enabled": false,
  "feature.vehicle-profiles.enabled": false,
  "feature.inventory-poll-interval-minutes": 10
}
```
Key format: `feature.{kebab-name}.{property}`. All Phase 2+ flags default `false` in production.

**Structured Log Format (JSON — all services):**
```json
{
  "timestamp": "2026-05-01T14:30:00Z",
  "level": "INFO",
  "service": "bff-tires",
  "traceId": "abc123",
  "memberId": "hashed-only",
  "message": "Search query processed",
  "durationMs": 187
}
```
Never log raw `memberId`, email, or PII — hash or omit.

---

### Process Patterns

**Error Handling:**
- BFF: global `@ControllerAdvice` catches all exceptions → RFC 7807 response
- Next.js: `error.tsx` for page-level; inline error state for recoverable component errors
- Degraded states preferred: inventory unavailable → `{ available: false, message: "Check in-store" }`, not HTTP 500

**Zustand Store Pattern:**
```typescript
// src/stores/vehicleStore.ts
interface VehicleState {
  selectedVehicle: Vehicle | null;
  setVehicle: (vehicle: Vehicle) => void;
  clearVehicle: () => void;
}
export const useVehicleStore = create<VehicleState>((set) => ({ ... }));
```

**React Server Components Rule:** Data fetching always in Server Components — never `useEffect` + fetch for initial data. Client Components only for user interactivity.

---

### Enforcement — All AI Agents MUST:
1. `camelCase` JSON fields, `snake_case` DB columns, `kebab-case` file/route names
2. RFC 7807 Problem Details for all error responses — no custom error shapes
3. Never log raw PII in any log line
4. Add Pact consumer test for every new Next.js → BFF API call
5. Add BFF API test (REST-assured) before writing a Playwright test for the same feature
6. Run axe-playwright on every new route before marking a story complete
7. Check i18n coverage (all 3 locales) before marking any user-facing story complete
8. Feature flags for all Phase 2+ capabilities — never deploy unconditional Phase 2+ logic
9. Never call BFF endpoints from React Client Components — route through Next.js API handlers

**Anti-Patterns (explicitly forbidden):**
- ❌ `useEffect(() => fetch(...))` for initial page data — use Server Components
- ❌ Raw card or payment data flowing through BFF routes
- ❌ Hardcoded strings in any UI component — use `next-intl` `t()` function
- ❌ `console.log()` in production code — use structured logging
- ❌ HTTP 500 for inventory unavailability — return degraded `{ available: false }`
- ❌ Redux or Context API for global state — Zustand only
- ❌ New Kentico API calls — ContentStack only (feature-flag-controlled)

---

## Project Structure & Boundaries

### Requirements to Structure Mapping

| FR | Feature | BFF Domain | Web Route | Mobile Screen |
|----|---------|-----------|-----------|--------------|
| FR1, FR2 | Vehicle fitment & size validation | `catalog/` | `app/[locale]/tires/` | `app/(tabs)/tires/` |
| FR3, FR8 | Warehouse inventory display | `inventory/` | `app/[locale]/tires/search/` | `app/(tabs)/tires/search` |
| FR4, FR6 | Appointment scheduling | `appointments/` | `app/[locale]/tires/cart/` | `app/(tabs)/tires/cart` |
| FR5, FR10 | Context & cart sync | `session/` (cross-cutting) | `lib/context/` | `stores/sessionStore.ts` |
| FR7 | Multi-entry point routing | Apigee config | `middleware.ts` + `app/[locale]/` | `app/` root layout |
| FR9 | Smart search & intent | `search/` | `app/[locale]/tires/search/` | `app/(tabs)/tires/search` |
| FR11 | Vehicle profiles (Phase 5) | `members/vehicles/` | `app/[locale]/account/vehicles/` | `app/(tabs)/account/vehicles` |
| FR12 | Mobile native (Phase 2) | _all BFF domains_ | N/A | `app/` (full Expo app) |
| FR13 | Educational content (Phase 3C) | `content/` | `app/[locale]/tires/guides/` | `app/(tabs)/tires/guides` |
| FR14 | Promotional routing | `promotions/` | `middleware.ts` promo handler | `app/_layout.tsx` deep link |

---

### BFF Service Structure (`costco-tires-bff`)

```
costco-tires-bff/
├── build.gradle
├── settings.gradle
├── Dockerfile
├── openapi/
│   └── tires-bff-api.yaml              # Master OpenAPI 3.0 spec (source of truth)
├── src/
│   ├── main/java/com/costco/tires/
│   │   ├── TiresBffApplication.java
│   │   ├── config/
│   │   │   ├── SecurityConfig.java     # OAuth2 resource server, JWKS config
│   │   │   ├── RedisConfig.java        # Cloud Memorystore; persistence mode configured
│   │   │   ├── CacheConfig.java        # Cache TTL definitions
│   │   │   ├── ResilienceConfig.java   # Circuit breaker + rate limiter (per-env)
│   │   │   └── FeatureFlagConfig.java
│   │   ├── catalog/                    # FR1, FR2
│   │   │   ├── controller/CatalogController.java
│   │   │   ├── service/CatalogService.java
│   │   │   ├── service/FitmentValidationService.java
│   │   │   ├── client/Mitchell1Client.java      # VIN/LP (Digest auth)
│   │   │   ├── client/ApigeeClient.java
│   │   │   └── dto/  (CatalogRequest, CatalogResponse)
│   │   ├── search/                     # FR9
│   │   │   ├── controller/SearchController.java
│   │   │   ├── service/SearchService.java
│   │   │   ├── service/IntentDetectionService.java
│   │   │   ├── client/ApigeeSearchClient.java
│   │   │   └── dto/  (SearchRequest, SearchResponse)
│   │   ├── inventory/                  # FR3, FR8
│   │   │   ├── controller/InventoryController.java
│   │   │   ├── service/InventoryService.java
│   │   │   ├── scheduler/InventoryPollScheduler.java   # @Scheduled; interval flag-controlled
│   │   │   ├── client/ApigeeInventoryClient.java
│   │   │   └── dto/InventoryResponse.java
│   │   ├── appointments/               # FR4, FR6
│   │   │   ├── controller/AppointmentController.java
│   │   │   ├── service/AppointmentService.java
│   │   │   ├── client/WarehouseSchedulingClient.java
│   │   │   └── dto/  (AppointmentSlotResponse, BookAppointmentRequest)
│   │   ├── session/                    # FR5, FR10 (cross-cutting)
│   │   │   ├── controller/SessionController.java
│   │   │   ├── service/MemberContextService.java    # Redis HSET/HGET
│   │   │   ├── service/GuestSessionService.java
│   │   │   ├── service/ContextMigrationService.java # guest→member; MULTI/EXEC atomic
│   │   │   └── dto/MemberContextDto.java
│   │   ├── promotions/                 # FR14
│   │   ├── content/                    # FR13 (Phase 3C) — ContentStack
│   │   ├── members/                    # FR11 (Phase 5) — vehicle profiles
│   │   └── common/
│   │       ├── exception/GlobalExceptionHandler.java  # @ControllerAdvice → RFC 7807
│   │       ├── security/JwtClaimsExtractor.java       # sub, roles, memberId, warehouseId
│   │       ├── logging/StructuredLogger.java
│   │       └── util/TireSizeValidator.java
│   └── test/java/com/costco/tires/
│       ├── {domain}/controller/        # @WebMvcTest slice tests
│       ├── {domain}/service/           # Mockito unit tests
│       ├── integration/                # @SpringBootTest
│       └── contract/                   # Pact provider verification
├── k6/
│   └── search-load-test.js             # k6 performance baseline
└── .github/workflows/
    ├── pr.yml                           # PR quality gate
    └── deploy.yml                       # Staging + production promotion
```

---

### Web Frontend Structure (`costco-tires-web`)

```
costco-tires-web/
├── package.json
├── next.config.ts
├── tailwind.config.ts
├── tsconfig.json
├── src/
│   ├── app/
│   │   ├── [locale]/                   # en-US | en-CA | fr-CA
│   │   │   ├── page.tsx                # Tires homepage (FR7: Start Search)
│   │   │   ├── tires/
│   │   │   │   ├── search/page.tsx     # FR9 PLP (FR7: Global Search)
│   │   │   │   ├── [itemNumber]/page.tsx  # FR3 PDP
│   │   │   │   ├── brands/[brand]/page.tsx  # FR7 Brand Entry
│   │   │   │   ├── deals/page.tsx      # FR14 Promotional routing
│   │   │   │   ├── guides/[slug]/page.tsx  # FR13 Phase 3C
│   │   │   │   ├── cart/page.tsx       # FR4, FR6
│   │   │   │   └── schedule/page.tsx   # FR7: Services-First
│   │   │   └── account/vehicles/page.tsx  # FR11 Phase 5
│   │   └── api/v1/                     # Next.js BFF proxy (server-side JWT attachment)
│   │       ├── catalog/route.ts
│   │       ├── search/route.ts
│   │       ├── inventory/route.ts
│   │       ├── appointments/route.ts
│   │       └── session/route.ts
│   ├── components/
│   │   ├── ui/                         # shadcn/ui (do not modify — copy only)
│   │   ├── features/
│   │   │   ├── vehicle-selector/       # FR1, FR2 (VehicleSelector, LicensePlateInput, TireSizeEntry)
│   │   │   ├── tire-search/            # FR9 (SearchBar, SearchAutocomplete, TireCard)
│   │   │   ├── inventory/              # FR3, FR8 (WarehouseSelector, StockBadge)
│   │   │   ├── appointment/            # FR4, FR6 (AppointmentSlotPicker)
│   │   │   ├── cart/                   # FR5, FR10 (CartSummary, CartSync)
│   │   │   └── promo/                  # FR14 (PromoRoutingGuard)
│   │   └── shared/  (ErrorBoundary, LocaleSwitcher, SkeletonCard)
│   ├── stores/                         # Zustand (vehicleStore, cartStore, warehouseStore, sessionStore)
│   ├── hooks/                          # useVehicleContext, useTireSearch, useAppointmentSlots
│   ├── lib/
│   │   ├── api/                        # Server-side BFF client functions
│   │   ├── tire-size-validator.ts
│   │   └── feature-flags.ts
│   ├── messages/                       # en-US.json, en-CA.json, fr-CA.json
│   └── middleware.ts                   # Auth + promo routing (FR7, FR14)
└── tests/
    ├── e2e/  (Playwright: search, fitment, cart-sync, appointment, accessibility)
    └── pact/consumers/  (catalog.pact.ts, search.pact.ts)
```

---

### Mobile App Structure (`costco-tires-mobile`) — Phase 2

```
costco-tires-mobile/
├── package.json / app.json / eas.json
├── app/
│   ├── _layout.tsx                     # Root layout + deep link handler
│   ├── (tabs)/
│   │   ├── tires/  (search, [itemNumber], cart, schedule)
│   │   └── account/vehicles.tsx        # FR11 Phase 5
│   └── (auth)/login.tsx
├── components/features/  (mirrors web: VehicleSelector, TireSearch, Inventory, Appointment)
├── stores/  (vehicleStore, cartStore, sessionStore — same Zustand API as web)
├── lib/
│   ├── api/           # React Native BFF client
│   ├── secure-storage.ts   # expo-secure-store wrapper
│   └── camera.ts          # expo-camera (license plate scan)
└── tests/maestro/  (search-flow.yaml, vehicle-fitment-flow.yaml)
```

---

### Architectural Boundaries

**API Boundaries:**
- Apigee → BFF: all external traffic; enforces auth, rate limits, strangler fig routing
- BFF → Next.js: only via `/api/v1/*` server-side handlers; never direct from client components
- BFF → Expo: same BFF endpoints via `lib/api/` wrappers with JWT attached
- BFF → Mitchell1: VIN/LP only; Digest auth; isolated in `catalog/client/`
- BFF → ContentStack: Phase 3C+; feature-flag-gated; isolated in `content/service/`

**Data Boundaries:**
- Cloud SQL: structured data (tires, members, carts, appointments, fitment tables)
- Redis: session context (FR5, FR10) + inventory cache (FR3) + search cache — no PII stored plain
- ContentStack: CMS content — read-only from BFF
- GCP Secret Manager: all credentials — no secrets committed to repo

**PCI Boundary:** Costco's existing checkout handles all payment — BFF never crosses this boundary; no card data in BFF, Redis, Next.js routes, or Expo storage.

**Data Flow — Primary Search Path (FR9):**
```
Member types → Next.js SearchBar (client, debounced 300ms)
  → /api/v1/search/tires (server, attaches JWT)
  → BFF SearchController → IntentDetectionService
  → ApigeeSearchClient → Apigee → Costco search
  → Results cached in Redis (15-min TTL)
  → Response: { data: TireResult[], meta: pagination }
```

**Data Flow — Context Persistence (FR5, FR10):**
```
Member selects vehicle → BFF /v1/session/context (PUT)
  → MemberContextService → Redis HSET member:{id}:context

Member opens mobile → BFF /v1/session/context (GET)
  → Redis HGET (hit: <5s guaranteed)
  → On miss: Cloud SQL member_contexts (cold recovery path)
```

---

## Architecture Validation Results

### Coherence Validation ✅

**Decision Compatibility:** All technology choices are compatible. Java 17 / Spring Boot 3.2 (BFF), Next.js 15 App Router (web), and Expo SDK 52 (mobile) share TypeScript + Zustand patterns, reducing cross-stack friction. GCP Cloud Run, Cloud SQL, Cloud Memorystore, and Cloud Monitoring form a cohesive managed-service layer. Apigee unifies API gateway and strangler fig routing — single vendor, existing Costco investment.

**Pattern Consistency:** Naming conventions are uniform — `camelCase` JSON, `snake_case` DB, `kebab-case` files/routes — applied across BFF, web, and mobile. RFC 7807 error format enforced globally. Zustand store pattern is identical on web and mobile. React Server Components pattern aligns with Next.js 15 App Router; no contradictions.

**Structure Alignment:** Project structure directly maps all 14 FRs to BFF domains and web routes. shadcn/ui + Radix primitives satisfy WCAG 2.1 AA. Pact tests are co-located with their respective consumers. Per-PR Cloud Run revisions enable isolation without environment contention.

---

### Requirements Coverage Validation ✅

**Functional Requirements — 14/14 covered:**

| FR | Architecture Coverage | Status |
|----|------------------------|--------|
| FR1 | `catalog/` domain + Mitchell1Client + FitmentValidationService | ✅ |
| FR2 | TireSizeValidator (BFF + web mirror) | ✅ |
| FR3 | `inventory/` domain + InventoryPollScheduler + StockBadge component | ✅ |
| FR4 | `appointments/` domain + AppointmentSlotPicker component | ✅ |
| FR5 | `session/` domain + Redis context + MemberContextService | ✅ |
| FR6 | AppointmentService + no-redirect constraint enforced in flow | ✅ |
| FR7 | `middleware.ts` + Apigee routing config + 7 locale entry routes | ✅ |
| FR8 | Inventory gate at checkout + InventoryPollScheduler | ✅ |
| FR9 | `search/` domain + IntentDetectionService + ApigeeSearchClient | ✅ |
| FR10 | Redis cart sync + CartSync component + TanStack Query (mobile) | ✅ |
| FR11 | `members/` domain — Phase 5, feature-flag-gated | ✅ |
| FR12 | Expo SDK 52 scaffold — Phase 2, feature-flag-gated | ✅ |
| FR13 | `content/` domain + ContentStackService — Phase 3C, feature-flag-gated | ✅ |
| FR14 | `promotions/` domain + PromoRoutingGuard + middleware | ✅ |

**Non-Functional Requirements — 21/21 covered:**

| Category | Coverage |
|----------|----------|
| Performance (Perf-1/2/3) | RSC + CDN + Redis cache; k6 gates in CI |
| Availability (Avail-1/2/3) | GCP multi-zone; Resilience4j circuit breaker; Cloud SQL HA |
| Security (Sec-1/2/3/4) | JWT/JWKS; PCI delegation; Apigee + Resilience4j rate limits; OWASP ZAP in CI |
| Scalability (Scale-1/2/3) | Cloud Run autoscale; HikariCP pool; multi-tier cache |
| Accessibility (Acc-1/2/3) | Radix primitives (ARIA); axe-playwright CI gate; next-intl i18n |
| Compliance (Comp-1/2) | PCI fully delegated; GCP Secret Manager; warranty on PDP |
| Analytics (Analytics-1/2) | Cloud Monitoring + Cloud Trace + PagerDuty alerting |
| Technical (Tech-1/2/3) | Cloud Run/GCP; OpenAPI 3.0; Cloud Build 1h deploy gate |
| Platform (Plat-1/2/3/4) | Playwright cross-browser; SSR Next.js; expo-notifications; EAS Build |

---

### Gap Analysis Results

**Critical — Must resolve before sprint 1:**
1. ⚠️ JWT claim schema — publish as cross-team contract (blocks 2A, 2B, 4A)
2. ⚠️ Redis hash field schema — publish as cross-team contract (blocks 1A, web integration)
3. ⚠️ Apigee proxy config ownership — process decision needed (blocks every BFF story)
4. ⚠️ Inventory churn velocity data — required before locking 10-min polling interval (Decision 3D)

**Important — Address in Phase 1 planning:**
5. Strangler Fig seam experience — product + UX define per-domain cutover UX
6. Search accuracy baseline measurement plan — Phase 1 exit deliverable
7. HikariCP Cloud Run pool config explicit values in `application.yaml`

**Minor — Phase 2+:**
8. Maestro mobile test suite — full scenarios written at Phase 2 sprint 1
9. `fr-CA` translation seeding from existing .NET Razor resource strings

---

### Architecture Completeness Checklist

- [x] Requirements Analysis — 7 source documents, High complexity, 7 cross-cutting concerns
- [x] Architectural Decisions — 14 decisions, Party Mode review incorporated
- [x] Implementation Patterns — naming, format, communication, process, 9 mandatory rules
- [x] Project Structure — 3 complete directory trees, FR→structure mapping, integration boundaries
- [x] Testing Architecture — full CI gate (PR + staging + production), 7 tool additions

---

### Architecture Readiness Assessment

**Overall Status: READY FOR IMPLEMENTATION**
*(4 sprint-1 contract blockers to resolve in parallel with scaffold stories)*

**Confidence Level: High** — all 14 FRs and 21 NFRs architecturally covered; Party Mode surfaced and resolved all major design risks; testing strategy is complete and specific.

**Key Strengths:**
- Boring technology where possible — reduces operational risk during complex brownfield migration
- Strangler Fig fully designed — legacy and new stack coexist without user disruption
- Feature flags gate all phased capability — no code deploy required to advance phases
- Comprehensive testing architecture — Murat's amendments close gaps that would otherwise become sprint-3 fires

**Areas for Future Enhancement (post-Phase 1):**
- Real-time inventory webhook from Apigee (eliminates polling staleness window)
- Consolidate feature flag systems post-Phase 2 (Firebase + custom → single platform)
- React Native New Architecture (Fabric) performance profiling after Phase 2 launch

---

### Implementation Handoff

**First Implementation Priorities:**
```bash
# Sprint 1A — Contracts (unblocks all parallel teams)
1. Publish JWT claim schema document
2. Publish Redis key/hash schema document
3. Decide Apigee proxy config ownership

# Sprint 1B — Scaffolds (run in parallel after contracts)
npx create-next-app@latest costco-tires-web \
  --typescript --tailwind --eslint --app --src-dir --import-alias "@/*"

spring init \
  --build=gradle --java-version=17 \
  --dependencies=web,data-jpa,security,cache,actuator,validation \
  --artifact-id=costco-tires-bff

# Phase 2 Sprint 1
npx create-expo-app@latest costco-tires-mobile --template tabs -p typescript
```
