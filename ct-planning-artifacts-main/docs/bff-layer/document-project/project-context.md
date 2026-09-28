---
project_name: 'Tires-Batteries 1'
user_name: 'Rupesh.Jain'
date: '2026-04-13'
sections_completed: ['technology_stack', 'language_rules', 'framework_rules', 'database_rules', 'testing_rules', 'code_quality_rules', 'critical_dont_miss_rules']
status: 'complete'
rule_count: 47
optimized_for_llm: true
existing_patterns_found: 12
---

# Project Context for AI Agents

_This file contains critical rules and patterns that AI agents must follow when implementing code in this project. Focus on unobvious details that agents might otherwise miss._

---

## Technology Stack & Versions

### Runtime & Framework
- .NET Framework **4.8** (classic full-framework — NOT .NET Core / .NET 5+)
- ASP.NET MVC **5** (Microsoft.AspNet.Mvc 5.2.4)
- Visual Studio 2022 (v17.2)

### Data
- SQL Server / Azure SQL — raw ADO.NET only (`Microsoft.Data.SqlClient 5.2.1`)
- Entity Framework **6.2.0** — used ONLY for ASP.NET Identity admin users
- StackExchange.Redis **2.0.601**

### Azure / Cloud
- Azure.Identity **1.11.3**
- Azure.Security.KeyVault.Secrets **4.6.0**
- Microsoft.ApplicationInsights **2.4.0** / .Web **2.4.1**
- Microsoft.Web.RedisSessionStateProvider-Bundled **2.2.0.5**

### Serialization / Utilities
- Newtonsoft.Json **11.0.1** (NOT System.Text.Json for domain serialization)
- HtmlAgilityPack **1.6.17**

### Identity / Security
- Microsoft.IdentityModel.Tokens **6.35.0**
- System.IdentityModel.Tokens.Jwt **6.35.0**

### Frontend (Costco web project)
- jQuery **3.3.1**, Bootstrap **3.0.0**, Moment.js **2.30.1 + Timezone 0.5.13**

### Testing
- MSTest (VS built-in) + Moq **4.18.4** + Castle.Core **5.1.1**

### External Integrations
- Mitchell1 (license plate / VIN lookup)
- Bing Maps
- Apigee (Costco inventory API)
- ContentStack (headless CMS)
- Kentico (CMS — REST API)

---

## Critical Implementation Rules

### Language-Specific Rules (C# / .NET 4.8)

#### Naming Conventions
- Classes, methods, properties, constants: **PascalCase**
- Local variables and parameters: **camelCase**
- Interfaces: always prefix with **`I`** (e.g., `ISearch`, `ITireInfo`, `IVehicleOperations`)
- Use `#region` blocks to organize large classes — this is the established pattern throughout

#### Namespaces
- Namespaces **mirror project/folder structure exactly** (max 3 levels deep)
- Pattern: `<ProjectName>.<SubFolder>.<SubFolder>` (e.g., `BusinessObjects.Library.Common`)
- Never create namespaces that don't map to a physical folder

#### Serialization
- Always use **Newtonsoft.Json** (`JsonConvert`, `[JsonProperty]`, etc.) — never `System.Text.Json`
- `System.Text.Json 4.7.2` is present only as a transitive dependency — do not use it directly

#### Async / Await
- The codebase is **predominantly synchronous** — do not introduce async without explicit need
- Never use `.Result` or `.Wait()` on async methods (sync-over-async anti-pattern exists in legacy code — do not replicate it)
- Async classes/controllers follow the `*Async` naming suffix (e.g., `WarehousesAsync`, `HttpBaseClientAsync`)

#### Error Handling
- **Universal pattern** — wrap every public method body:
  ```csharp
  try
  {
      // ...
  }
  catch (Exception ex)
  {
      Logger.LogError(ex,
          MethodBase.GetCurrentMethod().DeclaringType.ToString(),
          MethodBase.GetCurrentMethod().Name);
      return null; // or default(T)
  }
  ```
- Always import `System.Reflection` to access `MethodBase.GetCurrentMethod()`
- Return `null` (or zero/empty) on catch — never rethrow or swallow silently

