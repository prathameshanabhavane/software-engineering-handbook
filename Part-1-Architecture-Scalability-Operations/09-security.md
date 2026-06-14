# 🔒 9. Security & Data Safety — Defense in Depth

> **Security is like a castle, not a single locked door. There's a moat (firewall), walls (authentication), guards checking IDs at each gate (authorization), sealed vaults for treasure (encryption), and a logbook of who entered which room and when (audit logs).**

---

## 🏰 The Castle Model — Layered Security

```mermaid
graph TB
    Internet["🌐 Internet<br/>(Wild West)"]

    Internet -->|"Layer 1"| Firewall["🔥 Firewall / WAF<br/>Block malicious traffic<br/>IP blacklists, DDoS protection"]

    Firewall -->|"Layer 2"| RateLimit["🚦 Rate Limiter<br/>Max 100 requests/min per IP<br/>Prevents brute force & abuse"]

    RateLimit -->|"Layer 3"| TLS["🔐 TLS/HTTPS<br/>Encrypt all data in transit<br/>No eavesdropping"]

    TLS -->|"Layer 4"| Auth["🔑 Authentication<br/>WHO are you?<br/>(Login, JWT, OAuth)"]

    Auth -->|"Layer 5"| Authz["🛡️ Authorization<br/>WHAT can you do?<br/>(RBAC, permissions)"]

    Authz -->|"Layer 6"| Validation["✅ Input Validation<br/>Is the data safe?<br/>(SQL injection, XSS prevention)"]

    Validation -->|"Layer 7"| Encryption["🔒 Encryption at Rest<br/>Data encrypted in database<br/>Even if stolen, it's useless"]

    Encryption -->|"Layer 8"| Audit["📋 Audit Logging<br/>WHO did WHAT and WHEN<br/>Immutable trail"]

    style Internet fill:#161b22,stroke:#f85149,color:#fff
    style Firewall fill:#0d1117,stroke:#f85149,color:#c9d1d9
    style RateLimit fill:#0d1117,stroke:#d29922,color:#c9d1d9
    style TLS fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
    style Auth fill:#0d1117,stroke:#3fb950,color:#c9d1d9
    style Authz fill:#0d1117,stroke:#bc8cff,color:#c9d1d9
    style Validation fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
    style Encryption fill:#0d1117,stroke:#3fb950,color:#c9d1d9
    style Audit fill:#0d1117,stroke:#d29922,color:#c9d1d9
```

---

---

## 🔥 Layer 1: Firewall & DDoS Mitigation — Defense at the Edge

> **Defense starts at the boundary. DDoS attacks attempt to overwhelm you; a firewall blocks bad actors, and DDoS scrubbers filter malicious traffic before it reaches your compute servers.**

### What it is
*   **WAF (Web Application Firewall)**: Filters, monitors, and blocks HTTP/HTTPS traffic to and from a web application (e.g., blocking SQLi, XSS, bots).
*   **DDoS (Distributed Denial of Service) Mitigation**: The process of successfully protecting a targeted server or network from a distributed denial-of-service attack.

### How it works
1.  **Anycast DNS & CDN**: Incoming traffic is routed to the closest edge server globally. This disperses the volume of a DDoS attack across hundreds of locations.
2.  **Traffic Scrubbing**: Special traffic scrubbing centers analyze traffic signatures. Malicious requests (e.g., SYN floods, NTP amplification, bot requests) are dropped, and clean traffic is passed through.
3.  **WAF Rules**: WAF inspects HTTP headers, payloads, query parameters, and patterns (like OWASP Top 10) to block attacks.

### When to use
*   **Always** place a DDoS shielding layer (like Cloudflare, AWS Shield, or Akamai) in front of public-facing endpoints.

---

## 🚦 Layer 2: Rate Limiting — Traffic Flow Control

> **A rate limiter restricts the number of requests a user or IP can make in a given timeframe to prevent abuse, brute-force logins, and server exhaustion.**

