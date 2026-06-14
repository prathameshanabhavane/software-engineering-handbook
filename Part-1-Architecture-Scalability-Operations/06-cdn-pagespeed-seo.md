# 🌍 6. CDN, Page Speed & SEO — Fast, Discoverable, Global

> **A slow page is like a restaurant where the menu takes 10 minutes to arrive — most customers leave before ordering. Optimizing page speed is like having the menu, water, and bread ready the moment someone sits down.**

---

## 🌐 CDN — Content Delivery Network

### How a CDN Works

```mermaid
sequenceDiagram
    actor User as 👤 User (Mumbai)
    participant Edge as 🇮🇳 Mumbai Edge Node
    participant Origin as 🇺🇸 Origin Server (US)

    Note over User,Origin: First request from Mumbai region

    User->>Edge: GET /images/product.jpg
    Edge->>Edge: Check local cache
    Note over Edge: ❌ Cache MISS (first time)
    Edge->>Origin: Fetch from origin
    Origin-->>Edge: 200 OK + image data
    Edge->>Edge: Cache locally (TTL: 24h)
    Edge-->>User: Serve image (~200ms total)

    Note over User,Origin: Subsequent requests from Mumbai

    User->>Edge: GET /images/product.jpg
    Edge->>Edge: Check local cache
    Note over Edge: ✅ Cache HIT!
    Edge-->>User: Serve from edge (~20ms total!)

    Note over User,Origin: 10x faster because no round trip to US!
```

### CDN Global Distribution

```mermaid
graph TB
    Origin["🏢 Origin Server<br/>(US East)"] --> Edge1["🇮🇳 Mumbai Edge"]
    Origin --> Edge2["🇬🇧 London Edge"]
    Origin --> Edge3["🇯🇵 Tokyo Edge"]
    Origin --> Edge4["🇧🇷 São Paulo Edge"]
    Origin --> Edge5["🇦🇺 Sydney Edge"]
    Origin --> Edge6["🇺🇸 US West Edge"]

    U1["👤 Indian User"] -->|"~20ms"| Edge1
    U2["👤 UK User"] -->|"~15ms"| Edge2
    U3["👤 Japanese User"] -->|"~10ms"| Edge3

    U1 -.->|"~300ms without CDN"| Origin

    style Origin fill:#161b22,stroke:#d29922,color:#fff
    style Edge1 fill:#0d1117,stroke:#3fb950,color:#c9d1d9
    style Edge2 fill:#0d1117,stroke:#3fb950,color:#c9d1d9
    style Edge3 fill:#0d1117,stroke:#3fb950,color:#c9d1d9
    style Edge4 fill:#0d1117,stroke:#3fb950,color:#c9d1d9
    style Edge5 fill:#0d1117,stroke:#3fb950,color:#c9d1d9
    style Edge6 fill:#0d1117,stroke:#3fb950,color:#c9d1d9
```

### What CDNs Cache

| Content Type | Cache Strategy | TTL |
|-------------|---------------|-----|
| Images, videos | Cache aggressively | 1-30 days |
| CSS, JS (with hash) | Cache immutably | 1 year |
| Fonts | Cache aggressively | 30 days |
| HTML pages | Short cache or revalidate | 0-5 min |
| API responses | Varies by endpoint | 0-60 sec |
| User-specific data | Do NOT cache at CDN | N/A |

---

## ⚡ Page Speed — Core Web Vitals

### The Loading Timeline

```mermaid
gantt
    title Page Load Timeline — What the User Sees
    dateFormat X
    axisFormat %L ms

    section Network
    DNS Lookup          :dns, 0, 50
    TCP + TLS           :tcp, 50, 150
    Server Processing   :server, 150, 350
    Download HTML       :html, 350, 450

    section Rendering
    Parse HTML + DOM    :dom, 450, 550
    Download CSS        :css, 450, 600
    Parse CSS + CSSOM   :cssom, 600, 650
    FCP (First Paint)   :milestone, fcp, 650, 650
    Download Images     :img, 650, 1200
    Download JS         :js, 550, 900
    Execute JS          :exec, 900, 1400
    LCP (Main Content)  :milestone, lcp, 1200, 1200
    TTI (Interactive)   :milestone, tti, 1400, 1400
```

### Core Web Vitals Explained

