# Domain 1: Designing and Planning a Cloud Solution Architecture

> **Exam Weight: ~24%** — The largest domain. Expect 12–15 questions covering requirements gathering, solution design, and architectural trade-offs.

---

## 1.1 Domain Overview

This domain tests your ability to translate business and technical requirements into a well-architected GCP solution. You must demonstrate proficiency in selecting the right services, designing for scale and availability, and planning migrations.

**Key skills tested:**
- Mapping business requirements to technical solutions
- Applying the Google Cloud Well-Architected Framework
- Selecting compute, storage, and networking services
- Designing multi-region and hybrid architectures
- Planning AI/ML integration
- Migration strategy selection

---

## 1.2 Google Cloud Well-Architected Framework (2025)

Google's Well-Architected Framework has **six pillars**. Exam questions frequently ask you to identify which pillar a given design decision addresses.

### Pillar 1: Operational Excellence
- **Definition:** Ability to run, monitor, and improve systems efficiently
- Automate operational tasks with Cloud Build, Cloud Deploy, and Deployment Manager
- Use runbooks, SRE practices, and incident response procedures
- Implement Infrastructure as Code (Terraform, Config Connector)
- Enable structured logging and distributed tracing
- **Exam signal:** Questions about reducing operational overhead → look for managed services (Cloud Run, Cloud SQL, Spanner)

### Pillar 2: Security
- **Definition:** Protect data, systems, and assets; meet compliance requirements
- Apply least-privilege IAM; use service accounts with minimal permissions
- Encrypt data at rest (CMEK/CSEK) and in transit (TLS, mTLS)
- Use VPC Service Controls to create security perimeters
- Implement Defense in Depth: Cloud Armor + WAF + private endpoints
- **Exam signal:** "How do you ensure only authorized services access BigQuery?" → VPC Service Controls + authorized views

### Pillar 3: Reliability
- **Definition:** System performs its intended function correctly and consistently
- Design for failure: use multi-zone deployments as a minimum
- Implement health checks, circuit breakers, and retry logic with exponential backoff
- Use global load balancers with backend health checks
- **Exam signal:** "99.99% availability" → multi-region active-active architecture

### Pillar 4: Performance Efficiency
- **Definition:** Use compute resources efficiently to meet system requirements
- Right-size VMs; use custom machine types for unique workloads
- Cache aggressively: Cloud CDN, Memorystore (Redis/Valkey)
- Choose the right storage tier and database for access patterns
- **Exam signal:** Latency-sensitive workloads → Cloud Spanner, Bigtable, or Memorystore

### Pillar 5: Cost Optimization
- **Definition:** Avoid unnecessary costs; understand and control spending
- Use Committed Use Discounts (CUDs) for predictable workloads
- Preemptible/Spot VMs for batch and fault-tolerant jobs
- Implement lifecycle policies on Cloud Storage buckets
- Use budget alerts and billing exports to BigQuery
- **Exam signal:** "Reduce cost of stateless batch processing" → Spot VMs on GKE

### Pillar 6: Sustainability
- **Definition:** Minimize environmental impact of cloud workloads
- Choose regions with high renewable energy percentage (carbon-free energy score)
- Use carbon footprint reporting in the Google Cloud console
- Right-size and schedule workloads to reduce idle resource waste
- Prefer serverless (Cloud Run, Cloud Functions) to avoid over-provisioning
- **Exam signal:** New in 2025 — expect 1–2 questions about reducing carbon footprint

---

## 1.3 Business and Technical Requirements Analysis

### Business Requirements Mapping
| Business Need | GCP Architecture Signal |
|---|---|
| Reduce TCO | Managed services, CUDs, auto-scaling |
| Time to market | Serverless (Cloud Run), managed databases |
| Regulatory compliance | Regional data residency, CMEK, audit logs |
| Global customer base | Multi-region deployment, global load balancer |
| Legacy modernization | Migrate for Compute Engine → re-platform to GKE |
| Data analytics | BigQuery + Looker + Dataflow pipeline |
| Real-time processing | Pub/Sub + Dataflow + Bigtable |

### Technical Requirements Framework
When analyzing technical requirements, capture:
1. **Availability target** (99.9%, 99.99%, 99.999%) → drives multi-zone vs multi-region
2. **RTO / RPO** → drives backup strategy and DR architecture
3. **Data volume and velocity** → drives database and pipeline selection
4. **Latency requirements** → drives caching strategy and region selection
5. **Security/compliance requirements** → drives encryption, audit, and access control design
6. **Team expertise** → drives managed vs self-managed service selection

