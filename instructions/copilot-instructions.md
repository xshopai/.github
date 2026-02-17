# GitHub Copilot Instructions for xshopai

This file provides context and guidelines for GitHub Copilot to better assist with the xshopai microservices platform.

## Project Overview

xshopai is a comprehensive e-commerce platform built using a microservices architecture. The platform consists of 11 services implemented across 5 different technology stacks, designed for scalability, maintainability, and high performance.

### Core Business Domain

- **E-commerce Platform**: Online retail with AI-powered features
- **Multi-tenant Architecture**: Support for multiple vendors/sellers
- **Event-driven Design**: Asynchronous processing for scalability
- **Polyglot Persistence**: Different databases for different use cases

## Architecture Principles

### Microservices Design Patterns

- **Domain-Driven Design (DDD)**: Services organized by business capabilities
- **Database per Service**: Each service owns its data
- **Saga Pattern**: Choreography-based distributed transactions
- **Event Sourcing**: For order processing and audit trails
- **CQRS**: Command Query Responsibility Segregation where applicable

### Technology Stack Guidelines

- **Node.js/Express**: User-facing APIs, rapid development
- **C#/.NET 8**: Business-critical services requiring type safety
- **Python**: Data processing, AI/ML integration, analytics
- **Go**: High-performance, lightweight services
- **Java/Spring Boot**: Enterprise patterns, saga orchestration

## Service Portfolio

### Authentication & User Management

```typescript
// auth-service (Node.js/Express, Port 4000)
// Responsibilities: JWT authentication, MFA, social auth, session management
// Database: MongoDB
// Key patterns: Passport.js strategies, JWT refresh tokens, TOTP MFA

// user-service (Node.js/Express, Port 5000)
// Responsibilities: User profiles, preferences, address management
// Database: MongoDB
// Key patterns: CRUD operations, data validation, user lifecycle

// admin-service (Node.js/Express, Port 6000)
// Responsibilities: Administrative operations, user management
// Database: MongoDB
// Key patterns: Role-based access, bulk operations, reporting
```

### Product & Inventory Management

```python
# product-service (Python/FastAPI, Port 8000)
# Responsibilities: Product catalog, search, categories, metadata
# Database: MongoDB
# Key patterns: Full-text search, image handling, category hierarchies

# inventory-service (Python/Flask, Port 3000)
# Responsibilities: Stock levels, reservations, warehouse management
# Database: MySQL
# Key patterns: Concurrent stock updates, reservation system, alerts
```

### Order Processing

```csharp
// order-service (C#/.NET 8, Port 7000)
// Responsibilities: Order creation, status tracking, order history
// Database: PostgreSQL
// Key patterns: Entity Framework, FluentValidation, domain models

// payment-service (C#/.NET 8, Port 8080)
// Responsibilities: Payment processing, refunds, payment methods
// Database: SQL Server
// Key patterns: Stripe/PayPal integration, PCI compliance, idempotency
```

```java
// order-processor-service (Java/Spring Boot)
// Responsibilities: Saga orchestration, distributed transactions
// Database: PostgreSQL
// Key patterns: Spring Boot, choreography saga, event sourcing
```

### Supporting Services

```go
// cart-service (Go/Gin)
// Responsibilities: Shopping cart, session management
// Database: Redis
// Key patterns: Session-based storage, TTL, cart abandonment
```

```typescript
// review-service (Node.js/Express)
// Responsibilities: Product reviews, ratings, moderation
// Database: MongoDB
// Key patterns: Aggregation pipelines, content moderation, sentiment analysis

// notification-service (Node.js/Express)
// Responsibilities: Email, SMS, push notifications
// Database: MongoDB
// Key patterns: Template engine, queue processing, delivery tracking

// audit-service (TypeScript/Node.js)
// Responsibilities: Audit logging, compliance, data lineage
// Database: PostgreSQL
// Key patterns: Event logging, immutable records, retention policies
```

## Common Patterns & Standards

