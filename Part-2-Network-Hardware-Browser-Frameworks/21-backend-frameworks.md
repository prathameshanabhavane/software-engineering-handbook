# ⚙️ 21. Backend Frameworks & Server-Side Concepts

> **A backend framework is like a well-designed kitchen — it provides the oven (routing), utensils (utilities), and recipes (patterns) so you can focus on cooking (business logic) instead of building the kitchen from scratch.**

---

## 🔄 Request Lifecycle — How a Backend Processes a Request

```mermaid
sequenceDiagram
    participant Client
    participant Server as ⚙️ Server
    participant MW1 as 🔌 Middleware: Logger
    participant MW2 as 🔌 Middleware: Auth
    participant MW3 as 🔌 Middleware: Rate Limit
    participant Router as 🚏 Router
    participant Controller as 🎮 Controller
    participant Service as ⚙️ Service
    participant DB as 🗄️ Database

    Client->>Server: POST /api/orders {items, address}

    Server->>MW1: Log request
    MW1->>MW2: Check auth token
    alt Token invalid
        MW2-->>Client: 401 Unauthorized
    end
    MW2->>MW3: Check rate limit
    alt Rate exceeded
        MW3-->>Client: 429 Too Many Requests
    end
    MW3->>Router: Match route: POST /api/orders

    Router->>Controller: orderController.create(req, res)
    Controller->>Controller: Parse & validate request body
    Controller->>Service: orderService.createOrder(data)
    Service->>Service: Apply business rules
    Service->>DB: INSERT INTO orders ...
    DB-->>Service: Order created
    Service-->>Controller: Order object
    Controller-->>Client: 201 Created {orderId: 123}

    Note over MW1: Log response time & status
```

---

## 🔌 Middleware — The Pipeline

```mermaid
graph LR
    Request["📥 Request"] --> M1["🔍 Logger<br/>(log every request)"]
    M1 --> M2["🔐 Auth<br/>(verify JWT)"]
    M2 --> M3["🚦 Rate Limit<br/>(throttle abuse)"]
    M3 --> M4["📐 CORS<br/>(cross-origin headers)"]
    M4 --> M5["📦 Body Parser<br/>(parse JSON body)"]
    M5 --> M6["✅ Validator<br/>(validate input schema)"]
    M6 --> Route["🚏 Route Handler"]
    Route --> M7["❌ Error Handler<br/>(catch all errors)"]
    M7 --> Response["📤 Response"]

    style Request fill:#161b22,stroke:#58a6ff,color:#fff
    style Response fill:#161b22,stroke:#3fb950,color:#fff
```

### Express.js Middleware Example

```javascript
// Middleware executes in ORDER
app.use(logger());         // 1. Log every request
app.use(cors());           // 2. Handle CORS
app.use(express.json());   // 3. Parse JSON bodies
app.use(authenticate);     // 4. Verify auth token
app.use(rateLimiter);      // 5. Rate limiting

// Route handler
app.post('/api/orders', validate(orderSchema), orderController.create);

// Error handler (must be last — 4 args)
app.use((err, req, res, next) => {
  logger.error({ error: err.message, stack: err.stack });
  res.status(err.status || 500).json({ error: 'Internal server error' });
});
```

---

## 🗄️ ORM — Object-Relational Mapping

```mermaid
graph LR
    subgraph WITHOUT["❌ Raw SQL"]
        R1["db.query('SELECT * FROM users WHERE email = $1 AND status = $2', ['alice@ex.com', 'active'])"]
    end

    subgraph WITH["✅ ORM (Prisma/Sequelize/TypeORM)"]
        O1["User.findOne({<br/>  where: { email: 'alice@ex.com', status: 'active' }<br/>})"]
    end

    style WITHOUT fill:#0d1117,stroke:#d29922,color:#c9d1d9
    style WITH fill:#0d1117,stroke:#3fb950,color:#c9d1d9
```

### ORM Pros & Cons

| ✅ Pros | ❌ Cons |
|---------|---------|
| Type-safe queries | May generate inefficient SQL |
| Database-agnostic | Learning curve for complex queries |
| Auto-generates migrations | Hides what's actually happening |
| Prevents SQL injection | Performance overhead |
| Better developer experience | Complex joins can be awkward |

### When to Use Raw SQL vs ORM

| Scenario | Recommendation |
|----------|---------------|
| Simple CRUD operations | ORM ✅ |
| Complex reporting queries | Raw SQL ✅ |
| Rapid prototyping | ORM ✅ |
| Performance-critical queries | Raw SQL ✅ |
| Team with varying SQL skills | ORM ✅ |

---

## 🔑 Authentication Patterns