---

## 1.4 High Availability Patterns

### Availability Tier Design
| Availability | Architecture | Services |
|---|---|---|
| 99.9% (43.8 min/month downtime) | Single region, multi-zone | Regional MIG, Cloud SQL HA |
| 99.99% (4.4 min/month) | Multi-region active-passive | Global LB + failover routing |
| 99.999% (26 sec/month) | Multi-region active-active | Spanner, Bigtable multi-cluster |

### GCP High Availability Constructs
- **Managed Instance Groups (MIGs):** Regional MIGs span multiple zones; use autohealing with health checks
- **Cloud SQL HA:** Primary + standby in separate zones; automatic failover in ~60 seconds
- **Cloud Spanner:** Multi-region configurations provide 99.999% SLA; uses Paxos replication
- **Cloud Storage:** Dual-region and multi-region buckets for geo-redundancy
- **GKE Regional Clusters:** Control plane and nodes spread across 3 zones automatically

### Load Balancing for High Availability
```
Global HTTP(S) Load Balancer
├── Frontend: Anycast IP (global)
├── URL Map → Backend Services
│   ├── Backend 1: us-central1 MIG
│   ├── Backend 2: europe-west1 MIG
│   └── Backend 3: asia-east1 MIG
└── Cloud CDN (optional, for static content)
```

**Load Balancer Selection Guide:**
| Type | Use Case | Layer |
|---|---|---|
| Global External HTTP(S) LB | Internet-facing web apps, multi-region | L7 |
| Regional External HTTP(S) LB | Single-region web apps | L7 |
| Global External TCP/SSL Proxy | Non-HTTP TCP, SSL offload | L4 |
| Internal HTTP(S) LB | Microservices, internal APIs | L7 |
| Internal TCP/UDP LB | Internal VMs, non-HTTP | L4 |
| Network LB (pass-through) | Ultra-low latency, preserve source IP | L4 |

---

## 1.5 Scalability Patterns

### Horizontal vs Vertical Scaling
- **Horizontal (scale out):** Add more instances; preferred for stateless workloads
  - GKE HPA (Horizontal Pod Autoscaler) scales pods based on CPU/memory/custom metrics
  - MIG autoscaling based on CPU, load balancing serving capacity, or Cloud Monitoring metrics
- **Vertical (scale up):** Increase instance size; used for stateful workloads (databases)
  - GKE VPA (Vertical Pod Autoscaler) adjusts pod resource requests
  - Cloud SQL can scale compute vertically with minimal downtime

### Stateless vs Stateful Scaling
- **Stateless services:** Cloud Run (0→N auto-scaling), GKE Deployments, App Engine
- **Stateful services:** GKE StatefulSets with persistent disks, Spanner (add nodes), Bigtable (add nodes)

### Queue-Based Load Leveling
```
Client → Pub/Sub Topic → Subscription → Worker VMs/Cloud Run
                                     → Dead Letter Topic (failed messages)
```
Use Pub/Sub to decouple producers from consumers and handle traffic spikes.

---

## 1.6 Compute Selection Framework

### Decision Tree
```
Is the workload containerized?
├── YES → Is it event-driven/stateless?
│   ├── YES → Cloud Run (fully managed, HTTP/gRPC)
│   └── NO  → GKE (full Kubernetes control)
└── NO  → Does it need a specific OS or GPU?
    ├── YES → Compute Engine (GCE)
    └── NO  → Is it event-triggered code?
        ├── YES → Cloud Functions (gen 2)
        └── NO  → App Engine (Flex or Standard)
```

### Compute Service Comparison
| Service | Best For | Scaling | Pricing Model |
|---|---|---|---|
| **Compute Engine** | Custom OS, GPU/TPU, lift-and-shift | Manual or MIG autoscale | Per-second (min 1 min) |
| **GKE Standard** | Microservices, full K8s control | HPA/VPA/Cluster Autoscaler | Node pool VMs |
| **GKE Autopilot** | K8s without node management | Automatic | Per-pod resource |
| **Cloud Run** | HTTP/gRPC stateless services | 0→1000 instances | Per request + CPU/memory |
| **Cloud Functions** | Event-driven, short tasks (<9 min) | Automatic | Per invocation |
| **App Engine Standard** | Web apps, quick deploy | Automatic | Instance-hours |
| **App Engine Flexible** | Custom runtimes, long-running | Manual + scheduled | Per vCPU/memory-hour |

