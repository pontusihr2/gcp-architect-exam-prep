# GCP Big Data & Analytics Services — Exam Reference (2025)

---

## BigQuery

### Core Concepts

- **Serverless** data warehouse; columnar storage; petabyte-scale; SQL (ANSI 2011)
- Storage and compute are **fully decoupled** — can scale independently
- **Datasets**: Top-level namespace for tables/views; have a location (regional or multi-regional)
- **Tables**: Columnar; partitioned or clustered; max 10,000 columns; rows up to 100 MB

### Partitioning

| Partition Type | Column Type | Use Case |
|---------------|-------------|---------|
| **Ingestion time** | Automatic (_PARTITIONTIME) | Append-only streaming logs |
| **Column partitioning** | DATE, TIMESTAMP, DATETIME | Query by date range |
| **Integer range** | INTEGER column | Partition by ID range |

- Partitioning prunes scanned data → lower cost + faster queries
- **Partition expiration**: Auto-delete partitions older than N days
- Best practice: Always filter on partition column in WHERE clause

### Clustering

- Sort data within partitions by up to 4 columns
- Reduces bytes scanned for `WHERE`, `GROUP BY`, `ORDER BY` queries on clustered columns
- Works best with high-cardinality columns (e.g., `user_id`, `event_type`)
- Free to add clustering; automatically maintained on DML

### Slots & Reservations

- **Slot**: Unit of BigQuery compute (CPU + memory); 2,000 slots per project by default (on-demand)
- **Reservations**: Purchase dedicated slot capacity (100 slot increments); assign to projects/folders via **Assignments**
- **Commitments**: 1-year or 3-year slot commitments for discount (up to 25-55%)
- **Autoscaling slots**: Baseline + max slots; pay only for slots used above baseline
- **On-demand pricing**: $5/TB scanned (first 1 TB/month free); no slot management needed

### Performance Features

- **BI Engine**: In-memory analytics engine; caches frequently queried data; sub-second query response for Looker Studio/connected BI tools; billed per GB reserved
- **Materialized Views**: Pre-computed query results; auto-refreshed on base table changes; automatic query rewrite to use materialized views
- **BigQuery ML (BQML)**: Train and serve ML models using SQL; supported models: linear regression, logistic regression, k-means, matrix factorization, XGBoost, TensorFlow, Vertex AI AutoML
- **BigQuery Omni**: Run BigQuery queries against data in AWS S3 or Azure Blob Storage without data movement; uses Anthos

### Access Control

- **Authorized Views**: Share query results without exposing underlying tables; grant the view access to underlying datasets
- **Authorized Datasets**: Allow views/routines in one dataset to access tables in another
- **Row-Level Security**: Row access policies filter rows based on user/group identity
- **Column-Level Security**: Policy tags on columns; require DLP classification; restrict who can read sensitive columns
- **Dataset IAM**: Grant `roles/bigquery.dataViewer`, `dataEditor`, `dataOwner` at dataset level

### Other Key Features

- **Data Studio/Looker Studio integration**: Direct connector; use BI Engine for performance
- **Streaming inserts**: Low-latency row-by-row ingestion; best-effort deduplication; costs more than batch
- **BigQuery Storage API**: High-throughput reads from BigQuery into Dataflow, Spark, pandas
- **External tables**: Query data in GCS, Bigtable, Cloud SQL, Google Drive without loading
- **Scheduled queries**: Run SQL on a schedule; results to table; uses BigQuery Data Transfer Service

> **Exam Tip**: Partitioning reduces cost by scanning less data; clustering further reduces cost within partitions. Always combine both for large time-series tables.

> **Gotcha**: BigQuery does NOT update/delete rows efficiently at high frequency — it's optimized for append and analytical read patterns. For frequent row updates, consider Cloud SQL or Spanner.

---

## Dataflow

### Overview

- **Fully managed**, autoscaling data processing service based on **Apache Beam**
- Supports both **batch** and **streaming** pipelines with the same code
- Serverless: Google manages workers, scaling, and autoscaling (Horizontal Worker Autoscaling)
- **Runner**: Dataflow executes Apache Beam pipelines; portable (can also run on Spark, Flink runners)

### Key Concepts

- **Pipeline**: DAG of transforms applied to PCollections
- **PCollection**: Distributed, immutable dataset (bounded for batch; unbounded for streaming)
- **Transforms**: `ParDo` (element-wise), `GroupByKey`, `Combine`, `Flatten`, `Partition`, `CoGroupByKey`
- **Windows**: Tumbling, sliding, session windows for streaming aggregations; watermarks handle late data
- **Triggers**: When to emit results from a window; default: on watermark; can add speculative early firings

