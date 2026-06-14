# 🗄️ 7. Database Design & Data Safety

> **SQL is a well-organized filing cabinet — every drawer has labeled folders in a fixed structure. NoSQL is more like sticky notes on a wall — flexible, fast to add, great for rapidly-changing information.**

---

## 🔄 Database Selection Decision Flow

```mermaid
flowchart TD
    Start["🗄️ Need to store data"] --> Q1{"Is data highly<br/>relational?<br/>(joins, foreign keys)"}

    Q1 -->|"Yes"| Q2{"Need strict<br/>consistency (ACID)?"}
    Q1 -->|"No / Flexible schema"| Q3{"What's the<br/>access pattern?"}

    Q2 -->|"Yes"| SQL["🟦 SQL Database<br/>PostgreSQL, MySQL<br/>• Orders, payments, users<br/>• Financial records"]
    Q2 -->|"Eventual OK"| Q3

    Q3 -->|"Key → Document"| DocDB["🟧 Document DB<br/>MongoDB<br/>• Content management<br/>• User profiles<br/>• Product catalogs"]
    Q3 -->|"Key → Value<br/>(fast lookup)"| KV["🟥 Key-Value Store<br/>Redis, DynamoDB<br/>• Sessions, cache<br/>• Shopping carts"]
    Q3 -->|"Relationships<br/>& traversals"| GraphDB["🟪 Graph DB<br/>Neo4j<br/>• Social networks<br/>• Recommendation engines"]
    Q3 -->|"Time-ordered<br/>events"| TSDB["🟩 Time-Series DB<br/>InfluxDB, TimescaleDB<br/>• IoT sensor data<br/>• Metrics, logs"]
    Q3 -->|"Full-text<br/>search"| SearchDB["🔍 Search Engine<br/>Elasticsearch<br/>• Product search<br/>• Log analysis"]

    style Start fill:#1a1a2e,stroke:#e94560,color:#fff
    style SQL fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
    style DocDB fill:#0d1117,stroke:#d29922,color:#c9d1d9
    style KV fill:#0d1117,stroke:#f85149,color:#c9d1d9
    style GraphDB fill:#0d1117,stroke:#bc8cff,color:#c9d1d9
    style TSDB fill:#0d1117,stroke:#3fb950,color:#c9d1d9
    style SearchDB fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
```

---

## 🟦 SQL vs NoSQL — Side by Side

| Aspect | SQL (PostgreSQL, MySQL) | NoSQL (MongoDB, Redis, Cassandra) |
|--------|------------------------|----------------------------------|
| **Schema** | Fixed, predefined | Flexible, schema-less |
| **Relationships** | Built-in (JOINs, foreign keys) | Manual or embedded |
| **Consistency** | Strong (ACID) | Eventually consistent (BASE) |
| **Scaling writes** | Vertical (hard to shard) | Horizontal (designed for it) |
| **Query language** | SQL (standardized) | Varies per database |
| **Best for** | Complex queries, transactions | High volume, flexible data |
| **Analogy** | Filing cabinet (structured) | Sticky notes (flexible) |

### ACID vs BASE

```mermaid
graph LR
    subgraph ACID["🟦 ACID (SQL)"]
        A["Atomicity<br/>All or nothing"]
        C["Consistency<br/>Data always valid"]
        I["Isolation<br/>Transactions don't interfere"]
        D["Durability<br/>Committed = permanent"]
    end

    subgraph BASE["🟧 BASE (NoSQL)"]
        BA["Basically Available<br/>System always responds"]
        S["Soft state<br/>Data may change over time"]
        E["Eventually consistent<br/>All nodes converge eventually"]
    end

    style ACID fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
    style BASE fill:#0d1117,stroke:#d29922,color:#c9d1d9
```

---

## 📊 Indexing — The Textbook Index

### How an Index Works