### Authentication & Authorization

```javascript
// JWT Token Structure
const tokenPayload = {
  id: user.id,
  email: user.email,
  roles: user.roles, // ['user', 'admin', 'vendor']
  iat: issuedAt,
  exp: expiration,
};

// Role-based middleware pattern
export const authorizeRoles =
  (...roles) =>
  (req, res, next) => {
    if (!req.user || !roles.some((role) => req.user.roles?.includes(role))) {
      return next(new ErrorResponse('Forbidden: insufficient role', 403));
    }
    next();
  };

// Usage in routes
router.get('/admin-only', requireAuth, authorizeRoles('admin'), adminController);
```

### Correlation ID Pattern

```javascript
// Always include correlation ID in requests
class CorrelationIdHelper {
  static getCorrelationId(req) {
    return req.correlationId || 'unknown';
  }

  static createHeaders(req) {
    return {
      'X-Correlation-ID': this.getCorrelationId(req),
      'Content-Type': 'application/json',
    };
  }
}

// Inter-service HTTP calls
const response = await serviceClient.get(req, '/api/endpoint', {
  headers: CorrelationIdHelper.createHeaders(req),
});
```

### Error Handling Standards

```javascript
// Standard error response format
class ErrorResponse extends Error {
  constructor(message, statusCode, errors = null) {
    super(message);
    this.statusCode = statusCode;
    this.errors = errors;
    this.isOperational = true;
  }
}

// Usage
throw new ErrorResponse('User not found', 404);
throw new ErrorResponse('Validation failed', 400, validationErrors);
```

### Logging Standards

```javascript
// Structured logging with correlation ID
logger.info('Operation completed', {
  operation: 'CREATE_USER',
  userId: user.id,
  correlationId: CorrelationIdHelper.getCorrelationId(req),
  duration: Date.now() - startTime,
  metadata: { additionalContext },
});

// Error logging
logger.error('Operation failed', {
  operation: 'PROCESS_PAYMENT',
  error: error.message,
  stack: error.stack,
  correlationId: CorrelationIdHelper.getCorrelationId(req),
});
```

### Rate Limiting Patterns

```javascript
// Service-specific rate limits
const createRateLimit = (windowMs, max, message) =>
  rateLimit({
    windowMs,
    max,
    message,
    standardHeaders: true,
    legacyHeaders: false,
    handler: (req, res) => {
      logger.warn('Rate limit exceeded', {
        ip: req.ip,
        endpoint: req.path,
        correlationId: CorrelationIdHelper.getCorrelationId(req),
      });
      res.status(429).json({ error: message });
    },
  });

// Different limits for different operations
const profileRateLimit = createRateLimit(15 * 60 * 1000, 100, 'Too many profile requests');
const loginRateLimit = createRateLimit(15 * 60 * 1000, 5, 'Too many login attempts');
```

### Database Connection Patterns

```javascript
// MongoDB connection with retry logic
const connectDB = async () => {
  try {
    const conn = await mongoose.connect(process.env.MONGODB_URI, {
      useNewUrlParser: true,
      useUnifiedTopology: true,
      maxPoolSize: 10,
      serverSelectionTimeoutMS: 5000,
    });

    logger.info('MongoDB connected', { host: conn.connection.host });
    global.mongoUrl = process.env.MONGODB_URI;
  } catch (error) {
    logger.error('Database connection failed', { error: error.message });
    process.exit(1);
  }
};
```

### Health Check Implementation

```javascript
// Standard health check endpoints
export const health = async (req, res) => {
  const checks = {
    database: await checkDatabase(),
    redis: await checkRedis(),
    externalServices: await checkExternalServices(),
  };

  const isHealthy = Object.values(checks).every((check) => check.healthy);
  const status = isHealthy ? 'healthy' : 'unhealthy';

  res.status(isHealthy ? 200 : 503).json({
    status,
    timestamp: new Date().toISOString(),
    service: process.env.NAME,
    version: process.env.VERSION,
    checks,
  });
};
```