```mermaid
graph TB
    subgraph SESSION["🍪 Session-Based (Traditional)"]
        S1["Login → Server creates session → Stores in Redis"]
        S2["Server sends session ID as cookie"]
        S3["Every request: cookie sent → server looks up session"]
        S4["Stateful: server must store session data"]
    end

    subgraph TOKEN["🔑 Token-Based (JWT)"]
        T1["Login → Server creates JWT → Sends to client"]
        T2["Client stores JWT (cookie or localStorage)"]
        T3["Every request: JWT sent in header → server verifies signature"]
        T4["Stateless: server doesn't store anything"]
    end

    subgraph OAUTH["🔗 OAuth 2.0 (Third-Party)"]
        O1["User clicks 'Login with Google'"]
        O2["Redirect to Google → User authorizes"]
        O3["Google redirects back with auth code"]
        O4["Your server exchanges code for tokens"]
        O5["Create local session/JWT for user"]
    end

    style SESSION fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
    style TOKEN fill:#0d1117,stroke:#3fb950,color:#c9d1d9
    style OAUTH fill:#0d1117,stroke:#d29922,color:#c9d1d9
```

---

## 📡 API Design Patterns

### RESTful API Design

```mermaid
graph TB
    subgraph REST_RULES["REST Best Practices"]
        R1["Use nouns, not verbs<br/>✅ GET /users<br/>❌ GET /getUsers"]
        R2["Use HTTP methods correctly<br/>GET = read, POST = create<br/>PUT = replace, PATCH = partial update<br/>DELETE = remove"]
        R3["Use proper status codes<br/>200 OK, 201 Created<br/>400 Bad Request, 401 Unauthorized<br/>404 Not Found, 500 Server Error"]
        R4["Version your API<br/>/api/v1/users<br/>/api/v2/users"]
        R5["Pagination<br/>GET /users?page=2&limit=20"]
        R6["Filter & sort<br/>GET /users?role=admin&sort=name"]
    end

    style REST_RULES fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
```

### HTTP Status Codes Cheat Sheet

| Code | Meaning | When to Use |
|------|---------|-------------|
| **200** | OK | Successful GET, PUT, PATCH |
| **201** | Created | Successful POST (resource created) |
| **204** | No Content | Successful DELETE |
| **301** | Moved Permanently | URL has permanently changed |
| **304** | Not Modified | Cache is still valid |
| **400** | Bad Request | Invalid input / validation error |
| **401** | Unauthorized | Not logged in / bad token |
| **403** | Forbidden | Logged in but not permitted |
| **404** | Not Found | Resource doesn't exist |
| **409** | Conflict | Duplicate resource / version conflict |
| **422** | Unprocessable Entity | Valid JSON but semantic errors |
| **429** | Too Many Requests | Rate limit exceeded |
| **500** | Internal Server Error | Unhandled server error |
| **502** | Bad Gateway | Upstream service unreachable |
| **503** | Service Unavailable | Server overloaded / maintenance |

---

## 🔄 Background Jobs & Workers

```mermaid
graph TB
    subgraph WEB["🌐 Web Process (handles HTTP)"]
        API["API endpoints"]
        API -->|"Quick response"| User["👤 User"]
    end

    subgraph WORKERS["⚙️ Worker Process (background)"]
        W1["📧 Email Worker<br/>Send welcome emails"]
        W2["📊 Report Worker<br/>Generate PDFs"]
        W3["🖼️ Image Worker<br/>Resize & compress"]
        W4["🔄 Sync Worker<br/>Third-party integrations"]
    end

    API -->|"Enqueue job"| Queue["📬 Job Queue<br/>(Redis/Bull/BullMQ)"]
    Queue --> W1 & W2 & W3 & W4

    style WEB fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
    style WORKERS fill:#0d1117,stroke:#3fb950,color:#c9d1d9
    style Queue fill:#161b22,stroke:#d29922,color:#fff
```

---

## ⚠️ Edge Cases & Gotchas

1. **Don't put business logic in controllers** — Controllers should be thin: parse request → call service → send response. Business rules belong in the service layer.

2. **Error handling middleware must be last** — In Express, error handlers must have 4 arguments `(err, req, res, next)` and be registered after all routes.

3. **Database migrations in production** — Never run destructive migrations (drop column) without a multi-step approach. See [Chapter 7](../Part-1-Architecture-Scalability-Operations/07-database-design.md).

4. **Graceful shutdown** — When a server receives SIGTERM (during deploy), finish processing current requests before shutting down. Don't kill mid-request.

5. **Environment-specific config** — Never hardcode production URLs, keys, or credentials. Use environment variables that differ per environment (dev, staging, prod).

---

## 🔗 Connected Topics

| Topic | Connection |
|-------|-----------|
| [Clean Code](../Part-1-Architecture-Scalability-Operations/11-clean-modular-code.md) | Layered architecture: controller → service → repository |
| [Security](../Part-1-Architecture-Scalability-Operations/09-security.md) | Auth middleware, input validation, rate limiting |
| [Database](../Part-1-Architecture-Scalability-Operations/07-database-design.md) | ORM, connection pooling, migrations |
| [Latency](../Part-1-Architecture-Scalability-Operations/08-latency.md) | Middleware chain adds latency per layer |
| [Frontend Frameworks](20-frontend-frameworks.md) | API consumed by frontend, SSR integration |

---

**← Previous:** [20. Frontend Frameworks](20-frontend-frameworks.md) | **Next →** [22. CI/CD Pipeline](22-cicd-pipeline.md)
