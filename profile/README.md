# xshopai

<p align="center">
  <strong>An AI-powered, open-source e-commerce platform built with microservices architecture</strong>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/license-MIT-blue.svg" alt="License">
  <img src="https://img.shields.io/badge/PRs-welcome-brightgreen.svg" alt="PRs Welcome">
  <img src="https://img.shields.io/badge/AI--Powered-OpenAI-00A67E.svg" alt="AI Powered">
  <img src="https://img.shields.io/badge/microservices-16-orange.svg" alt="Microservices">
  <img src="https://img.shields.io/badge/languages-5-purple.svg" alt="Languages">
</p>

---

**xshopai** is a production-grade, polyglot e-commerce platform built across 16 independent microservices. Each service is implemented in the language and framework best suited for its domain — C#, Java, Python, TypeScript, and JavaScript — backed by polyglot persistence (MongoDB, PostgreSQL, SQL Server, MySQL, Redis) and orchestrated via [Dapr](https://dapr.io).

The platform is designed as both a functional e-commerce system and an architectural reference for distributed systems, event-driven design, and AI integration.

---

## Services

### Backend

| Service | Stack | Purpose |
|---------|-------|---------|
| [admin-service](https://github.com/xshopai/admin-service) | Node.js · Express | User and role administration |
| [audit-service](https://github.com/xshopai/audit-service) | Node.js · TypeScript · PostgreSQL | Immutable audit trail for all platform events |
| [auth-service](https://github.com/xshopai/auth-service) | Node.js · Express | Authentication, JWT issuance, session management |
| [cart-service](https://github.com/xshopai/cart-service) | Node.js · TypeScript · Redis | Shopping cart with guest and authenticated cart support |
| [chat-service](https://github.com/xshopai/chat-service) | Node.js · TypeScript · Azure OpenAI | AI-powered conversational shopping assistant |
| [inventory-service](https://github.com/xshopai/inventory-service) | Python · Flask · MySQL | Stock management, reservations, reorder alerts |
| [notification-service](https://github.com/xshopai/notification-service) | Node.js · TypeScript | Event-driven email and notification delivery |
| [order-processor-service](https://github.com/xshopai/order-processor-service) | Java · Spring Boot · PostgreSQL | Choreography-based saga for order fulfillment |
| [order-service](https://github.com/xshopai/order-service) | C# · ASP.NET Core · SQL Server | Order lifecycle management |
| [payment-service](https://github.com/xshopai/payment-service) | C# · ASP.NET Core · SQL Server | Multi-provider payment processing |
| [product-service](https://github.com/xshopai/product-service) | Python · FastAPI · MongoDB | Product catalog, search, and categories |
| [review-service](https://github.com/xshopai/review-service) | Node.js · Express · MongoDB | Product reviews and ratings |
| [user-service](https://github.com/xshopai/user-service) | Node.js · Express · MongoDB | User profiles, addresses, preferences |
| [web-bff](https://github.com/xshopai/web-bff) | Node.js · TypeScript | Backend for Frontend — API gateway for web UIs |

### Frontend

| Application | Stack | Purpose |
|-------------|-------|---------|
| [customer-ui](https://github.com/xshopai/customer-ui) | React 18 | Customer-facing storefront |
| [admin-ui](https://github.com/xshopai/admin-ui) | React 18 · TypeScript | Admin portal for operations and analytics |

---

## Documentation

| Resource | Location |
|----------|----------|
| Architecture Overview | [Wiki](https://github.com/xshopai/docs/wiki/Architecture-Overview) |
| Service Catalog | [Wiki](https://github.com/xshopai/docs/wiki/Service-Catalog) |
| API Reference | [Wiki](https://github.com/xshopai/docs/wiki/API-Reference) |
| Event Catalog | [Wiki](https://github.com/xshopai/docs/wiki/Event-Catalog) |
| Installation Guide | [Wiki](https://github.com/xshopai/docs/wiki/Installation-Guide) |
| Deployment Guide | [Wiki](https://github.com/xshopai/docs/wiki/Deployment-Guide) |
| Build Status | [BUILD_STATUS.md](./BUILD_STATUS.md) |

---

## Contributing

Contributions are welcome. Please review the [Contributing Guidelines](https://github.com/xshopai/docs/blob/main/CONTRIBUTING.md) before opening a pull request. For questions and discussions, use [GitHub Discussions](https://github.com/orgs/xshopai/discussions).

## License

[MIT](https://github.com/xshopai/docs/blob/main/LICENSE)
