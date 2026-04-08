# GCP Architecture Decision Trees
> Google Professional Cloud Architect — 2025 Edition

Use these trees to rapidly eliminate wrong answers on the exam. Start at the root question and follow the branches.

---

## 1. Compute Service Selection

```
START: What is your workload?
│
├─► Is it a legacy VM / requires specific OS / custom kernel / stateful VM?
│       YES → Compute Engine (GCE)
│             ├─ Short-lived batch? → Spot / Preemptible VMs
│             ├─ Steady-state? → Committed Use Discounts (CUDs)
│             └─ Stateful + HA? → MIG with stateful config + Regional PD
│
├─► Is it containerized?
│   │
│   ├─► Is it stateless + HTTP? (serve web traffic, APIs)
│   │   │
│   │   ├─► Minimal ops overhead? No cluster to manage?
│   │   │       YES → Cloud Run (fully managed)
│   │   │             ├─ Scale to zero needed? → Cloud Run (default)
│   │   │             └─ Always warm? → Cloud Run with min-instances
│   │   │
│   │   └─► Complex orchestration? Multi-container? Service mesh?
│   │       Stateful pods? Custom scheduling?
│   │           YES → GKE
│   │                 ├─ Fully managed nodes? → GKE Autopilot
│   │                 └─ Node customization / GPUs? → GKE Standard
│   │
│   └─► Is it a background job / batch container?
│           YES → Cloud Run Jobs (for containers)
│                 or GKE Jobs / CronJobs
│
├─► Is it event-driven? Short execution? Single function?
│   │
│   ├─► HTTP trigger / Pub/Sub / GCS event / Firestore event?
│   │       YES → Cloud Functions (Gen 2)
│   │             ├─ < 60 min execution? → Gen 2 (preferred)
│   │             └─ Need > 60 min? → Cloud Run instead
│   │
│   └─► Need complex workflow / multi-step orchestration?
│           YES → Cloud Workflows (+ Cloud Functions/Run for steps)
│
└─► Is it a web application?
    │
    ├─► Standard language runtime (Python/Java/Node/Go/PHP/Ruby)?
    │   Moderate traffic? Rapid deployment?
    │       YES → App Engine Standard
    │
    └─► Custom runtime / Docker container / long-running requests?
            YES → App Engine Flexible
                  (Note: Cloud Run usually preferred for new workloads)
```

---

## 2. Database Service Selection

```
START: What kind of data model do you need?
│
├─► SQL / Relational data?
│   │
│   ├─► Do you need GLOBAL distribution + strong consistency?
│   │   (multi-region writes, financial transactions globally)
│   │       YES → Cloud Spanner
│   │             ├─ Cost acceptable? (most expensive option) → Spanner
│   │             └─ PostgreSQL-compatible? → Spanner PostgreSQL dialect
│   │
│   ├─► High-performance PostgreSQL? AI/ML query acceleration?
│   │   Migrating AlloyDB workloads?
│   │       YES → AlloyDB for PostgreSQL
│   │
│   └─► Standard MySQL / PostgreSQL / SQL Server?
│       Single-region OLTP? Managed service?
│           YES → Cloud SQL
│                 ├─ MySQL → Cloud SQL for MySQL
│                 ├─ PostgreSQL → Cloud SQL for PostgreSQL
│                 └─ SQL Server → Cloud SQL for SQL Server
│
└─► NoSQL data?
    │
    ├─► What is the scale / throughput requirement?
    │
    ├─► Very high throughput (millions of ops/sec)?
    │   Flat / wide-column schema? Time-series / IoT / ad-tech?
    │       YES → Cloud Bigtable
    │             ├─ < 1 TB? → Bigtable is OVERKILL (use Firestore)
    │             ├─ HBase migration? → Bigtable (HBase-compatible API)
    │             └─ Analytics queries on top? → Bigtable → BigQuery
    │
    ├─► Document model? Mobile/web app? Real-time sync?
    │   User profiles, game state, product catalogs?
    │       YES → Firestore
    │             ├─ Mobile SDK needed? → Firestore (native iOS/Android SDK)
    │             ├─ Datastore legacy migration? → Firestore in Datastore mode
    │             └─ Global? → Firestore multi-region
    │
    └─► Key-value / cache / session store?
            YES → Memorystore
                  ├─ Redis? → Memorystore for Redis
                  └─ Memcached? → Memorystore for Memcached
```

---

## 3. Storage Service Selection

