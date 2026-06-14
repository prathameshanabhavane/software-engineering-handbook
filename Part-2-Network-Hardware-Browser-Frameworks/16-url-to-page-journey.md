# 🌐 16. The Complete Journey — What Happens When You Type a URL and Press Enter

> **This is THE most common interview question in system design. It touches almost every concept in this guide. Master this and you demonstrate understanding of the entire stack.**

---

## 🔄 The Complete Flow — Overview

```mermaid
graph LR
    A["⌨️ Type URL"] --> B["🔍 DNS Lookup"]
    B --> C["🤝 TCP Connection"]
    C --> D["🔐 TLS Handshake"]
    D --> E["📤 HTTP Request"]
    E --> F["⚙️ Server Processing"]
    F --> G["📥 HTTP Response"]
    G --> H["🏗️ HTML Parsing"]
    H --> I["🎨 Rendering"]
    I --> J["👁️ Page Visible!"]

    style A fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
    style J fill:#0d1117,stroke:#3fb950,color:#c9d1d9
```

---

## Step 1: URL Parsing & Browser Cache Check

```mermaid
flowchart TD
    Type["⌨️ User types: https://www.example.com/products"] --> Parse["Browser parses URL:<br/>Protocol: https<br/>Host: www.example.com<br/>Path: /products<br/>Port: 443 (implied)"]

    Parse --> HSTS{"HSTS list?<br/>(HTTP Strict Transport<br/>Security)"}
    HSTS -->|"In list"| ForceHTTPS["Force HTTPS<br/>(even if typed http://)"]
    HSTS -->|"Not in list"| Continue["Continue"]

    ForceHTTPS --> CheckCache{"Browser cache<br/>has response?"}
    Continue --> CheckCache

    CheckCache -->|"Cache HIT<br/>+ not expired"| ServeCache["⚡ Serve from cache<br/>No network needed!"]
    CheckCache -->|"Cache MISS<br/>or expired"| DNSStep["→ Step 2: DNS Lookup"]

    style Type fill:#161b22,stroke:#58a6ff,color:#fff
    style ServeCache fill:#0d1117,stroke:#3fb950,color:#c9d1d9
    style DNSStep fill:#0d1117,stroke:#d29922,color:#c9d1d9
```

---

## Step 2: DNS Resolution — Finding the IP Address

```mermaid
sequenceDiagram
    participant Browser
    participant OSCache as OS DNS Cache
    participant Router as Router Cache
    participant ISP as ISP DNS Resolver
    participant Root as Root DNS (.)
    participant TLD as TLD DNS (.com)
    participant Auth as Authoritative DNS<br/>(example.com)

    Browser->>Browser: Check browser DNS cache
    Note over Browser: ❌ Not cached

    Browser->>OSCache: Check OS DNS cache
    Note over OSCache: ❌ Not cached

    Browser->>Router: Check router cache
    Note over Router: ❌ Not cached

    Browser->>ISP: Resolve www.example.com
    Note over ISP: ❌ Not cached → recursive lookup

    ISP->>Root: Where is .com?
    Root-->>ISP: Ask TLD server at 192.12.94.30

    ISP->>TLD: Where is example.com?
    TLD-->>ISP: Ask authoritative server at 205.251.195.1

    ISP->>Auth: What is the IP for www.example.com?
    Auth-->>ISP: A record: 93.184.216.34 (TTL: 3600s)

    ISP-->>Browser: 93.184.216.34
    Note over Browser: Cache this for 3600s (1 hour)
```

### DNS Record Types

| Record | Purpose | Example |
|--------|---------|---------|
| **A** | Domain → IPv4 address | `example.com → 93.184.216.34` |
| **AAAA** | Domain → IPv6 address | `example.com → 2606:2800:220:1::` |
| **CNAME** | Domain → another domain | `www.example.com → example.com` |
| **MX** | Mail server | `example.com → mail.example.com` |
| **TXT** | Verification, SPF, DKIM | `example.com → "v=spf1 include:..."` |
| **NS** | Nameserver delegation | `example.com → ns1.cloudflare.com` |

