# Project Documentation Index

_Costco Tires & Batteries — Generated: 2026-04-13 | Scan Level: Deep_

---

## Project Overview

- **Type:** Monolith (.NET Framework 4.8 / ASP.NET MVC 5)
- **Primary Language:** C#
- **Architecture Pattern:** Layered Monolith with strict tier separation
- **Database:** Azure SQL Server (stored procedures only, raw ADO.NET)
- **Solution Projects:** 15 projects (1 MVC web app + 13 class libraries + 1 SQL project)
- **Markets:** en-US (USA), en-CA (Canada English), fr-CA (Canada French)
- **Domains:** Tires (main), Batteries (area), Admin (area)
- **Migration:** Strangler Fig → Java 17 / Spring Boot 3.2 BFF (133 APIs, 20-week plan)

---

## Quick Reference

| Item | Detail |
|---|---|
| Entry point | `Costco/Global.asax.cs` → `MvcApplication.Application_Start()` |
| Base controller | `Costco/Controllers/BaseController.cs` (tires) |
| Battery base | `Costco/Areas/Batteries/Controllers/BaseBatteryController.cs` |
| Object factory | `ApplicationFactory/FactoryCreator.cs` |
| All config keys | `DataAccess/WebConfigKeys.cs` |
| DB connection | `DataAccess/DbKeyVaultManager.cs` (Key Vault) or `Web.config DefaultConnection` |
| Redis singleton | `BusinessObjects/CommonHelper/RedisCacheHelper.cs` |
| Error logging | `ErrorLogging/Logger.cs` |
| CMS toggle | `WebConfigKeys.IsContentstackActive` → ContentStack or Kentico |
| Test project | `Costco.Tests/` (MSTest + Moq 4.18.4) |

---

## Documentation

### Core Docs

| Document | Description |
|---|---|
| [project-overview.md](./project-overview.md) | Project summary, purpose, markets, tech stack |
| [architecture.md](./architecture.md) | Full architecture: patterns, tiers, security, caching, deployment |
| [source-tree-analysis.md](./source-tree-analysis.md) | Annotated directory tree for all 15 projects |
| [development-guide.md](./development-guide.md) | Local setup, build configs, testing, deployment |

### API & Data Docs

| Document | Description |
|---|---|
| [api-contracts.md](./api-contracts.md) | All MVC routes, AJAX endpoints, external integration contracts |
| [data-models.md](./data-models.md) | Database tables (139), domain models, key stored procedures |

### Functional Specifications

| Document | Description |
|---|---|
| [deep-dive-functional-specification.md](./deep-dive-functional-specification.md) | Comprehensive functional specification — all domains: every controller flow, business logic, promotions engine, CMS integration, external integrations, caching architecture, localization, and all business rules |
| [deep-dive-user-personas-and-user-journeys.md](./deep-dive-user-personas-and-user-journeys.md) | User personas (6) and user journeys (9) — covers all user types (US member, CA member, Quebec French, battery shopper, QR technician, admin) with step-by-step journey maps, technical touchpoints, friction points, and UX implications |

### AI Agent Rules

| Document | Description |
|---|---|
| [../_bmad-output/project-context.md](../_bmad-output/project-context.md) | Critical implementation rules for AI agents (47 rules) |

### Migration Planning

| Document | Description |
|---|---|
| [../costco_tires_bff_api_summary.md](../costco_tires_bff_api_summary.md) | BFF migration: Java Spring Boot, 133 APIs, 15 domains, 20-week plan |

---

## Getting Started

### For Developers

1. Read [architecture.md](./architecture.md) for system understanding
2. Follow [development-guide.md](./development-guide.md) for local setup
3. Review [source-tree-analysis.md](./source-tree-analysis.md) to understand project layout
4. Check [api-contracts.md](./api-contracts.md) before adding new routes

### For AI Agents

1. **Always read** [../_bmad-output/project-context.md](../_bmad-output/project-context.md) before writing any code
2. Reference [architecture.md](./architecture.md) for tier boundaries
3. Reference [data-models.md](./data-models.md) when working with database or domain models
4. Reference [api-contracts.md](./api-contracts.md) when working with controllers or routes

### For New Features

- **Tire feature**: Add controller in `Costco/Controllers/`, extend `BaseController`, use `FactoryCreator`
- **Battery feature**: Add controller in `Costco/Areas/Batteries/Controllers/`, extend `BaseBatteryController`, use `BatteryFactoryCreator`
- **New DB query**: Define SP in `CWC_Tires` project, add `DataAccess` class, add `BusinessObjects` class, add interface to `ApplicationInterfaces`, register in factory
- **New config key**: Add to `Web.config`, `WebConfigKeys.cs`, AND ALL 6 `Web.{env}.config` XDT transforms

---

## Key Architecture Decisions

| Decision | Rationale |
|---|---|
| No IoC container | Legacy design; static factory pattern throughout |
| Stored procedures only (domain data) | Performance + DBA control; no ORM for domain queries |
| EF 6 for Identity only | Admin user management only |
| ContentStack/Kentico feature flag | CMS migration underway; switchable at runtime |
| Two-tier caching | In-process (HttpRuntime) + distributed (Redis) for different data lifetimes |
| Hard-coded admin credentials | Legacy; must be replaced — see `LoginController.cs` |

_Last Updated: 2026-04-13_
