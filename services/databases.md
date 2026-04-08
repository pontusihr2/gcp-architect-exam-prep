# GCP Database Services — Exam Reference (2025)

---

## Cloud SQL

### Supported Engines

| Engine | Max Storage | Max RAM | Max Connections | Notes |
|--------|-------------|---------|-----------------|-------|
| MySQL 8.0 | 64 TB | 624 GB | ~4,000 | GTID replication; InnoDB only |
| PostgreSQL 15/16 | 64 TB | 624 GB | ~1,000 | Extensions: pgvector, PostGIS, etc. |
| SQL Server 2019/2022 | 64 TB | 624 GB | varies | Enterprise/Standard/Web/Express editions |

### High Availability

- **HA Configuration**: Primary + standby instance in the **same region, different zone**
- Synchronous replication to standby; automatic failover in 1–2 minutes
- Failover is automatic; no manual intervention required
- HA standby instance is NOT accessible for reads (not a read replica)
- Costs 2× single instance (primary + standby billed separately)

### Read Replicas

- **Cross-region read replicas** supported; used for disaster recovery or global read scaling
- Read replicas in same region reduce primary load for read-heavy workloads
- Can be promoted to standalone instance (breaks replication)
- **Cascaded read replicas**: Read replica of a read replica (reduces primary replication load)
- Up to 10 read replicas per primary

### Backups & Maintenance

- **Automated backups**: Daily; retained 7 days (configurable up to 365); point-in-time recovery (PITR) using transaction logs
- **On-demand backups**: Manual; retained until deleted
- **Maintenance windows**: Weekly 1-hour window; defer minor vs major updates
- **Deletion protection**: Prevent accidental instance deletion (enable in production!)
- **Private IP**: Recommended for production; requires VPC; no public IP exposure

> **Exam Tip**: Cloud SQL HA standby is NOT readable — it's only for failover. For read scaling, add read replicas. For global distribution, consider Cloud Spanner.

> **Gotcha**: Cloud SQL PITR requires binary logging (MySQL) or WAL archiving (PostgreSQL) to be enabled. Enabling adds storage cost and slight write overhead.

---

## Cloud Spanner

### Architecture

- **Globally distributed**, horizontally scalable, strongly consistent relational database
- **TrueTime API**: GPS + atomic clocks in every Google datacenter; enables global external consistency; unique to Google
- ACID transactions across rows, tables, and databases globally
- SQL (ANSI 2011 + extensions) and mutations (batch writes) interfaces

### Instances & Processing Units

- **Instance**: Top-level resource; defines region configuration and capacity
- **Processing Units (PUs)**: Unit of compute; 1000 PUs = 1 node
- Instance configs:
  - *Regional*: Single region; 3+ read-write replicas; 5 nines availability (99.999%)
  - *Multi-region*: Replicated across multiple regions (e.g., `nam6`, `eur3`, `asia1`); 99.999% SLA; higher write latency
- Can scale PUs from 100 (smallest) to thousands online without downtime

### Schema Design

- **Interleaved tables**: Child table rows stored physically with parent row; eliminates joins for parent-child queries; up to 7 levels deep
- **Primary keys**: Must be carefully designed — monotonically increasing keys cause hotspots; use UUID, hash, or bit-reversal
- **Secondary indexes**: Stored as interleaved or non-interleaved; choosing interleaving affects performance
- **Commit timestamps**: `PENDING_COMMIT_TIMESTAMP()` for auto-timestamped writes

### Change Streams

- Capture data changes in real time; stream to Dataflow for CDC (Change Data Capture) pipelines

> **Exam Tip**: Cloud Spanner is the right choice when you need: (1) global ACID transactions, (2) > 2 TB relational data, or (3) multi-region active-active relational DB. It's 10–100× more expensive than Cloud SQL.

> **Gotcha**: Sequential primary keys in Spanner cause hot spots and poor performance. Always design for distributed key access.

---

## Firestore

### Native Mode vs Datastore Mode

| Feature | Native Mode | Datastore Mode |
|---------|------------|----------------|
| Real-time listeners | Yes | No |
| Mobile/web SDKs | Yes | No (server SDKs only) |
| Strongly consistent queries | Yes | Yes |
| Collections & documents | Yes | Kinds & entities |
| Legacy Datastore migration | No | Yes |
| Best for | Mobile, web, real-time apps | Server-side, migrating Datastore |

### Data Model

- **Documents**: JSON-like; max 1 MB; contain fields (scalars, arrays, maps, references)
- **Collections**: Groups of documents; no schema enforcement
- **Subcollections**: Collections nested inside documents
- **Document references**: Pointers to other documents (no actual joins)

### Queries & Indexes

- **Single-field indexes**: Auto-created for every field
- **Composite indexes**: Required for queries with multiple `where` clauses on different fields or combining `where` + `orderBy`; must be created manually (or via `firestore.indexes.json`)
- **Collection Group Queries**: Query across all collections with the same name
- No full-text search (use Algolia/Elasticsearch/Vertex AI Search as companion)
- **Limitations**: Cannot query based on partial string match; cannot count documents cheaply (use counter shards)

### Pricing

- Billed per read/write/delete operation; storage; network egress
- Reads are expensive at scale — design to minimize reads with denormalization and caching

> **Exam Tip**: Choose Firestore Native for real-time apps (Firebase mobile), Datastore mode for migrating from App Engine Datastore. Use composite indexes for complex queries.

> **Gotcha**: Firestore does NOT support multi-document transactions across different databases. Write throughput to a single document is limited to ~1 write/second (use sharded counters for high-write counters).

---

## Bigtable

### Architecture