---

## Step 3: TCP Three-Way Handshake

```mermaid
sequenceDiagram
    participant Client as 🌐 Browser
    participant Server as 🖥️ Server (93.184.216.34:443)

    Note over Client,Server: TCP Three-Way Handshake (~30ms)

    Client->>Server: SYN (seq=100)<br/>"Hey, I want to connect"
    Server-->>Client: SYN-ACK (seq=300, ack=101)<br/>"OK, I acknowledge. Let's connect"
    Client->>Server: ACK (ack=301)<br/>"Great, connection established!"

    Note over Client,Server: ✅ TCP connection established<br/>Reliable, ordered delivery guaranteed
```

---

## Step 4: TLS Handshake (HTTPS)

```mermaid
sequenceDiagram
    participant Client as 🌐 Browser
    participant Server as 🖥️ Server

    Note over Client,Server: TLS 1.3 Handshake (~50ms)

    Client->>Server: ClientHello<br/>• Supported cipher suites<br/>• Key share (ECDHE)

    Server-->>Client: ServerHello<br/>• Selected cipher suite<br/>• Server's key share<br/>• Certificate (proves identity)<br/>• Finished

    Note over Client: Verify certificate chain:<br/>Is it trusted? Not expired?<br/>Does domain match?

    Client->>Server: Finished<br/>• Client's key share

    Note over Client,Server: ✅ Encrypted tunnel established<br/>All data now encrypted with shared secret<br/>🔒 Nobody can eavesdrop
```

---

## Step 5: HTTP Request

```mermaid
graph LR
    subgraph REQUEST["HTTP Request"]
        Method["GET /products HTTP/2"]
        Headers["Headers:<br/>Host: www.example.com<br/>Accept: text/html<br/>Accept-Encoding: gzip, br<br/>Cookie: session=abc123<br/>User-Agent: Chrome/120"]
    end

    style REQUEST fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
```

---

## Step 6: Server Processing

```mermaid
flowchart TD
    Request["📥 Request arrives at server"] --> CDN{"CDN Edge?"}
    CDN -->|"Static asset"| CDNServe["Serve from CDN cache"]
    CDN -->|"Dynamic request"| LB["⚖️ Load Balancer"]

    LB --> WAF{"WAF Check<br/>(malicious?)"}
    WAF -->|"Blocked"| Block["❌ 403 Forbidden"]
    WAF -->|"OK"| Rate{"Rate limit<br/>exceeded?"}

    Rate -->|"Yes"| Limit["❌ 429 Too Many Requests"]
    Rate -->|"No"| App["⚙️ App Server"]

    App --> AuthCheck{"Auth token<br/>valid?"}
    AuthCheck -->|"No"| Unauth["❌ 401 Unauthorized"]
    AuthCheck -->|"Yes"| Process["Process business logic"]

    Process --> Cache{"Cache<br/>hit?"}
    Cache -->|"Yes"| CacheReturn["Return cached data"]
    Cache -->|"No"| DB["Query database"]
    DB --> CacheStore["Store in cache"]
    CacheStore --> Respond["📤 Build HTTP Response"]

    style Request fill:#161b22,stroke:#58a6ff,color:#fff
    style Respond fill:#0d1117,stroke:#3fb950,color:#c9d1d9
```

---

## Step 7: HTTP Response

```mermaid
graph LR
    subgraph RESPONSE["HTTP Response"]
        Status["HTTP/2 200 OK"]
        RHeaders["Headers:<br/>Content-Type: text/html<br/>Content-Encoding: br<br/>Cache-Control: no-cache<br/>Set-Cookie: session=xyz<br/>X-Request-Id: req-456"]
        Body["Body:<br/>&lt;!DOCTYPE html&gt;<br/>&lt;html&gt;...&lt;/html&gt;"]
    end

    style RESPONSE fill:#0d1117,stroke:#3fb950,color:#c9d1d9
```

