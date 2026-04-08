# Domain 4: Analyzing and Optimizing Technical and Business Processes

> **Exam Weight: ~18%** — Expect 9–12 questions on observability, cost optimization, SRE practices, and FinOps.

---

## 4.1 Domain Overview

This domain covers how to design and operate systems that are observable, cost-effective, and continuously improving. It spans the full observability stack (metrics, logs, traces), SLO-based reliability engineering, cost optimization strategies, and business process improvement.

**Key skills tested:**
- Designing monitoring, logging, and tracing solutions
- Defining and managing SLOs, SLAs, and SLIs
- Implementing cost optimization strategies across compute, storage, and networking
- Using FinOps practices for cloud financial management
- Optimizing performance (caching, database tuning, autoscaling)
- Using data and analytics tools for business process improvement

---

## 4.2 Cloud Monitoring

### Metrics Collection
- **Cloud Monitoring** is the central hub for all GCP metrics, custom metrics, and third-party metrics
- All GCP services emit metrics automatically (compute, storage, databases, networking)
- **Agents:**
  - **Ops Agent:** Single agent for VMs that collects both metrics and logs; replaces legacy Monitoring and Logging agents
  - Install on GCE VMs and on-premises servers for full observability

### Metric Types
| Type | Description | Example |
|---|---|---|
| **Gauge** | Current value at a point in time | CPU utilization, memory used |
| **Delta** | Change since last measurement | Request count per interval |
| **Cumulative** | Monotonically increasing total | Total bytes sent |

### Custom Metrics
- Use Cloud Monitoring API or OpenTelemetry to create custom metrics
- Business metrics: orders per minute, payment failures, active users
- Custom metrics are billed after the free tier (non-billable metrics from GCP services are free)

```python
from google.cloud import monitoring_v3
import time

client = monitoring_v3.MetricServiceClient()
project_name = f"projects/my-project"

series = monitoring_v3.TimeSeries()
series.metric.type = "custom.googleapis.com/orders/per_minute"
series.metric.labels["payment_method"] = "credit_card"
series.resource.type = "global"

now = time.time()
point = monitoring_v3.Point({
    "interval": {"end_time": {"seconds": int(now)}},
    "value": {"int64_value": 42}
})
series.points = [point]

client.create_time_series(name=project_name, time_series=[series])
```

### Monitoring Dashboards
- Create custom dashboards with charts for metrics, logs, and SLOs
- Use **Monitoring Query Language (MQL)** or PromQL for advanced metric queries
- Pre-built dashboards available for GCE, GKE, Cloud SQL, Pub/Sub, etc.
- Share dashboards via public URL or restrict to org members

### Uptime Checks
- Monitor external availability of URLs, IPs, or port-based services
- Check from multiple global locations simultaneously
- Alert if check fails from N out of M locations
- Creates synthetic monitoring — verifies user-facing availability, not just backend health

```yaml
# Uptime check config
displayName: "Production API Health"
monitoredResource:
  type: "uptime_url"
  labels:
    host: "api.myapp.com"
httpCheck:
  path: "/health"
  port: 443
  useSsl: true
  validateSsl: true
period: 60s
timeout: 10s
```

---

## 4.3 Cloud Logging

### Log Collection Architecture
```
Log Sources:
  ├── GCP Services (auto-collected)
  ├── Compute Engine VMs (via Ops Agent)
  ├── GKE pods (via Cloud Logging integration)
  ├── Applications (using Cloud Logging client libraries)
  └── On-premises / other clouds (via BindPlane or Logging API)
        ↓
Cloud Logging Ingestion
        ↓
Log Router (filters + sinks)
  ├── _Default bucket (30 days, standard storage)
  ├── _Required bucket (400 days, audit logs — cannot delete)
  ├── Custom bucket (configurable retention)
  └── Log Sinks:
      ├── Cloud Storage (archive)
      ├── BigQuery (analysis)
      └── Pub/Sub (SIEM / real-time)
```

### Log Sinks Configuration
```bash
# Create a sink to export audit logs to BigQuery for analysis
gcloud logging sinks create audit-to-bigquery \
  bigquery.googleapis.com/projects/my-project/datasets/audit_logs \
  --log-filter='logName:"cloudaudit.googleapis.com"' \
  --include-children \
  --organization=123456789

# Grant the sink's service account BigQuery Data Editor role
# (gcloud returns the SA email after sink creation)
gcloud projects add-iam-policy-binding my-project \
  --member "serviceAccount:SINK_SA_EMAIL" \
  --role "roles/bigquery.dataEditor"
```

