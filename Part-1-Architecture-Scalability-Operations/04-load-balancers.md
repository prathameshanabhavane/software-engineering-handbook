# ⚖️ 4. Load Balancers — The Traffic Director

> **A load balancer is like a restaurant host. Customers arrive at the door; the host looks at which tables are free and seats people accordingly. Without a host, everyone crowds around one table while others sit empty.**

---

## 🔄 How Load Balancing Works — The Complete Flow

```mermaid
sequenceDiagram
    actor User
    participant DNS
    participant LB as ⚖️ Load Balancer
    participant S1 as 🖥️ Server 1
    participant S2 as 🖥️ Server 2
    participant S3 as 🖥️ Server 3

    Note over LB: Continuously runs health checks
    LB->>S1: GET /health ✅
    LB->>S2: GET /health ✅
    LB->>S3: GET /health ❌ (down!)

    Note over LB: Server 3 removed from pool

    User->>DNS: Resolve api.example.com
    DNS-->>User: Returns LB IP address

    User->>LB: GET /api/products
    Note over LB: Algorithm picks server<br/>(Round Robin → Server 1)
    LB->>S1: Forward request
    S1-->>LB: Response (200 OK)
    LB-->>User: Response

    User->>LB: GET /api/orders
    Note over LB: Round Robin → Server 2<br/>(skips unhealthy Server 3)
    LB->>S2: Forward request
    S2-->>LB: Response (200 OK)
    LB-->>User: Response

    Note over S3: Server 3 recovers
    LB->>S3: GET /health ✅
    Note over LB: Server 3 added back to pool
```

---

## 🎯 Load Balancing Algorithms

### 1. Round Robin — Take Turns

```mermaid
graph LR
    subgraph FLOW["Round Robin: Requests distributed in order"]
        R1["Request 1"] --> S1["🖥️ Server 1"]
        R2["Request 2"] --> S2["🖥️ Server 2"]
        R3["Request 3"] --> S3["🖥️ Server 3"]
        R4["Request 4"] --> S1
        R5["Request 5"] --> S2
        R6["Request 6"] --> S3
    end

    style FLOW fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
```

| Best When | Not Ideal When |
|-----------|---------------|
| Servers are identical | Servers have different capacities |
| Requests are similar in cost | Some requests are much heavier than others |
| Simple setup needed | Need to account for current load |

### 2. Weighted Round Robin — Proportional Distribution

```mermaid
graph LR
    subgraph FLOW["Weighted: Powerful servers get more traffic"]
        R1["Request 1"] --> S1["🖥️ Server 1<br/>Weight: 5<br/>(powerful)"]
        R2["Request 2"] --> S1
        R3["Request 3"] --> S1
        R4["Request 4"] --> S1
        R5["Request 5"] --> S1
        R6["Request 6"] --> S2["🖥️ Server 2<br/>Weight: 3<br/>(medium)"]
        R7["Request 7"] --> S2
        R8["Request 8"] --> S2
        R9["Request 9"] --> S3["🖥️ Server 3<br/>Weight: 2<br/>(small)"]
        R10["Request 10"] --> S3
    end

    style FLOW fill:#0d1117,stroke:#3fb950,color:#c9d1d9
```

### 3. Least Connections — Send to the Least Busy

```mermaid
graph TB
    LB["⚖️ Load Balancer<br/>Next request arrives..."]

    LB -->|"❌ Skip"| S1["🖥️ Server 1<br/>Active: 15 connections"]
    LB -->|"❌ Skip"| S2["🖥️ Server 2<br/>Active: 12 connections"]
    LB -->|"✅ PICK THIS"| S3["🖥️ Server 3<br/>Active: 3 connections"]

    style LB fill:#161b22,stroke:#d29922,color:#fff
    style S1 fill:#0d1117,stroke:#f85149,color:#c9d1d9
    style S2 fill:#0d1117,stroke:#d29922,color:#c9d1d9
    style S3 fill:#0d1117,stroke:#3fb950,color:#c9d1d9
```

| Best When | Not Ideal When |
|-----------|---------------|
| Request processing times vary a lot | All requests are quick and similar |
| Long-lived connections (WebSocket) | Simple stateless APIs |
| Mixed workloads (heavy + light) | When simplicity matters most |

### 4. IP Hash / Sticky Sessions

