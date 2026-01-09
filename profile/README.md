# 🛍️ xshopai

<p align="center">
  <strong>🤖 An AI-powered, open-source e-commerce platform built with microservices architecture</strong>
</p>

<p align="center">
  <a href="#-features">Features</a> •
  <a href="#%EF%B8%8F-architecture-highlights">Architecture</a> •
  <a href="#-microservices-overview">Services</a> •
  <a href="#-contributing">Contributing</a>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/license-MIT-blue.svg" alt="License">
  <img src="https://img.shields.io/badge/PRs-welcome-brightgreen.svg" alt="PRs Welcome">
  <img src="https://img.shields.io/badge/AI--Powered-OpenAI-00A67E.svg" alt="AI Powered">
  <img src="https://img.shields.io/badge/microservices-15-orange.svg" alt="Microservices">
  <img src="https://img.shields.io/badge/languages-6-purple.svg" alt="Languages">
</p>

---

## 💡 What's in the Name?

> **x** • **shop** • **ai** — Each part of our name tells our story:
>
> | | | |
> |:---:|:---|:---|
> | **x** | ✖️ | **Cross-platform & polyglot** — Built with 6+ languages (Node.js, Python, Java, .NET, Go, TypeScript) proving that the right tool for each job creates the best system |
> | **shop** | 🛒 | **E-commerce at scale** — A full-featured online store inspired by Amazon and Microsoft's [eShop reference architecture](https://github.com/dotnet/eShop) |
> | **ai** | 🤖 | **AI-native** — Intelligent chatbot, smart recommendations, and natural language interfaces powered by OpenAI |

---

**xshopai** is an **AI-powered**, open-source e-commerce platform built with a modern, microservices-based architecture. Featuring an intelligent chatbot (powered by OpenAI), smart product recommendations, and natural language order tracking — it showcases how AI/ML can be seamlessly integrated into a production-grade distributed system.

Designed as a learning resource and architectural blueprint for developers, architects, and students exploring distributed systems, AI integration, full-stack development, and cloud-native design.

> 🎯 **Perfect for**: Learning microservices patterns, exploring AI/ML integration in e-commerce, understanding event-driven systems, and building your portfolio with production-grade code.

## 🌐 Features

- 🧱 **Microservices Architecture** — Each business capability is implemented as a separate microservice using the most suitable technology (Node.js, .NET, Python, Java, Go, etc.).
- 🖥️ **Frontend in React** — A sleek, responsive UI built with vanilla React, optimized for user experience.
- 📱 **Mobile-Ready** — Future support for native mobile apps using React Native or Flutter.
- ☁️ **Cloud-Native** — Deployable locally with Docker Compose or to cloud platforms like Azure AKS using Kubernetes and Helm.
- 🧠 **AI-First Design** — Designed with AI/ML in mind: recommendation engine, intelligent chatbot, and NLP services are core to the vision.
- ⚙️ **DevOps with GitHub Actions** — Full CI/CD automation with GitHub Actions and reusable workflows across all microservices.
- 🗃️ **Polyglot Persistence** — Uses SQL, NoSQL, key-value, and graph databases — chosen based on the nature of each service's data.
- 🔐 **Authentication & Authorization** — Supports secure login via email/password and OAuth2 (Google, Facebook, Twitter), JWT authentication, and role-based access control.
- 📡 **API-First & Event-Driven** — RESTful APIs and asynchronous communication using message queues for scalability and loose coupling.

## 🏗️ Architecture Highlights

<p align="center">
  <img src="./architecture-diagram.png" alt="xshopai Architecture Diagram" width="800">
  <br>
  <em>High-level architecture diagram (coming soon)</em>
</p>

- **🌐 Polyglot Architecture**: Each service uses the technology best suited for its requirements
- **🔄 Event-Driven Communication**: Services communicate via message queues and events
- **📊 Comprehensive Audit Logging**: Full activity tracking and compliance monitoring
- **⚡ High-Performance Services**: Go-based inventory service for speed-critical operations
- **🎯 Saga Pattern**: Distributed transaction management in order processing
- **📈 Scalable Data Storage**: PostgreSQL, MongoDB, and Redis for different data patterns
- **🛡️ Security-First Design**: JWT authentication, service-to-service tokens, and audit trails

## 🧩 Microservices Overview

Each microservice is hosted in its own GitHub repository for separation of concerns, independent scalability, and streamlined DevOps workflows. The platform demonstrates a polyglot architecture with different technologies chosen for each service's specific requirements:

### Backend Services

