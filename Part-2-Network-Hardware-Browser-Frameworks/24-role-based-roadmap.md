# 🎯 24. Role-Based Roadmap — What to Focus On

> **You don't need to learn everything at once. Focus on your role first, then expand outward.**

---

## 🎨 Frontend Developer Roadmap

```mermaid
graph TD
    %% Stage 1
    subgraph S1["🟢 Core (Master First)"]
        F1["HTML, CSS, JS Fundamentals"] --> F2["Browser Internals (Ch 19)<br/>DOM, CSSOM, Event Loop"]
        F2 --> F3["Frontend Frameworks (Ch 20)<br/>React, Vue, Angular"]
        F3 --> F4["Build Tools (Ch 20)<br/>Vite, Webpack"]
    end

    %% Stage 2
    subgraph S2["🟡 Expand (Grow Into)"]
        F5["Page Speed & SEO (Ch 6)<br/>Core Web Vitals"] --> F6["Caching & CDN (Ch 5-6)<br/>Browser Cache, Edge"]
        F6 --> F7["Networking (Ch 17)<br/>HTTP, REST, WebSockets"]
        F7 --> F8["Latency & Performance (Ch 8, 12)<br/>Bundle sizes, Lazy Loading"]
    end

    %% Stage 3
    subgraph S3["🔴 Advanced (Stand Out)"]
        F9["URL Journey (Ch 16)<br/>Full Request Lifecycle"] --> F10["Security (Ch 9)<br/>XSS, CSRF, CORS"]
        F10 --> F11["Accessibility (Ch 15)<br/>Semantic HTML & a11y"]
        F11 --> F12["Backend Basics (Ch 21)<br/>APIs, Databases"]
        F12 --> F13["CI/CD (Ch 22)<br/>Testing, deployment"]
    end

    %% Connect stages
    F4 --> F5
    F8 --> F9

    style S1 fill:#0d1117,stroke:#3fb950,color:#c9d1d9
    style S2 fill:#0d1117,stroke:#d29922,color:#c9d1d9
    style S3 fill:#0d1117,stroke:#f85149,color:#c9d1d9
```

---

## ⚙️ Backend Developer Roadmap

```mermaid
graph TD
    %% Stage 1
    subgraph S1["🟢 Core (Master First)"]
        B1["Server-side Language<br/>Node.js, Python, Java, Go"] --> B2["Backend Frameworks (Ch 21)<br/>Express, Django, Spring"]
        B2 --> B3["Database Design (Ch 7)<br/>SQL, Indexing, ACID"]
        B3 --> B4["API Design (Ch 21)<br/>REST, HTTP response flow"]
    end

    %% Stage 2
    subgraph S2["🟡 Expand (Grow Into)"]
        B5["Architecture (Ch 2)<br/>Monolith vs Microservices"] --> B6["Caching (Ch 5)<br/>Redis, Cache-aside"]
        B6 --> B7["Security (Ch 9)<br/>Auth, Encryption, Validation"]
        B7 --> B8["Scalability (Ch 3)<br/>Horizontal scaling, Stateless"]
        B8 --> B9["Load Balancers (Ch 4)<br/>Algorithms, health checks"]
    end

    %% Stage 3
    subgraph S3["🔴 Advanced (Stand Out)"]
        B10["Latency & Async (Ch 8)<br/>Message Queues, Workers"] --> B11["Monitoring (Ch 13)<br/>Metrics, Traces, Logs"]
        B11 --> B12["Networking (Ch 17)<br/>TCP, TLS, HTTP/2, gRPC"]
        B12 --> B13["Hardware & Cloud (Ch 18)<br/>Containers, K8s"]
        B13 --> B14["Governance (Ch 10)<br/>Compliance, data policies"]
    end

    %% Connect stages
    B4 --> B5
    B9 --> B10

    style S1 fill:#0d1117,stroke:#3fb950,color:#c9d1d9
    style S2 fill:#0d1117,stroke:#d29922,color:#c9d1d9
    style S3 fill:#0d1117,stroke:#f85149,color:#c9d1d9
```

---

## 🔄 Fullstack Developer Roadmap

```mermaid
graph TD
    %% Stage 1
    subgraph S1["🟢 Core (Master First)"]
        FS1["Frontend & Backend Fundamentals"] --> FS2["URL Journey (Ch 16)<br/>End-to-end understanding"]
        FS2 --> FS3["Database & API Design<br/>(Ch 7 & Ch 21)"]
        FS3 --> FS4["E2E Scenario (Ch 23)<br/>Trace a click through all layers"]
    end

    %% Stage 2
    subgraph S2["🟡 Expand"]
        FS5["Architecture Patterns (Ch 2)"] --> FS6["Caching & CDN (Ch 5-6)"]
        FS6 --> FS7["Security (Ch 9)"]
        FS7 --> FS8["Performance & Latency (Ch 8, 12)"]
        FS8 --> FS9["CI/CD Pipeline (Ch 22)"]
    end

    %% Stage 3
    subgraph S3["🔴 Advanced"]
        FS10["Scalability (Ch 3)"] --> FS11["Load Balancers (Ch 4)"]
        FS11 --> FS12["Monitoring & Logs (Ch 13)"]
        FS12 --> FS13["Networking (Ch 17)"]
        FS13 --> FS14["Hardware & Cloud (Ch 18)"]
        FS14 --> FS15["Governance (Ch 10)"]
    end

    %% Connect stages
    FS4 --> FS5
    FS9 --> FS10

    style S1 fill:#0d1117,stroke:#3fb950,color:#c9d1d9
    style S2 fill:#0d1117,stroke:#d29922,color:#c9d1d9
    style S3 fill:#0d1117,stroke:#f85149,color:#c9d1d9
```