#### Attributes
- Apply `[ExcludeFromCodeCoverage]` on every new class added to Costco, BusinessObjects, DataAccess, and Integration projects — this is the established codebase-wide convention

### Framework-Specific Rules (ASP.NET MVC 5)

#### Controller Rules
- All controllers inherit from **`BaseController`** (not directly from `Controller`)
- `BaseController` provides: culture/locale resolution, warehouse cookie read/write, Redis session access — use these instead of reimplementing
- Apply `[OutputCache(NoStore = true, Duration = 0)]` on every action method — HTTP-level caching is disabled; all caching is done explicitly in the business layer
- Never access `DataAccess` or database directly from controllers — always go through `BusinessObjects`
- Use `[Route("...")]` attribute routing for new action methods; keep consistent with existing routes

#### Dependency Creation (No IoC Container)
- This project has **no IoC/DI container** — never use constructor injection or `IServiceProvider`
- Always create dependencies via the static factory:
  - Tire domain: `FactoryCreator.GetSearchObject()`, `FactoryCreator.GetVehicleOperations()`, etc.
  - Battery domain: `BatteryFactoryCreator.Get*()` equivalents
- Each factory method returns the interface type — always assign to the interface, not the concrete type

#### MVC Areas
- Batteries functionality lives in **`Costco.Areas.Batteries`** — new battery features must be placed in this area
- Tires functionality lives in the main `Costco` project controllers/views

#### Configuration Access
- **Never call `ConfigurationManager.AppSettings[...]` directly** in controllers or business objects
- All `AppSettings` reads must go through the **`WebConfigKeys` static class** in `DataAccess`
- Add new config keys as static properties to `WebConfigKeys`, not inline reads
- Connection strings for domain data come from Azure Key Vault (via `DbKeyVaultManager`) in production; `Web.config` in local/dev

#### Localization / Multi-Region
- Supported locales: **en-US** (US), **en-CA** (Canada English), **fr-CA** (Canada French)
- Culture is resolved in `BaseController` from the `PreferredCulture` cookie and `DomainCultureMapping.xml`
- Never hardcode locale-specific strings in controllers or views — use `Resources` / `ResourcesBattery` projects
- Country (US vs Canada) is derived from culture — always check `BaseController` properties, never parse URLs manually

#### Routing & URLs
- URL construction helpers live in `IURLOperations` / `URLOperations` — use these, never build URLs by string concatenation
- Route config is in `App_Start/RouteConfig.cs` — conventional routes + attribute routes coexist

#### Caching Strategy
- **In-process cache** (`HttpRuntime.Cache`): vehicle lookups, general config, warehouse lists — short-lived
- **Redis** (`RedisCacheHelper`): session state and cross-server shared data
- Cache TTL values are configured in `AppSettings` (`CacheExpiration`, `CacheExpirationFrontEnd`, `RedisTimeOut`) — read via `WebConfigKeys`, never hardcode durations
- Cache keys follow a pattern — look at existing `RedisCacheHelper` usage before adding new cache entries

### Database Access Rules

#### Core Rule: Stored Procedures Only
- **ALL domain data access uses stored procedures** — never write inline SQL, LINQ-to-SQL, or use EF for domain data
- `CommandType.StoredProcedure` is always set — never `CommandType.Text` for domain queries
- Entity Framework 6 is present **only** for ASP.NET Identity (admin user management) — do not use it for any tire/battery/warehouse/vehicle data

#### ADO.NET Pattern
- Use the project's `SqlHelper` / `DataAccessHelper` utilities — do not create raw `SqlConnection` / `SqlCommand` objects in business object classes
- Build parameters as `SqlParameter[]` arrays and pass to `ExecuteDataset` or equivalent helper methods
- All data access classes live in the **`DataAccess`** project — never put SQL calls in `BusinessObjects` directly

