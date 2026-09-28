# Development Guide — Costco Tires & Batteries

_Generated: 2026-04-13_

---

## Prerequisites

| Requirement | Version | Notes |
|---|---|---|
| **.NET Framework** | **4.8** | Classic full framework — NOT .NET Core or .NET 5+ |
| **Visual Studio** | 2022 (v17.2+) | Required; VS Code does not support fully |
| **SQL Server** | 2019+ or Azure SQL | See database setup below |
| **Redis** | 5.x+ | For session state and caching; can use Azure Redis Cache |
| **Node.js** (optional) | LTS | Only needed for `npx bmad-method` tooling |
| **Azure CLI** (optional) | Latest | For Azure Key Vault access in dev if `ReadFromKeyVault=1` |

---

## Local Setup

### 1. Clone & Open Solution

```
git clone <repo>
cd "Tires-Batteries 1"
Costco.sln  ← open with Visual Studio 2022
```

### 2. NuGet Package Restore

Visual Studio auto-restores NuGet packages on solution open, or run:
```
nuget restore Costco.sln
```
All packages are in `packages/` folder (referenced locally).

### 3. Database Setup

1. Create SQL Server database (e.g., `CostcoTiresDev`)
2. Execute scripts in order:
   - `Database-Table-Script.sql` — creates all 139 tables
   - `Databse-StoredProcedure-script.sql` — creates all stored procedures
3. Optionally restore a data backup for reference data (tire catalog, configurations, vehicle fitment)

### 4. Redis Setup

For local dev, install Redis locally (Windows via WSL, or use Docker):
```bash
docker run -d -p 6379:6379 redis:latest
```

### 5. Web.config Configuration

Open `Costco/Web.config` and update these critical keys:

```xml
<connectionStrings>
  <add name="DefaultConnection"
       connectionString="Server=localhost;Database=CostcoTiresDev;..." />
</connectionStrings>

<appSettings>
  <!-- Set to 0 to use DefaultConnection from Web.config (not Key Vault) -->
  <add key="ReadFromKeyVault" value="0" />

  <!-- Redis connection string -->
  <add key="RedisCacheConnectionString" value="localhost:6379" />

  <!-- CMS Mode: false = use Kentico, true = use ContentStack -->
  <add key="IsContentstackActive" value="false" />

  <!-- ContentStack credentials (if using ContentStack) -->
  <add key="ContentStackApiKey" value="..." />
  <add key="ContentStackAccessToken" value="..." />

  <!-- Kentico REST API (if using Kentico) -->
  <add key="RestAPI" value="https://your-kentico-host/api/..." />
  <add key="BasicAuth" value="Basic base64encodedcredentials" />
</appSettings>
```

> **Important:** Never commit real credentials to `Web.config`. Use environment-specific XDT transforms for Azure deployments.

### 6. Build & Run

1. Set `Costco` as the startup project
2. Press **F5** (Debug) or **Ctrl+F5** (Run without debugger)
3. App starts at `https://localhost:44300` (or configured IIS Express port)

---

## Build Configurations

| Configuration | Target | Notes |
|---|---|---|
| `Debug` | Local | Full debug info, no optimization |
| `Release` | Local / CI | Optimized, no debug symbols |
| `AzureDev` | Azure Dev | XDT transforms `Web.AzureDev.config` |
| `AzureQA2` | Azure QA2 | XDT transforms `Web.AzureQA2.config` |
| `AzureStaging` | Azure Staging | XDT transforms `Web.AzureStaging.config` |
| `AzureUAT` | Azure UAT | XDT transforms `Web.AzureUAT.config` |
| `AzurePreProd` | Azure Pre-Prod | XDT transforms `Web.AzurePreProd.config` |
| `AzureProduction` | Azure Production | XDT transforms `Web.AzureProduction.config` |

**Critical:** When adding a new `<appSettings>` key, you MUST add the corresponding XDT `<add>` or `<xdt:Transform>` entry in ALL 6 Azure `Web.{env}.config` files or the key will be missing in deployed environments.

---

## Testing

### Running Tests

```
# In Visual Studio:
Test > Run All Tests (Ctrl+R, A)

# Or via CLI:
vstest.console.exe "Costco.Tests\bin\Debug\Costco.Tests.dll"
```

### Test Framework