```mermaid
graph TB
    subgraph WITHOUT["❌ Without Index — Full Table Scan"]
        Q1["SELECT * FROM users<br/>WHERE email = 'alice@example.com'"]
        Q1 --> Scan["Scan ALL 10 million rows<br/>⏱️ ~5 seconds"]
    end

    subgraph WITH["✅ With Index — B-Tree Lookup"]
        Q2["SELECT * FROM users<br/>WHERE email = 'alice@example.com'"]
        Q2 --> BTree["B-Tree index on email<br/>Jump to correct row<br/>⏱️ ~5 milliseconds"]
    end

    style WITHOUT fill:#0d1117,stroke:#f85149,color:#c9d1d9
    style WITH fill:#0d1117,stroke:#3fb950,color:#c9d1d9
```

### B-Tree Index Visualization

```mermaid
graph TB
    Root["Root Node<br/>[M]"] -->|"< M"| Left["[D, H]"]
    Root -->|"> M"| Right["[R, W]"]

    Left -->|"< D"| L1["[A, B, C]"]
    Left -->|"D-H"| L2["[E, F, G]"]
    Left -->|"> H"| L3["[I, J, K]"]

    Right -->|"< R"| R1["[N, O, P]"]
    Right -->|"R-W"| R2["[S, T, U]"]
    Right -->|"> W"| R3["[X, Y, Z]"]

    Search["🔍 Find 'F':<br/>Root → Left (< M)<br/>→ D-H → Found!<br/>Only 3 lookups<br/>vs scanning all 26"]

    style Root fill:#161b22,stroke:#d29922,color:#fff
    style Search fill:#0d1117,stroke:#3fb950,color:#c9d1d9
```

### Index Types & When to Use

| Index Type | Best For | Example |
|-----------|---------|---------|
| **B-Tree** (default) | Equality & range queries | `WHERE age > 25` |
| **Hash** | Exact equality only | `WHERE id = 123` |
| **GIN** (Generalized Inverted) | Full-text search, arrays, JSONB | `WHERE tags @> '{react}'` |
| **Composite** | Multi-column queries | `WHERE city = 'Mumbai' AND age > 25` |
| **Partial** | Subset of rows | `WHERE status = 'active'` (index only active rows) |

### Index Trade-off

```mermaid
graph LR
    More["More Indexes"] --> FasterReads["✅ Faster Reads"]
    More --> SlowerWrites["❌ Slower Writes<br/>(each write updates all indexes)"]
    More --> MoreStorage["❌ More Disk Space"]

    style FasterReads fill:#0d1117,stroke:#3fb950,color:#c9d1d9
    style SlowerWrites fill:#0d1117,stroke:#f85149,color:#c9d1d9
    style MoreStorage fill:#0d1117,stroke:#d29922,color:#c9d1d9
```

---

## 🔄 Replication — Copies for Safety & Speed

### Primary-Replica Architecture

```mermaid
graph TB
    App["⚙️ App Server"]

    App -->|"ALL WRITES"| Primary["🟢 Primary (Master)<br/>Handles writes<br/>Source of truth"]

    App -->|"ALL READS"| Replica1["🔵 Replica 1<br/>Read-only copy"]
    App -->|"ALL READS"| Replica2["🔵 Replica 2<br/>Read-only copy"]

    Primary -->|"Replication Stream<br/>(WAL / binlog)"| Replica1
    Primary -->|"Replication Stream"| Replica2

    subgraph FAILOVER["⚠️ If Primary Dies"]
        Replica1 -->|"Promoted to Primary"| NewPrimary["🟢 New Primary"]
    end

    style App fill:#161b22,stroke:#58a6ff,color:#fff
    style Primary fill:#0d1117,stroke:#3fb950,color:#c9d1d9
    style Replica1 fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
    style Replica2 fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
    style FAILOVER fill:#0d1117,stroke:#d29922,color:#c9d1d9
```

### Replication Lag — The Gotcha

```mermaid
sequenceDiagram
    actor User
    participant App
    participant Primary as 🟢 Primary
    participant Replica as 🔵 Replica

    User->>App: Update profile name to "Alice"
    App->>Primary: UPDATE users SET name='Alice'
    Primary-->>App: ✅ Written

    Note over Primary,Replica: ⏳ Replication lag: ~100ms

    User->>App: View my profile (immediate read)
    App->>Replica: SELECT * FROM users WHERE id=1
    Replica-->>App: name = "Bob" (OLD value!)
    App-->>User: Shows "Bob" instead of "Alice"! 😱

    Note over App: Solution: Read from Primary<br/>for "read-your-own-writes"
```

