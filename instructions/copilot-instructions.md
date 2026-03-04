# GitHub Copilot Instructions — xshopai Platform

This is the single authoritative instruction file for the xshopai microservices platform.
It consolidates guidelines from all individual service repositories.

---

## Platform Overview

xshopai is a polyglot e-commerce platform built on a microservices architecture using Dapr as the sidecar runtime for service invocation, pub/sub messaging, and state management.

- **Pattern**: Event-driven microservices with choreography-based saga for order fulfillment
- **Messaging**: Dapr pub/sub (RabbitMQ backend) — CloudEvents 1.0 format on all events
- **Service Discovery**: Dapr service invocation between services
- **Deployment targets**: Azure Container Apps, Azure App Service, Local Docker Compose
- **Dapr sidecar ports**: HTTP 3500, gRPC 50001 (per service, configured in `.dapr/`)

---

## Services Quick Reference

| Service                 | Language                      | Port | Database         | Dapr App ID               |
| ----------------------- | ----------------------------- | ---- | ---------------- | ------------------------- |
| product-service         | Python 3.11 / FastAPI         | 8001 | MongoDB 27019    | `product-service`         |
| user-service            | Node.js 20 / Express 5 ESM    | 8002 | MongoDB 27018    | `user-service`            |
| admin-service           | Node.js 20 / Express 5 ESM    | 8003 | None             | `admin-service`           |
| auth-service            | Node.js 20 / Express 5 ESM    | 8004 | None (stateless) | `auth-service`            |
| inventory-service       | Python 3.11 / Flask 3 + RESTX | 8005 | MySQL 3306       | `inventory-service`       |
| order-service           | C# 12 / .NET 8 ASP.NET Core   | 8006 | SQL Server 1434  | `order-service`           |
| order-processor-service | Java 17 / Spring Boot 3.3     | 8007 | PostgreSQL 5435  | `order-processor-service` |
| cart-service            | Node.js 20 / Express 4 TS     | 8008 | Redis 6379       | `cart-service`            |
| payment-service         | C# 12 / .NET 8 ASP.NET Core   | 8009 | SQL Server 1433  | `payment-service`         |
| review-service          | Node.js 20 / Express ESM      | 8010 | MongoDB 27020    | `review-service`          |
| notification-service    | Node.js 20 / Express TS       | 8011 | None (stateless) | `notification-service`    |
| audit-service           | Node.js 20 / Express TS       | 8012 | PostgreSQL 5434  | `audit-service`           |
| chat-service            | Node.js 20 / Express TS       | 8013 | None (stateless) | `chat-service`            |
| web-bff                 | Node.js 20 / Express 4 TS     | 8014 | None (stateless) | `web-bff`                 |
| customer-ui             | React 18 JSX (CRA)            | 3000 | —                | —                         |
| admin-ui                | React 18 TSX (CRA rewired)    | 3001 | —                | —                         |
| db-seeder               | Python 3.11 CLI               | —    | All DBs          | —                         |
| infrastructure          | Bicep / Bash                  | —    | —                | —                         |

---

## Universal Rules (All Services)

### Correlation IDs

- If `X-Correlation-ID` header is missing on an inbound request, generate a UUID
- Propagate `correlationId` in: all logs, all error responses, all outbound Dapr calls, all published events

### Error Response Contract

All services MUST return errors in this exact JSON structure:

```json
{
  "error": {
    "code": "STRING_CODE",
    "message": "Human readable message",
    "correlationId": "uuid"
  }
}
```

- Never expose stack traces in production
- Use centralized error middleware only — never inline error responses

### Logging Rules

All services use structured JSON logging. Every log entry MUST include:

```json
{
  "timestamp": "ISO8601",
  "level": "info|warn|error",
  "serviceName": "<service-name>",
  "correlationId": "uuid",
  "message": "..."
}
```