### Templates

| Template Type | Description |
|--------------|-------------|
| **Classic Templates** | Pre-built, parameterized; stored as JSON in GCS; limited customization |
| **Flex Templates** | Containerized (Docker); full customization; launched from Artifact Registry |

- Pre-built templates: GCS-to-BigQuery, Pub/Sub-to-BigQuery, Datastream CDC, text transforms

### Runner v2 (Streaming Engine)

- **Streaming Engine**: Moves windowing, grouping, and state/timer management off worker VMs to Google infrastructure; reduces VM size requirements and cost
- Enable with `--experiments=enable_streaming_engine`

> **Exam Tip**: Dataflow is the primary choice for ETL pipelines and streaming analytics on GCP. For Hadoop/Spark workloads that already exist, prefer Dataproc.

> **Gotcha**: Dataflow autoscaling can take several minutes to scale up. For very spiky streaming workloads, pre-provision workers or use `--numWorkers` minimum.

---

## Dataproc

### Overview

- **Managed Hadoop and Spark** service; clusters provision in ~90 seconds
- Supports: Hadoop MapReduce, Spark, Hive, Pig, Flink, Presto, Jupyter
- Significantly cheaper than self-managed Hadoop; integrates with GCS, BigQuery, Bigtable natively
- **Use case**: Lift-and-shift Hadoop/Spark workloads, existing Spark code, teams with Hadoop expertise

### Cluster Types

- **Standard cluster**: Long-running or ephemeral; 1 master + N workers
- **Single-node cluster**: Master + worker on same VM; dev/test only
- **High Availability cluster**: 3 masters; no single point of failure
- **Dataproc Serverless**: Submit Spark jobs without cluster management; auto-scales; billed per DPU-second

### Key Features

- **Autoscaling**: Scale workers based on YARN metrics; set min/max worker counts; requires autoscaling policy
- **Preemptible Workers**: Use Spot VMs for workers; significantly reduces cost; jobs must tolerate worker loss
- **Initialization Actions**: Scripts run on cluster startup; install extra dependencies, configure settings
- **Component Gateway**: Web interfaces (Jupyter, Spark UI, YARN) accessible without SSH tunnel
- **Dataproc Metastore**: Managed Apache Hive Metastore; share metadata across Dataproc clusters and BigQuery

> **Exam Tip**: For **new** data pipelines, prefer Dataflow (managed, serverless Beam). For migrating existing **Spark/Hadoop** code, use Dataproc. Dataproc Serverless is the best of both worlds for Spark.

---

## Pub/Sub

### Core Concepts

- **Fully managed**, globally durable message queue / event streaming service
- At-least-once delivery guarantee by default
- **Topics**: Named channels for publishing messages; producers publish to topics
- **Subscriptions**: Named channels for receiving messages; consumers subscribe; multiple subscriptions per topic = fan-out

### Push vs Pull

| | Pull | Push |
|--|------|------|
| Consumer initiates | Yes | No (Pub/Sub pushes to endpoint) |
| Endpoint | Any (App Engine, GKE, Cloud Run, etc.) | HTTPS endpoint (Cloud Run, App Engine URLs) |
| Scaling control | Consumer controls rate | Pub/Sub controls delivery rate |
| Offline consumers | Supported (messages buffered) | Requires always-on endpoint |
| Back-pressure | Easy | Managed by Pub/Sub retry |

### Advanced Features

- **Dead Letter Topics (DLT)**: Messages that fail delivery N times routed to DLT; set `maxDeliveryAttempts`
- **Message Ordering**: Ordering keys ensure messages with same key delivered in order; adds latency; single partition per key
- **Exactly-Once Delivery**: Available on pull subscriptions; uses ack IDs; higher cost; ~30% throughput reduction
- **Message Retention**: Configurable up to 7 days on topic (default: messages retained until acked on all subscriptions)
- **Snapshot & Seek**: Create point-in-time snapshots; replay messages from snapshot for recovery
- **BigQuery Subscriptions**: Write Pub/Sub messages directly to BigQuery table without Dataflow
- **Cloud Storage Subscriptions**: Write Pub/Sub messages directly to GCS files

### Pricing

- Per message per MB (minimum 1,000 bytes); subscription throughput; data retention fees

> **Exam Tip**: Use BigQuery subscriptions for simple streaming to BigQuery without Dataflow. Use dead letter topics for error handling. Ordering keys cause throughput reduction — only use when needed.