```mermaid
sequenceDiagram
    actor Attacker
    participant RateLimiter as 🚦 Rate Limiter
    participant API as ⚙️ API

    loop Normal usage (under limit)
        Attacker->>RateLimiter: Request 1-100 (in 1 minute)
        RateLimiter->>API: Forward ✅
        API-->>Attacker: 200 OK
    end

    Attacker->>RateLimiter: Request 101 (over limit!)
    RateLimiter-->>Attacker: 429 Too Many Requests ❌<br/>Retry-After: 60

    Note over RateLimiter: Different limits for different endpoints:
    Note over RateLimiter: Login: 5/min (prevent brute force)
    Note over RateLimiter: API: 100/min (prevent abuse)
    Note over RateLimiter: Search: 30/min (prevent scraping)
```

---

## 🔐 Layer 3: TLS/HTTPS — Encryption in Transit

> **HTTPS ensures that all communication between the client's browser and the server is encrypted, protecting data from eavesdropping and tampering.**

### How it works
*   During the **TLS handshake**, the server presents a certificate to prove its identity, and a secure session key is negotiated using asymmetric encryption. Subsequent traffic is encrypted using faster symmetric encryption.
*   **HSTS (HTTP Strict Transport Security)**: A response header that instructs browsers to *only* access the site via HTTPS, transforming any `http://` links to `https://` client-side.

---

## 🔑 Layer 4: Authentication — WHO Are You?

### Authentication Flow (JWT + OAuth)

```mermaid
sequenceDiagram
    actor User
    participant Client as 🌐 Client App
    participant Auth as 🔑 Auth Server
    participant API as ⚙️ API Server
    participant DB as 🗄️ Database

    User->>Client: Enter email + password
    Client->>Auth: POST /auth/login {email, password}
    Auth->>DB: Find user by email
    DB-->>Auth: User record (with hashed password)
    Auth->>Auth: bcrypt.compare(password, hash)

    alt Password matches ✅
        Auth->>Auth: Generate JWT token<br/>payload: {userId, role, exp}
        Auth-->>Client: 200 OK {accessToken, refreshToken}

        Note over Client: Store tokens securely

        Client->>API: GET /api/orders<br/>Authorization: Bearer <JWT>
        API->>API: Verify JWT signature<br/>Check expiration
        API->>DB: Fetch user's orders
        DB-->>API: Order data
        API-->>Client: 200 OK [orders]

    else Password wrong ❌
        Auth-->>Client: 401 Unauthorized
        Note over Auth: Log failed attempt<br/>After 5 failures → lock account
    end
```

### Password Storage — NEVER Plain Text

```mermaid
graph LR
    subgraph BAD["❌ WRONG — Plain Text"]
        PW1["Database stores:<br/>password = 'MyP@ssw0rd'<br/>If DB is hacked,<br/>attacker sees password"]
    end

    subgraph GOOD["✅ CORRECT — Hashed + Salted"]
        PW2["Database stores:<br/>hash = '$2b$10$K7L...'<br/>Even if DB is hacked,<br/>password can't be reversed"]
    end

    Input["User enters: 'MyP@ssw0rd'"] --> Hash["bcrypt.hash(password, 10)"]
    Hash --> Stored["$2b$10$K7L1OJ..."]

    style BAD fill:#0d1117,stroke:#f85149,color:#c9d1d9
    style GOOD fill:#0d1117,stroke:#3fb950,color:#c9d1d9
```

### JWT Token Structure

```mermaid
graph LR
    subgraph JWT["JWT Token"]
        Header["Header<br/>{alg: HS256, typ: JWT}"]
        Payload["Payload<br/>{userId: 123, role: admin,<br/>exp: 1700000000}"]
        Signature["Signature<br/>HMAC-SHA256(<br/>header + payload,<br/>SECRET_KEY)"]
    end

    Header --> Encode1["base64"]
    Payload --> Encode2["base64"]
    Signature --> Encode3["base64"]

    Encode1 & Encode2 & Encode3 --> Token["eyJhbG....<br/>.eyJ1c2...<br/>.SflKxw..."]

    style JWT fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
```

---