- **MSTest** (built-in Visual Studio test runner)
- **Moq 4.18.4** for mocking
- Test project: `Costco.Tests/`

> **Note:** Most production classes have `[ExcludeFromCodeCoverage]`. Current test coverage is near-zero. The `HomeControllerTest.cs` is fully commented out. Focus new tests on `BusinessObjects` logic only.

---

## Key Development Commands / Tasks

### Adding a New AppSettings Key

1. Add to `Costco/Web.config` `<appSettings>`
2. Add static property to `DataAccess/WebConfigKeys.cs`
3. Add XDT transform entry to all 6 `Web.{env}.config` files
4. Use the property via `WebConfigKeys.YourNewKey` — never call `ConfigurationManager.AppSettings` directly

### Adding a New Tire Controller

1. Create `ControllerName.cs` in `Costco/Controllers/`
2. Inherit from `BaseController` (NOT `Controller`)
3. Use `FactoryCreator.GetXxx()` to obtain business objects
4. Apply `[OutputCache(NoStore = true, Duration = 0)]` on all action methods
5. Apply `[ExcludeFromCodeCoverage]`
6. Wrap all method bodies in the standard `try/catch/Logger.LogError` pattern

### Adding a New Battery Controller

1. Create in `Costco/Areas/Batteries/Controllers/`
2. Inherit from `BaseBatteryController`
3. Use `BatteryFactoryCreator.GetXxx()` for business objects
4. Follow same patterns as tire controllers

### Adding a New Business Object

1. Define interface in `ApplicationInterfaces/`
2. Implement in `BusinessObjects/`
3. Register in static factory: `ApplicationFactory/FactoryCreator.cs` or `BatteryFactoryCreator.cs`
4. Add model to `ApplicationModels/` if needed

### Adding a New Stored Procedure

1. Define SP in `CWC_Tires` SQL project
2. Add data access method in `DataAccess/`
3. Call via `DataAccessHelper.ExecuteDataset(sp, CommandType.StoredProcedure, params)`
4. Build corresponding business object method in `BusinessObjects/`
5. Name pattern for new SPs: `API_SP_<Domain>_<Verb>`

---

## Deployment (Azure)

### CI/CD

- Deployments use Azure DevOps pipelines
- Eight named configurations map to eight Azure environments
- Build uses MSBuild with configuration parameter: `msbuild Costco.sln /p:Configuration=AzureProduction`

### Azure Key Vault (Production Only)

Set in `Web.AzureProduction.config`:
```xml
<add key="ReadFromKeyVault" value="1" />
<add key="AzKeyVaultUrl" value="https://your-keyvault.vault.azure.net/" />
<add key="Az_KV_DBConnectionStringSecret" value="db-connection-string-secret-name" />
```
The app uses `DefaultAzureCredential` (Azure Managed Identity) to authenticate to Key Vault — no credentials needed in config for production.

### Environment Web.config Transforms

XDT transform files handle environment-specific overrides for:
- Database connection strings
- Redis connection string
- API endpoints (Kentico, ContentStack, Apigee, Warehouse API)
- Feature flags (`IsContentstackActive`, `ReadFromKeyVault`, `ActivateSameSite`)
- App Insights instrumentation key

---

## Code Quality Tools

- **SonarLint**: `.sonarlint/costcotires_redesigncsharp.ruleset` — enforced across all build configs
- **Application Insights**: Automatic request/dependency/exception telemetry in Azure environments
- **System.Diagnostics.Trace**: All `Logger.LogError()`/`Logger.LogInfo()` calls write to Trace listeners

---

## Common Troubleshooting

| Issue | Cause | Fix |
|---|---|---|
| Redis connection error on startup | Redis not running | Start Docker Redis container or set valid `RedisCacheConnectionString` |
| Blank tire results | Missing DB reference data | Run the SP scripts and seed tire catalog / fitment tables |
| 500 on AppSettings key | Missing key in Web.config | Add key to `WebConfigKeys.cs` + `Web.config` |
| CMS content not loading | ContentStack/Kentico not configured | Set `IsContentstackActive` and matching credential keys |
| Assembly load error at startup | Strong-name mismatch | Ensure all `.snk` files are intact; don't add unsigned assembly references |
| Culture not resolving | `selectedLanguage` cookie missing | App defaults to `en-US`; test with `CultureCode` query param |
