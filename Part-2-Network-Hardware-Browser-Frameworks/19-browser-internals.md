# 🌐 19. Browser Internals — From HTML to Pixels

> **The browser is the most complex piece of software most users interact with — it's an operating system within an operating system. Understanding how it works makes you a better frontend developer.**

---

## 🏗️ Browser Architecture

```mermaid
graph TB
    subgraph BROWSER["🌐 Browser"]
        UI["🎨 Browser UI<br/>(address bar, tabs, bookmarks)"]
        Engine["⚙️ Browser Engine<br/>(bridges UI and rendering)"]

        subgraph RENDERER["🖼️ Rendering Engine (per tab)"]
            HTMLParser["HTML Parser"]
            CSSParser["CSS Parser"]
            JSEngine["⚡ JS Engine<br/>(V8 / SpiderMonkey)"]
            Layout["Layout Engine"]
            Painter["Painting Layer"]
            Compositor["Compositor<br/>(GPU-accelerated)"]
        end

        Network["🌐 Network Layer<br/>(HTTP, DNS, cache)"]
        Storage["💾 Storage<br/>(localStorage, IndexedDB,<br/>cookies, Cache API)"]
    end

    UI --> Engine --> RENDERER
    RENDERER --> Network
    RENDERER --> Storage

    style BROWSER fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
    style RENDERER fill:#0d1117,stroke:#3fb950,color:#c9d1d9
```

---

## 🎨 The Critical Rendering Path

```mermaid
graph TD
    HTML["📄 HTML bytes"] -->|"Parse"| DOM["🌳 DOM Tree<br/>(Document Object Model)"]

    CSS["🎨 CSS bytes"] -->|"Parse"| CSSOM["🎨 CSSOM Tree<br/>(CSS Object Model)"]

    DOM & CSSOM -->|"Combine"| RenderTree["🌲 Render Tree<br/>(Only visible elements<br/>display:none excluded)"]

    RenderTree -->|"Calculate"| Layout["📐 Layout / Reflow<br/>(Position, size of every element)"]

    Layout -->|"Fill in"| Paint["🎨 Paint<br/>(Colors, borders, shadows, text)"]

    Paint -->|"Optimize"| Composite["🖼️ Composite<br/>(Combine layers, send to GPU)"]

    Composite --> Screen["🖥️ Pixels on Screen!"]

    JS["⚡ JavaScript"] -->|"Can modify"| DOM
    JS -->|"Can modify"| CSSOM
    JS -->|"Triggers"| Layout

    style HTML fill:#0d1117,stroke:#f85149,color:#c9d1d9
    style CSS fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
    style JS fill:#0d1117,stroke:#d29922,color:#c9d1d9
    style Screen fill:#0d1117,stroke:#3fb950,color:#c9d1d9
```

### What Triggers What — Performance Cost

```mermaid
graph LR
    subgraph CHEAP["🟢 Cheap (Composite Only)"]
        CC1["transform: translateX()"]
        CC2["opacity: 0.5"]
    end

    subgraph MEDIUM["🟡 Medium (Paint + Composite)"]
        MC1["color: red"]
        MC2["background-color"]
        MC3["box-shadow"]
    end

    subgraph EXPENSIVE["🔴 Expensive (Layout + Paint + Composite)"]
        EC1["width, height"]
        EC2["margin, padding"]
        EC3["font-size"]
        EC4["position: top/left"]
    end

    style CHEAP fill:#0d1117,stroke:#3fb950,color:#c9d1d9
    style MEDIUM fill:#0d1117,stroke:#d29922,color:#c9d1d9
    style EXPENSIVE fill:#0d1117,stroke:#f85149,color:#c9d1d9
```

---

## ⚡ JavaScript Engine & Event Loop

```mermaid
graph TB
    subgraph JSENGINE["⚡ JS Engine (V8)"]
        CallStack["📚 Call Stack<br/>(LIFO — one thing at a time)<br/>JavaScript is SINGLE-THREADED"]
        Heap["📦 Heap<br/>(Memory allocation<br/>for objects)"]
    end

    subgraph WEBAPI["🌐 Web APIs (Browser-provided)"]
        Timer["setTimeout / setInterval"]
        Fetch["fetch / XMLHttpRequest"]
        DOM_API["DOM Events (click, scroll)"]
        Geolocation["Geolocation, WebSocket"]
    end

    subgraph QUEUES["📬 Queues"]
        MacroQ["Macro Queue<br/>(setTimeout, setInterval, I/O)"]
        MicroQ["Micro Queue ⚡<br/>(Promise.then, MutationObserver)<br/>Higher priority!"]
    end

    EventLoop["🔄 Event Loop<br/>(checks if Call Stack is empty,<br/>then moves tasks from queues)"]

    CallStack -->|"Offloads async work"| WEBAPI
    WEBAPI -->|"When done, callback goes to"| MacroQ
    WEBAPI -->|"Promise resolves"| MicroQ
    EventLoop -->|"Stack empty? Check micro first"| MicroQ
    EventLoop -->|"Micro empty? Check macro"| MacroQ
    MicroQ & MacroQ -->|"Push to"| CallStack

    style JSENGINE fill:#0d1117,stroke:#d29922,color:#c9d1d9
    style WEBAPI fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
    style QUEUES fill:#0d1117,stroke:#3fb950,color:#c9d1d9
    style EventLoop fill:#161b22,stroke:#f85149,color:#fff
```