### Log-Based Metrics
- Create **counter metrics** and **distribution metrics** from log entries
- Use for alerting on log patterns (e.g., error rate > 5/minute)
- Example: Count 5xx errors in access logs → alert if rate > threshold

```bash
# Create a log-based metric for HTTP 500 errors
gcloud logging metrics create http-500-errors \
  --description="Count of HTTP 500 errors" \
  --log-filter='resource.type="cloud_run_revision" httpRequest.status>=500'
```

### Log Retention Policies
| Log Bucket | Default Retention | Customizable? |
|---|---|---|
| `_Required` | 400 days | No (locked) |
| `_Default` | 30 days | Yes (1–3650 days) |
| Custom buckets | 1–3650 days | Yes |
| Cloud Storage sink | Object lifecycle policy | Yes (up to 7 years for compliance) |

### Structured Logging Best Practices
- Use **JSON-format logs** with consistent field names: `severity`, `message`, `timestamp`, `trace`, `labels`
- Include **trace IDs** in logs to correlate with Cloud Trace
- Include **request IDs** for end-to-end correlation across microservices

---

## 4.4 Cloud Trace

### What Cloud Trace Does
- **Distributed tracing** for latency analysis across microservices
- Automatically instruments: App Engine, Cloud Run, Cloud Functions, GKE (with OpenTelemetry)
- Shows end-to-end latency, breakdown by service, and identifies slow spans

### Trace Integration
```python
from opentelemetry import trace
from opentelemetry.exporter.cloud_trace import CloudTraceSpanExporter
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor

# Configure Cloud Trace exporter
tracer_provider = TracerProvider()
cloud_trace_exporter = CloudTraceSpanExporter()
tracer_provider.add_span_processor(BatchSpanProcessor(cloud_trace_exporter))
trace.set_tracer_provider(tracer_provider)

tracer = trace.get_tracer(__name__)

def process_order(order_id: str):
    with tracer.start_as_current_span("process_order") as span:
        span.set_attribute("order.id", order_id)
        # ... business logic
        with tracer.start_as_current_span("validate_payment"):
            # ... payment validation
            pass
```

### Trace Analysis Use Cases
- Identify the **slowest 99th percentile requests**
- Find which microservice is the **bottleneck**
- Correlate latency with specific code versions after deployment
- Set up **latency SLOs** using trace data

---

## 4.5 Cloud Profiler

### What Cloud Profiler Does
- **Continuous production profiler** — minimal overhead (~0.5%)
- Profiles: CPU time, heap allocation, threads, contention
- Compares profiles across versions to catch performance regressions
- Supports: Go, Java, Python, Node.js, Ruby

### When to Use Profiler
- Diagnosing high CPU usage on Cloud Run or GKE
- Identifying memory leaks causing OOM restarts
- Optimizing hot code paths in microservices
- Pre/post-deployment comparison after code changes

---

## 4.6 Error Reporting

### What Error Reporting Does
- Automatically aggregates and deduplicates exceptions across services
- Groups similar errors into **error groups**
- Tracks first occurrence, last occurrence, and occurrence frequency
- Integrates with Cloud Logging — parses stack traces automatically
- Sends notifications for new error types

### Error Reporting Integration
- **Automatic:** App Engine, Cloud Run, Cloud Functions (no code changes)
- **Manual:** Use `google-cloud-error-reporting` client library
- Errors visible in console and can trigger alerts via Cloud Monitoring

---

## 4.7 SLO, SLA, and SLI Framework

### Definitions
| Term | Full Name | Definition |
|---|---|---|
| **SLI** | Service Level Indicator | The metric you measure (e.g., % of successful requests) |
| **SLO** | Service Level Objective | The target for that metric (e.g., 99.9% of requests successful) |
| **SLA** | Service Level Agreement | Legal/business contract — consequences if SLO is breached |
| **Error Budget** | — | Time (or requests) you can fail while still meeting the SLO |

### Error Budget Calculation
```
SLO = 99.9% availability over 30 days

Total minutes in 30 days = 43,200 minutes
Allowed downtime (0.1%) = 43.2 minutes

Error budget = 43.2 minutes/month
  ├── If remaining budget is high → safe to deploy faster
  └── If budget nearly exhausted → freeze deployments, focus on reliability
```

### SLI Types and Corresponding Metrics
| SLI Type | Metric Example | Good For |
|---|---|---|
| **Availability** | HTTP 2xx responses / total responses | APIs, web services |
| **Latency** | % requests served in < 200ms | Interactive apps |
| **Error rate** | Error responses / total responses | Any service |
| **Throughput** | Messages processed / second | Streaming pipelines |
| **Freshness** | Age of most recently updated data | Data pipelines |
| **Durability** | % of data chunks successfully stored | Storage systems |