## 🛡️ Layer 5: Authorization — WHAT Can You Do?

### RBAC (Role-Based Access Control)

```mermaid
graph TB
    subgraph ROLES["Roles"]
        Admin["👑 Admin"]
        Editor["✏️ Editor"]
        User["👤 User"]
        Guest["👻 Guest"]
    end

    subgraph PERMISSIONS["Permissions"]
        Create["Create"]
        Read["Read"]
        Update["Update"]
        Delete["Delete"]
        ManageUsers["Manage Users"]
    end

    Admin -->|"✅"| Create & Read & Update & Delete & ManageUsers
    Editor -->|"✅"| Create & Read & Update
    User -->|"✅"| Read
    User -->|"✅ own only"| Update
    Guest -->|"✅"| Read

    style Admin fill:#0d1117,stroke:#f85149,color:#c9d1d9
    style Editor fill:#0d1117,stroke:#d29922,color:#c9d1d9
    style User fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
    style Guest fill:#0d1117,stroke:#3fb950,color:#c9d1d9
```

### Authorization Middleware Example

```javascript
// Middleware: Check if user has required role
function authorize(...allowedRoles) {
  return (req, res, next) => {
    const userRole = req.user.role; // from JWT

    if (!allowedRoles.includes(userRole)) {
      return res.status(403).json({ error: 'Forbidden: insufficient permissions' });
    }
    next();
  };
}

// Usage
app.delete('/api/users/:id', authorize('admin'), deleteUser);
app.put('/api/posts/:id', authorize('admin', 'editor'), updatePost);
app.get('/api/posts', authorize('admin', 'editor', 'user', 'guest'), getPosts);
```

---

## 🛡️ Layer 6: Input Validation & Sanitization — Payload Filtering

> **Input Validation verifies that the data fits expected formats, types, and lengths. Input Sanitization cleanses the data by stripping out executable scripts or database commands before they are executed or stored.**

### Common Attacks & Defenses

```mermaid
graph TD
    subgraph ATTACKS["⚔️ Common Attacks"]
        SQLi["SQL Injection<br/>Input: ' OR 1=1 --<br/>Drops tables, leaks data"]
        XSS["Cross-Site Scripting (XSS)<br/>Input: &lt;script&gt;steal(cookies)&lt;/script&gt;<br/>Runs malicious JS in browser"]
        CSRF["Cross-Site Request Forgery<br/>Tricks user into making<br/>unintended requests"]
    end

    subgraph DEFENSES["🛡️ Defenses"]
        Param["Parameterized Queries<br/>db.query('SELECT * WHERE id=$1', [id])"]
        Escape["Output Escaping<br/>Convert < to &amp;lt;<br/>Never render raw HTML"]
        CSRFToken["CSRF Tokens<br/>Unique token per form<br/>Verify on submission"]
        Validate["Server-side Validation<br/>Validate type, length, format<br/>Never trust client input"]
    end

    SQLi -->|"Prevented by"| Param
    XSS -->|"Prevented by"| Escape
    CSRF -->|"Prevented by"| CSRFToken
    SQLi & XSS & CSRF -->|"Reduced by"| Validate

    style ATTACKS fill:#0d1117,stroke:#f85149,color:#c9d1d9
    style DEFENSES fill:#0d1117,stroke:#3fb950,color:#c9d1d9
```

### 🔄 Input Validation vs. Sanitization
*   **Input Validation**: Checks format, type, and bounds (e.g., checking if `age` is a number > 0 and < 120).
    *   *Solution*: Use libraries like **Zod**, **Joi**, or standard validator libraries.
    *   *When*: On **every single API entry point**. Reject inputs that fail validation with a `400 Bad Request`.
*   **Input Sanitization**: Modifies and cleans the incoming data, stripping dangerous characters, scripts, or unwanted HTML tags.
    *   *Solution*: Use sanitization engines like **DOMPurify** (for rich text/HTML) or specialized escaping logic.
    *   *When*: Before storing input into a database if that data will ever be rendered to other users (especially rich HTML comments, bios, markdown).