```
START: What type of storage access pattern?
│
├─► OBJECT storage (files, blobs, unstructured data)?
│   │
│   ├─► How frequently is data accessed?
│   │
│   ├─► Accessed multiple times per day / real-time serving?
│   │       → Cloud Storage STANDARD
│   │         (website assets, active datasets, streaming media)
│   │
│   ├─► Accessed less than once per month?
│   │       → Cloud Storage NEARLINE
│   │         (monthly backups, DR data, analytics archives)
│   │
│   ├─► Accessed less than once per quarter?
│   │       → Cloud Storage COLDLINE
│   │         (quarterly DR tests, compliance archives)
│   │
│   ├─► Accessed less than once per year?
│   │       → Cloud Storage ARCHIVE
│   │         (regulatory retention, legal holds, 7+ year archives)
│   │
│   └─► Access pattern unpredictable / want auto-tiering?
│           → Cloud Storage with AUTOCLASS enabled
│
├─► BLOCK storage (VM disk, database volumes)?
│   │
│   ├─► Which performance tier?
│   │
│   ├─► General-purpose VM workloads, OS disks, dev/test?
│   │       → Balanced Persistent Disk (PD-Balanced)
│   │
│   ├─► High IOPS databases (MySQL, PostgreSQL, SQL Server)?
│   │       → SSD Persistent Disk (PD-SSD)
│   │
│   ├─► Throughput-optimized (Hadoop, sequential reads/writes)?
│   │       → Standard Persistent Disk (PD-Standard / HDD)
│   │
│   ├─► Highest IOPS available (Oracle, SAP HANA)?
│   │       → Hyperdisk Extreme
│   │
│   └─► Need disk shared across multiple VMs?
│           → Hyperdisk ML (read-only multi-attach for ML inference)
│           → Regional Persistent Disk (HA failover between zones)
│
└─► FILE storage (NFS shared filesystem)?
    │
    ├─► Lift-and-shift NAS workloads? Shared app data?
    │   Content management? Genomics?
    │       → Filestore
    │
    ├─► What performance tier?
    │   ├─ General workloads → Filestore Basic HDD
    │   ├─ Higher IOPS → Filestore Basic SSD
    │   └─ Enterprise + HA + Regional → Filestore Enterprise
    │
    └─► Cloud-native shared config / data?
            → Consider GCS FUSE (mount GCS as filesystem) — note latency tradeoffs
```

---

## 4. Network Connectivity Selection (On-Premises to GCP)

```
START: Do you need private (non-internet) connectivity to GCP?
│
├─► NO — Public internet is acceptable
│       → Cloud VPN (HA VPN) with encryption
│         ├─ BW < 3 Gbps per tunnel? → Single HA VPN
│         └─ More BW? → Multiple HA VPN tunnels (bond them)
│
└─► YES — Private, dedicated connectivity
    │
    ├─► Are you co-located in a Google PoP / data center?
    │   │
    │   └─► YES → Dedicated Interconnect
    │             ├─ 10 Gbps links (up to 8 = 80 Gbps)
    │             ├─ 100 Gbps links (up to 8 = 800 Gbps)
    │             └─ Need max SLA (99.99%)? → 4 circuits, 2 metros
    │
    └─► NO — Not co-located at Google PoP
        │
        ├─► Does a Partner Interconnect provider serve your location?
        │       YES → Partner Interconnect
        │             ├─ 50 Mbps – 10 Gbps options
        │             ├─ Layer 2 (direct VLAN) or Layer 3 (provider-managed BGP)
        │             └─ 99.99% SLA with redundant connections
        │
        └─► NO / Only need access to Google public APIs?
                → Direct Peering
                  (BGP peering — best-effort, no SLA, no GCP VPC access)

Additional Decision: Does your BW requirement exceed current connection?
├─ < 1 Gbps → HA VPN is cost-effective
├─ 1–10 Gbps → Evaluate VPN vs Partner Interconnect (break-even ~3 Gbps)
└─ > 10 Gbps → Dedicated Interconnect almost always wins on cost + latency
```

---

## 5. Data Processing Service Selection

