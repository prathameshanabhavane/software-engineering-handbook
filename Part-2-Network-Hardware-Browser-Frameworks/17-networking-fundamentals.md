# 🌐 17. Networking Fundamentals — How Data Travels

> **The internet is like a postal system. TCP is registered mail (guaranteed delivery). UDP is postcards (fast, no guarantee). HTTP is the language written on the letters.**

---

## 🏗️ The OSI Model vs TCP/IP — Layers of Communication

```mermaid
graph TB
    subgraph OSI["OSI Model (7 Layers)"]
        L7["7. Application<br/>(HTTP, FTP, DNS, WebSocket)"]
        L6["6. Presentation<br/>(SSL/TLS, encryption, compression)"]
        L5["5. Session<br/>(Connection management)"]
        L4["4. Transport<br/>(TCP, UDP)"]
        L3["3. Network<br/>(IP, routing)"]
        L2["2. Data Link<br/>(Ethernet, WiFi, MAC)"]
        L1["1. Physical<br/>(Cables, radio waves, fiber)"]
    end

    subgraph TCPIP["TCP/IP Model (4 Layers)"]
        T4["Application<br/>(HTTP, DNS, FTP)"]
        T3["Transport<br/>(TCP, UDP)"]
        T2["Internet<br/>(IP, ICMP)"]
        T1["Network Access<br/>(Ethernet, WiFi)"]
    end

    L7 & L6 & L5 -.-> T4
    L4 -.-> T3
    L3 -.-> T2
    L2 & L1 -.-> T1

    style OSI fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
    style TCPIP fill:#0d1117,stroke:#3fb950,color:#c9d1d9
```

### Data Encapsulation — How Data Wraps at Each Layer

```mermaid
graph LR
    App["Application Data:<br/>'Hello World'"]
    App -->|"+TCP header"| Segment["TCP Segment"]
    Segment -->|"+IP header"| Packet["IP Packet"]
    Packet -->|"+Ethernet header"| Frame["Ethernet Frame"]
    Frame -->|"Convert to"| Bits["⚡ Electrical signals / light / radio"]

    style App fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
    style Bits fill:#0d1117,stroke:#3fb950,color:#c9d1d9
```

---

## 📬 TCP vs UDP

```mermaid
graph TB
    subgraph TCP["📬 TCP — Registered Mail"]
        TC1["✅ Reliable delivery (guaranteed)"]
        TC2["✅ Ordered (packets arrive in sequence)"]
        TC3["✅ Error checking + retransmission"]
        TC4["❌ Slower (connection setup, acknowledgements)"]
        TC5["Use for: HTTP, email, file transfer, databases"]
    end

    subgraph UDP["📮 UDP — Postcards"]
        UC1["❌ No delivery guarantee"]
        UC2["❌ No ordering guarantee"]
        UC3["❌ No retransmission"]
        UC4["✅ Faster (no connection setup, no waiting)"]
        UC5["Use for: Video streaming, gaming, DNS, VoIP"]
    end

    style TCP fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
    style UDP fill:#0d1117,stroke:#3fb950,color:#c9d1d9
```

---

## 🔐 TLS/SSL — Securing Communication

```mermaid
sequenceDiagram
    participant Client as 🌐 Client
    participant Server as 🖥️ Server

    Note over Client,Server: TLS 1.3 Handshake (1 round trip!)

    Client->>Server: ClientHello<br/>• Supported cipher suites<br/>• Client key share (ECDHE)<br/>• SNI (which domain)

    Server-->>Client: ServerHello + Certificate + Finished<br/>• Selected cipher suite<br/>• Server key share<br/>• Certificate chain<br/>• Encrypted extensions

    Note over Client: Verify certificate:<br/>1. Signed by trusted CA?<br/>2. Not expired?<br/>3. Domain matches?

    Client->>Server: Finished + Application Data<br/>(can start sending data immediately!)

    Note over Client,Server: 🔒 All data now encrypted<br/>Using shared secret derived from key exchange
```

### Certificate Chain of Trust

```mermaid
graph TB
    Root["🏛️ Root CA<br/>(DigiCert, Let's Encrypt, etc.)<br/>Pre-installed in browser/OS"]
    Root -->|"Signs"| Intermediate["📜 Intermediate CA"]
    Intermediate -->|"Signs"| Leaf["📄 Your Certificate<br/>(example.com)"]

    Browser["🌐 Browser"] -->|"Validates chain"| Leaf
    Leaf -->|"Chain leads to"| Root
    Browser -->|"Trusts?"| Root

    style Root fill:#0d1117,stroke:#3fb950,color:#c9d1d9
    style Leaf fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
```

---

## 📡 HTTP Versions Compared

```mermaid
graph TB
    subgraph HTTP1["HTTP/1.1"]
        H1_1["One request per connection"]
        H1_2["Text-based headers"]
        H1_3["6 parallel connections per domain (limit)"]
        H1_4["Head-of-line blocking"]
    end

    subgraph HTTP2["HTTP/2"]
        H2_1["Multiplexing: many requests per connection"]
        H2_2["Binary protocol (faster parsing)"]
        H2_3["Header compression (HPACK)"]
        H2_4["Server push (deprecated)"]
        H2_5["Stream prioritization"]
    end

    subgraph HTTP3["HTTP/3"]
        H3_1["Uses QUIC (UDP-based, not TCP)"]
        H3_2["No head-of-line blocking"]
        H3_3["0-RTT connection resumption"]
        H3_4["Built-in encryption"]
        H3_5["Better for mobile (connection migration)"]
    end

    style HTTP1 fill:#0d1117,stroke:#f85149,color:#c9d1d9
    style HTTP2 fill:#0d1117,stroke:#d29922,color:#c9d1d9
    style HTTP3 fill:#0d1117,stroke:#3fb950,color:#c9d1d9
```