## Message Broker Patterns

### RabbitMQ Event Publishing

```javascript
// Event publisher pattern
class EventPublisher {
  constructor(connection) {
    this.connection = connection;
  }

  async publish(exchange, routingKey, message, options = {}) {
    const channel = await this.connection.createChannel();

    await channel.assertExchange(exchange, 'topic', { durable: true });

    const messageBuffer = Buffer.from(
      JSON.stringify({
        ...message,
        timestamp: new Date().toISOString(),
        correlationId: options.correlationId,
      }),
    );

    await channel.publish(exchange, routingKey, messageBuffer, {
      persistent: true,
      correlationId: options.correlationId,
    });

    await channel.close();
  }
}

// Usage
await eventPublisher.publish(
  'orders',
  'order.created',
  {
    orderId: order.id,
    userId: order.userId,
    amount: order.total,
  },
  { correlationId },
);
```

### Event Subscriber Pattern

```javascript
// Event subscriber pattern
class EventSubscriber {
  constructor(connection) {
    this.connection = connection;
  }

  async subscribe(exchange, pattern, handler) {
    const channel = await this.connection.createChannel();

    await channel.assertExchange(exchange, 'topic', { durable: true });
    const { queue } = await channel.assertQueue('', { exclusive: true });

    await channel.bindQueue(queue, exchange, pattern);

    await channel.consume(queue, async (msg) => {
      if (msg) {
        try {
          const data = JSON.parse(msg.content.toString());
          await handler(data);
          channel.ack(msg);
        } catch (error) {
          logger.error('Event processing failed', {
            error: error.message,
            pattern,
            correlationId: msg.properties.correlationId,
          });
          channel.nack(msg, false, false);
        }
      }
    });
  }
}
```

## API Design Guidelines

### REST API Conventions

```javascript
// Resource naming conventions
GET    /api/users              // List users
GET    /api/users/{id}         // Get specific user
POST   /api/users              // Create user
PUT    /api/users/{id}         // Update entire user
PATCH  /api/users/{id}         // Partial update
DELETE /api/users/{id}         // Delete user

// Nested resources
GET    /api/users/{id}/orders        // User's orders
POST   /api/users/{id}/addresses     // Add address to user
GET    /api/orders/{id}/items        // Order items

// Filtering and pagination
GET    /api/products?category=electronics&page=2&limit=20&sort=price:asc
```

### Response Formats

```javascript
// Success response format
{
  "success": true,
  "data": { /* resource data */ },
  "metadata": {
    "correlationId": "uuid",
    "timestamp": "2025-09-18T10:30:45.123Z"
  }
}

// Error response format
{
  "success": false,
  "error": {
    "message": "User not found",
    "code": "USER_NOT_FOUND",
    "statusCode": 404
  },
  "metadata": {
    "correlationId": "uuid",
    "timestamp": "2025-09-18T10:30:45.123Z"
  }
}

// Validation error response
{
  "success": false,
  "error": {
    "message": "Validation failed",
    "code": "VALIDATION_ERROR",
    "statusCode": 400,
    "details": [
      { "field": "email", "message": "Invalid email format" },
      { "field": "password", "message": "Password too weak" }
    ]
  }
}
```

## Environment Configuration

### Environment Variables Pattern

```bash
# Service identification
NAME=user-service
VERSION=1.0.0
NODE_ENV=development

# Database configuration
MONGODB_URI=mongodb://localhost:27017/user_service_dev_db
REDIS_URL=redis://localhost:6379
DATABASE_POOL_SIZE=10

# Authentication
JWT_SECRET=your_jwt_secret_minimum_32_characters
JWT_EXPIRES_IN=24h
BCRYPT_ROUNDS=12

# External services
AUTH_SERVICE_URL=http://localhost:8004
PRODUCT_SERVICE_URL=http://localhost:8001
PAYMENT_SERVICE_URL=http://localhost:8009

# Message broker
RABBITMQ_URL=amqp://admin:admin123@localhost:5672
RABBITMQ_EXCHANGE=xshopai.events

# Observability
LOG_LEVEL=debug
LOG_FORMAT=console
OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4318/v1/traces

# Rate limiting
RATE_LIMIT_WINDOW_MS=900000
RATE_LIMIT_MAX_REQUESTS=100
```