---

## 📊 Skills Matrix — What Each Role Should Know

| Topic | 🎨 Frontend | ⚙️ Backend | 🔄 Fullstack |
|-------|:-----------:|:----------:|:------------:|
| Requirements (Ch 1) | 🟡 | 🟢 | 🟢 |
| Architecture (Ch 2) | 🟡 | 🟢 | 🟢 |
| Scalability (Ch 3) | ⚪ | 🟢 | 🟡 |
| Load Balancers (Ch 4) | ⚪ | 🟢 | 🟡 |
| Caching (Ch 5) | 🟢 | 🟢 | 🟢 |
| CDN & SEO (Ch 6) | 🟢 | 🟡 | 🟢 |
| Database (Ch 7) | 🟡 | 🟢 | 🟢 |
| Latency (Ch 8) | 🟢 | 🟢 | 🟢 |
| Security (Ch 9) | 🟡 | 🟢 | 🟢 |
| Governance (Ch 10) | ⚪ | 🟡 | 🟡 |
| Clean Code (Ch 11) | 🟢 | 🟢 | 🟢 |
| Performance (Ch 12) | 🟢 | 🟢 | 🟢 |
| Monitoring (Ch 13) | 🟡 | 🟢 | 🟢 |
| URL Journey (Ch 16) | 🟢 | 🟡 | 🟢 |
| Networking (Ch 17) | 🟡 | 🟢 | 🟢 |
| Hardware (Ch 18) | ⚪ | 🟡 | 🟡 |
| Browser Internals (Ch 19) | 🟢 | ⚪ | 🟡 |
| Frontend Frameworks (Ch 20) | 🟢 | ⚪ | 🟢 |
| Backend Frameworks (Ch 21) | ⚪ | 🟢 | 🟢 |
| CI/CD (Ch 22) | 🟡 | 🟢 | 🟢 |

🟢 Must know | 🟡 Should know | ⚪ Nice to know

---

## 🧠 Senior & Staff Engineer Decision-Making Framework

> **A junior developer focuses on how to write code. A senior developer focuses on how to design components. A staff engineer or architect focuses on the trade-offs, business constraints, and system evolution.**

To make sound technical decisions and guide other engineers, you must master the fundamental trade-offs and rules of software architecture.

### 📈 Software Engineer Career & Competency Leveling Flowchart

```mermaid
graph TD
    %% Levels
    subgraph L1["🟢 L1: Junior Developer (Focus: Task Execution)"]
        A1["Syntax & Core Language Features"] --> A2["Clean Code & Basic Unit Tests"]
        A2 --> A3["Implement Isolated Components & Tasks"]
    end

    subgraph L2["🟡 L2: Mid-Level Engineer (Focus: Autonomy & Components)"]
        B1["Modular Component Design (Layered Arch)"] --> B2["Integration Testing & Profiling"]
        B2 --> B3["API & Database Design (REST, Indexing)"]
    end

    subgraph L3["🟠 L3: Senior Engineer (Focus: System Design & Strategy)"]
        C1["Scalable System Design (Caching, LB, Replicas)"] --> C2["NFR & Requirements Gathering (FR/NFR)"]
        C2 --> C3["Observability & Monitoring (Metrics, Traces, Logs)"]
    end

    subgraph L4["🔴 L4: Staff Engineer & Architect (Focus: Business & Alignment)"]
        D1["Analyze Deep Trade-offs (CAP, Latency/Throughput)"] --> D2["Build vs Buy Decisions & Technical Standards"]
        D2 --> D3["Manage Tech Debt & Long-term System Evolution"]
    end

    %% Progression
    A3 --> B1
    B3 --> C1
    C3 --> D1

    style L1 fill:#0d1117,stroke:#3fb950,color:#c9d1d9
    style L2 fill:#0d1117,stroke:#d29922,color:#c9d1d9
    style L3 fill:#0d1117,stroke:#e94560,color:#c9d1d9
    style L4 fill:#0d1117,stroke:#f85149,color:#c9d1d9
```

---

### 🗺️ Architectural Decision-Making Flowchart

