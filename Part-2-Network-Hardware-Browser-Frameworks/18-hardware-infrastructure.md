# 🖥️ 18. Hardware & Infrastructure Fundamentals

> **Understanding hardware is like understanding the engine of a car — you don't need to build one, but knowing how it works helps you drive better and diagnose problems.**

---

## 🏗️ Server Architecture — What's Inside

```mermaid
graph TB
    subgraph SERVER["🖥️ A Server"]
        CPU["🧠 CPU (Processor)<br/>The brain — executes code<br/>More cores = more parallel work"]
        RAM["💾 RAM (Memory)<br/>Fast, temporary storage<br/>Where running programs live<br/>Lost on restart"]
        DISK["💿 Disk (Storage)<br/>Permanent storage<br/>SSD: fast, HDD: cheap<br/>Survives restarts"]
        NIC["🌐 Network Interface<br/>Connects to network<br/>Bandwidth = throughput capacity"]
    end

    CPU -->|"Reads/writes"| RAM
    RAM -->|"Loads from / saves to"| DISK
    NIC -->|"Sends/receives data"| Network["🌐 Network"]

    style SERVER fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
```

### The Memory Hierarchy — Speed vs Size

```mermaid
graph TB
    L1["L1 Cache (~1ns)<br/>32-64 KB<br/>Fastest, smallest"]
    L2["L2 Cache (~4ns)<br/>256 KB - 1 MB"]
    L3["L3 Cache (~10ns)<br/>4-64 MB"]
    RAM_H["RAM (~100ns)<br/>8-512 GB"]
    SSD_H["SSD (~100μs)<br/>256 GB - 4 TB"]
    HDD_H["HDD (~10ms)<br/>1-16 TB"]
    NET["Network (~10-200ms)<br/>∞"]

    L1 --> L2 --> L3 --> RAM_H --> SSD_H --> HDD_H --> NET

    style L1 fill:#0d1117,stroke:#3fb950,color:#c9d1d9
    style L2 fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
    style L3 fill:#0d1117,stroke:#d29922,color:#c9d1d9
    style RAM_H fill:#0d1117,stroke:#f85149,color:#c9d1d9
    style SSD_H fill:#0d1117,stroke:#bc8cff,color:#c9d1d9
    style HDD_H fill:#0d1117,stroke:#f85149,color:#c9d1d9
```

### Latency Numbers Every Developer Should Know

| Operation | Time | Analogy |
|-----------|------|---------|
| L1 cache reference | 1 ns | Blink of an eye |
| L2 cache reference | 4 ns | |
| RAM reference | 100 ns | Grabbing a book from your desk |
| SSD random read | 100 μs | Walking to the bookshelf |
| HDD random read | 10 ms | Driving to the library |
| Send packet SF→NYC→SF | 40 ms | Mailing a letter cross-country |
| SSD read 1 MB | 1 ms | |
| Network read 1 MB | 10 ms | |

---

## 📦 Containers vs Virtual Machines

```mermaid
graph TB
    subgraph VM["Virtual Machines"]
        VM_Host["🖥️ Host OS"]
        VM_Hyper["Hypervisor (VMware, KVM)"]
        VM1["VM 1<br/>Full Guest OS<br/>App 1"]
        VM2["VM 2<br/>Full Guest OS<br/>App 2"]
        VM3["VM 3<br/>Full Guest OS<br/>App 3"]
        VM_Host --> VM_Hyper --> VM1 & VM2 & VM3
    end

    subgraph CONTAINER["Containers (Docker)"]
        C_Host["🖥️ Host OS"]
        C_Engine["Docker Engine"]
        C1["Container 1<br/>App 1<br/>(shared kernel)"]
        C2["Container 2<br/>App 2<br/>(shared kernel)"]
        C3["Container 3<br/>App 3<br/>(shared kernel)"]
        C_Host --> C_Engine --> C1 & C2 & C3
    end

    style VM fill:#0d1117,stroke:#f85149,color:#c9d1d9
    style CONTAINER fill:#0d1117,stroke:#3fb950,color:#c9d1d9
```

| Aspect | Virtual Machine | Container |
|--------|----------------|-----------|
| **Size** | GBs (full OS per VM) | MBs (shared OS kernel) |
| **Start time** | Minutes | Seconds |
| **Isolation** | Strong (full OS boundary) | Good (shared kernel) |
| **Overhead** | High (each VM runs full OS) | Low (shared kernel) |
| **Use case** | Different OS needs, strong isolation | Microservices, CI/CD, dev environments |

---

## ☸️ Container Orchestration (Kubernetes)