| Service                                                                               | Technology Stack                            | Description                                                                  |
| ------------------------------------------------------------------------------------- | ------------------------------------------- | ---------------------------------------------------------------------------- |
| 🛡️ [**admin-service**](https://github.com/xshopai/admin-service)                     | Node.js 20<br>Express 5<br>MongoDB 8        | Back-office operations, dashboard, analytics, and user management            |
| 📋 [**audit-service**](https://github.com/xshopai/audit-service)                     | Node.js 20<br>TypeScript 5<br>PostgreSQL 16 | Comprehensive audit logging, compliance tracking, and activity monitoring    |
| 🔐 [**auth-service**](https://github.com/xshopai/auth-service)                       | Node.js 20<br>Express 5<br>MongoDB 8        | Handles authentication, MFA, social login (OAuth2), and JWT issuance         |
| 🛒 [**cart-service**](https://github.com/xshopai/cart-service)                       | Java 21<br>Quarkus 3.6<br>Redis 7 (via Dapr)| Shopping cart management with Redis for session handling                     |
| 💬 [**chat-service**](https://github.com/xshopai/chat-service)                       | Node.js 20<br>TypeScript 5<br>OpenAI        | AI-powered chatbot for customer support and order inquiries                  |
| 📦 [**inventory-service**](https://github.com/xshopai/inventory-service)             | Python 3.11<br>Flask 3.0<br>MySQL 8         | High-performance inventory management, stock tracking, and reservations      |
| 📣 [**notification-service**](https://github.com/xshopai/notification-service)       | Node.js 20<br>Express 4<br>TypeScript 5     | Multi-channel notifications (email, SMS, push, WebSocket)                    |
| ⚙️ [**order-processor-service**](https://github.com/xshopai/order-processor-service) | Java 21<br>Spring Boot 3.3<br>PostgreSQL 16 | Asynchronous order processing with saga pattern and event sourcing           |
| 🧾 [**order-service**](https://github.com/xshopai/order-service)                     | .NET 8<br>ASP.NET Core 8<br>SQL Server 2022 | Order creation, validation, and lifecycle management                         |
| 💳 [**payment-service**](https://github.com/xshopai/payment-service)                 | .NET 8<br>ASP.NET Core 8<br>SQL Server 2022 | Payment processing, gateway integration, and transaction security            |
| 🛍️ [**product-service**](https://github.com/xshopai/product-service)                 | Python 3.11<br>FastAPI 0.104<br>MongoDB 8   | Handles product catalog, categories, attributes, search, and recommendations |
| ⭐ [**review-service**](https://github.com/xshopai/review-service)                   | Node.js 20<br>Express 4<br>MongoDB 8        | Product reviews, ratings, and customer feedback management                   |
| 👤 [**user-service**](https://github.com/xshopai/user-service)                       | Node.js 20<br>Express 5<br>MongoDB 8        | Manages user profiles, identity records, preferences, and account linking    |
| 🌐 [**web-bff**](https://github.com/xshopai/web-bff)                                 | Node.js 20<br>Express 4<br>TypeScript 5     | Backend for Frontend aggregating data from multiple microservices            |

> **Note:** Service-to-service communication and event-driven messaging is handled by [DAPR (Distributed Application Runtime)](https://dapr.io), eliminating the need for a separate message broker service.

### Frontend Applications

| Application                                                     | Technology Stack           | Description                                               |
| --------------------------------------------------------------- | -------------------------- | --------------------------------------------------------- |
| 🛍️ [**customer-ui**](https://github.com/xshopai/customer-ui)   | React 18                   | Customer-facing e-commerce web application                |
| 🛡️ [**admin-ui**](https://github.com/xshopai/admin-ui)         | React 18<br>TypeScript 4   | Admin dashboard for platform management and analytics     |

### 🚀 Planned Services

| Service                       | Technology            | Description                                            |
| ----------------------------- | --------------------- | ------------------------------------------------------ |
| 🤖 **recommendation-service** | Python + ML/AI        | AI-powered product recommendations and personalization |
|  **analytics-service**      | Python + Apache Spark | Business intelligence and real-time analytics          |
| 📱 **mobile-app**             | React Native/Flutter  | Native mobile application for iOS and Android          |

## 🤝 Contributing

We welcome contributions from the community! Whether it's:

- 🐛 **Bug fixes** - Found an issue? Submit a PR!
- ✨ **New features** - Have an idea? Let's discuss it!
- 📖 **Documentation** - Help us improve our docs
- 🧪 **Tests** - Increase our test coverage

Please read our [Contributing Guidelines](https://github.com/xshopai/.github/blob/main/CONTRIBUTING.md) before submitting a pull request.

## 📚 Documentation

- 📘 [Architecture Overview](https://github.com/xshopai/.github/wiki/Architecture)
- 🔧 [Development Setup](https://github.com/xshopai/.github/wiki/Development-Setup)
- 🚀 [Deployment Guide](https://github.com/xshopai/.github/wiki/Deployment)
- 📡 [API Reference](https://github.com/xshopai/.github/wiki/API-Reference)

## 💬 Community & Support

- 💡 [GitHub Discussions](https://github.com/orgs/xshopai/discussions) - Ask questions, share ideas
- 🐛 [Issue Tracker](https://github.com/xshopai/.github/issues) - Report bugs

## ⭐ Star History

If you find this project helpful, please consider giving it a star! It helps others discover this resource.

## 📃 License

MIT License — feel free to use, extend, and modify.

---

<p align="center">
  Made with ❤️ by the xshopai
</p>
