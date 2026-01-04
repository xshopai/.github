# 🛍️ xshop.ai

**xshop.ai** is a reference, open-source e-commerce platform built with a modern, microservices-based architecture. It is designed to serve as a learning resource and architectural blueprint for developers, architects, and students exploring distributed systems, full-stack development, and cloud-native design.

## 🌐 Features

**xshop.ai** is a modern, modular, and AI-ready e-commerce platform — similar in ambition to Amazon — with support for both responsive web and native mobile applications. It is built from the ground up for flexibility, scalability, and extensibility, following these core principles:

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

- **🌐 Polyglot Architecture**: Each service uses the technology best suited for its requirements
- **🔄 Event-Driven Communication**: Services communicate via message queues and events
- **📊 Comprehensive Audit Logging**: Full activity tracking and compliance monitoring
- **🔒 Multi-Factor Authentication**: Enhanced security with OTP and account linking
- **⚡ High-Performance Services**: Go-based inventory service for speed-critical operations
- **🎯 Saga Pattern**: Distributed transaction management in order processing
- **📈 Scalable Data Storage**: PostgreSQL, MongoDB, and Redis for different data patterns
- **🛡️ Security-First Design**: JWT authentication, service-to-service tokens, and audit trails

## 🧩 Microservices Overview

Each microservice is hosted in its own GitHub repository for separation of concerns, independent scalability, and streamlined DevOps workflows. The platform demonstrates a polyglot architecture with different technologies chosen for each service's specific requirements:

### Backend Services

| Service                                                                               | Technology Stack      | Description                                                                  |
| ------------------------------------------------------------------------------------- | --------------------- | ---------------------------------------------------------------------------- |
| 🔐 [**auth-service**](https://github.com/xshopai/auth-service)                       | Node.js 20<br>Express 5<br>MongoDB 8 | Handles authentication, MFA, social login (OAuth2), and JWT issuance         |
| 👤 [**user-service**](https://github.com/xshopai/user-service)                       | Node.js 20<br>Express 5<br>MongoDB 8 | Manages user profiles, identity records, preferences, and account linking    |
| 🛡️ [**admin-service**](https://github.com/xshopai/admin-service)                     | Node.js 20<br>Express 5<br>MongoDB 8 | Back-office operations, dashboard, analytics, and user management            |
| 📋 [**audit-service**](https://github.com/xshopai/audit-service)                     | Node.js 20<br>TypeScript 5<br>PostgreSQL 16  | Comprehensive audit logging, compliance tracking, and activity monitoring    |
| 🛍️ [**product-service**](https://github.com/xshopai/product-service)                 | Python 3.11<br>FastAPI 0.104<br>MongoDB 8      | Handles product catalog, categories, attributes, search, and recommendations |
| 📦 [**inventory-service**](https://github.com/xshopai/inventory-service)              | Python 3.11<br>Flask 3.0<br>MySQL 8              | High-performance inventory management, stock tracking, and reservations      |
| 🧾 [**order-service**](https://github.com/xshopai/order-service)                     | .NET 8<br>ASP.NET Core 8<br>SQL Server 2022 | Order creation, validation, and lifecycle management                         |
| 💳 [**payment-service**](https://github.com/xshopai/payment-service)                  | .NET 8<br>ASP.NET Core 8<br>SQL Server 2022 | Payment processing, gateway integration, and transaction security            |
| ⚙️ [**order-processor-service**](https://github.com/xshopai/order-processor-service) | Java 21<br>Spring Boot 3.3<br>PostgreSQL 16    | Asynchronous order processing with saga pattern and event sourcing           |
| 🛒 [**cart-service**](https://github.com/xshopai/cart-service)                       | Java 21<br>Quarkus 3.6<br>Redis 7 (via Dapr)        | Shopping cart management with Redis for session handling                     |
| ⭐ [**review-service**](https://github.com/xshopai/review-service)                   | Node.js 20<br>Express 4<br>MongoDB 8     | Product reviews, ratings, and customer feedback management                   |
| 📣 [**notification-service**](https://github.com/xshopai/notification-service)       | Node.js 20<br>Express 4<br>TypeScript 5     | Multi-channel notifications (email, SMS, push, WebSocket)                    |
| 🌐 [**web-bff**](https://github.com/xshopai/web-bff)                                 | Node.js 20<br>Express 4<br>TypeScript 5  | Backend for Frontend aggregating data from multiple microservices            |

> **Note:** Service-to-service communication and event-driven messaging is handled by [DAPR (Distributed Application Runtime)](https://dapr.io), eliminating the need for a separate message broker service.

### Frontend Applications

| Application                                                     | Technology Stack  | Description                                        |
| --------------------------------------------------------------- | ----------- | -------------------------------------------------- |
| 🛍️ [**customer-ui**](https://github.com/xshopai/customer-ui)   | React 18       | Customer-facing e-commerce web application         |
| 🛡️ [**admin-ui**](https://github.com/xshopai/admin-ui)         | React 18<br>TypeScript 4       | Admin dashboard for platform management and analytics |

### 🚀 Planned Services

| Service                       | Technology            | Description                                            |
| ----------------------------- | --------------------- | ------------------------------------------------------ |
| 🤖 **recommendation-service** | Python + ML/AI        | AI-powered product recommendations and personalization |
| 🔍 **search-service**         | Elasticsearch         | Advanced product search with filters and facets        |
| 📊 **analytics-service**      | Python + Apache Spark | Business intelligence and real-time analytics          |
| 💬 **chatbot-service**        | Python + NLP/AI       | Intelligent customer support chatbot                   |
| 📱 **mobile-app**             | React Native/Flutter  | Native mobile application for iOS and Android          |

## 📃 License

MIT License — feel free to use, extend, and modify.