### GKE Autopilot vs Standard
| Aspect | Standard | Autopilot |
|---|---|---|
| Node management | You manage nodes | Google manages nodes |
| Node pools | Configurable | Not configurable |
| Pricing | Pay for nodes | Pay for pods |
| Security posture | Configurable | Hardened by default |
| Best for | Custom node requirements | Most production workloads |

> **Exam Trap:** Autopilot does not support privileged containers or DaemonSets. If a question mentions these, the answer is GKE Standard.

---

## 1.7 Storage Selection Framework

### Decision Criteria
1. **Data structure:** Structured → relational DB or BigQuery; Semi-structured → Firestore/Bigtable; Unstructured → Cloud Storage
2. **Access pattern:** OLTP → Cloud SQL/Spanner; OLAP → BigQuery; Key-value → Bigtable/Firestore; Cache → Memorystore
3. **Scale:** Global scale → Spanner/Bigtable; Regional → Cloud SQL; Data lake → Cloud Storage + BigQuery

### Storage Services Quick Reference
| Service | Type | Scale | Latency | Best For |
|---|---|---|---|---|
| **Cloud Storage** | Object | Unlimited | ms–s | Blobs, backups, data lake |
| **Cloud SQL** | Relational | 96 vCPU, 624 GB RAM | ms | OLTP, PostgreSQL/MySQL/SQL Server |
| **Cloud Spanner** | Relational | Global, unlimited | ms | Global OLTP, financial, strong consistency |
| **Bigtable** | Wide-column NoSQL | Petabytes | <10 ms | Time-series, IoT, HBase-compatible |
| **Firestore** | Document NoSQL | Millions of docs | ms | Mobile/web apps, hierarchical data |
| **BigQuery** | Data warehouse | Petabytes | seconds | Analytics, SQL queries, ML |
| **Memorystore** | In-memory cache | Up to 300 GB | <1 ms | Session cache, leaderboards |
| **AlloyDB** | PostgreSQL-compatible | Regional | ms | High-performance PostgreSQL workloads |

### Cloud Storage Class Selection
| Class | Access Frequency | Min Storage Duration | Use Case |
|---|---|---|---|
| Standard | Frequent (daily) | None | Hot data, serving |
| Nearline | Monthly | 30 days | Backups, DR |
| Coldline | Quarterly | 90 days | Long-term backups |
| Archive | Yearly | 365 days | Compliance archives |

> **Exam Tip:** Storage class does NOT affect latency or throughput — only cost. Data retrieval is equally fast from all classes.

---

## 1.8 Network Design

### VPC Architecture Patterns

**Shared VPC (Recommended for enterprises):**
```
Host Project (VPC owner)
├── Subnet: 10.0.0.0/24 (us-central1) → shared to Service Project A
├── Subnet: 10.0.1.0/24 (us-east1)    → shared to Service Project B
└── Firewall rules, Cloud NAT, Cloud Router
```

**VPC Peering:**
- Non-transitive (A↔B, B↔C does NOT mean A↔C)
- Routes are not shared transitively
- Use for connecting separate business units or vendor networks

**Private Service Connect:**
- Access Google services (e.g., BigQuery, Cloud Storage) via a private endpoint in your VPC
- Preferred over VPC peering for accessing Google managed services

### Network Design Principles
- **Subnet strategy:** Use small subnets; avoid large flat networks (security and blast radius)
- **Cloud NAT:** Allow private VMs to access the internet without public IPs
- **Cloud Router:** BGP routing for Cloud Interconnect and Cloud VPN
- **Private Google Access:** Enable to allow VMs without public IPs to reach Google APIs
- **DNS:** Use Cloud DNS for internal resolution; use DNS peering for hybrid environments

### Connectivity Options
| Option | Bandwidth | Latency | Use Case |
|---|---|---|---|
| Cloud VPN (HA VPN) | 3 Gbps/tunnel | Variable | Encrypted, cost-effective |
| Dedicated Interconnect | 10/100 Gbps | Low, predictable | High-bandwidth, low-latency |
| Partner Interconnect | 50 Mbps–10 Gbps | Low | When dedicated not available |
| Cross-Cloud Interconnect | 10/100 Gbps | Low | Direct connect to AWS/Azure |

---

## 1.9 Multi-Region and Multi-Cloud Design