### Event Loop Execution Order

```javascript
console.log('1 - Sync');                    // 1st: immediate

setTimeout(() => console.log('2 - Macro'), 0); // 4th: macro queue

Promise.resolve().then(() => {
  console.log('3 - Micro');                 // 2nd: micro queue (higher priority!)
});

console.log('4 - Sync');                    // 3rd: immediate (sync before any queue)

// Output order: 1, 4, 3, 2
// Sync first → Microtasks → Macrotasks
```

---

## 🧵 Web Workers — True Parallelism

```mermaid
graph LR
    subgraph MAIN["🧵 Main Thread"]
        MT["UI rendering<br/>Event handling<br/>DOM manipulation<br/>⚠️ Must stay fast (60fps = 16ms per frame)"]
    end

    subgraph WORKER["🧵 Web Worker (Background Thread)"]
        WT["Heavy computation<br/>Data processing<br/>Image manipulation<br/>✅ Doesn't block UI"]
    end

    MAIN -->|"postMessage(data)"| WORKER
    WORKER -->|"postMessage(result)"| MAIN

    Note1["⚠️ Workers can't access DOM<br/>Communication via messages only"]

    style MAIN fill:#0d1117,stroke:#f85149,color:#c9d1d9
    style WORKER fill:#0d1117,stroke:#3fb950,color:#c9d1d9
```

---

## 💾 Browser Storage Options

```mermaid
graph TB
    subgraph STORAGE["Browser Storage Options"]
        Cookies["🍪 Cookies<br/>• 4KB limit<br/>• Sent with every HTTP request<br/>• Expiration date<br/>• httpOnly (no JS access)<br/>• Best for: auth tokens, sessions"]

        LS["📦 localStorage<br/>• 5-10MB limit<br/>• Persists forever<br/>• Sync API<br/>• Same origin only<br/>• Best for: preferences, themes"]

        SS["📋 sessionStorage<br/>• 5-10MB limit<br/>• Cleared on tab close<br/>• Same origin only<br/>• Best for: temp form data"]

        IDB["🗄️ IndexedDB<br/>• GBs of storage<br/>• Async API<br/>• Complex queries<br/>• Best for: offline data, large datasets"]

        CA["💾 Cache API<br/>• Used by Service Workers<br/>• Store HTTP responses<br/>• Best for: offline page cache (PWA)"]
    end

    style STORAGE fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
```

---

## 🔒 Same-Origin Policy & CORS

```mermaid
graph TB
    subgraph SAME["✅ Same Origin (allowed)"]
        S1["https://example.com/page1"]
        S2["https://example.com/page2"]
        S1 -->|"Can access"| S2
    end

    subgraph DIFF["❌ Different Origin (blocked by default)"]
        D1["https://example.com"]
        D2["https://api.other.com"]
        D1 -->|"❌ Blocked!"| D2
    end

    subgraph CORS["✅ CORS (explicitly allowed)"]
        C1["https://example.com"]
        C2["https://api.other.com<br/>Access-Control-Allow-Origin: https://example.com"]
        C1 -->|"✅ Allowed by CORS header"| C2
    end

    style SAME fill:#0d1117,stroke:#3fb950,color:#c9d1d9
    style DIFF fill:#0d1117,stroke:#f85149,color:#c9d1d9
    style CORS fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
```

---

## ⚠️ Edge Cases & Gotchas

1. **Long tasks block rendering** — If JS takes >50ms, the browser can't render frames = UI feels janky. Break long tasks into smaller chunks or use Web Workers.

2. **Layout thrashing** — Reading a layout property (offsetWidth) then immediately writing one (style.width) in a loop forces the browser to recalculate layout on each iteration. Batch reads and writes separately.

3. **Repaints are expensive** — Animating `width` triggers layout+paint+composite. Animating `transform` triggers only composite (GPU-accelerated). Always prefer `transform` and `opacity` for animations.

4. **Third-party scripts** — Each external script (analytics, ads, widgets) can block rendering and add latency. Load them `async` or `defer`.

5. **Memory leaks in SPAs** — Long-running single-page apps can leak memory through detached DOM nodes, event listeners, and closures. Clean up on component unmount.

---

## 🔗 Connected Topics

| Topic | Connection |
|-------|-----------|
| [URL Journey](16-url-to-page-journey.md) | Rendering is the final step |
| [CDN & Page Speed](../Part-1-Architecture-Scalability-Operations/06-cdn-pagespeed-seo.md) | CRP directly impacts Core Web Vitals |
| [Frontend Frameworks](20-frontend-frameworks.md) | Frameworks optimize DOM updates |
| [Performance](../Part-1-Architecture-Scalability-Operations/12-performance-optimization.md) | Layout thrashing, long tasks |
| [Caching](../Part-1-Architecture-Scalability-Operations/05-caching.md) | Browser cache, Service Worker cache |

---

**← Previous:** [18. Hardware & Infrastructure](18-hardware-infrastructure.md) | **Next →** [20. Frontend Frameworks](20-frontend-frameworks.md)
