# 🚀 22. CI/CD Pipeline — From Laptop to Production

> **CI/CD is like a factory assembly line with quality inspectors at each station. Code enters raw, gets built, tested, inspected, and only perfect products reach the customer.**

---

## 🔄 The Complete CI/CD Flow

```mermaid
graph LR
    Dev["👨‍💻 Developer<br/>writes code"] --> Commit["📝 Git Commit<br/>+ Push"]
    Commit --> CI["🔄 CI Pipeline<br/>(Continuous Integration)"]

    subgraph CI_STEPS["CI — Build & Test"]
        Lint["🔍 Lint<br/>(code style)"]
        UnitTest["🧪 Unit Tests"]
        Build["🔨 Build<br/>(compile, bundle)"]
        IntTest["🧪 Integration Tests"]
        Security["🔒 Security Scan<br/>(npm audit, SAST)"]
        Coverage["📊 Coverage Report"]
    end

    CI --> Lint --> UnitTest --> Build --> IntTest --> Security --> Coverage

    Coverage --> Review["👀 Code Review<br/>(Pull Request)"]
    Review -->|"Approved ✅"| Merge["🔀 Merge to main"]

    Merge --> CD["🚀 CD Pipeline<br/>(Continuous Deployment)"]

    subgraph CD_STEPS["CD — Deploy"]
        Staging["🟡 Deploy to Staging<br/>(test environment)"]
        SmokeTest["🧪 Smoke Tests<br/>(critical paths work?)"]
        Prod["🟢 Deploy to Production<br/>(real users)"]
        Monitor["👁️ Monitor<br/>(errors? rollback?)"]
    end

    CD --> Staging --> SmokeTest --> Prod --> Monitor

    Monitor -->|"Issues found"| Rollback["⏪ Rollback"]

    style Dev fill:#161b22,stroke:#58a6ff,color:#fff
    style Rollback fill:#0d1117,stroke:#f85149,color:#c9d1d9
    style Prod fill:#0d1117,stroke:#3fb950,color:#c9d1d9
```

---

## 🌳 Git Branching Strategy

```mermaid
gitGraph
    commit id: "initial"
    branch develop
    commit id: "feat-base"

    branch feature/user-auth
    commit id: "add login"
    commit id: "add signup"
    checkout develop
    merge feature/user-auth id: "merge auth"

    branch feature/checkout
    commit id: "cart logic"
    commit id: "payment"
    checkout develop
    merge feature/checkout id: "merge checkout"

    checkout main
    merge develop id: "release v1.2"
    commit id: "tag v1.2" type: HIGHLIGHT

    branch hotfix/security
    commit id: "patch XSS"
    checkout main
    merge hotfix/security id: "hotfix merge"
```

---

## 🧪 Testing Pyramid in CI

```mermaid
graph TB
    subgraph PYRAMID["Testing in CI Pipeline"]
        E2E["🔺 E2E Tests<br/>5-10 tests<br/>~5 min<br/>Full browser, real API"]
        Integration["🔶 Integration Tests<br/>50-100 tests<br/>~2 min<br/>API + DB + services"]
        Unit["🟩 Unit Tests<br/>500+ tests<br/>~30 sec<br/>Individual functions"]
    end

    Unit -->|"Run first<br/>(fastest feedback)"| Integration
    Integration -->|"Run second"| E2E

    style E2E fill:#0d1117,stroke:#f85149,color:#c9d1d9
    style Integration fill:#0d1117,stroke:#d29922,color:#c9d1d9
    style Unit fill:#0d1117,stroke:#3fb950,color:#c9d1d9
```

---

## 🚀 Deployment Strategies

### Blue-Green Deployment

```mermaid
graph TB
    LB["⚖️ Load Balancer"]

    subgraph BLUE["🔵 Blue (current - v1.0)"]
        B1["🖥️ Server 1"]
        B2["🖥️ Server 2"]
    end

    subgraph GREEN["🟢 Green (new - v1.1)"]
        G1["🖥️ Server 3"]
        G2["🖥️ Server 4"]
    end

    LB -->|"100% traffic"| BLUE
    LB -.->|"0% traffic<br/>(deploy & test here)"| GREEN

    GREEN -->|"Tests pass? Switch!"| Switch["⚖️ Switch traffic<br/>Blue → Green"]

    style BLUE fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
    style GREEN fill:#0d1117,stroke:#3fb950,color:#c9d1d9
```

### Canary Deployment

```mermaid
graph TB
    LB["⚖️ Load Balancer"]

    subgraph STABLE["🔵 Stable (v1.0)"]
        S1["🖥️"] & S2["🖥️"] & S3["🖥️"] & S4["🖥️"]
    end

    subgraph CANARY["🟡 Canary (v1.1)"]
        C1["🖥️"]
    end

    LB -->|"95% traffic"| STABLE
    LB -->|"5% traffic"| CANARY

    CANARY -->|"Monitor errors.<br/>If OK → gradually increase"| STABLE

    style STABLE fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
    style CANARY fill:#0d1117,stroke:#d29922,color:#c9d1d9
```

### Deployment Strategy Comparison

| Strategy | Downtime | Risk | Rollback Speed | Cost |
|----------|----------|------|---------------|------|
| **Rolling** | None | Medium | Moderate | Low (reuse servers) |
| **Blue-Green** | None | Low | Instant (switch LB) | High (double servers) |
| **Canary** | None | Lowest | Fast | Medium (1 extra) |
| **Recreate** | Yes ❌ | High | Slow | Low |

---

## 🏭 GitHub Actions Example

```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20' }
      - run: npm ci                    # Install dependencies
      - run: npm run lint              # Code style check
      - run: npm run test:unit         # Unit tests
      - run: npm run test:integration  # Integration tests
      - run: npm audit --production    # Security scan

  deploy-staging:
    needs: test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploy to staging..."
      - run: echo "Run smoke tests..."

  deploy-production:
    needs: deploy-staging
    runs-on: ubuntu-latest
    environment: production  # Requires manual approval!
    steps:
      - run: echo "Deploy to production..."
```

---

## ⚠️ Edge Cases & Gotchas

1. **"It works on my machine"** — CI ensures code is built and tested in a clean, reproducible environment. Use Docker in CI to match production.

2. **Flaky tests** — Tests that sometimes pass and sometimes fail destroy CI trust. Fix or quarantine flaky tests immediately.

3. **Long CI pipelines** — If CI takes 30 minutes, developers won't wait and will merge without checking results. Optimize: parallelize tests, cache dependencies.

4. **Database migrations in deploy** — Run migrations BEFORE deploying new code. New code expects new schema; if migration fails, old code still works.

5. **Secrets in CI** — Use CI/CD platform's secret management, not hardcoded values. Rotate secrets and audit who has access.

---

## 🔗 Connected Topics

| Topic | Connection |
|-------|-----------|
| [Clean Code](../Part-1-Architecture-Scalability-Operations/11-clean-modular-code.md) | Tests run in CI enforce code quality |
| [Monitoring](../Part-1-Architecture-Scalability-Operations/13-monitoring-observability.md) | Post-deploy monitoring catches issues |
| [Security](../Part-1-Architecture-Scalability-Operations/09-security.md) | Security scans in CI pipeline |
| [Governance](../Part-1-Architecture-Scalability-Operations/10-governance.md) | Branch protection, required reviews |
| [Hardware](18-hardware-infrastructure.md) | Containers, Kubernetes for deployment |

---

**← Previous:** [21. Backend Frameworks](21-backend-frameworks.md) | **Next →** [23. End-to-End Scenario](23-end-to-end-scenario.md)
