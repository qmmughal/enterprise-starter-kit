# 🏗️ EnterpriseKit — .NET 8 Enterprise Starter Kit

[![CI](https://github.com/qmmughal/enterprise-starter-kit/actions/workflows/ci.yml/badge.svg)](https://github.com/qmmughal/enterprise-starter-kit/actions/workflows/ci.yml)
[![.NET](https://img.shields.io/badge/.NET-8.0-512BD4?logo=dotnet)](https://dotnet.microsoft.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

> **.NET 8 LTS track.** For new greenfield work on **.NET 10**, prefer [enterprise-starter-kit-v2](https://github.com/qmmughal/enterprise-starter-kit-v2).
>
> Architecture, patterns, and conventions are **identical** between the two kits — Clean Architecture layering, the CQRS/MediatR pipeline, the transactional outbox pattern, and the "adding a new feature" checklist are documented once, in [v2's README](https://github.com/qmmughal/enterprise-starter-kit-v2#readme), rather than duplicated here. This document covers only what you need to run the **.NET 8** build and what differs from v2.

## What's identical to v2

- 🧱 **Clean Architecture** — same 4-layer separation (Domain / Application / Infrastructure / HttpApi)
- ⚡ **CQRS + MediatR** — same pipeline: `Request → LoggingBehaviour → ValidationBehaviour → TransactionBehaviour → Handler`
- 📬 **Transactional Outbox Pattern** — same at-least-once delivery guarantee via `OutboxRelayService`
- 🔐 **ABP Framework** — same multi-tenancy, identity, OpenIddict OIDC setup
- 📊 **Serilog** — same structured JSON logging
- 🐘 **PostgreSQL** + **Redis** + **MailHog** via the same Docker Compose stack

Full write-up of all of the above, including the architecture diagram and the step-by-step "adding a new feature" checklist: **[read it in the v2 README →](https://github.com/qmmughal/enterprise-starter-kit-v2#readme)**

## What's different on .NET 8

- Targets the **.NET 8.0 SDK** (LTS) instead of .NET 10.

---

## 🚀 Quick Start

### Prerequisites

- [.NET 8 SDK](https://dotnet.microsoft.com/download/dotnet/8.0)
- [Docker Desktop](https://www.docker.com/products/docker-desktop/)

### 1. Clone & spin up infrastructure

```bash
git clone https://github.com/qmmughal/enterprise-starter-kit.git
cd enterprise-starter-kit

docker compose up -d
```

This starts:
| Service | Port | Purpose |
|---|---|---|
| PostgreSQL 16 | `5432` | Primary database |
| Redis 7 | `6379` | Distributed cache |
| MailHog | `1025` / `8025` | SMTP mock + Web UI |

### 2. Apply database migrations

```bash
dotnet ef migrations add InitialCreate \
  --project src/Infrastructure/EnterpriseKit.Infrastructure \
  --startup-project src/HttpApi/EnterpriseKit.HttpApi

dotnet ef database update \
  --startup-project src/HttpApi/EnterpriseKit.HttpApi
```

### 3. Run the API

```bash
dotnet run --project src/HttpApi/EnterpriseKit.HttpApi
```

Swagger UI: **http://localhost:5000** (served at root in development)
MailHog UI: **http://localhost:8025**

---

## 🧪 Running Tests

```bash
# All tests
dotnet test

# Domain unit tests only
dotnet test tests/Domain.UnitTests

# Application unit tests only
dotnet test tests/Application.UnitTests
```

---

## 🗂️ Project Structure

```
EnterpriseKit/
├── src/
│   ├── Domain/EnterpriseKit.Domain/         ← ZERO deps, pure C#
│   │   ├── Common/                          ← Entity, AggregateRoot, ValueObject
│   │   ├── Exceptions/                      ← DomainException, NotFoundException
│   │   ├── Interfaces/Repositories/         ← Repository contracts
│   │   └── Orders/                          ← Order aggregate + Money VO + Events
│   │
│   ├── Application/EnterpriseKit.Application/
│   │   ├── Common/Behaviours/               ← Logging, Validation, Transaction
│   │   ├── Common/Mappings/                 ← AutoMapper profiles
│   │   └── Orders/
│   │       ├── Commands/                    ← PlaceOrder, CancelOrder
│   │       ├── Queries/                     ← GetOrderById, GetOrders (paged)
│   │       └── EventHandlers/               ← OrderPlacedEventHandler
│   │
│   ├── Infrastructure/EnterpriseKit.Infrastructure/
│   │   ├── Persistence/                     ← ApplicationDbContext, EF configs
│   │   ├── Outbox/                          ← OutboxMessage, OutboxRelayService
│   │   └── DependencyInjection/             ← InfrastructureServiceExtensions
│   │
│   └── HttpApi/EnterpriseKit.HttpApi/
│       ├── Controllers/                     ← OrdersController
│       ├── Middleware/                      ← GlobalExceptionMiddleware (RFC 7807)
│       ├── Program.cs
│       └── Dockerfile
│
├── tests/
│   ├── Domain.UnitTests/
│   └── Application.UnitTests/
│
├── infra/postgres/init.sql
├── docker-compose.yml
└── docker-compose.override.yml
```

---

## 🔐 Authentication

The API uses **JWT Bearer** authentication. Configure your identity provider in `appsettings.json`:

```json
{
  "Auth": {
    "Authority": "https://your-identity-server",
    "Audience": "enterprise-kit-api"
  }
}
```

For local development without an identity server, you can disable auth by removing `[Authorize]` from controllers or using a local JWT tool.

---

## 📄 License

MIT — see [LICENSE](LICENSE).