### ⚔️ Cross-Site Scripting (XSS) Deep Dive
*   **Stored XSS (Persistent)**: Malicious script is saved in the database (e.g., comment section) and executes in the browser of every visiting user.
    *   *Solution*: Sanitize content before saving, escape outputs when rendering, and use modern frameworks like React/Angular which auto-escape variables by default.
*   **Reflected XSS (Non-Persistent)**: Script is sent as part of a request parameter (e.g., search query) and reflected back in the HTML response.
    *   *Solution*: Escape query parameters rendered on screen.
*   **DOM-based XSS**: Client-side JS reads inputs directly from window/DOM (e.g., URL hash) and writes it unsafely into the page (`element.innerHTML`).
    *   *Solution*: Avoid `innerHTML` or `eval()`. Use `textContent` or use sanitizer libraries like DOMPurify.

### 🛡️ Global XSS Protections
1.  **HttpOnly Cookies**: Setting `HttpOnly` on session/JWT cookies makes them inaccessible via JavaScript (`document.cookie`), preventing session hijacking even if XSS is present.
2.  **CSP (Content Security Policy)**: A response header that instructs the browser which scripts/domains are allowed to load. E.g., `Content-Security-Policy: default-src 'self'`.

### ⚔️ SQL Injection Example
```javascript
// ❌ VULNERABLE — string concatenation
const query = `SELECT * FROM users WHERE email = '${userInput}'`;
// If userInput = "' OR 1=1 --"
// Query becomes: SELECT * FROM users WHERE email = '' OR 1=1 --'
// Returns ALL users!

// ✅ SAFE — parameterized query
const query = 'SELECT * FROM users WHERE email = $1';
const result = await db.query(query, [userInput]);
// Input is always treated as DATA, never as SQL code
```

### ⚔️ CSRF (Cross-Site Request Forgery) Deep Dive
*   **What it is**: Tricking an authenticated user's browser into executing an unwanted action on a trusted site because cookies are automatically sent.
*   **Mitigation**:
    1.  **Anti-CSRF Tokens**: Include a unique token in forms/headers. The server verifies this token on incoming write requests.
    2.  **SameSite Cookie Attribute**: Set `SameSite=Lax` or `SameSite=Strict` on cookies to ensure they aren't sent on cross-site requests.

---

## 🔒 Layer 7: Encryption at Rest — Data Shielding

```mermaid
graph TB
    subgraph TRANSIT["🔐 Encryption In Transit (TLS/HTTPS)"]
        Client["Client"] -->|"Encrypted tunnel<br/>🔒 HTTPS"| Server["Server"]
        T_Note["Even if intercepted,<br/>attacker sees gibberish"]
    end

    subgraph REST["🔒 Encryption At Rest"]
        App["Application"] -->|"Data"| Encrypt["Encryption Engine"]
        Encrypt -->|"Encrypted data"| DB["Database / Disk"]
        R_Note["Even if disk is stolen,<br/>data is unreadable without key"]
    end

    subgraph KEYS["🔑 Key Management"]
        K1["Never hardcode encryption keys in code"]
        K2["Use: AWS KMS, GCP KMS, HashiCorp Vault"]
        K3["Rotate keys periodically"]
    end

    style TRANSIT fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
    style REST fill:#0d1117,stroke:#3fb950,color:#c9d1d9
    style KEYS fill:#0d1117,stroke:#d29922,color:#c9d1d9
```

---

## 🔑 Layer 8: Secrets Management

```mermaid
graph TD
    subgraph BAD["❌ WRONG — Secrets in Code"]
        B1["const API_KEY = 'sk_live_abc123'<br/>const DB_PASSWORD = 'admin123'<br/>// Committed to Git! 😱"]
    end

    subgraph GOOD["✅ CORRECT — Secrets Manager"]
        G1["Environment Variables<br/>(injected at deploy time)"]
        G2["AWS Secrets Manager<br/>HashiCorp Vault<br/>GCP Secret Manager"]
        G3["Never in source code<br/>Never in Git history<br/>Rotated automatically"]
    end

    style BAD fill:#0d1117,stroke:#f85149,color:#c9d1d9
    style GOOD fill:#0d1117,stroke:#3fb950,color:#c9d1d9
```

