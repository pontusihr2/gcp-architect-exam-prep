# GCP Services Comparison Cheat Sheet
> Google Professional Cloud Architect — 2025 Edition

---

## 1. Compute Services Comparison

| Feature | Compute Engine (GCE) | Google Kubernetes Engine (GKE) | Cloud Run | App Engine | Cloud Functions |
|---|---|---|---|---|---|
| **Managed Level** | IaaS — you manage OS, runtime, patches | Semi-managed — GCP manages control plane; you manage node pools | Fully managed (serverless containers) | Fully managed (PaaS) | Fully managed (FaaS/serverless) |
| **Primary Use Case** | Lift-and-shift VMs, custom OS needs, stateful workloads | Microservices, complex containerized workloads, ML training | Stateless HTTP containers, APIs, microservices | Web apps, mobile backends (Standard: language runtimes; Flex: custom containers) | Event-driven, short-lived functions, webhooks, lightweight processing |
| **Scaling** | Manual or MIG autoscaling (CPU/custom metrics); 0→N but no scale-to-zero | HPA/VPA/Cluster Autoscaler; Autopilot scales to zero (Standard does not) | Scale to zero by default; concurrency-based; max instances cap | Standard: automatic scale-to-zero; Flex: manual min instances (no scale-to-zero) | Scale to zero; per-function concurrency (Gen 2: up to 1000 concurrent) |
| **Pricing Model** | Per-second (min 1 min), SUDs apply; Spot/Preemptible for 60–91% savings | Node VM cost + GKE management fee ($0.10/hr per cluster for Standard; Autopilot per-pod) | Per vCPU-second + memory-second + request; idle = $0 | Standard: instance-hours (free tier); Flex: underlying GCE VM cost | Per invocation + compute time (GB-seconds) + networking |
| **Max Execution Time** | Unlimited (long-running) | Unlimited (pods run continuously) | 60 min (HTTP) / no limit (jobs) | Standard: 10 min (background tasks); Flex: no hard limit | Gen 1: 9 min; Gen 2: 60 min |
| **State Management** | Stateful — Persistent Disk attached; full control | Stateful via StatefulSets + Persistent Volumes | Stateless by design; use Cloud Storage / Firestore for state | Standard: mostly stateless (Memorystore for sessions); Flex: more flexible | Stateless by design; external state via Firestore/GCS |
| **Cold Start** | N/A (always-on) | Pod startup (~seconds if node exists) | ~100ms–2s (Go/Node fastest); min-instances eliminates | Standard: fast (<250ms); Flex: minutes | Gen 1: ~1–3s; Gen 2: faster with min instances |
| **Concurrency** | N/A | Per-pod configuration | Up to 1000 requests/instance (configurable) | Standard: 1 req/instance (F-class); Flex: configurable | Gen 1: 1 req/function; Gen 2: up to 1000 |
| **GPU/TPU Support** | Yes (full range) | Yes (node pool config) | No | No | No |
| **Custom Runtimes** | Full control | Any container | Any container | Standard: limited runtimes; Flex: any container | Limited runtimes (Gen 2 improves this) |

> **Exam Tip:** When the scenario says "minimal operational overhead" + containers → Cloud Run. "Complex orchestration" + containers → GKE. "Legacy VM" or "custom kernel" → GCE.

---

## 2. Database Services Comparison