```mermaid
graph TB
    subgraph K8S["☸️ Kubernetes Cluster"]
        Master["🎮 Control Plane<br/>• API Server<br/>• Scheduler<br/>• Controller Manager<br/>• etcd (config store)"]

        subgraph Node1["Worker Node 1"]
            P1["Pod: App v2<br/>Container"]
            P2["Pod: App v2<br/>Container"]
        end

        subgraph Node2["Worker Node 2"]
            P3["Pod: App v2<br/>Container"]
            P4["Pod: API Gateway<br/>Container"]
        end

        subgraph Node3["Worker Node 3"]
            P5["Pod: App v2<br/>Container"]
            P6["Pod: Worker<br/>Container"]
        end

        Master --> Node1 & Node2 & Node3
    end

    LB["⚖️ Load Balancer"] --> P1 & P2 & P3 & P5

    style K8S fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
    style Master fill:#0d1117,stroke:#d29922,color:#c9d1d9
```

### What Kubernetes Does For You

| Feature | Without K8s | With K8s |
|---------|------------|----------|
| **Scaling** | Manual SSH, deploy, configure LB | `kubectl scale --replicas=10` |
| **Self-healing** | Container crashes = manual restart | Auto-restart + reschedule |
| **Rolling updates** | Downtime during deploy | Zero-downtime rolling update |
| **Service discovery** | Hardcoded IPs | DNS-based service names |
| **Secret management** | Env files on servers | K8s Secrets (encrypted) |

---

## ☁️ Cloud Infrastructure

```mermaid
graph TB
    subgraph CLOUD["☁️ Cloud Provider (AWS / GCP / Azure)"]
        subgraph Region1["🌍 Region: US-East"]
            subgraph AZ1["AZ 1 (Data Center 1)"]
                S1["🖥️ Server 1"]
                S2["🖥️ Server 2"]
            end
            subgraph AZ2["AZ 2 (Data Center 2)"]
                S3["🖥️ Server 3"]
                S4["🖥️ Server 4"]
            end
        end

        subgraph Region2["🌍 Region: EU-West"]
            subgraph AZ3["AZ 1"]
                S5["🖥️ Server 5"]
            end
        end
    end

    Note1["AZ = Availability Zone<br/>= separate data center<br/>(different power, cooling, network)<br/>If one AZ goes down, others continue"]

    style CLOUD fill:#0d1117,stroke:#58a6ff,color:#c9d1d9
    style Region1 fill:#0d1117,stroke:#3fb950,color:#c9d1d9
    style Region2 fill:#0d1117,stroke:#d29922,color:#c9d1d9
```

### Cloud Service Models

```mermaid
graph TB
    subgraph IaaS["IaaS — Infrastructure as a Service"]
        I1["You manage: OS, runtime, app, data"]
        I2["Provider manages: hardware, network, storage"]
        I3["Examples: AWS EC2, GCP Compute Engine"]
    end

    subgraph PaaS["PaaS — Platform as a Service"]
        P1["You manage: app code, data"]
        P2["Provider manages: OS, runtime, hardware"]
        P3["Examples: Heroku, Google App Engine, Vercel"]
    end

    subgraph SaaS["SaaS — Software as a Service"]
        S1["You manage: data, configuration"]
        S2["Provider manages: everything else"]
        S3["Examples: Gmail, Slack, Salesforce"]
    end

    MoreControl["← More Control"] -.-> IaaS
    MoreManaged["More Managed →"] -.-> SaaS

    style IaaS fill:#0d1117,stroke:#f85149,color:#c9d1d9
    style PaaS fill:#0d1117,stroke:#d29922,color:#c9d1d9
    style SaaS fill:#0d1117,stroke:#3fb950,color:#c9d1d9
```

---

## ⚠️ Edge Cases & Gotchas

1. **"The cloud is just someone else's computer"** — True, but with massive scale, redundancy, and managed services you could never build yourself cost-effectively.

2. **Noisy neighbors** — On shared cloud hardware, another tenant's heavy workload can impact your performance. Use dedicated instances for critical workloads.

3. **Region selection matters** — Deploy close to your users for low latency. Also consider data residency laws (GDPR may require EU data in EU regions).

4. **Spot/preemptible instances** — Up to 90% cheaper but can be terminated anytime. Great for batch jobs, terrible for web servers.

5. **Vendor lock-in** — Using proprietary cloud services (AWS Lambda, DynamoDB) makes it hard to switch providers. Use containers and open-source tools where possible for portability.

---

## 🔗 Connected Topics

| Topic | Connection |
|-------|-----------|
| [Scalability](../Part-1-Architecture-Scalability-Operations/03-scalability.md) | Hardware determines scaling limits |
| [Networking](17-networking-fundamentals.md) | Network interface connects to the internet |
| [Latency](../Part-1-Architecture-Scalability-Operations/08-latency.md) | Memory hierarchy directly impacts latency |
| [CI/CD](22-cicd-pipeline.md) | Containers are deployed through CI/CD |
| [Architecture](../Part-1-Architecture-Scalability-Operations/02-architecture-patterns.md) | Serverless vs containers vs VMs |

---

**← Previous:** [17. Networking Fundamentals](17-networking-fundamentals.md) | **Next →** [19. Browser Internals](19-browser-internals.md)