```mermaid
graph TB
    subgraph LCP["🎨 LCP — Largest Contentful Paint"]
        LCP_What["When does the MAIN content appear?"]
        LCP_Good["✅ Good: < 2.5s"]
        LCP_Bad["❌ Poor: > 4.0s"]
        LCP_Fix["Fix: Optimize images, preload fonts,<br/>use CDN, server-side render"]
    end

    subgraph INP["👆 INP — Interaction to Next Paint"]
        INP_What["How fast does UI respond to clicks?"]
        INP_Good["✅ Good: < 200ms"]
        INP_Bad["❌ Poor: > 500ms"]
        INP_Fix["Fix: Break up long tasks,<br/>use web workers, defer non-critical JS"]
    end

    subgraph CLS["📐 CLS — Cumulative Layout Shift"]
        CLS_What["Does content jump around while loading?"]
        CLS_Good["✅ Good: < 0.1"]
        CLS_Bad["❌ Poor: > 0.25"]
        CLS_Fix["Fix: Set image dimensions,<br/>reserve ad space, avoid dynamic injection"]
    end

    style LCP fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
    style INP fill:#0d1117,stroke:#3fb950,color:#c9d1d9
    style CLS fill:#0d1117,stroke:#d29922,color:#c9d1d9
```

### Page Speed Optimization Checklist

```mermaid
flowchart TD
    Start["🐌 Slow Page"] --> M1["1. Measure<br/>Chrome DevTools → Lighthouse"]

    M1 --> Images{"Images<br/>optimized?"}
    Images -->|"No"| ImgFix["• Convert to WebP/AVIF<br/>• Responsive srcset<br/>• Lazy load below fold<br/>• Set width/height attributes"]

    Images -->|"Yes"| JS{"JS bundle<br/>too large?"}
    JS -->|"Yes"| JSFix["• Code-split by route<br/>• Tree-shake unused code<br/>• Dynamic imports<br/>• Defer non-critical scripts"]

    JS -->|"No"| CSS{"Render-blocking<br/>CSS?"}
    CSS -->|"Yes"| CSSFix["• Inline critical CSS<br/>• Async load non-critical<br/>• Remove unused CSS"]

    CSS -->|"No"| Server{"Server<br/>response slow?"}
    Server -->|"Yes"| ServerFix["• Add caching (Redis)<br/>• Use CDN<br/>• SSR/SSG for HTML<br/>• Database indexes"]

    Server -->|"No"| Fonts{"Fonts causing<br/>FOUT/FOIT?"}
    Fonts -->|"Yes"| FontFix["• font-display: swap<br/>• Preload key fonts<br/>• Use system fonts as fallback"]

    Fonts -->|"No"| Done["✅ Page is fast!"]

    style Start fill:#161b22,stroke:#f85149,color:#fff
    style Done fill:#161b22,stroke:#3fb950,color:#fff
```

---

## 🔍 SEO — Search Engine Optimization

### How Search Engines See Your Page

```mermaid
graph LR
    subgraph CRAWLER["🕷️ Search Engine Crawler"]
        Discover["1. Discover<br/>(sitemap, links)"]
        Crawl["2. Crawl<br/>(download HTML)"]
        Render["3. Render<br/>(execute JS if needed)"]
        Index["4. Index<br/>(understand content)"]
        Rank["5. Rank<br/>(score & sort)"]
    end

    Discover --> Crawl --> Render --> Index --> Rank

    style CRAWLER fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
```

### SEO vs Rendering Strategy

```mermaid
graph TB
    subgraph CSR["❌ Client-Side Rendering (CSR)"]
        CSR_HTML["Server sends: empty HTML + JS bundle"]
        CSR_Browser["Browser downloads JS, executes, builds DOM"]
        CSR_SEO["Crawler sees: empty page or waits for JS<br/>⚠️ Poor SEO"]
    end

    subgraph SSR["✅ Server-Side Rendering (SSR)"]
        SSR_Server["Server builds complete HTML"]
        SSR_HTML["Sends: full HTML + hydration JS"]
        SSR_SEO["Crawler sees: complete content immediately<br/>✅ Great SEO"]
    end

    subgraph SSG["✅ Static Site Generation (SSG)"]
        SSG_Build["HTML pre-built at build time"]
        SSG_CDN["Served from CDN (fastest!)"]
        SSG_SEO["Crawler sees: complete content instantly<br/>✅ Best SEO + Speed"]
    end

    style CSR fill:#0d1117,stroke:#f85149,color:#c9d1d9
    style SSR fill:#0d1117,stroke:#3fb950,color:#c9d1d9
    style SSG fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
```

### SEO Technical Checklist

