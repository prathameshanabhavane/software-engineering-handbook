# ✅ 15. Checklist — Things That Are Often Missed

> **This is the list you review before launching, during code reviews, and when designing new features. Print it, bookmark it, come back to it.**

---

## 🔄 The Review Flow

```mermaid
flowchart TD
    Start["🚀 Before Launch /<br/>Design Review"] --> Resilience
    Resilience --> DataSafety
    DataSafety --> UX
    UX --> Operations
    Operations --> Compliance
    Compliance --> CostReady["✅ Ready to ship!"]

    subgraph Resilience["🛡️ Resilience"]
        R1["Graceful degradation"]
        R2["Idempotency"]
        R3["Circuit breakers"]
        R4["Rollback strategy"]
        R5["Feature flags"]
    end

    subgraph DataSafety["💾 Data Safety"]
        D1["Backups tested"]
        D2["Data retention policy"]
        D3["Encryption at rest"]
        D4["GDPR / compliance"]
    end

    subgraph UX["👤 User Experience"]
        U1["Accessibility (a11y)"]
        U2["Error messages are helpful"]
        U3["Loading states"]
        U4["Mobile responsive"]
    end

    subgraph Operations["⚙️ Operations"]
        O1["Monitoring & alerting"]
        O2["Documentation"]
        O3["Load testing done"]
        O4["CI/CD pipeline"]
    end

    subgraph Compliance["📜 Compliance"]
        C1["Security audit"]
        C2["Dependency scan"]
        C3["Rate limiting"]
        C4["Audit logging"]
    end

    style CostReady fill:#0d1117,stroke:#3fb950,color:#c9d1d9
```

---

## 🛡️ Resilience Checklist

| Item | Question | Why It Matters |
|------|----------|---------------|
| **Graceful degradation** | If cache/3rd-party goes down, does the site crash? | Users should see stale data or a friendly fallback, not a 500 error |
| **Idempotency** | If user double-clicks "Pay", are they charged twice? | Network retries and double-clicks must not cause duplicate actions |
| **Circuit breakers** | If a dependency is failing, does it cascade to everything? | Stop calling a failing service; fail fast instead of waiting |
| **Rollback strategy** | Can you revert a bad deployment in under 5 minutes? | Deploy → bug found → need instant rollback capability |
| **Feature flags** | Can you turn off a new feature without redeploying? | Ship features gradually; disable instantly if something breaks |
| **Retry with backoff** | Do retries use exponential backoff + jitter? | Without backoff, retries create a thundering herd on the failing service |
| **Timeout configuration** | Are timeouts set for every external call? | Without timeouts, one slow service can hang your entire system |
| **Dead letter queue** | Where do failed async messages go? | Messages that can't be processed shouldn't be lost silently |

### Idempotency Pattern

```mermaid
sequenceDiagram
    actor User
    participant App
    participant DB

    User->>App: POST /orders (idempotencyKey: "abc-123")
    App->>DB: Check: does order with key "abc-123" exist?
    DB-->>App: No

    App->>DB: Create order (key: "abc-123")
    App-->>User: 201 Created {orderId: 1}

    Note over User: Network hiccup — client retries!

    User->>App: POST /orders (idempotencyKey: "abc-123")
    App->>DB: Check: does order with key "abc-123" exist?
    DB-->>App: Yes! Order #1 already exists

    App-->>User: 200 OK {orderId: 1}
    Note over App: Same response, no duplicate order! ✅
```

### Circuit Breaker Pattern

```mermaid
stateDiagram-v2
    [*] --> Closed: Normal operation
    Closed --> Open: Failures > threshold<br/>(e.g., 5 failures in 30s)
    Open --> HalfOpen: Wait timeout<br/>(e.g., 30 seconds)
    HalfOpen --> Closed: Test request succeeds
    HalfOpen --> Open: Test request fails

    note right of Closed: All requests pass through
    note right of Open: All requests fail immediately<br/>(don't even try the service)
    note right of HalfOpen: Allow ONE test request<br/>to check if service recovered
```

---

## 💾 Data Safety Checklist

| Item | Action |
|------|--------|
| ✅ Automated backups | Daily full + hourly incremental, stored in different region |
| ✅ Restore tested | Actually restore from backup at least quarterly |
| ✅ Point-in-time recovery | Enabled for primary database |
| ✅ Data retention policy | Define how long each data type is kept |
| ✅ Encryption at rest | All sensitive data encrypted in database |
| ✅ Data deletion capability | Can fully delete a user's data (GDPR) |
| ✅ No PII in logs | Sensitive data scrubbed from log output |

---

## 👤 User Experience Checklist

| Item | Action |
|------|--------|
| ✅ Loading states | Show skeletons/spinners, not blank screens |
| ✅ Error messages | "Something went wrong" → "Payment failed — check card details" |
| ✅ Offline handling | PWA or graceful "you're offline" message |
| ✅ Mobile responsive | Works on all screen sizes |
| ✅ Accessibility (a11y) | Semantic HTML, ARIA labels, keyboard navigation, contrast |
| ✅ Empty states | What does the page look like with zero data? |
| ✅ Pagination | Never show "loading all 10,000 items..." |

---

## ⚙️ Operations Checklist

| Item | Action |
|------|--------|
| ✅ Health check endpoint | `/health` and `/ready` endpoints |
| ✅ Metrics dashboard | Request rate, error rate, latency, resources |
| ✅ Alerting configured | Critical alerts page on-call, warnings to Slack |
| ✅ Structured logging | JSON logs with request IDs, shipped to central store |
| ✅ Load testing | Simulate expected peak traffic before launch |
| ✅ Documentation | README, architecture diagram, runbooks |
| ✅ CI/CD pipeline | Tests → Build → Stage → Production automated |
| ✅ Secrets management | No secrets in code, use env vars or vault |

---

## 💰 Cost Checklist

| Item | Action |
|------|--------|
| ✅ Budget alerts | Get notified before spending exceeds limit |
| ✅ Auto-scaling limits | Set max instances to prevent runaway costs |
| ✅ Log retention limits | Don't store debug logs for years |
| ✅ Resource right-sizing | Don't run a $500/month server for a $5/month workload |
| ✅ Unused resource cleanup | Delete orphaned databases, storage buckets, IPs |

---

## 🔗 Connected Topics

Every item in this checklist maps back to a specific chapter — review the relevant chapter for deep details on any item.

---

**← Previous:** [14. Request Walkthrough](14-request-walkthrough.md) | **Next →** [Part 2: 16. URL to Page Journey](../Part-2-Network-Hardware-Browser-Frameworks/16-url-to-page-journey.md)
