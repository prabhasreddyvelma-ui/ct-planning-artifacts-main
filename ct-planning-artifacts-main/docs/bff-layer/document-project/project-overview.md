# Project Overview — Costco Tires & Batteries

_Generated: 2026-04-13 | Scan Level: Deep_

---

## Project Name

**Costco Tires & Batteries** (solution: `Tires-Batteries 1`)

## Purpose

A consumer-facing web application powering the tire and battery shopping experience on Costco.com. Customers can:
- Search for tires by vehicle (Year/Make/Model/Option), by tire size, by license plate (US), or by item/part number
- Compare up to 4 tires side-by-side
- View tire product detail pages with pricing, fitment, warranty, and features
- Select a Costco warehouse for local availability
- Search for and purchase batteries (Canada-focused; Automotive and Powersport categories)
- Access brand landing pages (Michelin, BFGoodrich, Bridgestone, Firestone, Pirelli)
- View CMS-managed static content (FAQ, warranty, tips, blogs)

## Markets

| Locale | Market |
|---|---|
| `en-US` | United States |
| `en-CA` | Canada (English) |
| `fr-CA` | Canada (French) |

---

## Tech Stack Summary

| Layer | Technology |
|---|---|
| Runtime | .NET Framework **4.8** (classic — NOT .NET Core) |
| Web Framework | ASP.NET MVC **5** |
| Database | Azure SQL (SQL Server) — stored procedures, raw ADO.NET |
| Cache | HttpRuntime.Cache (in-process) + Redis (StackExchange.Redis 2.0.601) |
| CMS | ContentStack (active) / Kentico REST API (legacy, feature-flag controlled) |
| External APIs | Mitchell1 (VIN/LP), Costco Inventory/Apigee, Bing Maps |
| Serialization | Newtonsoft.Json 11.0.1 |
| Secrets | Azure Key Vault |
| Monitoring | Azure Application Insights 2.4.x |
| Frontend | jQuery 3.3.1, Bootstrap 3.0.0, Razor views |
| Testing | MSTest + Moq 4.18.4 |
| IDE | Visual Studio 2022 (v17.2) |

---

## Architecture Type

**Layered Monolith** — single ASP.NET MVC 5 solution with strict tier separation:

```
Costco (MVC Web App)
  └── ApplicationFactory (static factory — no DI container)
  └── BusinessObjects (business logic)
       └── DataAccess (raw ADO.NET + stored procedures)
  └── ApplicationInterfaces (contracts)
  └── ApplicationModels (POCOs/DTOs)
  └── Integration (external HTTP client adapters)
  └── ErrorLogging (static Logger)
  └── Resources / ResourcesBattery (i18n strings)
  └── KenticoRestAPI (legacy CMS connector)
```

---

## Repository Structure

**Type:** Monolith (single repository, single deployable unit)  
**Solution Projects:** 15 projects (1 MVC web app + 14 class libraries + 1 SQL project)  
**Database Project:** `CWC_Tires` (.sqlproj) — 139 tables, hundreds of stored procedures

---

## Domain Areas

| Domain | Location | Notes |
|---|---|---|
| **Tires** | `Costco/Controllers/` | Main MVC project; en-US, en-CA, fr-CA |
| **Batteries** | `Costco/Areas/Batteries/` | MVC Area; Canada-heavy; US users redirected |
| **Admin** | `Costco/Controllers/Admin/` | Promotions management, dashboard |

---

## Key Links

| Document | Description |
|---|---|
| [Architecture](./architecture.md) | Full architecture decisions, patterns, security, deployment |
| [Source Tree Analysis](./source-tree-analysis.md) | Annotated project structure |
| [API Contracts](./api-contracts.md) | All MVC routes, AJAX endpoints, external integrations |
| [Data Models](./data-models.md) | Database tables, domain models, stored procedures |
| [Development Guide](./development-guide.md) | Setup, build, run, deploy |
| [Project Context (AI Rules)](../_bmad-output/project-context.md) | AI agent implementation rules |
| [BFF Migration Plan](../costco_tires_bff_api_summary.md) | Java Spring Boot BFF: 133 APIs, 20-week plan |

---

## Strategic Context

A Java 17 / Spring Boot 3.2 BFF migration is in progress using the **Strangler Fig** pattern. The ASP.NET MVC monolith will gradually be replaced by 133 REST API endpoints across 15 domains. See `costco_tires_bff_api_summary.md` for full details.