### Cloud Monitoring SLO Configuration
```yaml
# Define an SLO in Cloud Monitoring
displayName: "API Availability SLO"
goal: 0.999  # 99.9%
calendarPeriod: MONTH
serviceLevelIndicator:
  requestBased:
    goodTotalRatio:
      goodServiceFilter: |
        metric.type="run.googleapis.com/request_count"
        metric.labels.response_code_class="2xx"
      totalServiceFilter: |
        metric.type="run.googleapis.com/request_count"
```

### SRE Principles for the Exam
- **Embrace risk:** 100% reliability is impossible and too expensive; accept some failure
- **Error budgets:** Shared accountability between dev (features) and SRE (reliability)
- **Toil reduction:** Automate repetitive manual work; toil should be < 50% of SRE time
- **Monitoring simplicity:** Alert only on symptoms (user impact), not causes
- **Postmortems:** Blameless; focus on systemic issues, not individual blame

---

## 4.8 Cost Optimization Strategies

### Compute Cost Optimization

#### Committed Use Discounts (CUDs)
| Type | Discount | Commitment | Flexibility |
|---|---|---|---|
| **Resource-based CUD** | Up to 57% (1yr), 70% (3yr) | Specific VM family in specific region | Not flexible |
| **Spend-based CUD** | Up to 25% (1yr), 52% (3yr) | Dollar amount spent on compute | Flexible across VM types |

#### Preemptible vs Spot VMs
| Type | Price | Preemption Notice | Max Runtime |
|---|---|---|---|
| **Preemptible** | ~80% discount | 30 seconds | 24 hours |
| **Spot** | ~60–91% discount | 30 seconds | None (unlimited) |

**Spot VMs** are the successor to Preemptible VMs — lower cost, no 24-hour limit.

**When to use Spot/Preemptible VMs:**
- Batch processing jobs (ML training, data pipelines)
- Stateless web servers with proper shutdown handling
- GKE worker nodes for non-critical workloads
- CI/CD build agents

#### Rightsizing
- Use **VM Recommender** in Cloud Console to identify oversized VMs
- Cloud Monitoring CPU/memory metrics to baseline actual usage
- Use **custom machine types** to precisely match workload requirements (avoid over-provisioning)
- Example: Replace `n2-standard-8` (8 vCPU, 32 GB) with `n2-custom-6-20480` if workload uses 6 vCPU and 20 GB

### Storage Cost Optimization

#### Cloud Storage Lifecycle Policies
```json
{
  "rule": [
    {
      "action": {"type": "SetStorageClass", "storageClass": "NEARLINE"},
      "condition": {"age": 30, "matchesStorageClass": ["STANDARD"]}
    },
    {
      "action": {"type": "SetStorageClass", "storageClass": "COLDLINE"},
      "condition": {"age": 90, "matchesStorageClass": ["NEARLINE"]}
    },
    {
      "action": {"type": "SetStorageClass", "storageClass": "ARCHIVE"},
      "condition": {"age": 365, "matchesStorageClass": ["COLDLINE"]}
    },
    {
      "action": {"type": "Delete"},
      "condition": {"age": 2555}  // Delete after 7 years
    }
  ]
}
```

#### BigQuery Cost Optimization
| Strategy | Savings | Implementation |
|---|---|---|
| **Columnar queries** | High | SELECT only needed columns (avoid `SELECT *`) |
| **Partitioned tables** | High | Partition by date; queries scan only needed partitions |
| **Clustered tables** | Medium | Cluster by frequently-filtered columns |
| **Slot commitments** | ~25–50% | Purchase slot commitments for predictable workloads |
| **BigQuery editions** | Variable | Standard/Enterprise/Enterprise Plus based on features needed |
| **Table expiration** | Variable | Auto-delete temporary tables after TTL |
| **BI Engine** | (performance) | In-memory cache for Looker Studio reports |

#### Persistent Disk Optimization
- Use **pd-standard** (HDD) for archival data instead of pd-ssd
- Use **Hyperdisk** only for high IOPS workloads
- Set disk size accurately — do not massively over-provision SSD disks

### Networking Cost Optimization
- **Egress costs:** Moving data OUT of GCP regions costs money; design to minimize cross-region data movement
- **Cloud CDN:** Cache static content at edge; reduces load balancer backend traffic (and cost)
- **Cloud Interconnect vs VPN:** Interconnect has lower egress costs if volume is high
- **Internal load balancers:** No egress charges for internal traffic vs external LB