| Feature | Cloud SQL | Cloud Spanner | Firestore | Bigtable | AlloyDB |
|---|---|---|---|---|---|
| **Type** | Relational (MySQL, PostgreSQL, SQL Server) | Relational (globally distributed) | NoSQL document store | NoSQL wide-column (HBase-compatible) | Relational (PostgreSQL-compatible) |
| **Consistency** | Strong (single region); read replicas are eventually consistent | External consistency (strongest available — serializable + globally consistent) | Strong consistency (default); eventual optional | Strong within row; eventual across rows | Strong (ACID); ML-optimized |
| **Scale** | Up to ~64 TB per instance; read replicas for read scale; single-writer | Unlimited horizontal scale (petabytes); multi-region active-active | Scales automatically; 1 MB per document limit | Petabyte-scale; scales linearly with nodes (6 MB/s per node) | ~64 TB; read pool scales independently |
| **Latency** | Low (single region <5ms); cross-region adds latency | Single-digit ms reads; ~5–10ms writes globally | ~10–100ms; varies by region | ~1–10ms per read/write (optimized for low latency at scale) | Sub-millisecond for cached reads; ~ms writes |
| **Use Case** | Traditional OLTP, web apps, ERP, CMS | Global financial transactions, inventory (need strong consistency + global scale) | Mobile/web real-time apps, user profiles, game state | IoT time-series, ad tech, financial market data, large analytics tables | High-performance OLTP, migrating from AlloyDB or PostgreSQL at scale |
| **Pricing Model** | Per vCPU/RAM/storage/hour + network | Per node/hour + storage + network (expensive — justify with global requirement) | Per read/write/delete operation + storage | Per node/hour + storage (expensive at low scale) | Per vCPU/RAM/storage (premium over Cloud SQL) |
| **Multi-Region** | Read replicas only; not active-active writes | Native multi-region active-active | Multi-region (automatic replication) | Multi-cluster replication | Read replicas; not natively global |
| **Transactions** | Full ACID | Full ACID globally | ACID (multi-document transactions) | Single-row ACID only | Full ACID |
| **Best Avoid When** | Need global writes / petabyte scale | Cost sensitivity; regional-only workloads | Need SQL; relational joins | Need SQL; small datasets (<1 TB inefficient) | Cost constraints; simple workloads |

> **Exam Tip:** "Global + financial + strong consistency" → Spanner. "IoT + time-series + millions of writes/sec" → Bigtable. "Mobile app real-time sync" → Firestore. "Migrate existing MySQL/PostgreSQL" → Cloud SQL or AlloyDB. AlloyDB is the answer when the question involves "high-performance PostgreSQL" or "AI/ML integrated queries."

---

## 3. Storage Services Comparison

| Feature | Cloud Storage Standard | Cloud Storage Nearline | Cloud Storage Coldline | Cloud Storage Archive | Persistent Disk (PD) | Filestore |
|---|---|---|---|---|---|---|
| **Type** | Object storage | Object storage | Object storage | Object storage | Block storage | Managed NFS file storage |
| **Access Pattern** | Frequently accessed (hot data) | Accessed < once per month | Accessed < once per quarter | Accessed < once per year | Low-latency block I/O for VMs | Shared file access (NFS v3/v4.1) |
| **Latency** | ~10–100ms (first byte) | ~10–100ms (first byte; retrieval fee applies) | ~10–100ms (retrieval fee higher) | ~10–100ms (highest retrieval fee; 12hr min storage) | <1ms (SSD); ~1ms (HDD) | ~0.3–3ms (SSD); ~1–3ms (HDD) |
| **Cost Tier** | Highest storage cost; free retrieval | Lower storage cost; $0.01/GB retrieval | Lower storage cost; $0.02/GB retrieval | Lowest storage cost; $0.05/GB retrieval | SSD: ~$0.17/GB/mo; HDD: ~$0.04/GB/mo; Extreme: ~$0.12/GB/mo | Basic HDD: ~$0.20/GB/mo; Basic SSD: ~$0.30/GB/mo |
| **Durability** | 11 nines (99.999999999%) | 11 nines | 11 nines | 11 nines | 99.99%+ (regional replication) | 99.99%+ |
| **Minimum Storage Duration** | None | 30 days | 90 days | 365 days | N/A (pay while attached) | N/A |
| **Use Case** | Static website assets, media serving, data lakes, backups in rotation | Monthly backups, disaster recovery data, infrequently accessed logs | Quarterly DR, long-term backups | Regulatory archives, compliance retention (7+ years) | OS disks, database storage, application data requiring POSIX | Lift-and-shift NAS, shared app data, content management, genomics pipelines |
| **Multi-Attach** | N/A (object store) | N/A | N/A | N/A | Read-only multi-attach (hyperdisk) | Yes — shared NFS mount |
| **Replication** | Multi-region / Dual-region / Regional | Same options | Same options | Same options | Zonal (default); Regional PD (async) | Zonal only (Basic); Regional (Enterprise) |

### Cloud Storage Key Notes
- **Autoclass**: Automatically transitions objects between storage classes — cost-optimize without lifecycle rules
- **Object Lifecycle Management**: Rule-based transitions (e.g., move to Coldline after 90 days, delete after 7 years)
- **Uniform bucket-level access**: Disables ACLs; enforces IAM only — recommended for security
- **Retention Policies + Object Holds**: WORM compliance (Coldline/Archive + retention policy = regulatory archive)

---

## 4. Messaging & Eventing Comparison

