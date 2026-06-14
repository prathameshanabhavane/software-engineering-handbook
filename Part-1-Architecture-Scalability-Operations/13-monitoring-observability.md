# 👁️ 13. Monitoring, Logging & Observability

> **Observability is your car's dashboard. The speedometer and fuel gauge (metrics) tell you the car's current state. The trip log (logging) records what happened. A mechanic's diagnostic tool (tracing) shows exactly which component is causing the problem.**

---

## 🏗️ The Three Pillars of Observability

```mermaid
graph TB
    Observability["👁️ Observability"]

    Observability --> Metrics["📊 METRICS<br/>Numbers over time<br/>'What is happening now?'<br/>• Request rate: 500/sec<br/>• Error rate: 0.5%<br/>• P95 latency: 200ms<br/>• CPU: 65%"]

    Observability --> Logs["📝 LOGS<br/>Records of events<br/>'What happened?'<br/>• Error details<br/>• Request payloads<br/>• Business events<br/>• Audit trail"]

    Observability --> Traces["🔗 TRACES<br/>Request path across services<br/>'Where is it slow?'<br/>• Service A → B → C<br/>• Time at each hop<br/>• Which service failed"]

    Metrics -->|"Dashboard"| Grafana["Grafana / Datadog"]
    Logs -->|"Search"| ELK["ELK Stack / Loki"]
    Traces -->|"Visualize"| Jaeger["Jaeger / Zipkin"]

    style Observability fill:#161b22,stroke:#d29922,color:#fff
    style Metrics fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
    style Logs fill:#0d1117,stroke:#3fb950,color:#c9d1d9
    style Traces fill:#0d1117,stroke:#bc8cff,color:#c9d1d9
```

---

## 📊 Metrics — The Dashboard

### Key Metrics to Track (The RED & USE Methods)

```mermaid
graph LR
    subgraph RED["RED Method (Services)"]
        R["Rate<br/>Requests per second"]
        E["Errors<br/>Failed requests per second"]
        D["Duration<br/>Response time (p50, p95, p99)"]
    end

    subgraph USE["USE Method (Infrastructure)"]
        U["Utilization<br/>% of resource used<br/>(CPU, memory, disk)"]
        S["Saturation<br/>Queue depth, waiting tasks"]
        Er["Errors<br/>Hardware/system errors"]
    end

    style RED fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
    style USE fill:#0d1117,stroke:#3fb950,color:#c9d1d9
```

### Essential Dashboard Panels

| Panel | Metric | Alarm When |
|-------|--------|-----------|
| Request Rate | Requests/sec | Sudden drop (outage?) or spike (attack?) |
| Error Rate | 5xx errors/total | > 1% |
| Latency P95 | 95th percentile response time | > 500ms |
| CPU Usage | % across servers | > 80% sustained |
| Memory Usage | % across servers | > 85% |
| Cache Hit Rate | Hits / total lookups | < 80% |
| DB Connection Pool | Active / max connections | > 80% pool used |
| Queue Depth | Messages waiting | Growing continuously |
| Disk Usage | % full | > 85% |

---

## 📝 Logging — What Happened

### Structured vs Unstructured Logs

```mermaid
graph LR
    subgraph BAD["❌ Unstructured Log"]
        B1["Error: something went wrong with order 123 for user 456 at 2024-01-15"]
    end

    subgraph GOOD["✅ Structured Log (JSON)"]
        G1["{<br/>  level: 'error',<br/>  message: 'Order processing failed',<br/>  orderId: 123,<br/>  userId: 456,<br/>  error: 'Payment declined',<br/>  timestamp: '2024-01-15T10:30:00Z',<br/>  traceId: 'abc-123-def'<br/>}"]
    end

    style BAD fill:#0d1117,stroke:#f85149,color:#c9d1d9
    style GOOD fill:#0d1117,stroke:#3fb950,color:#c9d1d9
```

### Log Levels

```mermaid
graph TB
    subgraph LEVELS["Log Levels (Severity)"]
        DEBUG["🔍 DEBUG<br/>Detailed dev info<br/>Variable values, flow details<br/>⚠️ NEVER in production"]
        INFO["ℹ️ INFO<br/>Normal operations<br/>Server started, request processed<br/>User logged in"]
        WARN["⚠️ WARN<br/>Something unexpected<br/>High latency, retry attempt<br/>Deprecated API usage"]
        ERROR["❌ ERROR<br/>Something failed<br/>Exception caught, request failed<br/>Needs investigation"]
        FATAL["💀 FATAL<br/>System is crashing<br/>Out of memory, DB unreachable<br/>Immediate action needed"]
    end

    DEBUG --> INFO --> WARN --> ERROR --> FATAL

    style DEBUG fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
    style INFO fill:#0d1117,stroke:#3fb950,color:#c9d1d9
    style WARN fill:#0d1117,stroke:#d29922,color:#c9d1d9
    style ERROR fill:#0d1117,stroke:#f85149,color:#c9d1d9
    style FATAL fill:#0d1117,stroke:#f85149,color:#c9d1d9
```

### Logging Code Example

```javascript
const logger = require('pino')(); // Structured JSON logger

// ✅ Good logging practice
app.get('/api/orders/:id', async (req, res) => {
  const { id } = req.params;
  const requestId = req.headers['x-request-id'];

  logger.info({ requestId, orderId: id, action: 'getOrder' }, 'Fetching order');

  try {
    const order = await orderService.getOrder(id);
    logger.info({ requestId, orderId: id, duration: Date.now() - start }, 'Order fetched');
    res.json(order);
  } catch (error) {
    logger.error({ requestId, orderId: id, error: error.message, stack: error.stack },
                  'Failed to fetch order');
    res.status(500).json({ error: 'Internal error' });
  }
});
```