### GKE Cost Optimization
- Use **Autopilot** to pay per pod CPU/memory (not per node)
- Enable **cluster autoscaler** to scale down unused nodes
- Use **Spot node pools** for fault-tolerant workloads
- Use **resource quotas and limits** to prevent runaway resource consumption
- Implement **namespace-level cost attribution** with cost allocation labels

---

## 4.9 FinOps Practices

### FinOps Framework Applied to GCP
| FinOps Phase | GCP Tools |
|---|---|
| **Inform** | Billing reports, billing export to BigQuery, Cost Table, label-based reports |
| **Optimize** | VM Recommender, Idle Resource Recommender, CUDs, lifecycle policies |
| **Operate** | Budget alerts, FinOps team workflows, showback/chargeback models |

### Showback vs Chargeback
- **Showback:** Report cloud spend per team/project without billing them directly (awareness)
- **Chargeback:** Allocate cloud costs to internal teams/cost centers (accountability)
- GCP implementation:
  - Labels (`team`, `app`, `env`) → billing export to BigQuery → Looker Studio dashboard
  - Budget alerts at project or label level for automated notifications

### GCP Cost Management Tools
| Tool | Purpose |
|---|---|
| **Billing Reports** | Visual overview of current spend |
| **Billing Export to BigQuery** | Detailed analysis, custom queries |
| **Cost Breakdown** | Hierarchical view by service |
| **Recommenders** | AI-driven cost and performance recommendations |
| **Committed Use Discounts (CUDs)** | Discounts for committed resource usage |
| **Sustained Use Discounts (SUDs)** | Automatic discount for long-running VMs (GCE only) |
| **Active Assist** | Collection of all recommenders in one place |

### Sustained Use Discounts (SUDs)
- **Automatic** — no action required
- Applied when a GCE VM (N1, N2, N2D) runs for more than 25% of a month
- Up to 30% discount at 100% usage in a month
- Does **NOT** apply to: E2 VMs, A2 (GPU), committed use discounted instances

---

## 4.10 Performance Optimization

### Caching Strategies
| Cache Type | GCP Service | Typical Benefit |
|---|---|---|
| **CDN caching** | Cloud CDN | Serve static content from edge; reduce origin load by 80%+ |
| **Application cache** | Memorystore (Redis/Valkey) | Sub-millisecond reads; reduce DB load |
| **Database query cache** | BI Engine (BigQuery) | In-memory analytics cache; faster dashboard queries |
| **DNS caching** | Cloud DNS | Reduce DNS lookup latency |

### Database Performance
- **Read replicas:** Offload read traffic from Cloud SQL primary
- **Bigtable row key design:** Distribute writes evenly; avoid hotspotting
- **BigQuery partitioning:** Reduce bytes scanned; dramatically faster queries
- **Spanner interleaved tables:** Co-locate child rows with parent rows; reduce I/O
- **Connection pooling:** Use PgBouncer for Cloud SQL PostgreSQL or Cloud SQL connector

### Autoscaling for Performance
- **GKE HPA:** Scale pods based on CPU/memory or custom metrics (e.g., Pub/Sub queue depth)
- **GKE VPA:** Automatically adjust pod resource requests/limits based on actual usage
- **Cloud Run:** Scales to 0 and up to 1,000 instances automatically
- **MIG autoscaling:** Scale VM instances based on Cloud Monitoring metrics

---

## 4.11 Business Process Optimization

### Google Workspace + GCP Integration
- **AppSheet:** No-code app builder using Google Sheets or Drive data as backend
- **Google Forms + Apps Script + BigQuery:** Automate data collection and analysis workflows
- **Looker / Looker Studio:** BI and reporting connected to BigQuery, Cloud SQL, etc.
- **Google Meet / Chat APIs:** Automate notifications from GCP events

### Data-Driven Decision Making Architecture
```
Raw Data Sources (databases, logs, APIs)
    ↓
Ingestion (Pub/Sub, Cloud Storage, Cloud Data Fusion)
    ↓
Processing (Dataflow, Dataproc, BigQuery ML)
    ↓
Storage (BigQuery data warehouse)
    ↓
Reporting (Looker Studio dashboards, Looker LookML models)
    ↓
Decision (business stakeholders, automated alerts)
```

### Looker vs Looker Studio
| Feature | Looker | Looker Studio (formerly Data Studio) |
|---|---|---|
| **Target audience** | Enterprise analytics teams | Business users, small teams |
| **Data modeling** | LookML semantic layer | Direct connection |
| **Cost** | Paid (per user) | Free |
| **Embedded analytics** | Yes (Looker Embed) | Limited |
| **Governance** | Strong (semantic layer) | Weak (each report independent) |