| Feature | Pub/Sub | Cloud Tasks | Cloud Scheduler | Eventarc |
|---|---|---|---|---|
| **Primary Model** | Async message queue / fan-out (push or pull) | Explicit task queue with targeting | Cron-style time-based job invocation | Event-driven routing (Cloud events standard) |
| **Delivery** | At-least-once; ordering (with ordering keys) | Exactly-once attempt; retry with backoff | At-least-once per schedule | At-least-once; CloudEvents format |
| **Targets** | Any push endpoint, Cloud Run, Cloud Functions, BigQuery, GCS | Cloud Run, Cloud Functions, App Engine, HTTP URLs | Cloud Run, Cloud Functions, App Engine, HTTP | Cloud Run, Cloud Functions, GKE, Workflows |
| **Trigger** | Event-driven (producer pushes) | Explicit enqueue (programmatic) | Time-based (cron expression) | GCP service events (GCS, Pub/Sub, Audit Logs, custom) |
| **Ordering** | Optional (ordering keys per partition) | FIFO within queue | N/A (single shot) | N/A |
| **Rate Control** | No (consumers control pull rate) | Yes — max dispatches per second, max concurrent | Frequency: cron expression | N/A |
| **Use Case** | Decoupling microservices, streaming ingest, event fan-out | Deferred processing, rate-limited API calls, retryable background tasks | Scheduled reports, database cleanup, periodic ML model refresh | Trigger Cloud Run/Functions on GCP service events |
| **Message Retention** | 7 days default (up to 31 days) | Until acknowledged or deadline | N/A | N/A |
| **Dead Letter** | Dead Letter Topic support | Dead task queue | N/A | N/A |

> **Exam Tip:** "Decouple" + "fan-out" → Pub/Sub. "Rate limit API calls" or "retry with exponential backoff" → Cloud Tasks. "Run nightly at 2am" → Cloud Scheduler. "Trigger on GCS upload" → Eventarc (or direct GCS notification to Pub/Sub).

---

## 5. Analytics & Data Processing Comparison

| Feature | BigQuery | Dataflow | Dataproc | Datastream |
|---|---|---|---|---|
| **Type** | Serverless data warehouse + analytics engine | Managed Apache Beam (unified batch + stream) | Managed Hadoop/Spark/Flink/Hive | Change Data Capture (CDC) / replication service |
| **Processing Model** | SQL queries on stored data (interactive + scheduled) | Streaming or batch pipelines (code-driven transforms) | Batch (Spark/MapReduce) or stream (Spark Streaming) | Continuous CDC from source databases |
| **Managed Level** | Fully serverless (no cluster to manage) | Fully managed (Dataflow workers auto-scale) | Managed cluster (you size it; auto-scaling available) | Fully managed |
| **Latency** | Interactive: seconds to minutes; streaming: near-real-time inserts | Streaming: sub-second to seconds; batch: throughput-optimized | Batch: minutes to hours; depends on cluster size | Low-latency replication (seconds to minutes lag) |
| **Skill Required** | SQL | Apache Beam (Java/Python) | Spark/Hadoop ecosystem | Minimal (managed service) |
| **Scaling** | Automatic (slots-based; on-demand or reserved) | Horizontal autoscaling of workers | Manual or autoscaling node pools | Automatic |
| **Use Case** | Ad hoc analysis, BI dashboards, data warehouse, ML feature engineering | ETL/ELT pipelines, real-time fraud detection, log processing | Migrate existing Hadoop/Spark jobs, ML training at scale | Migrate databases to GCP; real-time replication to BigQuery |
| **Cost Model** | On-demand: $5/TB queried; flat-rate: reserved slots; storage: $0.02/GB/mo | Per vCPU/hr + memory/hr while pipeline runs | Per node/hr (standard or preemptible) + HDFS | Per GB replicated + compute |
| **Integration** | Native: GCS, Bigtable, Pub/Sub, Sheets; BQML built-in | Any GCP service via Beam I/O connectors | Works with GCS, Bigtable, BigQuery | Source: Oracle, MySQL, PostgreSQL, AlloyDB → Target: BigQuery, GCS |

> **Exam Tip:** "Analyze petabytes with SQL" → BigQuery. "Complex streaming pipeline with custom transforms" → Dataflow. "Existing Spark/Hadoop jobs" → Dataproc. "Real-time Oracle → BigQuery" → Datastream.

---

## 6. Network Connectivity Comparison