### HTTP/2 Multiplexing vs HTTP/1.1

```mermaid
graph LR
    subgraph H1["HTTP/1.1 — Sequential per connection"]
        C1["Conn 1: CSS ──────────────────"]
        C2["Conn 2: JS ─────────────"]
        C3["Conn 3: img1 ────────────────────"]
        C4["Conn 4: img2 ──────────"]
        C5["Conn 5: font ───────"]
        C6["Conn 6: img3 ────────────"]
    end

    subgraph H2["HTTP/2 — Multiplexed on ONE connection"]
        M1["One connection: CSS|JS|img1|img2|font|img3<br/>(all interleaved simultaneously)"]
    end

    style H1 fill:#0d1117,stroke:#f85149,color:#c9d1d9
    style H2 fill:#0d1117,stroke:#3fb950,color:#c9d1d9
```

---

## 🔌 WebSocket — Full-Duplex Communication

```mermaid
sequenceDiagram
    participant Client as 🌐 Client
    participant Server as 🖥️ Server

    Note over Client,Server: HTTP Upgrade Handshake
    Client->>Server: GET /chat HTTP/1.1<br/>Upgrade: websocket<br/>Connection: Upgrade

    Server-->>Client: HTTP/1.1 101 Switching Protocols<br/>Upgrade: websocket

    Note over Client,Server: ✅ WebSocket connection open!<br/>Full-duplex: both can send anytime

    Client->>Server: {"type": "message", "text": "Hello!"}
    Server-->>Client: {"type": "message", "text": "Hi there!"}
    Server-->>Client: {"type": "notification", "text": "New message from Alice"}

    Note over Client,Server: No polling needed!<br/>Server pushes updates instantly
```

### When to Use WebSocket vs HTTP

| Feature | HTTP | WebSocket |
|---------|------|-----------|
| Direction | Client → Server (request/response) | Both directions anytime |
| Connection | New per request (or keep-alive) | Persistent |
| Overhead | Headers on every request | Minimal per message |
| Best for | CRUD APIs, page loads | Chat, live updates, gaming, collaborative editing |

---

## 🏠 REST vs GraphQL vs gRPC

```mermaid
graph TB
    subgraph REST["REST"]
        R1["Multiple endpoints<br/>GET /users, POST /orders"]
        R2["Fixed response shape"]
        R3["Over-fetching common<br/>(get entire user when<br/>you only need name)"]
        R4["Simple, widely supported"]
    end

    subgraph GRAPHQL["GraphQL"]
        G1["Single endpoint<br/>POST /graphql"]
        G2["Client specifies exact fields<br/>query { user { name, email } }"]
        G3["No over/under-fetching"]
        G4["More complex, needs schema"]
    end

    subgraph GRPC["gRPC"]
        GR1["Binary protocol (Protocol Buffers)"]
        GR2["Very fast, low overhead"]
        GR3["Bidirectional streaming"]
        GR4["Best for service-to-service"]
    end

    style REST fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
    style GRAPHQL fill:#0d1117,stroke:#bc8cff,color:#c9d1d9
    style GRPC fill:#0d1117,stroke:#3fb950,color:#c9d1d9
```

---

## ⚠️ Edge Cases & Gotchas

1. **DNS propagation** — When you change DNS records, it can take up to 48 hours for all caches worldwide to update. Plan DNS changes well ahead of time.

2. **Mixed content** — Loading HTTP resources on an HTTPS page is blocked by browsers. All resources must use HTTPS.

3. **CORS** — Cross-Origin Resource Sharing blocks requests from different domains by default. Your API needs to send proper CORS headers.

4. **TCP slow start** — New TCP connections start slow and ramp up speed. This is why connection reuse (HTTP/2, keep-alive) matters.

5. **MTU and fragmentation** — IP packets larger than ~1500 bytes get fragmented, adding overhead. Keep payloads reasonable.

---

## 🔗 Connected Topics

| Topic | Connection |
|-------|-----------|
| [URL Journey](16-url-to-page-journey.md) | DNS, TCP, TLS are steps in that journey |
| [Security](../Part-1-Architecture-Scalability-Operations/09-security.md) | TLS encryption, HTTPS |
| [Latency](../Part-1-Architecture-Scalability-Operations/08-latency.md) | Network protocols directly impact latency |
| [Load Balancers](../Part-1-Architecture-Scalability-Operations/04-load-balancers.md) | L4 vs L7 load balancing |
| [Hardware](18-hardware-infrastructure.md) | Physical layer that carries these protocols |

---

**← Previous:** [16. URL to Page Journey](16-url-to-page-journey.md) | **Next →** [18. Hardware & Infrastructure](18-hardware-infrastructure.md)
