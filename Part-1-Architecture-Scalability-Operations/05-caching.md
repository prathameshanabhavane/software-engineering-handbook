# 💾 5. Caching — Don't Repeat Expensive Work

> **Caching is like keeping frequently-used spices on your kitchen counter instead of walking to the pantry (database) every time you cook. The pantry itself is closer than the supermarket (external API). Each layer you move data closer to where it's needed reduces time.**

---

## 🏗️ The Cache Hierarchy — Four Layers

```mermaid
graph TB
    User["👤 User"] -->|"1. Check"| BrowserCache["🌐 Browser Cache<br/>⏱️ ~0ms<br/>📍 User's device"]

    BrowserCache -->|"MISS → 2. Check"| CDNCache["🌍 CDN Cache<br/>⏱️ ~10-50ms<br/>📍 Edge server near user"]

    CDNCache -->|"MISS → 3. Check"| AppCache["💾 Application Cache (Redis)<br/>⏱️ ~1-5ms<br/>📍 In-memory store"]

    AppCache -->|"MISS → 4. Query"| DB["🗄️ Database<br/>⏱️ ~10-100ms<br/>📍 Disk-based storage"]

    DB -->|"Store result"| AppCache
    AppCache -->|"Return to CDN"| CDNCache
    CDNCache -->|"Return + cache headers"| BrowserCache
    BrowserCache -->|"Serve"| User

    style User fill:#161b22,stroke:#58a6ff,color:#fff
    style BrowserCache fill:#0d1117,stroke:#3fb950,color:#c9d1d9
    style CDNCache fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
    style AppCache fill:#0d1117,stroke:#d29922,color:#c9d1d9
    style DB fill:#0d1117,stroke:#f85149,color:#c9d1d9
```

### Speed Comparison

| Layer | Latency | Analogy |
|-------|---------|---------|
| Browser Cache | ~0ms | Spice on your kitchen counter |
| CDN Cache | ~10-50ms | Neighborhood store |
| Redis/Memcached | ~1-5ms | Pantry in your house |
| Database | ~10-100ms | Supermarket across town |
| External API | ~100-2000ms | Factory in another country |

---

## 🌐 Layer 1: Browser Cache

### How HTTP Caching Works

```mermaid
sequenceDiagram
    participant Browser
    participant Server

    Note over Browser,Server: First Visit

    Browser->>Server: GET /style.css
    Server-->>Browser: 200 OK<br/>Cache-Control: max-age=86400<br/>ETag: "abc123"
    Note over Browser: Stores locally for 24 hours

    Note over Browser,Server: Second Visit (within 24 hours)

    Browser->>Browser: Cache-Control says "still fresh"
    Note over Browser: ✅ Serve from cache<br/>No network request at all!

    Note over Browser,Server: Third Visit (after 24 hours)

    Browser->>Server: GET /style.css<br/>If-None-Match: "abc123"
    alt File hasn't changed
        Server-->>Browser: 304 Not Modified
        Note over Browser: ✅ Use cached version<br/>(saved bandwidth!)
    else File has changed
        Server-->>Browser: 200 OK (new file)<br/>ETag: "def456"
        Note over Browser: Update cache
    end
```

### Key HTTP Cache Headers

| Header | Purpose | Example |
|--------|---------|---------|
| `Cache-Control: max-age=86400` | Cache for 24 hours | Static assets (CSS, JS, images) |
| `Cache-Control: no-cache` | Always revalidate with server | Dynamic pages |
| `Cache-Control: no-store` | Never cache at all | Sensitive data (bank pages) |
| `ETag: "abc123"` | Fingerprint for change detection | Any cached resource |
| `Cache-Control: immutable` | Never revalidate (use with hashed filenames) | `app.a1b2c3.js` |

### Cache Busting with Hashed Filenames

```mermaid
graph LR
    subgraph OLD["Version 1"]
        O1["app.a1b2c3.js"] -->|"cached forever"| OC["Browser Cache"]
    end

    subgraph NEW["Version 2 (new deploy)"]
        N1["app.x7y8z9.js"] -->|"new hash = new URL<br/>forces fresh download"| NC["Browser Cache"]
    end

    style OLD fill:#0d1117,stroke:#d29922,color:#c9d1d9
    style NEW fill:#0d1117,stroke:#3fb950,color:#c9d1d9
```