```mermaid
graph TD
    SEO["🔍 SEO Essentials"] --> Structure["📄 Structure"]
    SEO --> Content["📝 Content"]
    SEO --> Technical["⚙️ Technical"]
    SEO --> Performance["⚡ Performance"]

    Structure --> S1["Semantic HTML (h1, nav, article, section)"]
    Structure --> S2["One h1 per page"]
    Structure --> S3["Clean URLs (/products/blue-shoes)"]
    Structure --> S4["Proper heading hierarchy (h1 > h2 > h3)"]

    Content --> C1["Unique title tag per page"]
    Content --> C2["Meta description (150-160 chars)"]
    Content --> C3["Open Graph tags (social sharing)"]
    Content --> C4["Structured data (Schema.org)"]

    Technical --> T1["sitemap.xml (list of pages)"]
    Technical --> T2["robots.txt (crawl rules)"]
    Technical --> T3["Canonical URLs (avoid duplicates)"]
    Technical --> T4["Mobile responsive (mobile-first indexing)"]

    Performance --> P1["LCP < 2.5s"]
    Performance --> P2["TLS/HTTPS (ranking signal)"]
    Performance --> P3["No broken links (404s)"]
    Performance --> P4["Fast server response (TTFB < 600ms)"]

    style SEO fill:#161b22,stroke:#58a6ff,color:#fff
```

### Semantic HTML vs Div Soup

```html
<!-- ❌ BAD: Div soup — search engines can't understand structure -->
<div class="header">
  <div class="logo">My Site</div>
  <div class="menu">
    <div class="menu-item">Home</div>
    <div class="menu-item">About</div>
  </div>
</div>
<div class="content">
  <div class="title">Welcome</div>
  <div class="text">This is my page</div>
</div>

<!-- ✅ GOOD: Semantic HTML — clear meaning for crawlers & screen readers -->
<header>
  <h1>My Site</h1>
  <nav>
    <a href="/">Home</a>
    <a href="/about">About</a>
  </nav>
</header>
<main>
  <article>
    <h2>Welcome</h2>
    <p>This is my page</p>
  </article>
</main>
```

---

## 🔗 CDN + Page Speed + SEO — How They Connect

```mermaid
graph TD
    CDN["🌍 CDN"] -->|"Faster static assets"| Speed["⚡ Page Speed"]
    Speed -->|"Better Core Web Vitals"| SEO["🔍 SEO Ranking"]
    SEO -->|"More organic traffic"| Users["👥 More Users"]
    Users -->|"More load"| CDN

    SSR["🖥️ SSR/SSG"] -->|"Fast first paint"| Speed
    SSR -->|"Crawlable HTML"| SEO

    Semantic["📄 Semantic HTML"] -->|"Screen readers"| A11y["♿ Accessibility"]
    Semantic -->|"Clear structure"| SEO

    style CDN fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
    style Speed fill:#0d1117,stroke:#3fb950,color:#c9d1d9
    style SEO fill:#0d1117,stroke:#d29922,color:#c9d1d9
    style Users fill:#0d1117,stroke:#bc8cff,color:#c9d1d9
    style SSR fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
    style Semantic fill:#0d1117,stroke:#3fb950,color:#c9d1d9
    style A11y fill:#0d1117,stroke:#d29922,color:#c9d1d9
```

---

## 📚 Analogy — The Library

SEO is like **organizing a library**:
- A well-labeled library (clear sections, a catalog, signs) lets visitors and librarians (search engines) quickly find and recommend the right book
- A library with books piled randomly in one room — even if it has the best books — will be ignored because no one can find anything

---

## ⚠️ Edge Cases & Gotchas

1. **CDN cache invalidation** — When you deploy changes, old cached versions may persist at edge nodes. Use versioned URLs or purge the CDN cache on deploy.

2. **Dynamic content at CDN** — Be careful caching user-specific data at CDN level. `Cache-Control: private` ensures CDN doesn't cache personal data.

3. **Over-relying on Lighthouse scores** — A perfect 100 score in development doesn't mean real users experience it that way. Use Real User Monitoring (RUM) data.

4. **SPA SEO trap** — Single Page Apps (React/Vue CSR) render HTML client-side. Many search engines struggle with this. Use SSR/SSG for content that needs to be indexed.

5. **Image CDN transforms** — Modern CDNs (Cloudinary, imgix, Cloudflare Images) can resize and convert images on-the-fly. Serve WebP to Chrome, AVIF to newer browsers, JPEG to old ones — all from one source.

---

## 🔗 Connected Topics

| Topic | Connection |
|-------|-----------|
| [Caching](05-caching.md) | CDN is layer 2 of the cache hierarchy |
| [Latency](08-latency.md) | CDN reduces network distance = reduced latency |
| [Browser Internals](../Part-2-Network-Hardware-Browser-Frameworks/19-browser-internals.md) | Page speed depends on how the browser renders |
| [Frontend Frameworks](../Part-2-Network-Hardware-Browser-Frameworks/20-frontend-frameworks.md) | SSR/SSG/CSR choice impacts both speed and SEO |
| [Monitoring](13-monitoring-observability.md) | Track Core Web Vitals as metrics |

---

**← Previous:** [5. Caching](05-caching.md) | **Next →** [7. Database Design](07-database-design.md)