### Multi-Region Patterns
| Pattern | Description | Trade-off |
|---|---|---|
| **Active-Active** | Traffic served from multiple regions simultaneously | Complex data sync; highest availability |
| **Active-Passive** | One region active, other on standby | Simpler; failover lag |
| **Pilot Light** | Minimal passive region (core infra only) | Cheapest DR; slowest failover |

### Multi-Cloud Considerations (2025 Exam Focus)
- **Anthos (now GKE Enterprise):** Deploy and manage Kubernetes clusters across GCP, on-premises, AWS, and Azure
- **BigQuery Omni:** Query data in AWS S3 or Azure Blob without moving data
- **Vertex AI:** Train on GCP, deploy inference endpoints anywhere
- **Reasons for multi-cloud:** Avoid vendor lock-in, regulatory data residency, leverage best-of-breed services

### Data Residency and Sovereignty
- Use **Organization Policies** to restrict resource creation to specific regions
- Use **BigQuery dataset location** to pin data to specific geographic areas
- Cloud Storage **dual-region** options: us-central1 + us-east1, or eu configurations

---

## 1.10 AI/ML Integration Planning

### When to Use Each ML Approach
| Approach | When to Use | GCP Service |
|---|---|---|
| Pre-built API | No ML expertise, standard use case | Vision AI, Natural Language AI, Translation AI, Speech-to-Text |
| AutoML | Moderate data, custom model, no ML code | Vertex AI AutoML (Tables, Vision, Text, Video) |
| Custom training | Large data, unique requirements, full control | Vertex AI Training + custom containers |
| Foundation models | LLM/generative AI use cases | Vertex AI Gemini API, Model Garden |

### Vertex AI Architecture
```
Data Ingestion (BigQuery / Cloud Storage)
    ↓
Feature Engineering (Dataflow / BigQuery ML)
    ↓
Model Training (Vertex AI Training / AutoML)
    ↓
Model Registry (Vertex AI Model Registry)
    ↓
Endpoint Deployment (Vertex AI Prediction)
    ↓
Monitoring (Vertex AI Model Monitoring)
```

### MLOps Considerations
- **Vertex AI Pipelines:** Orchestrate ML workflows (based on KFP or TFX)
- **Feature Store:** Centralized feature management for training and serving consistency
- **Explainable AI:** Built into Vertex AI; required for regulated industries
- **Data labeling:** Vertex AI Data Labeling Service

---

## 1.11 Migration Planning

### Migration Strategies (The 6 Rs)
| Strategy | Description | Effort | Risk |
|---|---|---|---|
| **Retire** | Decommission unused apps | Low | None |
| **Retain** | Keep on-premises (for now) | None | None |
| **Rehost (Lift & Shift)** | Move VMs as-is to GCE | Low | Low |
| **Replatform** | Minor modifications for cloud benefits (Cloud SQL, GKE) | Medium | Medium |
| **Repurchase** | Switch to SaaS (e.g., move to Google Workspace) | Medium | Medium |
| **Refactor/Re-architect** | Redesign for cloud-native (microservices, serverless) | High | High |

### Migration Wave Planning
1. **Discovery:** Use Migrate for Compute Engine discovery agent, or StratoZone (now part of Google Cloud Migration Center)
2. **Assessment:** Migration Center analyzes VMs, databases, and dependencies
3. **Pilot wave:** Migrate low-risk, non-production workloads first
4. **Production waves:** Migrate in dependency order; use cutover windows

### Key Migration Tools
| Tool | Purpose |
|---|---|
| **Migrate for Compute Engine** | VM lift-and-shift from VMware/AWS/Azure/Hyper-V |
| **Database Migration Service (DMS)** | Migrate MySQL, PostgreSQL, Oracle to Cloud SQL/AlloyDB/Spanner |
| **BigQuery Data Transfer Service** | Move data from Teradata, Redshift, S3, SaaS apps |
| **Storage Transfer Service** | Move data from S3, Azure Blob, HTTP sources, on-premises |
| **Transfer Appliance** | Physical device for large (>10 TB) offline data transfers |

### On-Premises Integration Patterns
- **Hybrid connectivity:** HA VPN (encrypted, lower bandwidth) or Dedicated Interconnect (higher bandwidth)
- **Extending AD:** Use AD on GCE VMs, or use Google Cloud Directory Sync (GCDS)
- **Anthos Config Management:** Consistent policy enforcement across GCP and on-premises
- **Pub/Sub with on-prem:** Use VPN/Interconnect + Pub/Sub for event streaming