### Solutions for Replication Lag

| Pattern | How It Works | Use When |
|---------|-------------|----------|
| **Read-your-own-writes** | After a write, read from primary for that user | User updates their own profile |
| **Monotonic reads** | Same user always reads from same replica | Avoid "going back in time" |
| **Synchronous replication** | Primary waits for replicas to confirm | Need strict consistency (rare) |

---

## 🔀 Sharding — Splitting Data Across Machines

### How Sharding Works

```mermaid
graph TB
    App["⚙️ App / Router"] -->|"User ID 1-1M"| Shard1["🗄️ Shard 1<br/>Users 1 - 1,000,000"]
    App -->|"User ID 1M-2M"| Shard2["🗄️ Shard 2<br/>Users 1,000,001 - 2,000,000"]
    App -->|"User ID 2M-3M"| Shard3["🗄️ Shard 3<br/>Users 2,000,001 - 3,000,000"]

    subgraph HASH["Hash-Based Sharding"]
        H["shard = hash(user_id) % num_shards"]
    end

    subgraph RANGE["Range-Based Sharding"]
        R["shard = user_id range<br/>(A-M → shard1, N-Z → shard2)"]
    end

    style App fill:#161b22,stroke:#d29922,color:#fff
    style Shard1 fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
    style Shard2 fill:#0d1117,stroke:#3fb950,color:#c9d1d9
    style Shard3 fill:#0d1117,stroke:#bc8cff,color:#c9d1d9
```

### Sharding Challenges

```mermaid
graph TD
    Sharding["🔀 Sharding"] --> C1["❌ Cross-shard JOINs<br/>Can't easily join data<br/>across shards"]
    Sharding --> C2["❌ Resharding<br/>Adding a new shard means<br/>redistributing data"]
    Sharding --> C3["❌ Hot shards<br/>Uneven data distribution<br/>(celebrity user on one shard)"]
    Sharding --> C4["❌ Transactions<br/>No ACID across shards"]
    Sharding --> C5["❌ Complexity<br/>Routing logic, shard map,<br/>operational overhead"]

    style Sharding fill:#161b22,stroke:#f85149,color:#fff
```

### When to Shard (Last Resort!)

```mermaid
flowchart TD
    Slow["🐌 Database is slow"] --> Q1{"Added<br/>indexes?"}
    Q1 -->|"No"| AddIndex["Add indexes first"]
    Q1 -->|"Yes"| Q2{"Added<br/>caching?"}
    Q2 -->|"No"| AddCache["Add Redis cache"]
    Q2 -->|"Yes"| Q3{"Added<br/>read replicas?"}
    Q3 -->|"No"| AddReplica["Add read replicas"]
    Q3 -->|"Yes"| Q4{"Optimized<br/>queries?"}
    Q4 -->|"No"| OptimizeQ["Use EXPLAIN, fix N+1"]
    Q4 -->|"Yes"| Q5{"Archived<br/>old data?"}
    Q5 -->|"No"| Archive["Move old data to archive DB"]
    Q5 -->|"Yes"| Shard["🔀 NOW consider sharding<br/>(last resort!)"]

    style Slow fill:#161b22,stroke:#f85149,color:#fff
    style Shard fill:#0d1117,stroke:#d29922,color:#c9d1d9
```

---

## 💾 Backups & Disaster Recovery