```mermaid
graph TB
    LB["⚖️ Load Balancer<br/>hash(client IP) % servers"]

    U1["👤 User A<br/>IP: 1.2.3.4"] -->|"hash = always Server 1"| S1["🖥️ Server 1"]
    U2["👤 User B<br/>IP: 5.6.7.8"] -->|"hash = always Server 2"| S2["🖥️ Server 2"]
    U3["👤 User C<br/>IP: 9.10.11.12"] -->|"hash = always Server 3"| S3["🖥️ Server 3"]

    Note1["⚠️ Better to make app stateless<br/>and use Round Robin or Least Connections"]

    style LB fill:#161b22,stroke:#d29922,color:#fff
    style Note1 fill:#0d1117,stroke:#f85149,color:#c9d1d9
```

### Algorithm Comparison at a Glance

| Algorithm | Complexity | Best For | Weakness |
|-----------|-----------|----------|----------|
| Round Robin | Simple | Equal servers, similar requests | Ignores server load |
| Weighted Round Robin | Simple | Mixed server capacities | Static weights need tuning |
| Least Connections | Medium | Variable request costs | Slightly more overhead |
| IP Hash | Simple | Sticky sessions needed | Uneven if IP distribution is skewed |
| Least Response Time | Complex | Latency-sensitive apps | Needs continuous monitoring |
| Random | Simplest | Statistically large pool | Can be uneven with few servers |

---

## 🏗️ Layer 4 vs Layer 7 Load Balancing

```mermaid
graph TB
    subgraph L4["🔵 Layer 4 (Transport Layer)"]
        L4_What["Routes based on:<br/>• IP address<br/>• Port number<br/>• TCP/UDP protocol"]
        L4_How["Doesn't inspect content<br/>Very fast, low overhead"]
        L4_Ex["Example:<br/>All traffic on :443 → server pool"]
    end

    subgraph L7["🟣 Layer 7 (Application Layer)"]
        L7_What["Routes based on:<br/>• URL path<br/>• HTTP headers<br/>• Cookies<br/>• Request body"]
        L7_How["Inspects content<br/>Smarter but slightly slower"]
        L7_Ex["Example:<br/>/api/* → API servers<br/>/images/* → Image servers<br/>/admin/* → Admin servers"]
    end

    style L4 fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
    style L7 fill:#0d1117,stroke:#bc8cff,color:#c9d1d9
```

### Layer 7 Content-Based Routing Example

```mermaid
graph TB
    Client["👤 Client Request"] --> L7LB["⚖️ Layer 7 Load Balancer"]

    L7LB -->|"/api/search/*"| SearchPool["🔍 Search Servers<br/>(optimized for Elasticsearch)"]
    L7LB -->|"/api/images/*"| ImagePool["🖼️ Image Servers<br/>(GPU-enabled, high memory)"]
    L7LB -->|"/api/payments/*"| PaymentPool["💳 Payment Servers<br/>(PCI-compliant, extra security)"]
    L7LB -->|"/api/*"| GeneralPool["⚙️ General API Servers"]
    L7LB -->|"/static/*"| CDN["🌍 CDN / Static Servers"]

    style Client fill:#161b22,stroke:#58a6ff,color:#fff
    style L7LB fill:#161b22,stroke:#d29922,color:#fff
    style SearchPool fill:#0d1117,stroke:#3fb950,color:#c9d1d9
    style ImagePool fill:#0d1117,stroke:#bc8cff,color:#c9d1d9
    style PaymentPool fill:#0d1117,stroke:#f85149,color:#c9d1d9
    style GeneralPool fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
    style CDN fill:#0d1117,stroke:#d29922,color:#c9d1d9
```

---

## ❤️ Health Checks — Detecting Unhealthy Servers

```mermaid
sequenceDiagram
    participant LB as ⚖️ Load Balancer
    participant S1 as 🖥️ Server 1
    participant S2 as 🖥️ Server 2

    loop Every 10 seconds
        LB->>S1: GET /health
        S1-->>LB: 200 OK {"status": "healthy", "db": "ok", "cache": "ok"}

        LB->>S2: GET /health
        S2-->>LB: ❌ Connection timeout
    end

    Note over LB: After 3 consecutive failures...<br/>Remove Server 2 from pool

    LB->>LB: Route all traffic to Server 1 only

    Note over S2: Server 2 recovers...

    loop Health check resumes
        LB->>S2: GET /health
        S2-->>LB: 200 OK {"status": "healthy"}
    end

    Note over LB: After 2 consecutive successes...<br/>Add Server 2 back to pool
```