---

## 1.12 Common Exam Question Patterns and Traps

### Pattern 1: Service Selection
**Question stem:** "A company needs a globally consistent, highly available relational database..."
**Answer:** Cloud Spanner (not Cloud SQL — Cloud SQL is regional)

### Pattern 2: Migration Strategy
**Question stem:** "Minimize migration effort and cost for existing VMs..."
**Answer:** Migrate for Compute Engine (rehost/lift-and-shift)

### Pattern 3: Event-Driven Architecture
**Question stem:** "Process uploaded files asynchronously with unpredictable volume..."
**Answer:** Cloud Storage trigger → Pub/Sub → Cloud Run or Cloud Functions

### Pattern 4: Availability Design
**Question stem:** "Design for 99.99% availability..."
**Answer:** Multi-region deployment with global load balancer (not just multi-zone)

### Common Traps
- **Cloud SQL ≠ global:** Cloud SQL is regional; for global relational DB → Spanner
- **VPC peering is non-transitive:** Cannot route A→B→C; need mesh peering or Shared VPC
- **App Engine Standard vs Flex:** Standard scales to 0, Flex does not (minimum 1 instance)
- **Cloud Functions timeout:** Gen 1: 9 min max; Gen 2: 60 min max
- **Autopilot restrictions:** No privileged containers, no DaemonSets, no custom node images
- **Archive storage:** Still instantly accessible; cost is in retrieval fees, not speed

---

## 1.13 Key Decision Frameworks

### The "Right Service" Framework
1. What is the **data structure**? (structured / unstructured / time-series)
2. What is the **access pattern**? (OLTP / OLAP / cache / stream)
3. What is the **scale requirement**? (regional / global)
4. What are the **operational constraints**? (team skill, managed vs self-managed)
5. What are the **cost constraints**? (reserved / on-demand / serverless)

### The "High Availability" Framework
1. Identify the **availability SLA** (99.9% / 99.99% / 99.999%)
2. Design **redundancy** at each tier (compute, storage, network)
3. Eliminate **single points of failure** (SPOF)
4. Implement **health checks and autohealing**
5. Test with **chaos engineering / Game Days**

---

## 1.14 Summary Table

| Topic | Key Service/Concept | Exam Signal Words |
|---|---|---|
| Global relational DB | Cloud Spanner | "global", "strong consistency", "horizontal scale" |
| Serverless HTTP | Cloud Run | "containerized", "scale to zero", "stateless" |
| Event-driven | Cloud Functions / Pub/Sub | "trigger", "event", "asynchronous" |
| Data warehouse | BigQuery | "analytics", "petabyte", "SQL queries" |
| Object storage | Cloud Storage | "blobs", "unstructured", "data lake" |
| Key-value / time-series | Bigtable | "IoT", "HBase", "millions of writes/sec" |
| Global load balancing | Cloud HTTP(S) LB | "multi-region", "anycast", "CDN" |
| Hybrid connectivity | HA VPN / Interconnect | "on-premises", "hybrid", "BGP" |
| Container orchestration | GKE (Standard or Autopilot) | "Kubernetes", "microservices", "container" |
| VM migration | Migrate for Compute Engine | "lift-and-shift", "VMware", "existing VMs" |
| DB migration | Database Migration Service | "migrate MySQL/PostgreSQL", "minimal downtime" |
| ML platform | Vertex AI | "ML", "train", "deploy model", "AutoML" |
| Caching | Memorystore (Redis/Valkey) | "cache", "session", "sub-millisecond" |
| Message queue | Pub/Sub | "decouple", "at-least-once", "fan-out" |
| Cost-effective batch | Spot VMs | "fault-tolerant", "batch", "cost savings" |

---

## 1.15 Exam Tips

- **Read the entire question** before looking at answers — look for constraints (budget, time, team skills)
- **"Managed" always beats "self-managed"** when operational overhead reduction is a requirement
- **"Minimum cost"** often points to serverless + Spot VMs + lifecycle policies
- **"Minimum effort"** often points to lift-and-shift + managed databases
- **The Well-Architected Framework** is referenced explicitly in some questions — know all six pillars
- When asked about **sustainability**, look for serverless options and high-carbon-free energy regions (Iowa, Finland, Oregon)
- **Spanner** is the answer to almost every "globally consistent relational database" question
- **Bigtable** is the answer to almost every "high-throughput, low-latency, wide-column, time-series" question
