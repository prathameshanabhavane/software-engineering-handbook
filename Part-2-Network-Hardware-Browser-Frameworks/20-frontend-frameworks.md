# ⚛️ 20. Frontend Frameworks & Build Tools

> **Frameworks solve the problem of keeping the UI in sync with data. Without them, you'd manually find and update DOM elements every time data changes — which becomes a nightmare at scale.**

---

## 🔄 The Core Problem Frameworks Solve

```mermaid
graph LR
    subgraph VANILLA["❌ Without Framework"]
        Data1["Data changes"] --> Find["Find DOM elements<br/>by ID/class"]
        Find --> Update["Manually update<br/>innerHTML / textContent"]
        Update --> Bug["Bugs: missed updates,<br/>stale state, spaghetti code"]
    end

    subgraph FRAMEWORK["✅ With Framework"]
        Data2["Data changes"] --> Auto["Framework automatically<br/>updates only what changed"]
        Auto --> Fast["Fast, consistent,<br/>predictable UI"]
    end

    style VANILLA fill:#0d1117,stroke:#f85149,color:#c9d1d9
    style FRAMEWORK fill:#0d1117,stroke:#3fb950,color:#c9d1d9
```

---

## ⚛️ Virtual DOM — How React Updates Efficiently

```mermaid
sequenceDiagram
    participant State as 📦 State
    participant VDOM1 as 🌲 Virtual DOM (old)
    participant VDOM2 as 🌲 Virtual DOM (new)
    participant Diff as 🔍 Diffing Algorithm
    participant RealDOM as 🖥️ Real DOM

    State->>VDOM2: State changes → create new Virtual DOM

    Diff->>VDOM1: Compare old tree
    Diff->>VDOM2: Compare new tree
    Diff->>Diff: Find minimal changes<br/>(what actually changed?)

    Diff->>RealDOM: Apply ONLY the changes<br/>(not entire tree!)

    Note over RealDOM: Only 2 DOM nodes updated<br/>instead of re-rendering entire page
```

### Virtual DOM vs Direct DOM

```mermaid
graph TB
    subgraph DIRECT["❌ Direct DOM Manipulation"]
        DD1["Change 1 → Update DOM → Browser reflows"]
        DD2["Change 2 → Update DOM → Browser reflows"]
        DD3["Change 3 → Update DOM → Browser reflows"]
        DD4["3 separate reflows = slow!"]
    end

    subgraph VDOM["✅ Virtual DOM (Batched)"]
        VD1["Change 1, 2, 3 → Update Virtual DOM (in memory)"]
        VD2["Diff → Find minimal real changes"]
        VD3["One batched DOM update → One reflow"]
        VD4["1 reflow = fast!"]
    end

    style DIRECT fill:#0d1117,stroke:#f85149,color:#c9d1d9
    style VDOM fill:#0d1117,stroke:#3fb950,color:#c9d1d9
```

---

## 🔄 Rendering Strategies — CSR vs SSR vs SSG vs ISR

```mermaid
graph TB
    subgraph CSR["🔵 CSR — Client-Side Rendering"]
        CSR_How["Server sends empty HTML + JS bundle<br/>Browser downloads JS, executes, builds DOM"]
        CSR_Good["✅ Rich interactivity<br/>✅ Smooth page transitions"]
        CSR_Bad["❌ Slow initial load (wait for JS)<br/>❌ Bad SEO (empty HTML)"]
        CSR_Use["Use for: Dashboards, admin panels, apps behind login"]
    end

    subgraph SSR["🟢 SSR — Server-Side Rendering"]
        SSR_How["Server builds complete HTML for each request<br/>Browser receives ready-to-display HTML + hydration JS"]
        SSR_Good["✅ Fast FCP (content visible immediately)<br/>✅ Great SEO"]
        SSR_Bad["❌ Server cost (renders per request)<br/>❌ TTFB depends on server speed"]
        SSR_Use["Use for: E-commerce products, news articles, social feeds"]
    end

    subgraph SSG["🟡 SSG — Static Site Generation"]
        SSG_How["HTML pre-built at BUILD TIME<br/>Served directly from CDN (fastest!)"]
        SSG_Good["✅ Fastest possible (pre-built)<br/>✅ Cheapest (just CDN)"]
        SSG_Bad["❌ Content is static until rebuild<br/>❌ Can't personalize per user"]
        SSG_Use["Use for: Blogs, docs, marketing pages, landing pages"]
    end

    subgraph ISR["🟣 ISR — Incremental Static Regeneration"]
        ISR_How["Pre-built like SSG, but re-generates<br/>in background after TTL expires"]
        ISR_Good["✅ Fast like SSG<br/>✅ Content stays fresh"]
        ISR_Bad["❌ Slight staleness during TTL<br/>❌ Next.js specific (mostly)"]
        ISR_Use["Use for: Product pages, frequently updated content"]
    end

    style CSR fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
    style SSR fill:#0d1117,stroke:#3fb950,color:#c9d1d9
    style SSG fill:#0d1117,stroke:#d29922,color:#c9d1d9
    style ISR fill:#0d1117,stroke:#bc8cff,color:#c9d1d9
```