```
START: Is your processing batch, streaming, or both?
│
├─► BATCH only?
│   │
│   ├─► Do you have existing Hadoop/Spark/Hive/Pig jobs?
│   │       YES → Dataproc (managed Hadoop/Spark)
│   │             ├─ Ephemeral clusters (create/process/delete)? → Best practice
│   │             └─ Long-running cluster? → Use autoscaling
│   │
│   ├─► Is the data already in BigQuery / GCS?
│   │   Do you need SQL-based transformations?
│   │       YES → BigQuery (SQL queries, scheduled queries, BQML)
│   │             └─ Transform + load? → BigQuery DML or dbt on BigQuery
│   │
│   └─► Custom complex pipeline? Language preference Java/Python?
│           YES → Dataflow (Apache Beam) — batch mode
│
├─► STREAMING only or STREAMING + BATCH (unified)?
│   │
│   ├─► Low-latency, custom transformations, complex windowing?
│   │       YES → Dataflow (Apache Beam) — streaming mode
│   │             (exactly-once semantics, auto-scaling workers)
│   │
│   ├─► Existing Spark Streaming jobs to migrate?
│   │       YES → Dataproc (with Spark Structured Streaming)
│   │
│   └─► Ingest + simple transforms → store → query?
│           YES → Pub/Sub → BigQuery direct subscription
│                 or Pub/Sub → Dataflow → BigQuery
│
├─► What is the latency requirement?
│   ├─ Real-time (<1 second) → Dataflow streaming
│   ├─ Near-real-time (1–60 seconds) → Dataflow or Pub/Sub → BigQuery
│   ├─ Minutes → Dataflow batch or Dataproc
│   └─ Hours/overnight → BigQuery scheduled queries or Dataproc batch
│
└─► Data warehouse / analytics queries (not pipeline)?
        → BigQuery
          ├─ Ad hoc SQL analysis → On-demand pricing
          ├─ Predictable heavy workloads → Slot reservations (flat-rate)
          └─ ML on data → BigQuery ML (BQML)
```

---

## 6. Migration Strategy Selection

```
START: What are your migration goals?
│
├─► Fastest migration to cloud? Minimal changes?
│   (time-to-migrate is priority over optimization)
│       → LIFT AND SHIFT (Rehost)
│         ├─ VMs as-is → Migrate to Virtual Machines (formerly Velostrata)
│         ├─ Databases as-is → Cloud SQL or GCE + database software
│         └─ Containers as-is → GKE or Cloud Run (if already containerized)
│
├─► Modernize the platform but keep application logic?
│   (medium complexity, medium risk, medium benefit)
│       → RE-PLATFORM (Lift-and-Optimize)
│         ├─ MySQL on GCE → Cloud SQL (managed DB)
│         ├─ App on VMs → App Engine or Cloud Run (containerize app)
│         ├─ Hadoop on VMs → Dataproc (managed Hadoop)
│         └─ Self-managed cache → Memorystore
│
├─► Redesign the architecture for cloud-native benefits?
│   (high complexity, high risk, maximum benefit)
│       → RE-ARCHITECT (Refactor / Cloud-native)
│         ├─ Monolith → Microservices on GKE/Cloud Run
│         ├─ Batch ETL → Streaming with Pub/Sub + Dataflow
│         ├─ On-prem Oracle → Cloud Spanner (global)
│         └─ File-based workflow → GCS + Cloud Functions/Eventarc
│
├─► Decision Factors:
│   ├─ Timeline pressure → Lift and Shift
│   ├─ Budget constraints → Re-platform (targeted savings)
│   ├─ Technical debt → Re-architect (long-term cost savings)
│   ├─ Compliance requirements → Evaluate all options for data residency
│   └─ Vendor lock-in concerns → Re-architect with open standards (GKE, Dataflow/Beam)
│
└─► Risk Assessment:
    ├─ Business criticality HIGH → Start with Re-platform (lower risk than Re-architect)
    ├─ Application well-understood → Re-architect for max cloud benefit
    └─ Application legacy / poor documentation → Lift and Shift first, then modernize
```

---

## 7. Disaster Recovery Strategy Selection

```
START: What are your RTO and RPO requirements?
│
├─► RPO: Hours acceptable? RTO: Hours acceptable?
│       → BACKUP AND RESTORE
│         ├─ GCS object storage for backups
│         ├─ Cloud SQL automated backups
│         └─ Lowest cost, highest recovery time
│
├─► RPO: Minutes/Hours? RTO: Hours/Day?
│       → PILOT LIGHT
│         ├─ Core data replicated continuously (Cloud SQL replicas)
│         ├─ Minimal compute provisioned (start VMs on failover)
│         └─ Medium cost, medium recovery time
│
├─► RPO: Seconds/Minutes? RTO: Minutes?
│       → WARM STANDBY
│         ├─ Reduced-capacity secondary environment always running
│         ├─ Regional Persistent Disk for data replication
│         ├─ Scale up secondary on failover event
│         └─ Higher cost, faster recovery
│
└─► RPO: Near-zero? RTO: Seconds?
        → ACTIVE-ACTIVE / HOT STANDBY
          ├─ Cloud Spanner (global, always-on)
          ├─ Multi-region Cloud Run / GKE (Global LB auto-failover)
          ├─ Global External Application LB (auto-routes to healthy backends)
          └─ Highest cost, near-zero downtime
```