---

## 🔗 Distributed Tracing — Follow the Request

```mermaid
sequenceDiagram
    participant Client
    participant Gateway as API Gateway
    participant OrderSvc as Order Service
    participant PaySvc as Payment Service
    participant NotifySvc as Notification Service

    Note over Client,NotifySvc: Trace ID: abc-123 (follows the entire request)

    Client->>Gateway: POST /orders (traceId: abc-123)
    Gateway->>OrderSvc: Create order (traceId: abc-123, spanId: span-1)
    Note over OrderSvc: ⏱️ 50ms

    OrderSvc->>PaySvc: Charge payment (traceId: abc-123, spanId: span-2)
    Note over PaySvc: ⏱️ 300ms ← Bottleneck found!

    PaySvc-->>OrderSvc: Payment OK
    OrderSvc->>NotifySvc: Send confirmation (traceId: abc-123, spanId: span-3)
    Note over NotifySvc: ⏱️ 20ms

    NotifySvc-->>OrderSvc: Notified
    OrderSvc-->>Gateway: Order created
    Gateway-->>Client: 201 Created

    Note over Client,NotifySvc: Total: 420ms<br/>Trace shows PaySvc is 71% of total time
```

### What a Trace Reveals

```mermaid
gantt
    title Trace: POST /api/orders (Total: 420ms)
    dateFormat X
    axisFormat %L ms

    section API Gateway
    Route + Auth          :gateway, 0, 50

    section Order Service
    Validate order        :validate, 50, 80
    Save to DB           :save, 80, 120

    section Payment Service
    Connect to Stripe    :connect, 120, 200
    Process payment      :process, 200, 370
    Save receipt         :receipt, 370, 400

    section Notification
    Send email           :email, 400, 420
```

---

## 🚨 Alerting — Know Before Users Do

```mermaid
flowchart TD
    Metric["📊 Metric Collected"] --> Check{"Threshold<br/>breached?"}

    Check -->|"No"| OK["✅ All good"]
    Check -->|"Yes"| Severity{"Severity?"}

    Severity -->|"Warning<br/>(latency > 300ms)"| Slack["💬 Slack notification"]
    Severity -->|"Critical<br/>(error rate > 5%)"| PagerDuty["📟 PagerDuty<br/>(wake someone up)"]
    Severity -->|"Informational<br/>(disk > 70%)"| Dashboard["📊 Dashboard flag"]

    style Metric fill:#161b22,stroke:#58a6ff,color:#fff
    style Slack fill:#0d1117,stroke:#d29922,color:#c9d1d9
    style PagerDuty fill:#0d1117,stroke:#f85149,color:#c9d1d9
    style Dashboard fill:#0d1117,stroke:#3fb950,color:#c9d1d9
```

### Alert Anti-Patterns

| Anti-Pattern | Problem | Fix |
|-------------|---------|-----|
| **Alert fatigue** | Too many alerts → team ignores all | Only alert on actionable items |
| **Missing context** | "Error rate high" but no details | Include links to dashboards, logs |
| **No runbook** | Alert fires, nobody knows what to do | Attach runbook to every alert |
| **Alerting on averages** | Average hides outliers | Alert on P95/P99 percentiles |

---

## ❤️ Health Checks

```javascript
// /health — simple liveness check
app.get('/health', (req, res) => {
  res.status(200).json({ status: 'ok' });
});

// /ready — readiness check (are dependencies available?)
app.get('/ready', async (req, res) => {
  try {
    await db.query('SELECT 1');
    await redis.ping();
    res.status(200).json({
      status: 'ready',
      database: 'connected',
      cache: 'connected',
    });
  } catch (err) {
    res.status(503).json({
      status: 'not ready',
      error: err.message,
    });
  }
});
```

---

## ⚠️ Edge Cases & Gotchas

1. **Logging sensitive data** — Never log passwords, tokens, credit card numbers, or PII. Scrub sensitive fields before logging.

2. **Log volume explosion** — Debug-level logging in production can generate terabytes. Use INFO in production, DEBUG only for local development.

3. **Cardinality explosion in metrics** — Creating a metric label for every user ID creates millions of time series. Use bounded labels (status codes, endpoints, regions).

4. **Monitoring the monitoring** — If your monitoring system goes down, who monitors it? Have a simple external uptime check (e.g., Pingdom) as a last resort.

5. **Not correlating across pillars** — The real power is connecting metrics → logs → traces. When an alert fires (metric), find the relevant logs, then trace the failing request across services.

---

## 🔗 Connected Topics

| Topic | Connection |
|-------|-----------|
| [Latency](08-latency.md) | Track p50/p95/p99 latency as metrics |
| [Scalability](03-scalability.md) | Auto-scaling triggers based on metrics |
| [Load Balancers](04-load-balancers.md) | LB health checks + LB metrics |
| [Security](09-security.md) | Audit logs, intrusion detection |
| [Performance](12-performance-optimization.md) | Profiling data feeds into optimization |

---

**← Previous:** [12. Performance Optimization](12-performance-optimization.md) | **Next →** [14. Request Walkthrough](14-request-walkthrough.md)
