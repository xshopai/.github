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

| Service | Stack | Purpose | CI |
|---------|-------|---------|:--:|
| [admin-service](https://github.com/xshopai/admin-service) | Node.js · Express | User and role administration | [![CI](https://github.com/xshopai/admin-service/actions/workflows/ci-app-service.yml/badge.svg)](https://github.com/xshopai/admin-service/actions/workflows/ci-app-service.yml) |
| [audit-service](https://github.com/xshopai/audit-service) | Node.js · TypeScript · PostgreSQL | Immutable audit trail for all platform events | [![CI](https://github.com/xshopai/audit-service/actions/workflows/ci-app-service.yml/badge.svg)](https://github.com/xshopai/audit-service/actions/workflows/ci-app-service.yml) |
| [auth-service](https://github.com/xshopai/auth-service) | Node.js · Express | Authentication, JWT issuance, session management | [![CI](https://github.com/xshopai/auth-service/actions/workflows/ci-app-service.yml/badge.svg)](https://github.com/xshopai/auth-service/actions/workflows/ci-app-service.yml) |
| [cart-service](https://github.com/xshopai/cart-service) | Node.js · TypeScript · Redis | Shopping cart with guest and authenticated cart support | [![CI](https://github.com/xshopai/cart-service/actions/workflows/ci-app-service.yml/badge.svg)](https://github.com/xshopai/cart-service/actions/workflows/ci-app-service.yml) |
| [chat-service](https://github.com/xshopai/chat-service) | Node.js · TypeScript · Azure OpenAI | AI-powered conversational shopping assistant | [![CI](https://github.com/xshopai/chat-service/actions/workflows/ci-app-service.yml/badge.svg)](https://github.com/xshopai/chat-service/actions/workflows/ci-app-service.yml) |
| [inventory-service](https://github.com/xshopai/inventory-service) | Python · Flask · MySQL | Stock management, reservations, reorder alerts | [![CI](https://github.com/xshopai/inventory-service/actions/workflows/ci-app-service.yml/badge.svg)](https://github.com/xshopai/inventory-service/actions/workflows/ci-app-service.yml) |
| [notification-service](https://github.com/xshopai/notification-service) | Node.js · TypeScript | Event-driven email and notification delivery | [![CI](https://github.com/xshopai/notification-service/actions/workflows/ci-app-service.yml/badge.svg)](https://github.com/xshopai/notification-service/actions/workflows/ci-app-service.yml) |
| [order-processor-service](https://github.com/xshopai/order-processor-service) | Java · Spring Boot · PostgreSQL | Choreography-based saga for order fulfillment | [![CI](https://github.com/xshopai/order-processor-service/actions/workflows/ci-app-service.yml/badge.svg)](https://github.com/xshopai/order-processor-service/actions/workflows/ci-app-service.yml) |
| [order-service](https://github.com/xshopai/order-service) | C# · ASP.NET Core · SQL Server | Order lifecycle management | [![CI](https://github.com/xshopai/order-service/actions/workflows/ci-app-service.yml/badge.svg)](https://github.com/xshopai/order-service/actions/workflows/ci-app-service.yml) |
| [payment-service](https://github.com/xshopai/payment-service) | C# · ASP.NET Core · SQL Server | Multi-provider payment processing | [![CI](https://github.com/xshopai/payment-service/actions/workflows/ci-app-service.yml/badge.svg)](https://github.com/xshopai/payment-service/actions/workflows/ci-app-service.yml) |
| [product-service](https://github.com/xshopai/product-service) | Python · FastAPI · MongoDB | Product catalog, search, and categories | [![CI](https://github.com/xshopai/product-service/actions/workflows/ci-app-service.yml/badge.svg)](https://github.com/xshopai/product-service/actions/workflows/ci-app-service.yml) |
| [review-service](https://github.com/xshopai/review-service) | Node.js · Express · MongoDB | Product reviews and ratings | [![CI](https://github.com/xshopai/review-service/actions/workflows/ci-app-service.yml/badge.svg)](https://github.com/xshopai/review-service/actions/workflows/ci-app-service.yml) |
| [user-service](https://github.com/xshopai/user-service) | Node.js · Express · MongoDB | User profiles, addresses, preferences | [![CI](https://github.com/xshopai/user-service/actions/workflows/ci-app-service.yml/badge.svg)](https://github.com/xshopai/user-service/actions/workflows/ci-app-service.yml) |
| [web-bff](https://github.com/xshopai/web-bff) | Node.js · TypeScript | Backend for Frontend — API gateway for web UIs | [![CI](https://github.com/xshopai/web-bff/actions/workflows/ci-app-service.yml/badge.svg)](https://github.com/xshopai/web-bff/actions/workflows/ci-app-service.yml) |

### Frontend

| Application | Stack | Purpose | CI |
|-------------|-------|---------|:--:|
| [customer-ui](https://github.com/xshopai/customer-ui) | React 18 | Customer-facing storefront | [![CI](https://github.com/xshopai/customer-ui/actions/workflows/ci-app-service.yml/badge.svg)](https://github.com/xshopai/customer-ui/actions/workflows/ci-app-service.yml) |
| [admin-ui](https://github.com/xshopai/admin-ui) | React 18 · TypeScript | Admin portal for operations and analytics | [![CI](https://github.com/xshopai/admin-ui/actions/workflows/ci-app-service.yml/badge.svg)](https://github.com/xshopai/admin-ui/actions/workflows/ci-app-service.yml) |

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

---

## Contributing

Contributions are welcome. Please review the [Contributing Guidelines](https://github.com/xshopai/docs/blob/main/CONTRIBUTING.md) before opening a pull request. For questions and discussions, use [GitHub Discussions](https://github.com/orgs/xshopai/discussions).

## License

[MIT](https://github.com/xshopai/docs/blob/main/LICENSE)