#### Connection Strings
- Never hardcode connection strings
- Local / dev: read from `Web.config` `<connectionStrings>` section
- Production (Azure): retrieved from **Azure Key Vault** via `DbKeyVaultManager` (toggled by `ReadFromKeyVault=1` in `AppSettings`)
- Always access the connection string through `DbKeyVaultManager` / `WebConfigKeys` — never `ConfigurationManager.ConnectionStrings[...]` inline

#### Stored Procedure Naming
- Existing SP names are inconsistent (`API_GetWarehouseInfo`, `API_SP_GetWarehouseInfo_v1`, `Availability_Invetry_data`)
- For new SPs, follow the pattern: `API_SP_<Domain>_<Verb>` (e.g., `API_SP_Tire_GetByWarehouse`)
- All SP definitions belong in the **`CWC_Tires`** SQL Server Database Project (`.sqlproj`)

#### Database-Driven Configuration
- Runtime feature flags and config values live in the `Mst_GeneralConfiguration` table
- Fetched via `Configurations.SetDatabaseCostcoWebConfigKey()` — cached in `HttpRuntime.Cache`
- Never read `Mst_GeneralConfiguration` directly; use the existing config abstraction

#### DataSet / DataTable Handling
- Data access layer returns `DataSet` — business objects extract rows via `DataSet.Tables[0].Rows`
- Always null-check the `DataSet` and `Tables[0]` before accessing rows
- Map `DataRow` fields to model properties manually — no auto-mapping libraries

### Testing Rules

#### Test Framework
- **MSTest** (Visual Studio built-in) — use `[TestClass]`, `[TestMethod]`, `[TestInitialize]`, `[TestCleanup]`
- **Moq 4.18.4** for mocking — always mock via the interface (`Mock<ISearch>`, not `Mock<Search>`)
- All test classes belong in the **`Costco.Tests`** project

#### Test Structure
- Test class naming: `<ClassUnderTest>Test` (e.g., `HomeControllerTest`)
- Test method naming: `<MethodName>_<Scenario>_<ExpectedResult>` (e.g., `GetTires_ValidVehicle_ReturnsResults`)
- One `[TestClass]` per production class under test

#### Mocking Conventions
- Use `FactoryCreator` interfaces as mock targets — never mock concrete classes
- Setup mocks in `[TestInitialize]` method, not inline in each test
- Use `Mock<T>.Object` to pass mocked instances to constructors or method calls

#### Coverage Expectations
- Most production classes carry `[ExcludeFromCodeCoverage]` — **do not remove this attribute** from existing classes
- Focus new tests on business logic in `BusinessObjects` — controllers and `DataAccess` classes are intentionally excluded
- Aim to test the interface contract (inputs → outputs), not internal implementation details

#### What NOT to Test
- Do not write tests for `DataAccess` classes — SQL/stored procedure calls are not unit-testable in this codebase
- Do not write tests for `ErrorLogging` or `ApplicationFactory` static helpers
- Do not write integration tests that require live DB, Redis, or Key Vault connections

### Code Quality & Style Rules

#### Code Analysis
- **SonarLint** is configured via `.sonarlint\costcotires_redesigncsharp.ruleset` — all builds enforce these rules
- Do not suppress SonarLint warnings inline without a documented reason
- Run SonarLint analysis before committing new code

#### Assembly Signing
- All projects use **Strong Name signing** (`.snk` files) — every new class library project must include a `.snk` file and set `<SignAssembly>true</SignAssembly>` in its `.csproj`
- Never change or delete existing `.snk` files

#### Project Organization
- New classes in a project go in a subfolder matching their logical domain (e.g., `Battery/`, `Configurations/`, `Admin/`)
- Interfaces in `ApplicationInterfaces` mirror the same subfolder structure as their implementations
- Models in `ApplicationModels` mirror the same subfolder structure

#### Constants
- Domain constants (DB column names, SP names, config key strings) belong in dedicated constants classes (`DBConstants`, `CommonUtility`, `WebConfigKeys`)
- Never use magic strings for config keys, SP names, or cache keys inline in methods

#### Frontend (Razor Views)
- Use Bootstrap **3.x** class conventions — not Bootstrap 4/5
- jQuery **3.3.1** is the only JS library guaranteed present — do not add new JS framework dependencies without explicit approval
- Use Moment.js for all date/time formatting in JavaScript