- **NoSQL wide-column store**; low-latency at massive scale (petabytes, millions of QPS)
- Ideal for time-series, IoT, financial ticks, AdTech, ML feature serving
- HBase-compatible API; can migrate HBase workloads directly

### Data Model

- **Row key**: Sole indexed dimension; queries must use row key (prefix scans supported); no secondary indexes
- **Column families**: Group of related columns; defined at table creation; compaction settings per family
- **Cells**: Intersection of row + column; stores value + timestamp; multiple timestamped versions per cell
- **Tablets**: Rows sorted and split into tablets; stored on distributed nodes

### Row Key Design (Critical!)

- Good row keys enable efficient scans; bad keys cause hotspots
- **Anti-patterns**: Sequential timestamps as prefix, sequential user IDs, domain names left-to-right
- **Good patterns**: Reversed domain names (`com.example`), salted/hashed prefixes, reversed timestamps for recency queries, composite keys (e.g., `userId#timestamp`)

### Cluster Configuration

- **Instances** contain 1+ clusters; clusters are in specific zones
- **Multi-cluster routing**: Active-active replication for HA and global reads; eventual consistency between clusters
- **Replication**: Synchronously maintained by Bigtable; ~1–5 second lag between clusters
- Node count determines throughput/storage capacity; scale nodes without downtime

> **Exam Tip**: Bigtable is NOT a relational DB and has no SQL. It's optimized for single-row lookups and sequential scans. Design row keys as your query access patterns.

> **Gotcha**: Bigtable does NOT support multi-row ACID transactions. Cannot join data. All queries must use the row key.

---

## Memorystore

### Redis vs Valkey

| | Memorystore for Redis | Memorystore for Valkey |
|--|----------------------|------------------------|
| Protocol | Redis | Valkey (Redis fork, open source) |
| Tiers | Basic, Standard | Standard only |
| HA | Standard tier: HA with replica | Yes |
| Max size | 300 GB (Standard) | 1 TB |
| Use case | Caching, session, pub/sub | Same; preferred for new workloads |

### Tiers (Redis)

- **Basic**: Single node; no HA; data loss on failure; dev/test; less expensive
- **Standard**: Primary + replica; automatic failover; 99.9% SLA; required for production

### Key Features

- **RDB snapshots** and **AOF persistence** for data durability
- **IAM auth** and **in-transit TLS encryption**
- VPC-native; access via private IP within VPC
- **Read replicas**: Up to 5 read replicas for read-scaling (Standard tier)
- No direct internet access — must be within VPC

> **Exam Tip**: Use Memorystore Standard tier for any production caching or session management requiring HA. Valkey is the forward-looking choice for new deployments.

---

## AlloyDB

- **Fully managed PostgreSQL-compatible** database built for demanding workloads
- **Columnar Engine**: In-memory columnar store for analytical queries (HTAP); auto-selects row vs column storage
- Up to 4× faster than standard PostgreSQL for transactional workloads; up to 100× for analytical
- **AlloyDB Omni**: Run AlloyDB on any cloud or on-premises; same engine, self-managed
- **Cross-region replication**: Continuous replication for DR
- Compatible with PostgreSQL tools, drivers, extensions (pgvector included)
- **Migration**: Use Database Migration Service (DMS) for live migrations from PostgreSQL

> **Exam Tip**: Choose AlloyDB over Cloud SQL PostgreSQL when you need: higher performance, HTAP workloads (transactional + analytical in same DB), or pgvector at scale.

---

## Database Migration Service (DMS)

- Managed service for migrating databases to GCP
- **Supports**: MySQL → Cloud SQL, PostgreSQL → Cloud SQL/AlloyDB, Oracle → Cloud SQL PostgreSQL (via Ora2Pg), SQL Server → Cloud SQL
- **Migration types**: Continuous (CDC for minimal downtime) or one-time
- Uses Change Data Capture (CDC) for near-zero downtime migrations
- Migration jobs track progress; source connection via Cloud SQL Auth Proxy or IP allowlisting

---

## Database Decision Matrix

| Use Case | Recommended Database |
|----------|---------------------|
| OLTP relational (MySQL/PostgreSQL/SQL Server) < 30TB | Cloud SQL |
| Global ACID transactions, horizontal scale | Cloud Spanner |
| High-performance PostgreSQL + analytics (HTAP) | AlloyDB |
| Mobile/web app with real-time sync | Firestore (Native Mode) |
| Migrating from App Engine Datastore | Firestore (Datastore Mode) |
| Time-series, IoT, AdTech (millions of QPS) | Bigtable |
| In-memory caching, session storage | Memorystore (Redis/Valkey) |
| Pub/Sub messaging (not actually a DB) | Pub/Sub |
| Analytics data warehouse | BigQuery |
| Document search with full-text | Firestore + Vertex AI Search |
| Oracle license portability | Bare Metal Solution |
| Migrating existing DB with minimal downtime | Database Migration Service |

---

## Common Exam Gotchas

1. **Cloud SQL HA standby** is NOT readable — it exists only for failover
2. **Spanner hotspots** caused by sequential primary keys — always use distributed keys
3. **Firestore** requires manual composite index creation for multi-field queries — missing indexes cause errors
4. **Bigtable** has no secondary indexes — all queries must leverage the row key
5. **Memorystore Basic** tier loses data on failure — never use for production
6. **AlloyDB** columnar engine accelerates `SELECT` analytics automatically — no ETL to BigQuery needed for moderate analytics
7. **DMS** uses CDC for live migration — source DB remains active during migration
8. **Firestore write throughput** to a single document is ~1 write/second — shard high-write counters
