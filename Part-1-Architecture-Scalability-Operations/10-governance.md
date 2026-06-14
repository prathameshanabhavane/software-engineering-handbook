# 🏛️ 10. Governance — Rules, Compliance & Accountability

> **Governance is like a country's constitution. Without it, a company is just a group of people doing whatever they think is best — fine with 3 people, chaos with 300.**

---

## 🔄 Governance Areas — The Five Pillars

```mermaid
graph TB
    Gov["🏛️ Governance"] --> Data["📊 Data Governance<br/>Who can access what data?<br/>How long is data kept?<br/>What sensitivity level?"]
    Gov --> Reg["📜 Regulatory Compliance<br/>GDPR, HIPAA, PCI-DSS, SOC2<br/>Legal requirements<br/>User rights"]
    Gov --> Access["🔐 Access Governance<br/>Who can deploy?<br/>Who can access prod DB?<br/>Approval workflows"]
    Gov --> Code["📝 Code Governance<br/>Code review required?<br/>Branch protection?<br/>Automated testing?"]
    Gov --> Version["🔄 Versioning<br/>API versioning<br/>Database migrations<br/>Change management"]

    style Gov fill:#161b22,stroke:#d29922,color:#fff
    style Data fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
    style Reg fill:#0d1117,stroke:#f85149,color:#c9d1d9
    style Access fill:#0d1117,stroke:#3fb950,color:#c9d1d9
    style Code fill:#0d1117,stroke:#bc8cff,color:#c9d1d9
    style Version fill:#0d1117,stroke:#d29922,color:#c9d1d9
```

---

## 📊 Data Governance — Classification & Access

```mermaid
graph TD
    subgraph CLASSIFICATION["Data Classification Levels"]
        Public["🟢 PUBLIC<br/>Marketing content, blog posts<br/>Anyone can access"]
        Internal["🟡 INTERNAL<br/>Employee directory, docs<br/>Employees only"]
        Confidential["🟠 CONFIDENTIAL<br/>Customer PII, emails<br/>Authorized teams only"]
        Restricted["🔴 RESTRICTED<br/>Passwords, payment cards,<br/>health records<br/>Strict access + encryption + audit"]
    end

    Public -->|"Access: Open"| AC1["No restrictions"]
    Internal -->|"Access: Auth required"| AC2["Login required"]
    Confidential -->|"Access: Role-based"| AC3["RBAC + encryption"]
    Restricted -->|"Access: Strict controls"| AC4["RBAC + encryption + audit log<br/>+ MFA + approval workflow"]

    style Public fill:#0d1117,stroke:#3fb950,color:#c9d1d9
    style Internal fill:#0d1117,stroke:#d29922,color:#c9d1d9
    style Confidential fill:#0d1117,stroke:#f85149,color:#c9d1d9
    style Restricted fill:#0d1117,stroke:#f85149,color:#c9d1d9
```

### Data Retention Policy

| Data Type | Retention | After Expiry | Why |
|-----------|----------|-------------|-----|
| Active user data | While active | Archive/delete | GDPR right to be forgotten |
| Transaction records | 7 years | Archive | Legal/tax requirements |
| Server logs | 90 days | Delete | Cost + security |
| Analytics | 2 years | Aggregate | Privacy + cost |
| Backups | 30 days | Delete | Storage cost |
| Audit logs | 3 years | Archive | Compliance |

---

## 📜 Regulatory Compliance

```mermaid
graph TB
    subgraph GDPR["🇪🇺 GDPR (EU Users)"]
        G1["Right to access data"]
        G2["Right to delete data"]
        G3["Right to data portability"]
        G4["Consent before data collection"]
        G5["Data breach notification (72 hrs)"]
        G6["Privacy by design"]
    end

    subgraph HIPAA["🏥 HIPAA (Health Data, US)"]
        H1["Encrypt health data at rest + transit"]
        H2["Strict access controls"]
        H3["Audit trails required"]
        H4["Business associate agreements"]
    end

    subgraph PCI["💳 PCI-DSS (Payment Cards)"]
        P1["Never store full card numbers"]
        P2["Encrypt cardholder data"]
        P3["Quarterly vulnerability scans"]
        P4["Restrict access to card data"]
    end

    subgraph SOC2["🏢 SOC 2 (B2B/Enterprise)"]
        S1["Security controls documented"]
        S2["Availability SLAs"]
        S3["Processing integrity"]
        S4["Privacy controls"]
    end

    style GDPR fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
    style HIPAA fill:#0d1117,stroke:#3fb950,color:#c9d1d9
    style PCI fill:#0d1117,stroke:#d29922,color:#c9d1d9
    style SOC2 fill:#0d1117,stroke:#bc8cff,color:#c9d1d9
```

### GDPR "Right to Be Forgotten" — Technical Implementation