```mermaid
flowchart TD
    Start["📋 New System / Component Design"] --> Q1{"Is it a core business differentiator?"}
    
    Q1 -->|"No"| Buy["🛒 BUY / SaaS<br/>(managed service, Auth0, Stripe, Algolia)"]
    Q1 -->|"Yes (Build)"| Q2{"What is the read:write ratio?"}
    
    Q2 -->|"Read-heavy (e.g. 95:5)"| ReadOpt["⚡ Read-Optimized Pattern<br/>• Heavy caching (Redis/CDN)<br/>• Read replicas<br/>• Search indexer (Elasticsearch)"]
    Q2 -->|"Write-heavy (e.g. 20:80)"| WriteOpt["📬 Write-Optimized Pattern<br/>• Message queues (Kafka/RabbitMQ)<br/>• Append-only stores (NoSQL/Log-structured)<br/>• Async workers"]
    
    ReadOpt & WriteOpt --> Q3{"Are strict ACID transactions required?"}
    
    Q3 -->|"Yes (CP)"| SQL["🗄️ Relational DB<br/>(PostgreSQL, MySQL)<br/>• Strong consistency"]
    Q3 -->|"No (AP)"| NoSQL["📦 NoSQL DB<br/>(MongoDB, Cassandra)<br/>• Eventual consistency"]
    
    SQL & NoSQL --> Q4{"How complex is the business logic?"}
    
    Q4 -->|"Low/Moderate"| Mono["🏛️ Modular Monolith<br/>• Shared codebase<br/>• Simple deployment"]
    Q4 -->|"High Complexity & Team Scale"| Micro["⚙️ Microservices / Serverless<br/>• Isolated domains<br/>• Separate deployments"]
    
    style Start fill:#161b22,stroke:#58a6ff,color:#fff
    style Buy fill:#0d1117,stroke:#3fb950,color:#c9d1d9
    style ReadOpt fill:#0d1117,stroke:#d29922,color:#c9d1d9
    style WriteOpt fill:#0d1117,stroke:#d29922,color:#c9d1d9
    style SQL fill:#0d1117,stroke:#e94560,color:#c9d1d9
    style NoSQL fill:#0d1117,stroke:#e94560,color:#c9d1d9
```

---

### ⚖️ The 4 Fundamental Trade-offs

Every system design choice is a trade-off. There is no single "best" architecture, only the "right" trade-off for your specific requirements.

```mermaid
mindmap
  root(("⚖️ Core Trade-offs"))
    Consistency vs Availability
      Strong Consistency: ACID, slower writes
      Eventual Consistency: fast, high availability
    Latency vs Throughput
      Low Latency: optimize response speed
      High Throughput: optimize batch operations
    Read vs Write Optimization
      Read-Heavy: cache-aside, replicas, indexing
      Write-Heavy: append-only, queues, log-structured
    Build vs Buy
      Build: proprietary IP, full custom control
      Buy: managed cloud service, fast delivery
```

| Trade-off | Option A | Option B | When to Choose A | When to Choose B |
|-----------|----------|----------|------------------|------------------|
| **CAP Theorem** | Strong Consistency (CP) | High Availability (AP) | Financial ledgers, checkouts | Social feeds, catalog views |
| **Optimization** | Low Latency | High Throughput | Live chat apps, trading platforms | Analytical reports, logging systems |
| **Operations** | Read-Optimized | Write-Optimized | Storefronts, catalog searches | IoT telemetry, clickstream logs |
| **Implementation**| Build Custom System | Buy Managed Service | Core business IP, custom limits | Standard utilities, quick release |

---

### 🏛️ The 3 Golden Rules of System Architecture

#### 1. YAGNI (You Aren't Gonna Need It)
*   **Rule**: Never design for scale you don't have and won't have in the near future.
*   **Reasoning**: Over-engineering adds complexity, operational costs, and latency. Build a modular monolith first; split into microservices only when team scale or traffic limits demand it.

#### 2. Design for Failure (Fault Tolerance)
*   **Rule**: Assume everything that *can* fail, *will* fail.
*   **Reasoning**: Implement circuit breakers, retries with exponential backoff and jitter, fallbacks (graceful degradation), and dead-letter queues to isolate failures.

#### 3. Single Source of Truth (SSOT)
*   **Rule**: For every piece of business data, there must be exactly one primary system of record.
*   **Reasoning**: Avoid dual-writes. Use message-driven event streams (e.g., Change Data Capture) to synchronize replicas and caches from the single source of truth.

---

### 🛠️ Technical Debt Management Guide

As a senior/staff engineer, you must teach teams how to borrow tech debt responsibly:

```mermaid
graph TD
    Debt["Is this Technical Debt?"]
    Debt -->|"Deliberate & Prudent"| Good["✅ Strategic Debt<br/>Ship fast to validate market<br/>Refactor immediately after validation"]
    Debt -->|"Reckless & Inadvertent"| Bad["❌ Toxic Debt<br/>Messy code, no tests, poor patterns<br/>Slows down engineering indefinitely"]

    style Good fill:#0d1117,stroke:#3fb950,color:#fff
    style Bad fill:#0d1117,stroke:#f85149,color:#fff
```

*   **How to manage it**: Maintain a **Tech Debt Backlog**. Allocate 15-20% of every sprint/cycle to refactoring and paying down high-risk debt (e.g., code bottlenecks, security vulnerabilities, stale packages).

---

**← Previous:** [23. End-to-End Scenario](23-end-to-end-scenario.md) | **Next →** [25. Master Checklist](25-master-checklist.md)