#### Logging
- Use **`Logger.LogInfo()`** for informational traces and **`Logger.LogError()`** for exceptions — no other logging calls
- Application Insights telemetry is automatic via the SDK; no manual `TelemetryClient` calls needed for basic request tracking
- Never log sensitive data (PII, connection strings, tokens)

### Critical Don't-Miss Rules

#### Architecture Anti-Patterns — NEVER DO THESE
- **Never skip tiers**: Controllers must not call `DataAccess` directly; `BusinessObjects` must not call `HttpClient` / external APIs directly — use `Integration` project adapters
- **Never use EF for domain data**: Entity Framework is for `AspNetUsers` / Identity only — any tire, battery, warehouse, or vehicle data query must use raw ADO.NET + stored procedures
- **Never use `System.Text.Json`** for domain serialization — always `Newtonsoft.Json`
- **Never call `ConfigurationManager` directly** in controllers or business objects — always `WebConfigKeys`
- **Never introduce a DI container** (Autofac, Unity, Microsoft.Extensions.DI, etc.) — the static factory pattern is intentional and must be preserved

#### Security Gotchas
- **Cookie `HttpOnly`** is deliberately set to `false` on some cookies (JavaScript-readable) — do not blindly set `HttpOnly=true` on all cookies; check `BaseController` cookie logic first
- **SameSite cookie rewriting** is handled in `App_Start/SameSiteCookieRewriter.cs` for cross-site scenarios — do not add duplicate SameSite logic elsewhere
- **Never store secrets in `Web.config` for production** — DB connection strings and API keys must go through Azure Key Vault; use `WebConfigKeys.ReadFromKeyVault` flag pattern
- **Never log request parameters or user data** — Application Insights auto-collects telemetry; avoid adding manual traces that could capture PII

#### Data Access Gotchas
- **Always null-check `DataSet` returns** before accessing `.Tables[0]` — a failed SP call returns `null`, not an empty `DataSet`
- **Redis connection is a singleton** (`RedisCacheHelper` uses lazy `ConnectionMultiplexer`) — never create a new `ConnectionMultiplexer` per request
- **Key Vault reads are cached in-memory** with sliding expiration — do not call `DbKeyVaultManager` in a tight loop; it is designed for infrequent reads

#### Multi-Region / Localization Gotchas
- **Canada has two locales** (en-CA and fr-CA) — any feature that works for en-US must be verified for both Canadian locales
- **Domain-to-culture mapping** is in `DomainCultureMapping.xml` — if adding a new environment URL, update this file
- **Resource strings for batteries** are in `ResourcesBattery`, not `Resources` — using the wrong resource project will cause missing string errors

#### Build & Deployment Gotchas
- **8 Azure build configurations** exist (AzureDev, AzureQA2, AzureStaging, AzureUAT, AzurePreProd, AzureProduction) — any new `Web.config` key must have a corresponding XDT transform entry in **every** `Web.{env}.config` file
- **Assembly references are signed** — adding a new reference to an unsigned assembly will break the strong-name build
- **`ApplicationInsights.config`** instrumentation key is set programmatically in `Global.asax.cs` — do not set it in the config file directly for Azure environments

#### Performance Gotchas
- **Do not call `HttpRuntime.Cache` and Redis for the same data** — pick one layer per data type and be consistent
- **Synchronous HTTP calls** (`.Result` on `GetAsync`) exist in legacy integration clients — do not replicate this pattern; use `await` properly in any new async integration code
- **`HtmlAgilityPack`** is used for HTML parsing — do not use regex for HTML parsing

---

## Usage Guidelines

**For AI Agents:**
- Read this file before implementing any code in this project
- Follow ALL rules exactly as documented
- When in doubt, prefer the more restrictive option
- Update this file if new patterns emerge during implementation

**For Humans:**
- Keep this file lean and focused on agent needs
- Update when technology stack or patterns change
- Review quarterly for outdated rules
- Remove rules that become obvious over time

_Last Updated: 2026-04-13_