## Testing Patterns

### Unit Testing Standards

```javascript
// Jest test structure
describe('UserService', () => {
  let userService;
  let mockUserRepository;

  beforeEach(() => {
    mockUserRepository = {
      findById: jest.fn(),
      create: jest.fn(),
      update: jest.fn(),
      delete: jest.fn(),
    };
    userService = new UserService(mockUserRepository);
  });

  describe('createUser', () => {
    it('should create user with valid data', async () => {
      // Arrange
      const userData = {
        email: 'test@example.com',
        password: 'ValidPassword123!',
        firstName: 'John',
        lastName: 'Doe',
      };
      mockUserRepository.create.mockResolvedValue({ id: 1, ...userData });

      // Act
      const result = await userService.createUser(userData);

      // Assert
      expect(result).toBeDefined();
      expect(result.email).toBe(userData.email);
      expect(mockUserRepository.create).toHaveBeenCalledWith(
        expect.objectContaining({
          email: userData.email,
          password: expect.any(String), // hashed password
        }),
      );
    });
  });
});
```

### Integration Testing

```javascript
// Supertest integration tests
describe('User API', () => {
  let app;
  let authToken;

  beforeAll(async () => {
    app = await createTestApp();
    authToken = await getTestAuthToken();
  });

  afterAll(async () => {
    await cleanupTestData();
    await closeTestApp(app);
  });

  describe('POST /api/users', () => {
    it('should create user with valid data', async () => {
      const userData = {
        email: 'newuser@example.com',
        password: 'ValidPassword123!',
        firstName: 'Jane',
        lastName: 'Smith',
      };

      const response = await request(app)
        .post('/api/users')
        .set('Authorization', `Bearer ${authToken}`)
        .send(userData)
        .expect(201);

      expect(response.body.success).toBe(true);
      expect(response.body.data.email).toBe(userData.email);
    });
  });
});
```

## Code Quality Standards

### ESLint Configuration Preferences

```javascript
// Prefer explicit error handling
try {
  const result = await riskyOperation();
  return result;
} catch (error) {
  logger.error('Operation failed', { error: error.message });
  throw new ErrorResponse('Operation failed', 500);
}

// Prefer async/await over promises
// ✅ Good
const user = await userService.findById(userId);

// ❌ Avoid
userService.findById(userId).then((user) => {
  // handle user
});

// Prefer explicit return types in TypeScript
async function createUser(userData: CreateUserRequest): Promise<UserResponse> {
  // implementation
}
```

### Security Best Practices

```javascript
// Input validation
const validateCreateUser = [
  body('email').isEmail().normalizeEmail(),
  body('password')
    .isLength({ min: 8 })
    .matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/),
  body('firstName').trim().isLength({ min: 2, max: 50 }),
  body('lastName').trim().isLength({ min: 2, max: 50 }),
];

// Sanitize output
const sanitizeUser = (user) => ({
  id: user.id,
  email: user.email,
  firstName: user.firstName,
  lastName: user.lastName,
  createdAt: user.createdAt,
  // Never include password, internal IDs, etc.
});

// Rate limiting for sensitive operations
const sensitiveOperationsSlowDown = slowDown({
  windowMs: 15 * 60 * 1000, // 15 minutes
  delayAfter: 2, // Allow 2 requests per windowMs without delay
  delayMs: 500, // Add 500ms delay per request after delayAfter
  maxDelayMs: 20000, // Max delay of 20 seconds
});
```

## Monitoring & Observability

### OpenTelemetry Instrumentation