---

## 🌍 Layer 2: CDN Cache

See [Chapter 6 — CDN, Page Speed & SEO](06-cdn-pagespeed-seo.md) for full details. Quick summary:

```mermaid
graph TB
    subgraph WORLD["🌍 CDN Edge Nodes Worldwide"]
        Mumbai["🇮🇳 Mumbai Edge"]
        London["🇬🇧 London Edge"]
        Tokyo["🇯🇵 Tokyo Edge"]
        NYC["🇺🇸 New York Edge"]
    end

    Origin["🏢 Origin Server<br/>(Your data center)"]

    UserIN["👤 User in India"] -->|"Served from Mumbai<br/>~20ms"| Mumbai
    UserUK["👤 User in UK"] -->|"Served from London<br/>~15ms"| London

    Mumbai -->|"First request:<br/>fetch from origin"| Origin
    London -->|"First request:<br/>fetch from origin"| Origin

    Mumbai -->|"Subsequent requests:<br/>serve from edge cache"| UserIN

    style WORLD fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
    style Origin fill:#161b22,stroke:#d29922,color:#fff
```

---

## 💾 Layer 3: Application Cache (Redis / Memcached)

### Cache-Aside Pattern (Most Common)

```mermaid
flowchart TD
    Request["📥 Request: Get User #123"] --> CheckCache{"Check Redis<br/>user:123"}

    CheckCache -->|"✅ Cache HIT"| ReturnCached["⚡ Return cached data<br/>(~1ms)"]

    CheckCache -->|"❌ Cache MISS"| QueryDB["Query Database<br/>(~50ms)"]

    QueryDB --> StoreCache["Store in Redis<br/>SET user:123 data TTL 300"]

    StoreCache --> ReturnFresh["Return fresh data"]

    style Request fill:#161b22,stroke:#58a6ff,color:#fff
    style CheckCache fill:#161b22,stroke:#d29922,color:#fff
    style ReturnCached fill:#0d1117,stroke:#3fb950,color:#c9d1d9
    style QueryDB fill:#0d1117,stroke:#f85149,color:#c9d1d9
    style StoreCache fill:#0d1117,stroke:#d29922,color:#c9d1d9
    style ReturnFresh fill:#0d1117,stroke:#3fb950,color:#c9d1d9
```

### Code Example (Node.js)

```javascript
async function getUser(userId) {
  const cacheKey = `user:${userId}`;

  // 1. Check cache first
  const cached = await redis.get(cacheKey);
  if (cached) {
    return JSON.parse(cached); // ⚡ Cache HIT (~1ms)
  }

  // 2. Cache MISS → query database
  const user = await db.query('SELECT * FROM users WHERE id = $1', [userId]);

  // 3. Store in cache for next time (TTL: 5 minutes)
  await redis.set(cacheKey, JSON.stringify(user), 'EX', 300);

  return user;
}
```

### Write-Through Pattern

```mermaid
flowchart TD
    Write["📝 Write: Update User #123"] --> UpdateDB["1. Update Database"]
    UpdateDB --> UpdateCache["2. Update Redis Cache<br/>(immediately)"]
    UpdateCache --> Done["✅ Cache and DB always in sync"]

    style Write fill:#161b22,stroke:#58a6ff,color:#fff
    style UpdateDB fill:#0d1117,stroke:#f85149,color:#c9d1d9
    style UpdateCache fill:#0d1117,stroke:#d29922,color:#c9d1d9
    style Done fill:#0d1117,stroke:#3fb950,color:#c9d1d9
```

### Write-Behind (Write-Back) Pattern

```mermaid
flowchart TD
    Write["📝 Write: Update User #123"] --> UpdateCache["1. Update Redis Cache<br/>(respond to user immediately)"]
    UpdateCache --> Response["⚡ 200 OK (fast!)"]
    UpdateCache --> Queue["2. Queue DB write<br/>(async, background)"]
    Queue --> UpdateDB["3. Batch write to DB<br/>(eventually)"]

    style Write fill:#161b22,stroke:#58a6ff,color:#fff
    style UpdateCache fill:#0d1117,stroke:#3fb950,color:#c9d1d9
    style Response fill:#0d1117,stroke:#3fb950,color:#c9d1d9
    style Queue fill:#0d1117,stroke:#d29922,color:#c9d1d9
    style UpdateDB fill:#0d1117,stroke:#f85149,color:#c9d1d9
```

