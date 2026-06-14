# 🏛️ 2. Architecture Patterns — Choosing Your System's Structure

> **The wrong architecture is like wearing a spacesuit to go grocery shopping — technically it works, but it's massively over-engineered for the job.**

---

## 🔄 Architecture Decision Flow

```mermaid
flowchart TD
    Start["🚀 New Project"] --> Q1{"Team size?"}

    Q1 -->|"1-10 people"| Q2{"Domain well<br/>understood?"}
    Q1 -->|"10-50 people"| Q3{"Parts need to<br/>scale differently?"}
    Q1 -->|"50+ people"| MS["🔶 Microservices<br/>+ Event-Driven"]

    Q2 -->|"No / MVP"| Mono["🟢 Monolith"]
    Q2 -->|"Yes, stable"| Q4{"Traffic<br/>predictable?"}

    Q3 -->|"Yes"| MS
    Q3 -->|"No"| ModMono["🟢 Modular Monolith"]

    Q4 -->|"Steady traffic"| ModMono
    Q4 -->|"Spiky / unpredictable"| Q5{"Need real-time<br/>response?"}

    Q5 -->|"Yes"| ModMono
    Q5 -->|"No, event-based OK"| SL["🟣 Serverless<br/>+ Event-Driven"]

    Mono -->|"As you grow..."| ModMono
    ModMono -->|"When genuinely needed..."| MS

    style Start fill:#1a1a2e,stroke:#e94560,color:#fff
    style Mono fill:#0d1117,stroke:#3fb950,color:#fff
    style ModMono fill:#0d1117,stroke:#58a6ff,color:#fff
    style MS fill:#0d1117,stroke:#d29922,color:#fff
    style SL fill:#0d1117,stroke:#bc8cff,color:#fff
```

---

## 🟢 2.1 Monolith — The Food Truck

### What
One single codebase and deployable unit containing UI, business logic, and database access — everything runs as one application.

### How It Works

```mermaid
graph TB
    subgraph MONOLITH["🟢 Monolith — Single Deployment"]
        direction TB
        UI["🖥️ UI Layer<br/>(Templates/Views)"]
        Auth["🔑 Auth Module"]
        Orders["📦 Orders Module"]
        Payments["💳 Payments Module"]
        Notify["📧 Notifications Module"]
        DB_Layer["🗄️ Database Layer"]
    end

    Client["👤 Client"] -->|"HTTP"| UI
    UI --> Auth
    UI --> Orders
    UI --> Payments
    Orders --> Notify
    Auth --> DB_Layer
    Orders --> DB_Layer
    Payments --> DB_Layer
    DB_Layer --> DB[("🗄️ Single Database")]

    style MONOLITH fill:#0d1117,stroke:#3fb950,color:#c9d1d9
    style Client fill:#161b22,stroke:#58a6ff,color:#fff
    style DB fill:#161b22,stroke:#bc8cff,color:#fff
```