```javascript
// Trace instrumentation
import { trace } from '@opentelemetry/api';

const tracer = trace.getTracer('user-service', '1.0.0');

async function createUser(userData) {
  const span = tracer.startSpan('createUser');

  try {
    span.setAttributes({
      'user.email': userData.email,
      'operation.type': 'CREATE_USER',
    });

    const user = await userRepository.create(userData);

    span.setStatus({ code: trace.SpanStatusCode.OK });
    return user;
  } catch (error) {
    span.recordException(error);
    span.setStatus({
      code: trace.SpanStatusCode.ERROR,
      message: error.message,
    });
    throw error;
  } finally {
    span.end();
  }
}
```

## Common Gotchas & Best Practices

### Database Transactions

```javascript
// MongoDB transactions
const session = await mongoose.startSession();
try {
  await session.withTransaction(async () => {
    const user = await User.create([userData], { session });
    await Profile.create([profileData], { session });
    return user[0];
  });
} finally {
  await session.endSession();
}
```

### Memory Management

```javascript
// Prevent memory leaks in event listeners
process.on('SIGTERM', gracefulShutdown);
process.on('SIGINT', gracefulShutdown);

async function gracefulShutdown() {
  logger.info('Shutting down gracefully...');

  // Close database connections
  await mongoose.connection.close();

  // Close Redis connection
  await redisClient.quit();

  // Close message broker connection
  await messageBroker.close();

  process.exit(0);
}
```

### Performance Optimization

```javascript
// Database query optimization
const users = await User.find({ active: true })
  .select('id email firstName lastName') // Only select needed fields
  .limit(50) // Limit results
  .lean(); // Return plain objects instead of Mongoose documents

// Caching strategy
const getCachedUser = async (userId) => {
  const cacheKey = `user:${userId}`;

  // Try cache first
  let user = await redis.get(cacheKey);
  if (user) {
    return JSON.parse(user);
  }

  // Fetch from database
  user = await User.findById(userId).lean();
  if (user) {
    await redis.setex(cacheKey, 300, JSON.stringify(user)); // Cache for 5 minutes
  }

  return user;
};
```

## Development Workflow

### Git Conventions

```bash
# Commit message format
feat(user-service): add email verification endpoint
fix(auth-service): resolve JWT token expiration issue
docs(api): update authentication documentation
test(order-service): add integration tests for order creation
refactor(common): extract correlation ID helper to shared library

# Branch naming
feature/user-email-verification
bugfix/jwt-token-expiration
hotfix/critical-payment-issue
```

### Code Review Checklist

- [ ] Authentication and authorization properly implemented
- [ ] Correlation ID propagated through all operations
- [ ] Error handling with proper HTTP status codes
- [ ] Input validation and sanitization
- [ ] Logging with appropriate level and context
- [ ] Rate limiting on public endpoints
- [ ] Unit tests cover main functionality
- [ ] Database queries optimized and indexed
- [ ] Memory leaks prevented (event listener cleanup)
- [ ] Environment variables used for configuration

## Documentation Standards

### API Documentation

```javascript
/**
 * Create a new user account
 *
 * @route POST /api/users
 * @param {CreateUserRequest} userData - User registration data
 * @returns {Promise<UserResponse>} Created user data
 * @throws {ValidationError} When input data is invalid
 * @throws {ConflictError} When email already exists
 *
 * @example
 * const userData = {
 *   email: 'user@example.com',
 *   password: 'SecurePass123!',
 *   firstName: 'John',
 *   lastName: 'Doe'
 * };
 * const user = await createUser(userData);
 */
```

### README Structure

Each service should have:

- Purpose and responsibilities
- API endpoints documentation
- Environment setup instructions
- Database schema/models
- Testing instructions
- Deployment guidelines
- Troubleshooting guide

This instruction file should help GitHub Copilot provide more accurate and contextually relevant suggestions for the xshopai platform. Update this file as the architecture evolves or new patterns are adopted.