---

## 🧠 Cache Invalidation — The Hard Problem

> "There are only two hard things in Computer Science: cache invalidation and naming things." — Phil Karlton

```mermaid
graph TD
    subgraph PROBLEM["⚠️ The Problem"]
        direction TB
        P1["Data changes in DB"]
        P2["Cache still has OLD data"]
        P3["Users see stale information!"]
        P1 --> P2 --> P3
    end

    subgraph SOLUTIONS["✅ Solutions"]
        direction TB
        Sol1["🕐 TTL (Time-to-Live)<br/>Cache expires after X seconds<br/>Simple but data can be stale until expiry"]
        Sol2["📝 Write-Through<br/>Update cache on every write<br/>Always fresh but complex"]
        Sol3["🗑️ Cache-Aside Invalidation<br/>Delete cache key on write<br/>Next read rebuilds cache"]
        Sol4["📢 Event-Based<br/>DB change triggers cache update<br/>Near real-time but complex"]
    end

    style PROBLEM fill:#0d1117,stroke:#f85149,color:#c9d1d9
    style SOLUTIONS fill:#0d1117,stroke:#3fb950,color:#c9d1d9
```

### Invalidation Strategy Comparison

| Strategy | Freshness | Complexity | Best For |
|----------|-----------|-----------|----------|
| **TTL only** | Stale for up to TTL | Very Low | Content that's OK being slightly stale (news, catalog) |
| **Write-through** | Always fresh | Medium | Critical data (user profile, balance) |
| **Delete on write** | Fresh after first read | Low | Most general-purpose use cases |
| **Event-based** | Near real-time | High | Multi-service systems with shared data |

### Cache Invalidation Code Example

```javascript
// Strategy: Delete on write (Cache-Aside Invalidation)
async function updateUser(userId, newData) {
  // 1. Update the database (source of truth)
  await db.query('UPDATE users SET name = $1 WHERE id = $2', [newData.name, userId]);

  // 2. Delete the cached version (forces next read to rebuild)
  await redis.del(`user:${userId}`);

  // Also invalidate related caches!
  await redis.del(`user-profile:${userId}`);
  await redis.del(`user-orders:${userId}`);
}
```

---

## 🗄️ Layer 4: Database Query Cache

```mermaid
graph LR
    subgraph DB_CACHE["Database-Level Caching"]
        Query["SELECT * FROM products<br/>WHERE category = 'shoes'"] --> QCache{"Query Cache"}
        QCache -->|"HIT"| Cached["Return cached result set"]
        QCache -->|"MISS"| Execute["Execute query on disk"]
        Execute --> Store["Store result in cache"]
    end

    subgraph MAT_VIEW["Materialized Views"]
        MV["Pre-computed view:<br/>top_selling_products<br/>(refreshed every 5 min)"]
        MV -->|"Dashboard reads<br/>this instead of<br/>heavy real-time query"| Fast["⚡ Fast reads"]
    end

    style DB_CACHE fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
    style MAT_VIEW fill:#0d1117,stroke:#3fb950,color:#c9d1d9
```

---

## 📊 Cache Hit Rate — The Key Metric

```mermaid
graph LR
    subgraph METRICS["Cache Performance"]
        HR90["90% hit rate<br/>✅ Good<br/>DB handles 10% of reads"]
        HR99["99% hit rate<br/>🌟 Excellent<br/>DB handles 1% of reads"]
        HR50["50% hit rate<br/>⚠️ Poor<br/>DB still handles half"]
    end

    Formula["Hit Rate =<br/>Cache Hits / Total Requests × 100"]

    style METRICS fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
    style Formula fill:#161b22,stroke:#d29922,color:#fff
```

### What Affects Hit Rate?

| Factor | Impact |
|--------|--------|
| TTL too short | Items expire before being reused → more misses |
| TTL too long | Stale data but higher hit rate |
| Cache size too small | Items evicted to make room → "cache thrashing" |
| Data too unique | Each user sees different data → hard to cache |
| Insufficient warming | Empty cache after restart → thundering herd |

---

## ⚡ The Thundering Herd Problem