> **Gotcha**: Pub/Sub is NOT a queue — messages are delivered at-least-once by default. Multiple subscriptions on a topic each receive ALL messages independently (fan-out), not split delivery.

---

## Cloud Composer

### Overview

- **Managed Apache Airflow** on GKE; orchestrate complex workflows and pipelines
- **Environments**: Composer 1 (Airflow 1/2) or Composer 2 (Airflow 2; autoscaling workers); Composer 3 (GKE Autopilot-based)
- **DAGs (Directed Acyclic Graphs)**: Python files defining workflow tasks and dependencies
- Workers, schedulers, and web servers run on GKE; DAGs stored in Cloud Storage bucket

### Key Features

- **Operators**: PythonOperator, BashOperator, BigQueryOperator, DataflowTemplateOperator, GCSToGCSOperator, etc.
- **Sensors**: Wait for external events (GCSObjectExistenceSensor, ExternalTaskSensor, etc.)
- **Variables & Connections**: Store config in Airflow UI or via Secret Manager integration
- **Scaling**: Composer 2 autoscales Airflow workers (Celery executor); Composer 3 uses Kubernetes executor

> **Exam Tip**: Cloud Composer is for orchestrating complex multi-step pipelines with dependencies, retries, and scheduling. For simple scheduling, use BigQuery Scheduled Queries or Cloud Scheduler + Cloud Functions.

---

## Looker & Looker Studio

### Looker Studio (formerly Data Studio)

- **Free** BI and dashboarding tool; connects to BigQuery, Cloud SQL, Sheets, and 700+ connectors
- Reports and dashboards; no semantic layer; direct query or extract
- Best for: Ad-hoc reporting, sharing dashboards with non-technical users

### Looker

- **Enterprise BI platform**; semantic layer (LookML models) for consistent business metrics
- **LookML**: SQL-based modeling language defining dimensions, measures, and joins
- Embedded analytics, API access, data governance
- Best for: Large organizations needing governed, consistent metrics across teams
- Looker available on Google Cloud Marketplace or as standalone SaaS

---

## Vertex AI Feature Store & BigQuery ML

- **Vertex AI Feature Store**: Centralized repository for ML features; serve features at low latency for online prediction; offline feature serving via BigQuery; time-travel for training
- **BigQuery ML**: Train/evaluate/predict ML models using SQL; models: linear, logistic, DNN, XGBoost, k-means, ARIMA, AutoML, TensorFlow import; export models to Vertex AI

---

## Decision Matrix: Batch vs Streaming & Data Architecture

| Scenario | Recommended Service |
|----------|-------------------|
| Ad-hoc SQL analytics on petabytes | BigQuery |
| Real-time streaming analytics | Dataflow + Pub/Sub |
| Batch ETL (new pipeline) | Dataflow |
| Batch ETL (existing Spark/Hadoop) | Dataproc |
| Spark jobs without cluster mgmt | Dataproc Serverless |
| Complex multi-system pipeline orchestration | Cloud Composer |
| Simple scheduled query | BigQuery Scheduled Queries |
| Event streaming / message bus | Pub/Sub |
| BI dashboards (free, quick) | Looker Studio |
| Governed enterprise BI | Looker |
| Stream to BigQuery (simple) | Pub/Sub BigQuery Subscription |
| CDC (change data capture) | Datastream → BigQuery/GCS |
| Data warehouse | BigQuery |
| Data lake | Cloud Storage |
| Data lakehouse | Cloud Storage + BigQuery External Tables |
| ML feature serving | Vertex AI Feature Store |

---

## Common Exam Gotchas

1. **BigQuery on-demand** pricing is per TB scanned — partitioning and clustering are critical for cost control
2. **BigQuery slots** are for compute; storage is separate — reservations don't affect storage costs
3. **Dataflow autoscaling** takes minutes — not suitable for sub-second spiky loads
4. **Pub/Sub fan-out**: Each subscription independently receives all messages — not round-robin distribution
5. **Pub/Sub ordering keys** significantly reduce throughput — use only when necessary
6. **Dataproc preemptible workers** can be interrupted mid-job — job must be fault-tolerant or use checkpointing
7. **Cloud Composer** is expensive for simple scheduling needs — consider Cloud Scheduler + Cloud Functions
8. **BigQuery external tables** are slower than native tables — use for exploratory queries, not production dashboards
9. **BI Engine** accelerates specific query patterns — not a replacement for proper partitioning/clustering