```mermaid
flowchart TD
    Request["User requests: 'Delete my data'"] --> Verify["Verify identity"]
    Verify --> Find["Find ALL user data across:<br/>• Main database<br/>• Search indexes<br/>• Cache (Redis)<br/>• Message queues<br/>• Backups<br/>• Analytics<br/>• Third-party integrations<br/>• Logs"]
    Find --> Delete["Delete or anonymize<br/>in ALL systems"]
    Delete --> Confirm["Confirm deletion to user<br/>within 30 days"]
    Delete --> Log["Log deletion event<br/>(for audit trail — don't log the deleted data!)"]

    style Request fill:#161b22,stroke:#58a6ff,color:#fff
    style Find fill:#0d1117,stroke:#f85149,color:#c9d1d9
    style Delete fill:#0d1117,stroke:#d29922,color:#c9d1d9
    style Confirm fill:#0d1117,stroke:#3fb950,color:#c9d1d9
```

---

## 🔐 Access Governance — Who Can Do What

```mermaid
graph TD
    subgraph PRINCIPLE["Principle of Least Privilege"]
        P["Every person and system<br/>gets ONLY the access<br/>they absolutely need"]
    end

    subgraph EXAMPLES["Examples"]
        Dev["Developer<br/>✅ Read code<br/>✅ Push to branch<br/>❌ Deploy to prod<br/>❌ Access prod DB"]
        Senior["Senior Engineer<br/>✅ All dev permissions<br/>✅ Approve PRs<br/>✅ Deploy to staging<br/>❌ Access prod DB directly"]
        SRE["SRE / DevOps<br/>✅ Deploy to prod<br/>✅ Read prod logs<br/>✅ Emergency DB access<br/>❌ Modify business logic"]
        Admin["Admin<br/>✅ Everything<br/>⚠️ All actions audited"]
    end

    style PRINCIPLE fill:#0d1117,stroke:#d29922,color:#c9d1d9
    style Dev fill:#0d1117,stroke:#3fb950,color:#c9d1d9
    style Senior fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
    style SRE fill:#0d1117,stroke:#bc8cff,color:#c9d1d9
    style Admin fill:#0d1117,stroke:#f85149,color:#c9d1d9
```

---

## 📝 Code Governance

```mermaid
flowchart LR
    Code["Developer<br/>writes code"] --> Branch["Push to<br/>feature branch"]
    Branch --> PR["Create<br/>Pull Request"]
    PR --> Review{"Code Review<br/>(min 1 approver)"}
    Review -->|"Changes requested"| Code
    Review -->|"Approved ✅"| CI["CI Pipeline<br/>• Tests pass?<br/>• Lint pass?<br/>• Build pass?"]
    CI -->|"Fail ❌"| Code
    CI -->|"Pass ✅"| Merge["Merge to main"]
    Merge --> Deploy["Auto-deploy<br/>to staging"]

    style Code fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
    style PR fill:#0d1117,stroke:#d29922,color:#c9d1d9
    style Review fill:#0d1117,stroke:#3fb950,color:#c9d1d9
    style CI fill:#0d1117,stroke:#bc8cff,color:#c9d1d9
    style Merge fill:#0d1117,stroke:#3fb950,color:#c9d1d9
```

---

## 🔄 API Versioning

```mermaid
graph LR
    subgraph VERSIONS["API Versions"]
        V1["v1 (deprecated)<br/>/api/v1/users<br/>Old format"]
        V2["v2 (current)<br/>/api/v2/users<br/>New format + fields"]
        V3["v3 (beta)<br/>/api/v3/users<br/>Breaking changes"]
    end

    OldClient["📱 Old Mobile App"] -->|"Still works"| V1
    CurrentClient["💻 Current Web App"] --> V2
    BetaClient["🧪 Beta Testers"] --> V3

    V1 -->|"Sunset date:<br/>2025-06-01"| Deprecated["⚠️ Will be removed"]

    style V1 fill:#0d1117,stroke:#f85149,color:#c9d1d9
    style V2 fill:#0d1117,stroke:#3fb950,color:#c9d1d9
    style V3 fill:#0d1117,stroke:#d29922,color:#c9d1d9
```

---

## ⚠️ Edge Cases & Gotchas

1. **"Move fast and break things" vs governance** — Governance isn't about slowing down. It's about guardrails that let you move fast safely (like highway guardrails let cars go faster than country roads).

2. **Shadow IT** — Teams spinning up databases or services without going through proper channels. Creates security blind spots.

3. **Compliance != security** — You can be compliant with regulations and still be insecure. Compliance is the minimum bar, not the ceiling.

4. **Data residency** — Some regulations require data to stay in specific countries (EU data in EU servers). This affects cloud region selection.

5. **Third-party risk** — Your security is only as strong as your weakest vendor. If you share user data with a third-party service, their breach is your breach.

---

## 🔗 Connected Topics

| Topic | Connection |
|-------|-----------|
| [Security](09-security.md) | Governance defines the security policies |
| [Database](07-database-design.md) | Data classification, retention, and encryption |
| [CI/CD](../Part-2-Network-Hardware-Browser-Frameworks/22-cicd-pipeline.md) | Code governance enforced through CI/CD pipeline |
| [Clean Code](11-clean-modular-code.md) | Coding standards are part of code governance |
| [Monitoring](13-monitoring-observability.md) | Audit logging and compliance monitoring |

---

**← Previous:** [9. Security](09-security.md) | **Next →** [11. Clean & Modular Code](11-clean-modular-code.md)