> **Exam Tip:** If the question mentions "consistent business definitions across the company" or "governed metrics," the answer is **Looker** (with LookML). For quick self-service dashboards, Looker Studio is fine.

---

## 4.12 Alerting Policy Design

### Alerting Best Practices (SRE Approach)
- **Alert on symptoms, not causes:** Alert on "high error rate" (user impact), not "high CPU" (potential cause)
- **Page only for actionable alerts:** Non-actionable alerts cause alert fatigue
- **Set appropriate thresholds:** Base on SLO burn rate, not arbitrary values

### SLO Burn Rate Alerting
Burn rate = how fast you're consuming your error budget relative to the SLO window
```
SLO: 99.9% availability (30-day window)
Error budget: 43.2 minutes

Burn rate alerts:
  - 14.4x burn rate for 1 hour → page immediately (consumes 2% of monthly budget in 1 hour)
  - 6x burn rate for 6 hours → page (consumes 5% of monthly budget in 6 hours)
  - 3x burn rate for 3 days → ticket (slow burn, catches gradual degradation)
```

### Alert Notification Channels
| Channel | Use Case |
|---|---|
| Email | Low-urgency notifications |
| SMS | High-urgency, simple message |
| PagerDuty | On-call rotation management |
| Slack / Google Chat | Team notifications |
| Pub/Sub | Programmatic / automated response |
| Webhooks | Custom integrations |

---

## 4.13 Common Exam Question Patterns and Traps

### Pattern 1: Cost Reduction
**Q:** "Reduce cost of ML training jobs that can tolerate interruption..."
**A:** Move training to Spot VMs or Spot node pools on GKE; save up to 91%

### Pattern 2: SLO Breach Response
**Q:** "Error budget is 80% consumed with 2 weeks left in the month..."
**A:** Freeze feature deployments; focus engineering on reliability improvements

### Pattern 3: Log Analysis
**Q:** "Need to analyze 90 days of application logs with SQL queries..."
**A:** Create a log sink to BigQuery; use SQL queries for analysis; set log retention to 90+ days

### Pattern 4: Performance Bottleneck
**Q:** "Application latency spike identified but cause unclear..."
**A:** Use Cloud Trace to identify slow spans; use Cloud Profiler to find CPU/memory bottlenecks

### Common Traps
- **SUD vs CUD:** SUDs are automatic; CUDs require commitment. SUDs don't apply to E2 VMs.
- **Preemptible ≠ Spot:** Preemptible has a 24-hour max runtime; Spot VMs do not
- **Log-based metrics vs custom metrics:** Log-based metrics are derived from logs; custom metrics require code instrumentation
- **Looker vs Looker Studio:** Looker = governed enterprise BI; Looker Studio = free, self-service
- **Uptime checks ≠ health checks:** Uptime checks are external synthetic monitors; health checks are internal LB health probes
- **Data Access audit logs must be explicitly enabled:** Not on by default; forgetting this is a compliance risk

---

## 4.14 Summary Table

| Topic | Key Concept | Exam Signal Words |
|---|---|---|
| Observability stack | Monitoring + Logging + Trace + Profiler | "end-to-end visibility", "latency", "errors" |
| SLO/error budget | Error budget = allowed failure | "reliability target", "deploy cadence", "burn rate" |
| SLI types | Availability, latency, error rate | "measure", "track", "indicator" |
| Cost: compute | CUDs, Spot VMs, rightsizing | "reduce cost", "batch", "predictable load" |
| Cost: storage | Lifecycle policies, storage classes | "archive", "infrequent access", "reduce storage cost" |
| Cost: BigQuery | Partitioning, clustering, slot CUDs | "BigQuery cost", "scan reduction", "analytics" |
| FinOps | Labels + BigQuery billing export | "showback", "chargeback", "team cost" |
| Performance | Cloud CDN, Memorystore, HPA | "latency", "cache", "scale out" |
| Alerting | Burn rate alerts, symptom-based | "alert fatigue", "SLO-based alerts", "on-call" |
| Log routing | Log sinks to BQ/GCS/Pub/Sub | "export logs", "archive", "analyze logs" |
| Profiling | Cloud Profiler | "CPU bottleneck", "memory leak", "production" |
| Tracing | Cloud Trace + OpenTelemetry | "distributed tracing", "microservices latency" |
| Dashboards | Looker (governed) vs Looker Studio | "business metrics", "consistent definitions" |
| Sustained discounts | SUDs (automatic) | "long-running VM", "no commitment needed" |