### Rendering Decision Flow

```mermaid
flowchart TD
    Start["What type of content?"] --> Q1{"Content changes<br/>per request?"}

    Q1 -->|"No — static"| Q2{"Changes<br/>frequently?"}
    Q1 -->|"Yes — dynamic"| Q3{"Needs SEO?"}

    Q2 -->|"Rarely"| SSG["🟡 SSG"]
    Q2 -->|"Every few minutes"| ISR["🟣 ISR"]

    Q3 -->|"Yes"| SSR["🟢 SSR"]
    Q3 -->|"No (behind auth)"| CSR["🔵 CSR"]

    style SSG fill:#0d1117,stroke:#d29922,color:#c9d1d9
    style ISR fill:#0d1117,stroke:#bc8cff,color:#c9d1d9
    style SSR fill:#0d1117,stroke:#3fb950,color:#c9d1d9
    style CSR fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
```

---

## 📦 State Management

```mermaid
graph TB
    subgraph LOCAL["🔵 Local State (Component)"]
        LS["useState / ref<br/>UI toggles, form inputs<br/>Scoped to one component"]
    end

    subgraph SHARED["🟢 Shared State (Cross-Component)"]
        SS["Context API / Props drilling<br/>Theme, auth status<br/>Shared by many components"]
    end

    subgraph GLOBAL["🟣 Global State (App-Wide)"]
        GS["Redux / Zustand / Jotai<br/>Shopping cart, user session<br/>Accessible anywhere"]
    end

    subgraph SERVER["🟡 Server State"]
        SVS["React Query / SWR<br/>API data with caching<br/>Handles loading, error, stale data"]
    end

    style LOCAL fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
    style SHARED fill:#0d1117,stroke:#3fb950,color:#c9d1d9
    style GLOBAL fill:#0d1117,stroke:#bc8cff,color:#c9d1d9
    style SERVER fill:#0d1117,stroke:#d29922,color:#c9d1d9
```

---

## 🛠️ Build Tools — What They Do

```mermaid
graph LR
    Source["📁 Source Code<br/>• JSX/TSX files<br/>• CSS/SCSS modules<br/>• Images, fonts<br/>• npm packages"] --> Bundler

    subgraph Bundler["📦 Build Tool (Vite / Webpack)"]
        Transpile["Transpile<br/>JSX → JS<br/>TS → JS<br/>Modern → Compatible"]
        Bundle["Bundle<br/>1000 files → few bundles<br/>Tree-shake unused code"]
        Optimize["Optimize<br/>Minify code<br/>Compress images<br/>Generate hashes"]
        Split["Code Split<br/>Split by route<br/>Lazy load chunks"]
    end

    Bundler --> Output["📁 Build Output<br/>• app.a1b2c3.js (85KB)<br/>• vendor.x7y8z9.js (120KB)<br/>• style.m4n5o6.css (15KB)<br/>• images/ (optimized)"]

    style Source fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
    style Bundler fill:#0d1117,stroke:#d29922,color:#c9d1d9
    style Output fill:#0d1117,stroke:#3fb950,color:#c9d1d9
```

---

## ⚠️ Edge Cases & Gotchas

1. **Hydration mismatch** — When SSR HTML doesn't match what React generates client-side, you get hydration errors. Avoid rendering random values or dates differently on server vs client.

2. **Bundle size creep** — Every `npm install` adds to your bundle. Use `npm ls --prod` and tools like bundlephobia.com to check package sizes before adding.

3. **Over-rendering** — In React, parent re-rendering re-renders all children. Use `React.memo`, `useMemo`, `useCallback` to prevent unnecessary renders — but only when profiling shows it's a bottleneck.

4. **SEO with client-side routing** — SPA client-side navigation doesn't trigger new page loads. Crawlers may not follow client-side routes. Use SSR/SSG for SEO-critical pages.

5. **State in URL** — Shareable state (filters, search queries, pagination) should be in the URL, not just in component state. Use query params: `/products?category=shoes&page=2`.

---

## 🔗 Connected Topics

| Topic | Connection |
|-------|-----------|
| [Browser Internals](19-browser-internals.md) | Frameworks sit on top of the browser's DOM |
| [CDN & SEO](../Part-1-Architecture-Scalability-Operations/06-cdn-pagespeed-seo.md) | Rendering strategy impacts SEO and page speed |
| [Performance](../Part-1-Architecture-Scalability-Operations/12-performance-optimization.md) | Code splitting, lazy loading, bundle optimization |
| [Caching](../Part-1-Architecture-Scalability-Operations/05-caching.md) | ISR is a form of caching at the page level |
| [Backend Frameworks](21-backend-frameworks.md) | SSR requires server-side framework integration |

---

**← Previous:** [19. Browser Internals](19-browser-internals.md) | **Next →** [21. Backend Frameworks](21-backend-frameworks.md)