```mermaid
graph TB
    subgraph BACKUP_STRATEGY["Backup Strategy"]
        Full["📦 Full Backup<br/>Every Sunday at 2 AM<br/>Complete copy of everything"]
        Incremental["📝 Incremental Backup<br/>Every hour<br/>Only changes since last backup"]
        WAL["📜 WAL / Transaction Log<br/>Continuous<br/>Every single write recorded"]
    end

    subgraph RECOVERY["Recovery Options"]
        RestoreFull["Restore full backup<br/>(lose up to 1 week of data)"]
        RestoreIncremental["Restore full + incremental<br/>(lose up to 1 hour)"]
        PointInTime["Point-in-time recovery<br/>(lose almost nothing!)"]
    end

    Full --> RestoreFull
    Full --> RestoreIncremental
    Incremental --> RestoreIncremental
    WAL --> PointInTime

    subgraph STORAGE["Where to Store Backups"]
        Local["Same server ❌<br/>(server dies = backup dies)"]
        Remote["Different region ✅<br/>(S3, GCS, separate DC)"]
        Encrypted["Encrypted at rest 🔒<br/>(especially personal data)"]
    end

    style BACKUP_STRATEGY fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
    style RECOVERY fill:#0d1117,stroke:#3fb950,color:#c9d1d9
    style STORAGE fill:#0d1117,stroke:#d29922,color:#c9d1d9
```

### The Backup Golden Rule

```
A backup you've never restored is a backup you don't have.
```

> **Test your restores regularly.** Many companies discover their backups were broken only during an actual emergency.

### Backup Schedule Template

| Backup Type | Frequency | Retention | Storage |
|------------|-----------|-----------|---------|
| Full | Weekly | 4 weeks | Remote (different region) |
| Incremental | Hourly | 7 days | Remote |
| Transaction log (WAL) | Continuous | 3 days | Remote |
| Monthly archive | Monthly | 1 year | Cold storage (S3 Glacier) |

---

## 🔄 Database Migration Safety

```mermaid
flowchart TD
    Change["Need to change DB schema"] --> Q1{"Is it backward<br/>compatible?"}

    Q1 -->|"Yes (add column, add table)"| Safe["✅ Safe to deploy<br/>1. Run migration<br/>2. Deploy new code"]

    Q1 -->|"No (rename column,<br/>delete column, change type)"| Multi["⚠️ Multi-step migration needed"]

    Multi --> Step1["Step 1: Add new column<br/>(keep old column)"]
    Step1 --> Step2["Step 2: Deploy code that<br/>writes to BOTH columns"]
    Step2 --> Step3["Step 3: Backfill old data<br/>to new column"]
    Step3 --> Step4["Step 4: Deploy code that<br/>reads from new column only"]
    Step4 --> Step5["Step 5: Remove old column<br/>(after verification)"]

    style Change fill:#161b22,stroke:#58a6ff,color:#fff
    style Safe fill:#0d1117,stroke:#3fb950,color:#c9d1d9
    style Multi fill:#0d1117,stroke:#d29922,color:#c9d1d9
```

---

## ⚠️ Edge Cases & Gotchas

1. **The N+1 query problem** — Fetching a list of 100 orders, then for each order making a separate query for the customer = 101 queries. One query with a JOIN gets the same data.

2. **Missing database connection pooling** — Creating a new DB connection for every request is expensive (~50ms per connection). Use a pool that reuses connections.

3. **Unbounded queries** — `SELECT * FROM logs` on a table with 500 million rows will crash your DB. Always use LIMIT/pagination.

4. **Not using transactions for multi-step writes** — Transferring money: debit account A and credit account B should be atomic. If the process crashes after debiting but before crediting, money disappears.

5. **Choosing NoSQL because "it's faster"** — NoSQL isn't inherently faster. A well-indexed PostgreSQL query is as fast as MongoDB for most use cases. Choose based on data model, not speed myths.

---

## 🔗 Connected Topics

| Topic | Connection |
|-------|-----------|
| [Caching](05-caching.md) | Redis cache sits in front of DB to reduce load |
| [Scalability](03-scalability.md) | Replication and sharding are DB-level scaling |
| [Security](09-security.md) | Encryption at rest, access control, audit logging |
| [Performance](12-performance-optimization.md) | Query optimization, indexing, N+1 prevention |
| [Latency](08-latency.md) | DB queries are a major latency source |

---

**← Previous:** [6. CDN, Page Speed & SEO](06-cdn-pagespeed-seo.md) | **Next →** [8. Latency](08-latency.md)