---

## 📋 Layer 9: Audit Logging — Tamper-Proof History


```javascript
// Log sensitive actions
const auditLog = {
  timestamp: new Date().toISOString(),
  userId: req.user.id,
  action: 'DELETE_USER',
  targetId: req.params.userId,
  ip: req.ip,
  userAgent: req.headers['user-agent'],
  result: 'SUCCESS',
};

// Store in append-only, tamper-proof log
await auditService.log(auditLog);
```

### What to Audit Log

| Action | Why |
|--------|-----|
| Login success/failure | Detect brute force, compromised accounts |
| Permission changes | Track who granted/revoked access |
| Data exports/downloads | Detect data exfiltration |
| Sensitive data access | Compliance (GDPR, HIPAA) |
| Configuration changes | Debug and accountability |
| Deletion of records | Detect accidental or malicious deletion |

---

## 🔄 The Complete Security Flow

```mermaid
flowchart TD
    Request["📥 Incoming Request"] --> Firewall{"🔥 Firewall<br/>Blocked IP?"}
    Firewall -->|"Blocked"| Reject1["❌ 403 Forbidden"]
    Firewall -->|"OK"| RateLimit{"🚦 Rate Limited?"}

    RateLimit -->|"Over limit"| Reject2["❌ 429 Too Many"]
    RateLimit -->|"OK"| TLS{"🔐 HTTPS?"}

    TLS -->|"No"| Redirect["301 → HTTPS"]
    TLS -->|"Yes"| Auth{"🔑 Valid token?"}

    Auth -->|"Invalid/expired"| Reject3["❌ 401 Unauthorized"]
    Auth -->|"Valid"| Authz{"🛡️ Has permission?"}

    Authz -->|"No"| Reject4["❌ 403 Forbidden"]
    Authz -->|"Yes"| Validate{"✅ Valid input?"}

    Validate -->|"Invalid"| Reject5["❌ 400 Bad Request"]
    Validate -->|"Valid"| Process["⚙️ Process Request"]

    Process --> AuditLog["📋 Audit Log"]
    Process --> Response["✅ 200 OK"]

    style Request fill:#161b22,stroke:#58a6ff,color:#fff
    style Process fill:#0d1117,stroke:#3fb950,color:#c9d1d9
    style Response fill:#0d1117,stroke:#3fb950,color:#c9d1d9
```

---

## ⚠️ Edge Cases & Gotchas

1. **Client-side validation is NOT security** — It's UX. Any validation in the browser can be bypassed. Always validate on the server.

2. **Don't roll your own auth** — Use established libraries (Passport.js, NextAuth, Firebase Auth). Writing your own login system is inviting vulnerabilities.

3. **JWT in localStorage is risky** — XSS can steal it. Consider httpOnly cookies for storing tokens (not accessible via JavaScript).

4. **API keys in frontend code** — Any key in client-side JavaScript is visible to anyone. Use server-side proxying for sensitive API calls.

5. **Dependency vulnerabilities** — 70%+ of code in most apps comes from npm/pip packages. Run `npm audit` regularly. One compromised dependency = your app is compromised.

6. **Forgot password flow** — Tokens should be single-use, short-lived (15-30 min), and invalidated after password change.

---

## 🔗 Connected Topics

| Topic | Connection |
|-------|-----------|
| [Governance](10-governance.md) | Security policies, compliance, access governance |
| [Latency](08-latency.md) | Security checks add latency (TLS handshake, token validation) |
| [Monitoring](13-monitoring-observability.md) | Monitor for suspicious activity, failed login spikes |
| [Load Balancers](04-load-balancers.md) | LB handles TLS termination, first line of defense |
| [Database](07-database-design.md) | Encryption at rest, backup encryption, access control |

---

**← Previous:** [8. Latency](08-latency.md) | **Next →** [10. Governance](10-governance.md)