---

## Step 8-9: Browser Rendering Pipeline

```mermaid
graph TB
    HTML["📄 HTML"] -->|"Parse"| DOM["DOM Tree"]
    CSS["🎨 CSS"] -->|"Parse"| CSSOM["CSSOM Tree"]
    DOM & CSSOM --> RenderTree["🌳 Render Tree<br/>(DOM + styles, minus hidden elements)"]
    RenderTree --> Layout["📐 Layout<br/>(Calculate position & size)"]
    Layout --> Paint["🎨 Paint<br/>(Fill in pixels: colors, text, images)"]
    Paint --> Composite["🖼️ Composite<br/>(Combine layers, GPU acceleration)"]
    Composite --> Screen["🖥️ Pixels on Screen!"]

    JS["⚡ JavaScript"] -->|"Can modify"| DOM
    JS -->|"Can modify"| CSSOM

    style HTML fill:#0d1117,stroke:#f85149,color:#c9d1d9
    style CSS fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
    style JS fill:#0d1117,stroke:#d29922,color:#c9d1d9
    style Screen fill:#0d1117,stroke:#3fb950,color:#c9d1d9
```

---

## 📊 Full Timeline — Where Time Goes

```mermaid
gantt
    title Full Journey: URL → Visible Page
    dateFormat X
    axisFormat %L ms

    section Network
    DNS Lookup (cached)        :dns, 0, 5
    TCP Handshake              :tcp, 5, 35
    TLS Handshake              :tls, 35, 85
    HTTP Request sent          :req, 85, 90

    section Server
    Load Balancer              :lb, 90, 92
    App Processing             :app, 92, 110
    Cache/DB lookup            :db, 110, 130
    Response generated         :resp, 130, 135

    section Response
    Download HTML              :html_dl, 135, 160
    Download CSS (parallel)    :css_dl, 160, 190
    Download JS (parallel)     :js_dl, 160, 220

    section Rendering
    Parse HTML → DOM           :dom, 160, 200
    Parse CSS → CSSOM          :cssom, 190, 210
    FCP                        :milestone, fcp, 210, 210
    Execute JS                 :exec, 220, 300
    Download images            :img, 210, 350
    LCP (hero image)           :milestone, lcp, 350, 350
    TTI (interactive)          :milestone, tti, 300, 300
```

---

## ⚠️ Edge Cases

1. **Service Worker intercept** — If a PWA has a service worker, it can intercept the request before it even hits the network, serving from its own cache.

2. **HTTP/2 Push (deprecated in most browsers)** — Server could push resources before browser asked for them. Now replaced by `103 Early Hints`.

3. **Prefetch/Preconnect** — `<link rel="preconnect" href="https://api.example.com">` starts the TCP+TLS handshake early for known domains.

4. **CORS preflight** — Cross-origin API calls may trigger an OPTIONS preflight request before the actual request, adding latency.

---

## 🔗 Connected Topics

| Step | Related Chapters |
|------|-----------------|
| DNS | [17. Networking Fundamentals](17-networking-fundamentals.md) |
| TCP/TLS | [17. Networking Fundamentals](17-networking-fundamentals.md) |
| CDN | [5. Caching](../Part-1-Architecture-Scalability-Operations/05-caching.md), [6. CDN](../Part-1-Architecture-Scalability-Operations/06-cdn-pagespeed-seo.md) |
| Server processing | [14. Request Walkthrough](../Part-1-Architecture-Scalability-Operations/14-request-walkthrough.md) |
| Browser rendering | [19. Browser Internals](19-browser-internals.md) |

---

**← Previous:** [15. Checklist](../Part-1-Architecture-Scalability-Operations/15-checklist.md) | **Next →** [17. Networking Fundamentals](17-networking-fundamentals.md)
