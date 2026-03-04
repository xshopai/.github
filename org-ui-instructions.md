# xshopai Platform — Copilot Custom Instructions (Org UI)

This is the condensed version for the GitHub org Settings > Copilot > Customization UI (4000 char limit).
Copy the content between the --- markers below into the UI text box.

---

xshopai is a polyglot e-commerce microservices platform using Dapr for service invocation, pub/sub (RabbitMQ), and state management. All events use CloudEvents 1.0 format.

## Services
| Service | Language/Framework | Port | DB |
|---|---|---|---|
| product-service | Python/FastAPI | 8001 | MongoDB |
| user-service | Node.js/Express 5 ESM | 8002 | MongoDB |
| admin-service | Node.js/Express 5 ESM | 8003 | None |
| auth-service | Node.js/Express 5 ESM | 8004 | None |
| inventory-service | Python/Flask-RESTX | 8005 | MySQL |
| order-service | C#/.NET 8 | 8006 | SQL Server |
| order-processor-service | Java/Spring Boot 3 | 8007 | PostgreSQL |
| cart-service | Node.js/Express TS | 8008 | Redis |
| payment-service | C#/.NET 8 | 8009 | SQL Server |
| review-service | Node.js/Express ESM | 8010 | MongoDB |
| notification-service | Node.js/Express TS | 8011 | None |
| audit-service | Node.js/Express TS | 8012 | PostgreSQL |
| chat-service | Node.js/Express TS | 8013 | None |
| web-bff | Node.js/Express TS | 8014 | None |
| customer-ui | React 18 JSX (CRA) | 3000 | — |
| admin-ui | React 18 TSX (CRA) | 3001 | — |

## Universal Rules
- Generate UUID for missing X-Correlation-ID header. Propagate correlationId in all logs, errors, outbound calls, events.
- All errors: `{"error":{"code":"STRING","message":"...","correlationId":"uuid"}}`
- Structured JSON logging: timestamp, level, serviceName, correlationId, message. Never log JWTs, secrets, passwords.
- CloudEvents: specversion 1.0, type "service.entity.action", source, id (uuid), time (ISO8601), data.
- JWT must be validated before controller logic. Validate all request bodies. Sanitize inputs. Rate limit routes.
- Never trust client-provided user IDs — derive from validated JWT.

## Key Patterns
- Node.js services: ESM modules, Express 5.1+ async error handling, Winston logging, custom ErrorResponse class.
- Python services: FastAPI async (product), Flask-RESTX with Marshmallow (inventory). SQLAlchemy ORM for MySQL, Motor for MongoDB.
- .NET services: C# 12, EF Core 8, FluentValidation, Serilog, Dapr.AspNetCore SDK.
- Java service: Spring Boot 3.3, Lombok, Spring Data JPA, Flyway migrations, Dapr Java SDK.
- UIs: React 18 functional components, TailwindCSS, Redux Toolkit + TanStack Query. All API calls via web-bff only.
- admin-service/auth-service: No database. Delegate to user-service via Dapr service invocation.
- audit-service: Append-only. No UPDATE/DELETE ever.
- order-processor-service: Choreography-based saga. States: PENDING_PAYMENT → PAYMENT_CONFIRMED → INVENTORY_RESERVED → SHIPPING_INITIATED → COMPLETED.
- payment-service: IPaymentProvider abstraction (Stripe/PayPal/Square/Simulation). Use Simulation in dev/test.
- cart-service: StorageFactory selects Dapr state store or direct Redis via PLATFORM_MODE.

## Testing
- Node.js: Jest (+ ts-jest for TS). Mock Dapr/DB.
- Python: pytest + pytest-asyncio. Mock DB.
- .NET: xUnit + Moq + FluentAssertions.
- Java: JUnit 5 + Mockito.
- UIs: React Testing Library + Jest. Playwright for E2E.
- Never call real DBs or downstream services in unit tests.

## Dapr
- Service invocation: http://localhost:3500/v1.0/invoke/{app-id}/method/{path}
- Forward X-Correlation-ID and Authorization headers on outbound calls.
- Pub/sub topics per service documented in repo-level instructions.

---