```mermaid
sequenceDiagram
    participant C1 as User 1
    participant C2 as User 2
    participant C3 as User 3
    participant C4 as User N...
    participant Cache as Redis
    participant DB as Database

    Note over Cache: Popular cache key EXPIRES

    C1->>Cache: GET product:popular
    Cache-->>C1: MISS ❌
    C2->>Cache: GET product:popular
    Cache-->>C2: MISS ❌
    C3->>Cache: GET product:popular
    Cache-->>C3: MISS ❌
    C4->>Cache: GET product:popular
    Cache-->>C4: MISS ❌

    Note over DB: 💥 ALL requests hit DB simultaneously!<br/>Database overloaded!

    C1->>DB: SELECT * FROM products...
    C2->>DB: SELECT * FROM products...
    C3->>DB: SELECT * FROM products...
    C4->>DB: SELECT * FROM products...

    Note over Cache,DB: Solution: Cache locking<br/>Only first request queries DB,<br/>others wait for cache to be filled
```

### Solution: Cache Locking

```javascript
async function getWithLock(key) {
  // 1. Try cache
  let data = await redis.get(key);
  if (data) return JSON.parse(data);

  // 2. Try to acquire lock
  const lockKey = `lock:${key}`;
  const acquired = await redis.set(lockKey, '1', 'NX', 'EX', 10); // NX = only if not exists

  if (acquired) {
    // 3. I got the lock → query DB and fill cache
    data = await db.query(/* ... */);
    await redis.set(key, JSON.stringify(data), 'EX', 300);
    await redis.del(lockKey);
    return data;
  } else {
    // 4. Someone else has the lock → wait and retry
    await sleep(100);
    return getWithLock(key); // retry
  }
}
```

---

## 🍔 Real-World Example — News Website

```mermaid
graph TB
    subgraph WITHOUT_CACHE["❌ Without Caching"]
        W_Users["1M page views/hour"] --> W_DB["Database<br/>1M queries/hour<br/>💥 Overloaded!"]
    end

    subgraph WITH_CACHE["✅ With Caching (TTL: 2 min)"]
        C_Users["1M page views/hour"] --> C_Cache["Redis Cache<br/>999,970 cache hits<br/>⚡ Fast!"]
        C_Cache --> C_DB["Database<br/>30 queries/hour<br/>✅ Relaxed"]
    end

    style WITHOUT_CACHE fill:#0d1117,stroke:#f85149,color:#c9d1d9
    style WITH_CACHE fill:#0d1117,stroke:#3fb950,color:#c9d1d9
```

The "Top 10 trending articles" is recalculated every 2 minutes. Instead of running a heavy ranking query per page load:
- **Without cache**: 1M heavy queries/hour → DB collapses
- **With 2-min TTL**: 30 queries/hour (once per 2 minutes) → 99.997% reduction

---

## ⚠️ Edge Cases & Gotchas

1. **Cache poisoning** — If bad data gets into cache, it's served to everyone for the TTL duration. Always validate data before caching.

2. **Cache stampede on restart** — When Redis restarts, all cached data is gone. Thousands of requests simultaneously hit the DB. Solution: warm the cache before routing traffic.

3. **Serialization overhead** — Storing complex objects in Redis requires serialization (JSON.stringify). For very hot paths, this overhead matters.

4. **Cache memory limits** — Redis has finite memory. Use eviction policies (LRU = Least Recently Used) so old/unused items are evicted when memory is full.

5. **Different cache TTLs for different data** — User sessions: 30 min. Product catalog: 5 min. Static config: 1 hour. Don't use one TTL for everything.

---

## 🔗 Connected Topics

| Topic | Connection |
|-------|-----------|
| [CDN](06-cdn-pagespeed-seo.md) | CDN is a geographic cache layer for static + dynamic content |
| [Database Design](07-database-design.md) | Caching reduces database load; materialized views are DB-level caches |
| [Latency](08-latency.md) | Each cache layer eliminates a slower lookup |
| [Scalability](03-scalability.md) | Caching is the cheapest way to scale reads |
| [Performance](12-performance-optimization.md) | Cache hit rate is a critical performance metric |

---

**← Previous:** [4. Load Balancers](04-load-balancers.md) | **Next →** [6. CDN, Page Speed & SEO](06-cdn-pagespeed-seo.md)