| Feature | Cloud VPN (HA VPN) | Dedicated Interconnect | Partner Interconnect | Direct Peering | CDN Interconnect |
|---|---|---|---|---|---|
| **Connection Type** | Encrypted IPsec tunnel over public internet | Private physical connection at Google PoP | Private connection via service provider | BGP peering with Google network (public IPs) | Reduced-cost egress via CDN partners |
| **Throughput** | Up to 3 Gbps per tunnel (multiple tunnels for more) | 10 Gbps or 100 Gbps per link; up to 8 links (80 Gbps / 800 Gbps) | 50 Mbps – 10 Gbps (provider-dependent) | Best-effort (no SLA) | Varies by partner |
| **SLA** | 99.99% (with 2 tunnels, 2 gateways) | 99.99% (with redundant connections) | 99.99% (with redundancy) | None | Varies |
| **Latency** | Higher (internet routing + encryption overhead) | Lowest (private, predictable) | Low (private via provider) | Variable (internet) | N/A |
| **Encryption** | Yes (IPsec) | No (encrypted separately if needed) | No (encrypted separately if needed) | No | No |
| **Minimum Bandwidth** | None (pay per tunnel) | 10 Gbps | 50 Mbps | N/A | N/A |
| **Use Case** | Backup connectivity, lower-bandwidth hybrid, dev/test | High-bandwidth, low-latency production hybrid (>1 Gbps) | Dedicated interconnect not available at your colocation; medium bandwidth | Accessing Google public APIs without traversing public internet | Large-scale media content delivery cost optimization |
| **Setup Time** | Minutes | Weeks (physical provisioning) | Days to weeks | Days | Varies |
| **Cost** | Per tunnel/hr + egress | Port fee + VLAN attachment + egress (cheaper than VPN at high BW) | Per VLAN attachment + egress (provider fees extra) | Free (no data transfer charges to Google) | Reduced egress rates |
| **When to Choose** | BW < 1 Gbps, or budget-conscious, or compliance allows internet | BW > 1 Gbps, strict latency SLA, not in Google PoP colocation | Not collocated with Google PoP but need private connectivity | Only need access to Google public APIs | Heavy CDN egress optimization |

---

## 7. Load Balancer Comparison

| Load Balancer | Scope | Protocol | SSL Termination | Backend Types | Key Notes |
|---|---|---|---|---|---|
| **Global External Application LB** (formerly HTTP(S) LB) | Global (anycast) | HTTP/HTTPS, HTTP/2, gRPC | Yes (Google-managed or self-managed certs) | Instance groups, NEGs (serverless, internet, hybrid), GCS buckets | CDN integration; URL maps for routing; supports Cloud Armor WAF |
| **Regional External Application LB** | Regional | HTTP/HTTPS | Yes | Instance groups, regional NEGs | Lower latency within region; no global anycast |
| **Internal Application LB** (Layer 7) | Regional (internal) | HTTP/HTTPS, gRPC | Yes (internal certs) | Instance groups, serverless NEGs | VPC-internal; replaces Internal HTTP(S) LB |
| **External Proxy Network LB** (TCP/SSL) | Global | TCP, SSL | Yes (SSL Proxy) | Instance groups, zonal NEGs | SSL offload; global anycast; for non-HTTP TCP |
| **Internal Proxy Network LB** (TCP) | Regional | TCP | No | Instance groups | Internal TCP Layer 4 proxy |
| **External Passthrough Network LB** (formerly Network LB) | Regional | TCP, UDP, ESP, ICMP | No (pass-through) | Instance groups, target pools | Preserves source IP; no SSL termination; health checks per protocol |
| **Internal Passthrough Network LB** (ILB) | Regional | TCP, UDP | No | Instance groups | Classic internal LB for 3-tier architectures; preserves source IP |

### Load Balancer Quick Decision Guide
```
Need HTTP/HTTPS routing? → Application LB
  ├─ Global users + CDN + WAF? → Global External Application LB
  ├─ Regional only? → Regional External Application LB
  └─ Internal microservices? → Internal Application LB

Need TCP/UDP?
  ├─ Global + SSL offload? → External Proxy Network LB
  ├─ Preserve source IP (external)? → External Passthrough Network LB
  └─ Internal VPC traffic? → Internal Passthrough Network LB
```

> **Exam Tip:** "Global users + CDN + WAF (Cloud Armor)" → always Global External Application LB. "Preserve client IP" → Passthrough LB. "gRPC internal" → Internal Application LB. Load balancers with "Proxy" in name terminate connections; "Passthrough" do not.