### Health Check Endpoint Example (Node.js)

```javascript
// GET /health
app.get('/health', async (req, res) => {
  try {
    // Check database connection
    await db.query('SELECT 1');

    // Check Redis connection
    await redis.ping();

    // Check disk space
    const diskFree = await checkDiskSpace();

    res.status(200).json({
      status: 'healthy',
      uptime: process.uptime(),
      database: 'connected',
      cache: 'connected',
      diskFree: diskFree,
      timestamp: new Date().toISOString()
    });
  } catch (error) {
    res.status(503).json({
      status: 'unhealthy',
      error: error.message
    });
  }
});
```

---

## 🔄 Load Balancer High Availability

The load balancer itself shouldn't be a single point of failure:

```mermaid
graph TB
    DNS["DNS: api.example.com"] -->|"Active IP"| LB1["⚖️ Primary LB<br/>Active ✅"]
    DNS -.->|"Standby IP"| LB2["⚖️ Secondary LB<br/>Standby 💤"]

    LB1 <-->|"Heartbeat<br/>(every 1s)"| LB2

    LB1 --> S1["🖥️ Server 1"] & S2["🖥️ Server 2"] & S3["🖥️ Server 3"]
    LB2 -.-> S1 & S2 & S3

    Note1["If Primary LB fails:<br/>Secondary takes over<br/>via Virtual IP (VRRP)"]

    style LB1 fill:#161b22,stroke:#3fb950,color:#fff
    style LB2 fill:#161b22,stroke:#d29922,color:#fff
    style DNS fill:#161b22,stroke:#58a6ff,color:#fff
```

---

## 🏪 Real-World Example — E-Commerce Flash Sale

```mermaid
graph TB
    subgraph BEFORE["Before Sale (2 servers)"]
        B_LB["⚖️ LB"] --> B_S1["🖥️"] & B_S2["🖥️"]
        B_Traffic["Traffic: 100 req/sec"]
    end

    subgraph DURING["During Sale (8 servers)"]
        D_LB["⚖️ LB"] --> D_S1["🖥️"] & D_S2["🖥️"] & D_S3["🖥️ NEW"] & D_S4["🖥️ NEW"] & D_S5["🖥️ NEW"] & D_S6["🖥️ NEW"] & D_S7["🖥️ NEW"] & D_S8["🖥️ NEW"]
        D_Traffic["Traffic: 5,000 req/sec"]
    end

    subgraph AFTER["After Sale (2 servers)"]
        A_LB["⚖️ LB"] --> A_S1["🖥️"] & A_S2["🖥️"]
        A_Traffic["Traffic: 150 req/sec"]
    end

    BEFORE -->|"Auto-scale triggered"| DURING
    DURING -->|"Cool-down period"| AFTER

    style BEFORE fill:#0d1117,stroke:#3fb950,color:#c9d1d9
    style DURING fill:#0d1117,stroke:#f85149,color:#c9d1d9
    style AFTER fill:#0d1117,stroke:#3fb950,color:#c9d1d9
```

---

## ⚠️ Edge Cases & Gotchas

1. **The load balancer IS the single point of failure** — unless you have LB redundancy (active-passive or active-active pair). Cloud providers (AWS ALB, GCP LB) handle this for you.

2. **Health checks can lie** — A server can respond to `/health` while its actual endpoints are broken. Make health checks test real dependencies (DB, cache, disk).

3. **Connection draining** — When removing a server, don't kill active connections immediately. Let existing requests finish ("drain") before removing the server from the pool.

4. **WebSocket and long connections** — Round Robin doesn't work well for WebSocket connections since they're long-lived. Use Least Connections or a sticky approach.

5. **SSL/TLS termination** — The LB can handle HTTPS decryption ("TLS termination"), so app servers only deal with HTTP internally. This simplifies cert management and offloads crypto work.

---

## 🔗 Connected Topics

| Topic | Connection |
|-------|-----------|
| [Scalability](03-scalability.md) | Load balancers enable horizontal scaling |
| [Caching](05-caching.md) | LB can route to cache layer before app servers |
| [Latency](08-latency.md) | LB adds a small hop but prevents overloaded servers (much worse latency) |
| [Security](09-security.md) | LB can terminate TLS, act as first line of defense |
| [Monitoring](13-monitoring-observability.md) | LB provides metrics: request rate, error rate, latency |

---

**← Previous:** [3. Scalability](03-scalability.md) | **Next →** [5. Caching](05-caching.md)