### When to Use
- Startups, MVPs, small-to-medium teams
- Domain not yet well understood (you'll refactor as you learn)
- Speed of development matters more than independent scaling
- Most successful companies started here (even Shopify runs a monolith serving millions)

### Analogy
A monolith is like a **food truck** — one team, one kitchen, everything in one place. Fast to set up, easy to manage, great when you're small.

### ✅ Pros & ❌ Cons

| ✅ Pros | ❌ Cons |
|---------|---------|
| Simple to develop, test, deploy | One bug can crash everything |
| No network calls between modules | Harder to scale individual parts |
| Easy to debug (one process) | Can become a "big ball of mud" if not organized |
| Fast internal communication | Deployments are all-or-nothing |
| Easy to refactor early on | Technology lock-in (one language/framework) |

---

## 🔶 2.2 Microservices — The Food Court

### What
The application is split into multiple independent services, each with its own codebase, database, and deployment pipeline, communicating over the network.

### How It Works

```mermaid
graph TB
    Client["👤 Client"] -->|"HTTPS"| Gateway["🚪 API Gateway"]

    subgraph SERVICES["🔶 Independent Services"]
        direction TB
        AuthSvc["🔑 Auth Service<br/><i>Node.js</i>"]
        OrderSvc["📦 Order Service<br/><i>Java</i>"]
        PaySvc["💳 Payment Service<br/><i>Go</i>"]
        NotifySvc["📧 Notification Service<br/><i>Python</i>"]
        SearchSvc["🔍 Search Service<br/><i>Elasticsearch</i>"]
    end

    Gateway --> AuthSvc
    Gateway --> OrderSvc
    Gateway --> PaySvc
    Gateway --> SearchSvc

    AuthSvc --> AuthDB[("Auth DB")]
    OrderSvc --> OrderDB[("Order DB")]
    PaySvc --> PayDB[("Payment DB")]
    SearchSvc --> SearchDB[("Search Index")]

    OrderSvc -->|"Event: OrderPlaced"| MQ["📬 Message Queue"]
    MQ --> NotifySvc
    MQ --> PaySvc

    NotifySvc --> NotifyDB[("Notification DB")]

    style Client fill:#161b22,stroke:#58a6ff,color:#fff
    style Gateway fill:#161b22,stroke:#d29922,color:#fff
    style SERVICES fill:#0d1117,stroke:#d29922,color:#c9d1d9
    style MQ fill:#161b22,stroke:#3fb950,color:#fff
```

### When to Use
- Large team (one team per service — "two-pizza teams")
- Domain is well understood and stable
- Different parts genuinely need to scale differently
- You need independent deployments (change payments without redeploying search)
- You can afford the operational complexity

### Analogy
A **food court** — each stall (pizza, sushi, burgers) operates independently with its own staff and supplies. If the sushi stall has a long line, it doesn't slow down the burger stall.

### ⚠️ The Hidden Costs

```mermaid
graph TD
    MS["Adopting Microservices"] --> Cost1["🌐 Network Latency<br/>Every service call = network hop"]
    MS --> Cost2["🐛 Distributed Debugging<br/>A bug could be in any of 20 services"]
    MS --> Cost3["🔄 Data Consistency<br/>No ACID across services"]
    MS --> Cost4["📦 Deployment Complexity<br/>20 services = 20 CI/CD pipelines"]
    MS --> Cost5["🔍 Service Discovery<br/>Services need to find each other"]
    MS --> Cost6["👥 Team Overhead<br/>Need DevOps, SRE, platform team"]
    MS --> Cost7["🧪 Integration Testing<br/>Testing across services is hard"]

    style MS fill:#161b22,stroke:#f85149,color:#fff
    style Cost1 fill:#0d1117,stroke:#f85149,color:#c9d1d9
    style Cost2 fill:#0d1117,stroke:#f85149,color:#c9d1d9
    style Cost3 fill:#0d1117,stroke:#f85149,color:#c9d1d9
    style Cost4 fill:#0d1117,stroke:#f85149,color:#c9d1d9
    style Cost5 fill:#0d1117,stroke:#f85149,color:#c9d1d9
    style Cost6 fill:#0d1117,stroke:#f85149,color:#c9d1d9
    style Cost7 fill:#0d1117,stroke:#f85149,color:#c9d1d9
```

### ✅ Pros & ❌ Cons

| ✅ Pros | ❌ Cons |
|---------|---------|
| Independent scaling per service | Huge operational complexity |
| Independent deployments | Network latency between services |
| Technology diversity (use best tool per service) | Distributed debugging is hard |
| Fault isolation (one service crash ≠ total crash) | Data consistency across services is complex |
| Teams can work independently | Need robust monitoring, tracing, service mesh |

---

## 🟣 2.3 Serverless — The Uber of Computing

### What
You write functions, the cloud provider runs them on-demand, scales automatically, and you pay only for execution time.

### How It Works

```mermaid
graph TB
    subgraph TRIGGERS["⚡ Event Triggers"]
        HTTP["HTTP Request<br/>(API Gateway)"]
        Upload["File Upload<br/>(S3 Bucket)"]
        Schedule["Cron Schedule<br/>(Every hour)"]
        Queue["Queue Message<br/>(SQS/SNS)"]
        DB_Event["DB Change<br/>(DynamoDB Stream)"]
    end

    subgraph SERVERLESS["🟣 Serverless Functions"]
        F1["λ processOrder()"]
        F2["λ resizeImage()"]
        F3["λ sendReport()"]
        F4["λ processPayment()"]
        F5["λ syncInventory()"]
    end

    HTTP --> F1
    Upload --> F2
    Schedule --> F3
    Queue --> F4
    DB_Event --> F5

    F1 --> DynamoDB[("DynamoDB")]
    F2 --> S3["S3 Bucket"]
    F3 --> SES["Email Service"]
    F4 --> Stripe["Stripe API"]
    F5 --> DB[("Database")]

    subgraph SCALING["📈 Auto-Scaling"]
        Zero["0 traffic = 0 cost<br/>💤 Sleeping"]
        Low["Low traffic = 1 instance<br/>💰 Pennies"]
        High["High traffic = 1000 instances<br/>💰💰 Still cheap"]
    end

    style TRIGGERS fill:#0d1117,stroke:#bc8cff,color:#c9d1d9
    style SERVERLESS fill:#0d1117,stroke:#bc8cff,color:#c9d1d9
    style SCALING fill:#0d1117,stroke:#3fb950,color:#c9d1d9
```

### When to Use
- Event-driven tasks (image processing, email sending, webhooks)
- Unpredictable or spiky traffic
- Want to avoid managing servers entirely
- APIs with low-to-moderate, irregular traffic
- Scheduled jobs (cron)

### Analogy
Calling an **Uber** instead of owning a car — you don't maintain the vehicle, you just pay per ride. As many "cars" as needed appear when demand spikes.

### ⚠️ Watch Out: Cold Starts

```mermaid
sequenceDiagram
    participant User
    participant Gateway as API Gateway
    participant Lambda as λ Function

    Note over Lambda: Function is COLD<br/>(no recent invocations)

    User->>Gateway: POST /api/order
    Gateway->>Lambda: Trigger function
    Note over Lambda: ⏳ Cold Start!<br/>- Download code<br/>- Initialize runtime<br/>- Load dependencies<br/>- ~200ms-2000ms delay
    Lambda->>Lambda: Execute function
    Lambda-->>Gateway: Response
    Gateway-->>User: 200 OK

    Note over Lambda: Function is now WARM

    User->>Gateway: POST /api/order
    Gateway->>Lambda: Trigger function
    Note over Lambda: ⚡ Warm Start!<br/>- Already initialized<br/>- ~5-50ms
    Lambda->>Lambda: Execute function
    Lambda-->>Gateway: Response
    Gateway-->>User: 200 OK (much faster!)
```

---

## 📢 2.4 Event-Driven Architecture — The Notice Board

### What
Components communicate by producing and consuming **events** through a message broker, rather than calling each other directly.

### How It Works

```mermaid
graph LR
    subgraph PRODUCERS["📤 Producers"]
        OrderSvc["Order Service"]
        UserSvc["User Service"]
        PaySvc["Payment Service"]
    end

    subgraph BROKER["📬 Message Broker (Kafka / RabbitMQ)"]
        direction TB
        T1["Topic: order.placed"]
        T2["Topic: user.registered"]
        T3["Topic: payment.received"]
    end

    subgraph CONSUMERS["📥 Consumers"]
        EmailSvc["Email Service"]
        AnalyticsSvc["Analytics Service"]
        InventorySvc["Inventory Service"]
        RewardsSvc["Rewards Service"]
        AuditSvc["Audit Log Service"]
    end

    OrderSvc -->|"publish"| T1
    UserSvc -->|"publish"| T2
    PaySvc -->|"publish"| T3

    T1 -->|"subscribe"| EmailSvc
    T1 -->|"subscribe"| AnalyticsSvc
    T1 -->|"subscribe"| InventorySvc
    T2 -->|"subscribe"| EmailSvc
    T2 -->|"subscribe"| RewardsSvc
    T3 -->|"subscribe"| InventorySvc
    T3 -->|"subscribe"| AuditSvc
    T3 -->|"subscribe"| AnalyticsSvc

    style PRODUCERS fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
    style BROKER fill:#0d1117,stroke:#d29922,color:#c9d1d9
    style CONSUMERS fill:#0d1117,stroke:#3fb950,color:#c9d1d9
```

### Key Event-Driven Patterns

```mermaid
graph TD
    subgraph PATTERN1["Pattern: Fire and Forget"]
        P1_Prod["Producer"] -->|"Publish event"| P1_Queue["Queue"]
        P1_Queue -->|"Consume"| P1_Con["Consumer"]
        P1_Prod -.->|"Doesn't wait<br/>for response"| P1_Note["✅ Producer<br/>responds immediately"]
    end

    subgraph PATTERN2["Pattern: Event Sourcing"]
        P2_Cmd["Command"] -->|"Append"| P2_Log["Event Log<br/>(immutable)"]
        P2_Log -->|"Replay"| P2_State["Current State"]
        P2_Log -->|"Audit"| P2_History["Complete History"]
    end

    subgraph PATTERN3["Pattern: Saga (Distributed Transaction)"]
        P3_Start["Order Placed"] --> P3_Pay["Payment"]
        P3_Pay -->|"Success"| P3_Inv["Update Inventory"]
        P3_Inv -->|"Success"| P3_Ship["Schedule Shipping"]
        P3_Pay -->|"Failure"| P3_Comp1["❌ Cancel Order<br/>(Compensating Action)"]
        P3_Inv -->|"Failure"| P3_Comp2["❌ Refund Payment<br/>(Compensating Action)"]
    end

    style PATTERN1 fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
    style PATTERN2 fill:#0d1117,stroke:#3fb950,color:#c9d1d9
    style PATTERN3 fill:#0d1117,stroke:#d29922,color:#c9d1d9
```

### When to Use
- Workflows with multiple steps that don't need instant response
- Order processing, notifications, analytics
- When you want to decouple services (add new consumers without changing producers)
- When resilience matters (if email service is down, events wait in queue)

### Analogy
A **notice board** in an office — when HR posts "New employee starting Monday," IT, Facilities, and the Team Lead all see it and act independently. HR doesn't have to call each department individually.

---

## 🔀 Side-by-Side Comparison

```mermaid
graph TB
    subgraph MONO["🟢 Monolith"]
        M1["All code in one place"]
        M2["Single deployment"]
        M3["One database"]
        M4["Internal function calls"]
    end

    subgraph MICRO["🔶 Microservices"]
        MS1["Code split by service"]
        MS2["Independent deployments"]
        MS3["Database per service"]
        MS4["Network calls (API/Queue)"]
    end

    subgraph SERVER["🟣 Serverless"]
        S1["Code as functions"]
        S2["No server management"]
        S3["Managed databases"]
        S4["Event-triggered"]
    end

    subgraph EVENT["📢 Event-Driven"]
        E1["Async communication"]
        E2["Decoupled producers/consumers"]
        E3["Event log as source of truth"]
        E4["Eventual consistency"]
    end

    style MONO fill:#0d1117,stroke:#3fb950,color:#c9d1d9
    style MICRO fill:#0d1117,stroke:#d29922,color:#c9d1d9
    style SERVER fill:#0d1117,stroke:#bc8cff,color:#c9d1d9
    style EVENT fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
```

| Aspect | Monolith | Microservices | Serverless | Event-Driven |
|--------|----------|---------------|------------|-------------|
| **Complexity** | Low | Very High | Medium | High |
| **Scaling** | Scale entire app | Scale per service | Auto per function | Scale consumers independently |
| **Team size** | 1-15 devs | 15+ devs | Any | Any (complements others) |
| **Latency** | Lowest (in-process) | Higher (network) | Variable (cold starts) | Eventual (async) |
| **Cost at low traffic** | Fixed server cost | High (many services) | Near zero | Medium |
| **Debugging** | Easy | Hard | Medium | Hard (async flows) |
| **Best for** | MVPs, small teams | Large orgs, complex domains | Spiky/event workloads | Decoupled workflows |

---

## 🏗️ The Evolution Path

Most successful systems follow this journey:

```mermaid
graph LR
    A["🟢 Simple Monolith<br/><i>Day 1: Build fast,<br/>validate the idea</i>"] -->|"Growing pains..."| B["🟢 Modular Monolith<br/><i>Organize internally,<br/>keep single deploy</i>"]
    B -->|"If genuinely needed..."| C["🔶 Extract Critical Services<br/><i>Split out what MUST<br/>scale independently</i>"]
    C -->|"At massive scale..."| D["🔶 Full Microservices<br/>+ 📢 Event-Driven<br/><i>Independent teams,<br/>independent services</i>"]

    A -.->|"⚠️ DON'T jump here<br/>on Day 1"| D

    style A fill:#0d1117,stroke:#3fb950,color:#fff
    style B fill:#0d1117,stroke:#58a6ff,color:#fff
    style C fill:#0d1117,stroke:#d29922,color:#fff
    style D fill:#0d1117,stroke:#f85149,color:#fff
```

---

## ⚠️ Edge Cases & Gotchas

1. **"Netflix uses microservices, so should we"** — Netflix has 10,000+ engineers. They didn't start with microservices. Don't copy the destination without considering your starting point.

2. **The distributed monolith trap** — If your "microservices" all share one database and must be deployed together, you've built a distributed monolith — all the complexity of microservices with none of the benefits.

3. **Service boundary mistakes** — Splitting services along the wrong boundaries (e.g., by technical layer instead of business domain) creates chatty services that constantly call each other.

4. **Ignoring the network** — In a monolith, a function call takes nanoseconds. In microservices, a network call takes milliseconds + can fail, timeout, or return garbage. Every internal boundary is now a potential failure point.

5. **Not having an API Gateway** — Without a gateway, clients need to know about every service's URL, handle auth for each, and deal with cross-cutting concerns separately.

---

## 🔗 Connected Topics

| Topic | Connection |
|-------|-----------|
| [Requirements](01-requirements.md) | NFRs (team size, scale, traffic pattern) determine the architecture choice |
| [Scalability](03-scalability.md) | Architecture determines how you can scale |
| [Load Balancers](04-load-balancers.md) | Essential once you have multiple instances (any pattern) |
| [Latency](08-latency.md) | Microservices add network latency; event-driven adds processing delay |
| [Monitoring](13-monitoring-observability.md) | More services = more critical observability becomes |

---

**← Previous:** [1. Requirements](01-requirements.md) | **Next →** [3. Scalability](03-scalability.md)