- Winston (Node.js), SLF4J/Logback (Java), Serilog (C#), Python `logging` with JSON formatter
- **Never log**: JWT tokens, refresh tokens, passwords, API keys, secrets, connection strings

### CloudEvents 1.0 Format

All published events MUST follow this structure:

```json
{
  "specversion": "1.0",
  "type": "service.entity.action",
  "source": "service-name",
  "id": "uuid",
  "time": "ISO8601",
  "datacontenttype": "application/json",
  "data": {}
}
```

### Security Rules (Universal)

- JWT MUST be validated before any controller logic executes
- All request bodies MUST be validated before reaching service logic
- Sanitize all inputs
- Rate limiting MUST be applied
- Never trust client-provided user IDs — always derive identity from the validated JWT

---

## Service-Specific Instructions

---

### product-service

**Language**: Python 3.11+ | **Framework**: FastAPI + Uvicorn | **Port**: 8001 | **DB**: MongoDB 27019 (Motor async driver)

#### Architecture

- Clean layered: routes → services → repositories → Motor/MongoDB
- JWT Bearer tokens validated via middleware
- Dapr pub/sub: publishes `product.created`, `product.updated`, `product.deleted`

#### Project Structure

```
product-service/
├── src/
│   ├── routes/          # FastAPI route definitions
│   ├── services/        # Business logic
│   ├── repositories/    # Data access (Motor/MongoDB)
│   ├── models/          # Pydantic v2 models
│   ├── middlewares/     # Auth, logging, tracing
│   ├── events/          # Dapr pub/sub event publishing
│   └── core/            # Config, logger, errors
├── tests/unit/ tests/integration/
├── .dapr/components/
└── main.py
```

#### Code Conventions

- Use `async/await` everywhere — FastAPI + Motor are fully async
- Use **Pydantic v2** models for all request/response validation
- Type hints on all function signatures; PEP 8 style
- Use `Depends()` for services and repos (FastAPI DI)
- Config via `pydantic-settings` `BaseSettings`
- MongoDB `ObjectId` for document IDs; serialize as strings in responses
- Use aggregation pipelines for search and category queries

#### Testing

- pytest + pytest-asyncio; `httpx.AsyncClient` for FastAPI routes; `mongomock-motor` for unit tests
- Run: `pytest tests/unit/ -v` | Coverage: `pytest --cov=src`

#### Non-Goals

- Does NOT manage reviews/ratings (→ review-service)
- Does NOT manage inventory stock (→ inventory-service)

---

### user-service

**Language**: Node.js 20+ ESM | **Framework**: Express 5.1+ | **Port**: 8002 | **DB**: MongoDB 27018 (Mongoose)

#### Architecture

- Layered MVC: routes → controllers → services → models (Mongoose)
- JWT Bearer tokens validated via middleware
- Dapr pub/sub: publishes `user.created`, `user.updated`, `user.deleted`

#### Project Structure

```
user-service/
├── src/
│   ├── controllers/     # Request handlers
│   ├── services/        # Business logic
│   ├── models/          # Mongoose schemas/models
│   ├── schemas/         # Reusable subdocuments
│   ├── events/          # Dapr pub/sub publishing
│   ├── middlewares/     # Auth, logging, tracing, correlation ID
│   ├── validators/      # Input validation (Joi/express-validator)
│   ├── routes/          # Route definitions
│   ├── core/            # Config, logger, errors
│   └── database/        # MongoDB connection setup
├── tests/unit/ tests/integration/ tests/e2e/
└── .dapr/components/
```

#### Code Conventions

- ESM modules (`import/export`) — NOT CommonJS
- Express 5.1+ async error handling (no try-catch wrappers needed)
- Mongoose schemas with built-in validation; subdocument schemas in `src/schemas/`
- Use `lean()` for read-only queries
- Timestamps enabled (`createdAt`, `updatedAt`) on all schemas
- `const` by default; arrow functions for callbacks; `async/await` over `.then()`

#### Security

- Ownership checks MUST be enforced — users may only access/modify their own profile
- Admin endpoints MUST verify `role === 'admin'`

#### Testing

- Jest; mongodb-memory-server or jest mocks; mock Dapr calls via `jest.mock()`
- Run: `npm test` | Unit: `npm run test:unit` | Coverage: `npm run test:coverage`

#### Non-Goals

- Does NOT issue JWTs (→ auth-service)
- Does NOT perform admin operations (→ admin-service)

---

### admin-service

**Language**: Node.js 20+ ESM | **Framework**: Express 5.1+ | **Port**: 8003 | **DB**: None

#### Architecture

- No database — all user operations delegated to user-service via Dapr service invocation
- All routes enforce admin role verification
- All outbound service calls forward the incoming JWT
- Dapr pub/sub: publishes `admin.user.updated`, `admin.user.deleted`

#### Project Structure

```
admin-service/
├── src/
│   ├── controllers/     # Admin endpoint handlers
│   ├── clients/         # Dapr service invocation clients
│   ├── middlewares/     # Auth, logging, tracing
│   ├── validators/      # Input validation
│   ├── routes/          # Route definitions
│   └── core/            # Config, logger, errors
├── tests/unit/ tests/e2e/
└── .dapr/components/
```

#### Code Conventions

- ESM modules; Express 5.1+ with async error handling
- Admin-only access: ALL routes require JWT with `role === 'admin'`
- Custom `ErrorResponse` class for error handling

#### Security

- JWT MUST be validated before any controller logic
- Admin role MUST be verified using `role === 'admin'`
- Never trust client-provided user IDs
- All outbound Dapr calls MUST include `X-Correlation-ID` and forwarded JWT

#### Testing

- Jest + Supertest; mock Dapr calls in unit tests; do NOT call real user-service
- All controllers MUST have unit tests; all routes MUST have e2e tests

#### Non-Goals

- Does NOT authenticate or issue JWTs (→ auth-service)
- Does NOT store user data (→ user-service)
- Does NOT manage user passwords
- Does NOT connect to any database — EVER

---

### auth-service

**Language**: Node.js 20+ ESM | **Framework**: Express 5.1+ | **Port**: 8004 | **DB**: None (stateless)

#### Architecture

- Stateless authentication gateway — no own database
- Issues and validates JWT access + refresh tokens
- Calls user-service via Dapr for user lookup/creation
- Dapr pub/sub: publishes `auth.login`, `auth.register`, `auth.logout`

#### Project Structure

```
auth-service/
├── src/
│   ├── controllers/     # Auth endpoint handlers
│   ├── middlewares/     # Auth middleware, logging, tracing
│   ├── validators/      # Input validation
│   ├── routes/          # Route definitions
│   ├── clients/         # Dapr service invocation clients
│   └── core/            # Config, logger, errors
├── tests/unit/ tests/integration/ tests/e2e/
└── .dapr/components/
```

#### Code Conventions

- ESM modules; Express 5.1+
- bcrypt for password hashing (salt rounds = 12)
- JWT: access token (short-lived) + refresh token (long-lived); rotation enforced
- Token payload: `{ id, email, roles, iat, exp }` — nothing else
- Rate limiting MUST be applied to the login endpoint

#### Security

- All request bodies MUST be validated before controller logic
- Passwords MUST use bcrypt with 12 salt rounds — never store or compare plain-text
- Refresh token rotation: invalidate old token on each use
- Never log JWT tokens, refresh tokens, or plain-text passwords

#### Testing

- Jest; mock Dapr calls and user-service invocations via `jest.mock()`
- Run: `npm test` | Coverage: `npm run test:coverage`

#### Non-Goals

- Does NOT manage user profiles (→ user-service)
- Does NOT enforce admin roles (→ admin-service)
- Does NOT store persistent user state — stateless

---

### inventory-service

**Language**: Python 3.11+ | **Framework**: Flask 3.0+ with Flask-RESTX | **Port**: 8005 | **DB**: MySQL 3306 (SQLAlchemy ORM + Flask-Migrate)

#### Architecture

- Layered MVC: Flask-RESTX Resources → services → SQLAlchemy models
- Auto-generated Swagger/OpenAPI via Flask-RESTX
- JWT Bearer tokens + service token validation
- Dapr pub/sub: publishes `inventory.reserved`, `inventory.released`, `inventory.low-stock`

#### Project Structure

```
inventory-service/
├── src/
│   ├── controllers/         # Flask-RESTX Resource classes
│   ├── services/            # Business logic
│   ├── models/              # SQLAlchemy models (InventoryItem, Reservation, StockMovement)
│   ├── middlewares/         # Auth, logging, tracing
│   └── utils/               # Schemas (Marshmallow), event publisher, error codes
├── migrations/              # Alembic migration scripts
├── tests/unit/ tests/integration/
├── config.py
└── main.py
```

#### Code Conventions

- Use Flask-RESTX `Resource` classes (auto-generates Swagger)
- Use SQLAlchemy ORM models with type hints
- Use Marshmallow schemas for request/response validation
- Use Flask-Migrate (Alembic) for database migrations
- `@require_admin` decorator for admin-only endpoints
- `@require_service_token` decorator for service-to-service calls
- SSL handling for Azure MySQL: strip `ssl_mode` from URL, use `connect_args`

#### Database Models

- `InventoryItem`: `sku` (unique, indexed), `quantity_available`, `quantity_reserved`, `reorder_level`, `max_stock`, `cost_per_unit`
- `Reservation`: order-based stock reservations
- `StockMovement`: audit trail of quantity changes

#### Testing

- pytest + pytest-cov; mock SQLAlchemy/MySQL in unit tests
- Run: `pytest tests/ -v` | Coverage: `pytest --cov=src`
- Target: ~91% code coverage

#### Non-Goals

- Does NOT manage product catalog (→ product-service)
- Does NOT process orders or payments

---

### order-service

**Language**: C# 12 / .NET 8 | **Framework**: ASP.NET Core 8 Web API | **Port**: 8006 | **DB**: SQL Server 1434 (EF Core 8)

#### Architecture

- Clean layered: Controllers → Services → Repositories → EF Core DbContext
- RESTful with Swagger/OpenAPI via Swashbuckle
- JWT Bearer tokens via ASP.NET Core Authentication
- Embedded BackgroundService subscribes to Dapr topics
- Dapr pub/sub: publishes `order.created`, `order.status.changed`, `order.cancelled`, `return.*`

#### Project Structure (Multi-project solution)

```
order-service/
├── OrderService.Api/
│   ├── Program.cs
│   ├── Controllers/
│   └── Migrations/          # EF Core migrations
├── OrderService.Core/
│   ├── Data/                # DbContext, entity configs
│   ├── Models/              # Domain entities + DTOs
│   ├── Repositories/        # Data access interfaces + implementations
│   ├── Services/            # Business logic
│   ├── Messaging/           # IMessagingProvider (Dapr + RabbitMQ)
│   ├── Events/              # Event models + publishers + consumers
│   ├── Validators/          # FluentValidation validators
│   ├── Extensions/          # DI registration extensions
│   └── Utils/               # StandardLogger, helpers
└── OrderService.Tests/      # xUnit test project
```

#### Code Conventions

- C# 12 with nullable reference types enabled
- EF Core 8 code-first migrations in `OrderService.Api/Migrations/`
- FluentValidation for request validation
- Serilog for structured logging
- `Dapr.AspNetCore` + `Dapr.Client` for pub/sub and service invocation
- `ICurrentUserService` extracts user from JWT claims via `HttpContextAccessor`
- `System.Text.Json` with `JsonStringEnumConverter`
- OpenTelemetry + Zipkin tracing

#### Database

- Entities: `Order`, `OrderItem`, `OrderReturn`, `Event` (outbox pattern)
- Order statuses: `Pending → Confirmed → Processing → Shipped → Delivered / Cancelled`
- Return workflow: `ReturnRequested → ReturnApproved → ReturnReceived → Refunded`

#### Testing

- xUnit + Moq + FluentAssertions
- Run: `dotnet test`

#### Non-Goals

- Does NOT process payments (→ payment-service)
- Does NOT orchestrate saga (→ order-processor-service)

---

### order-processor-service

**Language**: Java 17 | **Framework**: Spring Boot 3.3.1 | **Port**: 8007 | **DB**: PostgreSQL 5435 (Spring Data JPA + Flyway)

#### Architecture

- Choreography-based saga orchestrator
- Listens to order events, coordinates payment/inventory/shipping steps
- Dapr Java SDK for pub/sub; REST endpoints at `/dapr/events/{topic}`

#### Project Structure

```
order-processor-service/
├── src/main/java/com/xshopai/orderprocessor/
│   ├── config/              # DaprConfig, SecurityConfig, WebConfig
│   ├── controller/          # REST + Admin endpoints
│   ├── events/
│   │   ├── consumer/        # @PostMapping /dapr/events/* handlers
│   │   └── publisher/       # DaprEventPublisher
│   ├── messaging/           # MessagingProvider (Dapr vs RabbitMQ)
│   ├── model/entity/        # JPA entities (OrderProcessingSaga)
│   ├── repository/          # Spring Data JPA repositories
│   └── service/             # SagaOrchestratorService, SagaMetricsService
├── src/main/resources/
│   ├── application.yml      # Default (local)
│   ├── application-dapr.yml # Dapr profile
│   └── db/migration/        # Flyway SQL migrations
└── pom.xml
```

#### Code Conventions

- Lombok: `@RequiredArgsConstructor`, `@Slf4j`, `@Data`, `@Builder`
- Spring Data JPA repositories (extend `JpaRepository`)
- `@Transactional` on service methods that modify saga state
- `@EnableAsync` for async event processing
- Spring profiles: `default` (local/direct), `dapr` (with Dapr sidecar)
- Event consumers are `@RestController` with `@PostMapping("/dapr/events/{topic}")`
- Jackson with `JavaTimeModule` for Java 8 date/time

#### Saga Lifecycle

```
PENDING_PAYMENT_CONFIRMATION → PAYMENT_CONFIRMED → INVENTORY_RESERVED → SHIPPING_INITIATED → COMPLETED
```

- Admin confirms payment manually
- Compensating transactions on failure (release inventory, cancel payment)
- Saga stores serialized order items as JSON columns
- Max retry attempts configurable via `saga.retry.max-attempts`

#### Event Subscriptions

| Event                | Action                                        |
| -------------------- | --------------------------------------------- |
| `order.created`      | Start new saga (PENDING_PAYMENT_CONFIRMATION) |
| `order.cancelled`    | Cancel saga, compensate                       |
| `payment.processed`  | Advance to inventory reservation              |
| `inventory.reserved` | Advance to shipping                           |
| `return.approved`    | Handle return processing                      |

#### Testing

- JUnit 5 + Spring Boot Test + Mockito
- Run: `mvn test` | Integration: `mvn verify`

#### Non-Goals

- Does NOT manage the order record directly (→ order-service)
- Does NOT process payments directly (→ payment-service)

---

### payment-service

**Language**: C# 12 / .NET 8 | **Framework**: ASP.NET Core 8 Web API | **Port**: 8009 | **DB**: SQL Server 1433 (EF Core 8)

#### Architecture

- Provider abstraction pattern: `IPaymentProvider` with Stripe, PayPal, Square, Simulation implementations
- RESTful with Swagger/OpenAPI via Swashbuckle
- Dapr pub/sub: publishes `payment.processed`, `payment.failed`, `payment.refunded`

#### Project Structure

```
payment-service/
├── PaymentService/
│   ├── Program.cs
│   ├── Controllers/
│   ├── Models/                       # Entity models + DTOs
│   ├── Data/                         # DbContext, entity configs
│   ├── Services/
│   │   ├── Providers/
│   │   │   ├── StripePaymentProvider.cs
│   │   │   ├── PayPalPaymentProvider.cs
│   │   │   ├── SquarePaymentProvider.cs
│   │   │   └── SimulationPaymentProvider.cs
│   │   └── PaymentService.cs
│   ├── Messaging/
│   ├── Events/Publishers/
│   ├── Middlewares/
│   └── Migrations/
└── PaymentService.csproj
```

#### Code Conventions

- C# 12 with nullable reference types enabled
- EF Core 8 with SQL Server; FluentValidation for request validation
- Serilog for structured logging; `System.Text.Json` with camelCase naming
- Provider selected via `PaymentProviders:ActiveProvider` config
- `SimulationPaymentProvider` MUST be used in dev/test — NEVER call real providers
- Payment statuses: `Pending → Processing → Completed / Failed / Refunded`

#### Security

- Payment provider API keys MUST be stored in Dapr secrets store or Azure Key Vault
- Never expose raw payment provider error messages to API callers
- Never log full payment card details

#### Testing

- xUnit + Moq; use SimulationPaymentProvider in all tests
- Run: `dotnet test`

#### Non-Goals

- Does NOT manage orders (→ order-service)
- Does NOT orchestrate saga (→ order-processor-service)

---

### cart-service

**Language**: Node.js 20+ TypeScript | **Framework**: Express 4.18+ | **Port**: 8008 | **DB**: Redis 6379 (ioredis + Dapr state store)

#### Architecture

- StorageFactory selects Dapr state store or direct Redis based on `PLATFORM_MODE`
- User ID derived from `X-User-Id` header (set by BFF/gateway)
- Guest cart support: UUID-based, shorter TTL, transferable to authenticated carts
- Dapr pub/sub: publishes `cart.updated`, `cart.cleared`

#### Project Structure

```
cart-service/
├── src/
│   ├── controllers/
│   ├── services/
│   │   ├── cart.service.ts
│   │   ├── dapr.service.ts        # Dapr state + pub/sub
│   │   ├── redis.service.ts       # Direct Redis (App Service mode)
│   │   └── storage.factory.ts     # Dapr vs Redis selector
│   ├── models/          # TypeScript interfaces (Cart, CartItem)
│   ├── routes/
│   ├── middlewares/     # Trace context propagation
│   └── core/            # Config, logger, errors, Consul
├── tests/unit/ tests/integration/ tests/e2e/
└── .dapr/components/
```

#### Code Conventions

- TypeScript strict mode; ESM via `tsc`
- `interface` for all data shapes in `src/models/`
- `PLATFORM_MODE=dapr` → Dapr state store; `PLATFORM_MODE=direct` → ioredis
- ETag-based optimistic concurrency for cart state
- `asyncHandler` wrapper for Express route handlers
- W3C Trace Context propagation via `traceparent` header
- Cart transfer: `POST /api/v1/cart/transfer` merges guest cart into user cart

#### Limits

- `maxItems` and `maxItemQuantity` per cart — configurable, enforced on all operations
- Cart TTL: configurable days for authenticated (default 30) and guest (default 7) carts

#### Testing

- Jest + ts-jest with `--experimental-vm-modules` for ESM support
- Run: `npm test` | E2E: `npm run test:e2e` | Coverage: `npm run test:coverage`

#### Non-Goals

- Does NOT validate JWT (handled by BFF)
- Does NOT manage product pricing (→ product-service)

---

### review-service

**Language**: Node.js 20+ ESM | **Framework**: Express with Mongoose ODM | **Port**: 8010 | **DB**: MongoDB 27020

#### Architecture

- Layered MVC: routes → controllers → models (Mongoose)
- JWT Bearer tokens for authentication
- Dapr pub/sub: publishes review events

#### Project Structure

```
review-service/
├── src/
│   ├── controllers/
│   ├── models/          # Mongoose schemas (Review)
│   ├── routes/
│   ├── middlewares/     # Auth, validation
│   └── core/            # Config, logger, errors
├── tests/unit/
└── .dapr/components/
```

#### Code Conventions

- ESM modules with Babel transpilation
- Review model: rating (1-5), title, body, productId, userId, verified purchase flag
- Compound index on `productId + userId` — one review per product per user
- Aggregation pipelines for average ratings and review statistics
- Timestamps enabled

#### Security

- Users may only submit one review per product per user account
- Users may only edit or delete their own reviews

#### Testing

- Jest; Babel config for ESM→CJS test transformation; mock MongoDB operations
- Run: `npm test`

#### Non-Goals

- Does NOT manage product catalog (→ product-service)

---

### notification-service

**Language**: Node.js 20+ TypeScript | **Framework**: Express | **Port**: 8011 | **DB**: None (stateless)

#### Architecture

- Terminal event consumer — subscribes to Dapr pub/sub topics, sends notifications
- No end-user auth — service-to-service only
- Email via Nodemailer (SMTP/Mailpit for dev) + Azure Communication Services (production)
- Event handlers MUST be idempotent (duplicate events must not send duplicate emails)

#### Project Structure

```
notification-service/
├── src/
│   ├── controllers/     # Subscription handlers
│   ├── services/        # Notification sending logic
│   ├── templates/       # Email templates
│   ├── middlewares/     # Logging, tracing
│   ├── routes/          # Route + subscription definitions
│   └── core/            # Config, logger
├── tests/unit/
└── .dapr/components/
```

#### Event Subscriptions

| Event                  | Action                    |
| ---------------------- | ------------------------- |
| `auth.register`        | Send welcome email        |
| `auth.login`           | Send login notification   |
| `user.updated`         | Send profile update email |
| `order.created`        | Send order confirmation   |
| `order.status.changed` | Send order status update  |
| `payment.completed`    | Send payment receipt      |

#### Dev Tip

Use **Mailpit** (port 8025 web UI, port 1025 SMTP) to view sent emails during development.

#### Testing

- Jest + ts-jest; mock Nodemailer transport; test idempotent handling
- Run: `npm test` | Coverage: `npm run test:coverage`

#### Non-Goals

- Does NOT publish business events
- Does NOT store notification history
- Does NOT provide real-time push notifications

---

### audit-service

**Language**: Node.js 20+ TypeScript | **Framework**: Express | **Port**: 8012 | **DB**: PostgreSQL 5434 (Knex.js)

#### Architecture

- Terminal event consumer — subscribes to ALL Dapr pub/sub events, writes immutable audit records
- Read-only query endpoints + Dapr subscription endpoints
- Audit records are **append-only** — no UPDATE or DELETE operations — EVER

#### Project Structure

```
audit-service/
├── src/
│   ├── controllers/     # Query + subscription handlers
│   ├── services/        # Audit record business logic
│   ├── repositories/    # Data access (Knex.js)
│   ├── middlewares/     # Auth, logging, tracing
│   ├── routes/          # Route + subscription definitions
│   ├── migrations/      # Knex database migrations
│   └── core/            # Config, logger
├── tests/unit/
└── .dapr/components/
```

#### Code Conventions

- TypeScript strict mode
- Knex.js for PostgreSQL queries (NOT an ORM)
- Audit table: `id`, `event_type`, `source`, `subject`, `data` (JSONB), `correlation_id`, `created_at`
- Index on `event_type`, `source`, `created_at`
- Handlers MUST be idempotent — deduplicate by event ID
- All event data stored as JSONB

#### Event Subscriptions

Subscribes to ALL platform events: `auth.*`, `user.*`, `admin.*`, `order.*`, `payment.*`, `product.*`, `cart.*`, `inventory.*`

#### Testing

- Jest + ts-jest; mock PostgreSQL (Knex); test idempotency (duplicate event IDs must not create duplicate records)
- Run: `npm test`

#### Non-Goals

- Does NOT publish business events
- Does NOT modify or delete audit records — append-only
- Does NOT perform business logic on events

---

### chat-service

**Language**: Node.js 20+ TypeScript | **Framework**: Express | **Port**: 8013 | **DB**: None (stateless)

#### Architecture

- AI orchestration service — Azure OpenAI GPT-4o with function calling
- Receives user messages, calls Azure OpenAI, executes function tools via Dapr service invocation
- Conversation history managed in-memory per session

#### Project Structure

```
chat-service/
├── src/
│   ├── controllers/     # Chat endpoint handlers
│   ├── services/        # AI orchestration, conversation management
│   ├── tools/           # Function calling tool definitions (JSON Schema)
│   ├── clients/         # Dapr service invocation clients
│   ├── middlewares/     # Auth, logging
│   ├── routes/
│   └── core/            # Config, logger
└── .dapr/components/
```

#### AI Function Calling Tools

| Tool                | Description                 | Calls           |
| ------------------- | --------------------------- | --------------- |
| `searchProducts`    | Search product catalog      | product-service |
| `getProductDetails` | Get product by ID           | product-service |
| `getCategories`     | List product categories     | product-service |
| `getMyOrders`       | Get user's order history    | order-service   |
| `getOrderDetails`   | Get specific order          | order-service   |
| `trackOrder`        | Track order delivery status | order-service   |

#### Security

- JWT MUST be validated before any chat endpoint
- Conversation history MUST be session-scoped — never leak one user's history to another
- Never log user message content or AI response content
- Never include raw Azure OpenAI error responses in API responses

#### Testing

- Jest + ts-jest; mock Azure OpenAI responses and Dapr calls
- Run: `npm test`

#### Non-Goals

- Does NOT store chat history persistently — in-memory per session only
- Does NOT process orders or payments directly

---

### web-bff

**Language**: Node.js 20+ TypeScript | **Framework**: Express 4.19+ | **Port**: 8014 | **DB**: None (stateless)

#### Architecture

- Backend for Frontend — aggregates multiple microservice calls into UI-optimized responses
- JWT validation + cookie-based auth forwarding
- `PLATFORM_MODE=dapr` → Dapr SDK; `PLATFORM_MODE=direct` → direct HTTP via ServiceResolver

#### Project Structure

```
web-bff/
├── src/
│   ├── clients/           # Per-service client classes (typed)
│   │   ├── auth.client.ts
│   │   ├── user.client.ts
│   │   ├── product.client.ts
│   │   ├── order.client.ts
│   │   ├── cart.client.ts
│   │   └── inventory.client.ts
│   ├── core/
│   │   ├── config.ts
│   │   ├── daprClient.ts
│   │   ├── serviceInvoker.ts      # Dapr vs direct HTTP abstraction
│   │   ├── serviceResolver.ts     # App ID → URL mapping
│   │   └── baseServiceClient.ts
│   ├── middleware/         # Auth, error handling, trace context
│   ├── routes/
│   ├── validators/         # Config validation (blocking on startup)
│   └── tracing.ts          # OpenTelemetry + Zipkin
└── .dapr/components/
```

#### Downstream Services

| Service           | Dapr App ID         | Direct Port |
| ----------------- | ------------------- | ----------- |
| auth-service      | `auth-service`      | 8004        |
| user-service      | `user-service`      | 8002        |
| product-service   | `product-service`   | 8001        |
| order-service     | `order-service`     | 8006        |
| cart-service      | `cart-service`      | 8008        |
| inventory-service | `inventory-service` | 8005        |

#### Code Conventions

- TypeScript strict mode
- Path aliases: `@/` → `src/`, resolved via `tsc-alias`
- Use `BaseServiceClient` abstract class for all downstream service clients
- JWT config fetched from auth-service and cached
- CORS MUST restrict to configured `ALLOWED_ORIGINS`
- Helmet middleware MUST be applied
- Never expose internal service URLs or Dapr app IDs in API responses

#### Testing

- Jest + ts-jest; mock downstream service clients
- Test: auth failure, authorization failure, service unavailability scenarios
- Run: `npm test`

#### Non-Goals

- Does NOT own a database
- Does NOT issue JWT tokens (→ auth-service)
- Does NOT publish or consume domain events

---

### customer-ui

**Language**: JavaScript JSX | **Framework**: React 18.2 with CRA (react-scripts 5) | **Port**: 3000

#### Architecture

- SPA — all API calls via Web BFF (port 8014)
- Redux Toolkit (global state) + Zustand (lightweight stores) + TanStack Query v5 (server state)
- React Router DOM v6; Axios HTTP client; TailwindCSS 3 + Heroicons

#### Project Structure

```
customer-ui/
├── src/
│   ├── api/             # Axios API client configuration
│   ├── components/      # Reusable UI components
│   ├── contexts/        # React context providers
│   ├── data/            # Static data / constants
│   ├── hooks/           # Custom React hooks
│   ├── pages/           # Page-level components (route targets)
│   ├── store/           # Redux Toolkit slices + Zustand stores
│   ├── telemetry/       # Application Insights
│   └── utils/
├── nginx.conf
└── docker-entrypoint.sh  # Injects runtime env vars for Nginx
```

#### Code Conventions

- JavaScript JSX — NOT TypeScript
- Functional components with hooks exclusively
- TailwindCSS for all styling — no CSS modules
- Redux Toolkit `createSlice` for global state
- TanStack Query `useQuery`/`useMutation` for API data fetching
- Zustand for lightweight local state (cart, auth)
- React Router v6 BrowserRouter
- Axios with base URL and interceptors
- Icons via `@heroicons/react`
- Toast notifications via `react-toastify`
- Azure Application Insights SDK for telemetry
- Docker multi-stage: Node.js build → Nginx

#### Security

- All API calls MUST go through Web BFF — NEVER call microservices directly from browser
- Use `httpOnly` cookies for JWT — NEVER store JWTs in localStorage
- Handle 401 → redirect to login; 403 → show access-denied

#### Testing

- React Testing Library + Jest; Playwright for E2E
- Run: `npm test` (unit) | `npm run test:e2e` (Playwright)

---

### admin-ui

**Language**: TypeScript TSX | **Framework**: React 18.2 with react-app-rewired (CRA overrides) | **Port**: 3001

#### Architecture

- SPA — all API calls via Web BFF (port 8014)
- Redux Toolkit + Zustand + TanStack Query v5
- React Router DOM v6; Axios; TailwindCSS 3 + HeadlessUI + Heroicons
- TanStack Table v8 for data grids; Recharts for analytics dashboards

#### Project Structure

```
admin-ui/
├── src/
│   ├── components/      # Reusable UI components
│   ├── contexts/        # React context providers
│   ├── pages/           # Page-level components
│   ├── services/        # API service classes
│   ├── store/           # Redux Toolkit slices + Zustand stores
│   ├── telemetry/       # Application Insights
│   ├── types/           # TypeScript type definitions
│   └── utils/
├── config-overrides.js  # CRA config overrides (react-app-rewired)
├── nginx.conf
└── docker-entrypoint.sh
```

#### Code Conventions

- TypeScript strict mode (TSX)
- Functional components with hooks exclusively
- TailwindCSS utility classes for all styling
- HeadlessUI (`@headlessui/react`) for accessible dropdown, dialog, transition
- TanStack Table for sortable/filterable data tables
- Recharts for charts and graphs
- Type definitions in `src/types/`; API service classes in `src/services/`
- Admin role required for all operations
- Azure Application Insights for telemetry
- Docker multi-stage: Node.js build → Nginx

#### Security

- All API calls MUST go through Web BFF — NEVER call microservices directly
- `httpOnly` cookies for JWT — NEVER store in localStorage
- Admin portal MUST enforce authentication before rendering any route

#### Testing

- React Testing Library + Jest (via react-app-rewired); mock API service calls
- Run: `npm test`

---

### db-seeder

**Language**: Python 3.11+ | **Type**: CLI utility (no server, no Dapr)

#### Purpose

Seeds users, products, and inventory data for demo/development environments. Direct database connections.

#### CLI Usage

```bash
python seed.py                  # Seed all data
python seed.py --users          # Seed users only
python seed.py --products       # Seed products only
python seed.py --inventory      # Seed inventory only
python seed.py --clear          # Clear ALL databases
```

#### Databases Targeted

| Database   | Service           | Default                                               |
| ---------- | ----------------- | ----------------------------------------------------- |
| MongoDB    | user-service      | `mongodb://admin:admin123@localhost:27018`            |
| MongoDB    | product-service   | `mongodb://admin:admin123@localhost:27019`            |
| MongoDB    | review-service    | `mongodb://admin:admin123@localhost:27020`            |
| MySQL      | inventory-service | `admin:admin123@localhost:3306/inventory_service_db`  |
| PostgreSQL | audit-service     | `postgres:postgres@localhost:5434/audit-service`      |
| PostgreSQL | order-processor   | `postgres:postgres@localhost:5435/order_processor_db` |
| SQL Server | order-service     | `sa:Admin123!@localhost:1434/OrderServiceDb`          |
| SQL Server | payment-service   | `sa:Admin123!@localhost:1433/PaymentServiceDb`        |

#### Demo Credentials

- Admin: `admin@xshopai.com` / `admin`
- Guest: `guest@xshopai.com` / `guest`

#### Security

- Never use production credentials in `.env` or committed config
- Use `fetch-azure-secrets.sh` for Azure deployments (fetches from Key Vault)
- Bcrypt-hashed passwords MUST be used — never plain-text in seed data files
- Run `--clear` only against non-production databases

#### Non-Goals

- NOT a microservice — no server, no HTTP API, no Dapr
- NOT for production data

---

### infrastructure

**Language**: Bicep (ARM DSL) + Bash + PowerShell | **Target**: Azure

#### Deployment Targets

- `app-service/bicep/` — Azure App Service with Bicep modules
- `app-service-docker/bicep/` — Azure App Service with Docker containers
- `aca/bash/` — Azure Container Apps (bash scripts)
- `local/` — Local Docker Compose

#### Azure Resources Deployed

- Azure App Service / Container Apps (16 services + 2 UIs)
- Azure Container Registry (ACR)
- Azure Service Bus or RabbitMQ container
- Azure Cache for Redis
- Azure Database for MySQL, PostgreSQL
- Azure SQL Database
- Azure Cosmos DB for MongoDB
- Azure Key Vault
- Azure Application Insights + Log Analytics Workspace

#### Conventions

- Modular Bicep: one module per Azure resource type; `main.bicep` as orchestrator
- Environment-specific parameter files in `parameters/` directory
- Bash scripts: use `set -euo pipefail` for strict error handling
- Tags on ALL Azure resources: `environment`, `project`, `managedBy`
- OIDC authentication for GitHub Actions (no stored credentials)
- Connection strings/API keys MUST NOT appear in Bicep `outputs`

#### Security

- Secret values MUST NOT be hardcoded in Bicep templates — use Key Vault references
- GitHub Actions MUST use OIDC for Azure auth — never store long-lived credentials
- Production `*.bicepparam` files MUST NOT be committed with real secrets

---

## Dapr Integration Summary

### Service Invocation

```
http://localhost:3500/v1.0/invoke/{app-id}/method/{path}
```

- Always forward `X-Correlation-ID` and `Authorization` headers on outbound calls

### Pub/Sub Topics Published per Service

| Service              | Topics Published                                                       |
| -------------------- | ---------------------------------------------------------------------- |
| auth-service         | `auth.login`, `auth.register`, `auth.logout`                           |
| user-service         | `user.created`, `user.updated`, `user.deleted`                         |
| admin-service        | `admin.user.updated`, `admin.user.deleted`                             |
| product-service      | `product.created`, `product.updated`, `product.deleted`                |
| inventory-service    | `inventory.reserved`, `inventory.released`, `inventory.low-stock`      |
| order-service        | `order.created`, `order.status.changed`, `order.cancelled`, `return.*` |
| payment-service      | `payment.processed`, `payment.failed`, `payment.refunded`              |
| cart-service         | `cart.updated`, `cart.cleared`                                         |
| audit-service        | _(none — terminal consumer only)_                                      |
| notification-service | _(none — consumer only)_                                               |

---

## Environment Variables (All Services)

```bash
# Shared across most services
JWT_SECRET=<shared-secret>
DAPR_HTTP_PORT=3500
DAPR_GRPC_PORT=50001
NODE_ENV=development              # Node.js services
ASPNETCORE_ENVIRONMENT=Development  # .NET services
FLASK_ENV=development             # Python Flask/RESTX
```

Full per-service env vars are documented in each service's `.env.example` or README.
