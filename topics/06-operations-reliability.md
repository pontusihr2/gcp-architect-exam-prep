# Domain 6: Ensuring Solution and Operations Reliability

> **Exam Weight: ~14%** — Expect 7–9 questions covering SRE practices, disaster recovery, autoscaling, incident management, and multi-region reliability.

---

## 6.1 Domain Overview

This domain covers how to design and operate highly reliable systems. It combines SRE principles, disaster recovery planning, backup strategies, autoscaling, incident management, and multi-region architectures.

**Key skills tested:**
- Applying SRE principles: error budgets, toil reduction, blameless postmortems
- Designing disaster recovery architectures based on RTO/RPO requirements
- Implementing backup and restore strategies for different services
- Configuring autoscaling for GKE, MIGs, and Cloud Run
- Designing multi-region architectures with load balancing and CDN
- Managing incidents and driving operational improvements

---

## 6.2 SRE Principles and Error Budgets

### Core SRE Principles
| Principle | Description |
|---|---|
| **Embrace risk** | Accept some failure rate; pursuing 100% reliability is too costly |
| **Error budgets** | Quantify acceptable unreliability; align dev and SRE incentives |
| **Eliminate toil** | Automate repetitive manual work; toil should be <50% of SRE time |
| **Monitor for symptoms** | Alert on user-visible impact, not internal causes |
| **Automate responses** | Runbooks → automation; reduce human intervention |
| **Release engineering** | Reliable, repeatable builds and deployments |
| **Simplicity** | Complexity breeds failure; prefer simple, well-understood systems |

### Error Budget Deep Dive

**Error budget = 1 - SLO target**

```
Example: API with 99.9% availability SLO

Monthly error budget:
  Total requests in 30 days at 1,000 req/sec:
    = 1,000 × 60 × 60 × 24 × 30 = 2,592,000,000 requests
  Allowed failures (0.1%):
    = 2,592,000,000 × 0.001 = 2,592,000 failures

Time-based budget:
  Total minutes in 30 days = 43,200
  Allowed downtime (0.1%) = 43.2 minutes

Error budget consumption rate:
  If a 30-minute outage occurs → consumed 69.4% of monthly budget in one incident
```

### Error Budget Policy
| Budget Remaining | Action |
|---|---|
| > 50% | Normal release velocity; new features, experiments |
| 25–50% | Caution; prioritize reliability work alongside features |
| < 25% | Slow releases; mandatory reliability improvements |
| 0% | Freeze all feature deployments; all engineering on reliability |
| Negative (SLO breached) | Incident review; root cause analysis; reliability sprint |

### Toil vs Engineering Work
| Toil | Engineering Work |
|---|---|
| Manual, repetitive tasks | Automation, tooling |
| No lasting value | Reduces future toil |
| Grows with load | Does not grow proportionally |
| Examples: manual deployments, restarting services | Examples: write auto-healing scripts, improve monitoring |

---

## 6.3 Disaster Recovery (DR) Planning

### Key DR Metrics
| Metric | Definition | Examples |
|---|---|---|
| **RTO** (Recovery Time Objective) | Max acceptable time to restore service after a disaster | 4 hours, 15 minutes, 1 minute |
| **RPO** (Recovery Point Objective) | Max acceptable data loss measured in time | 24 hours, 1 hour, 5 minutes, 0 (no data loss) |
| **MTTR** (Mean Time to Recover) | Average time to restore service after a failure | Measured from incident start to resolution |
| **MTBF** (Mean Time Between Failures) | Average time between incidents | Indicates system stability |

### DR Tier Models
| Tier | Strategy | RTO | RPO | Cost |
|---|---|---|---|---|
| **Cold standby** | Backup in another region; restore when needed | Hours–days | Hours–days | Lowest |
| **Warm standby (Pilot Light)** | Core infra running in DR region; scale up on failover | Minutes–hours | Minutes–hours | Medium |
| **Warm standby (Active-Passive)** | Full replica running but not serving traffic | Minutes | Near-zero | High |
| **Hot standby (Active-Active)** | Traffic served from multiple regions simultaneously | Near-zero | Zero | Highest |

### DR Architecture Decision Framework
```
What is the RTO requirement?
  > 4 hours → Cold standby (backups to Cloud Storage + restore scripts)
  1–4 hours → Warm standby / pilot light (replicated DB, IaC to rebuild compute)
  < 1 hour  → Active-passive (full replica, automatic failover)
  < 5 min   → Active-active (global load balancing, multi-region databases)

What is the RPO requirement?
  > 1 hour  → Scheduled backups (Cloud SQL automated backups)
  < 1 hour  → Continuous replication (Cloud SQL read replicas with PITR)
  < 1 min   → Synchronous replication (Cloud Spanner multi-region)
  Zero      → Active-active with synchronous writes (Spanner, Bigtable)
```

### Regional vs Global DR
| Scenario | DR Design |
|---|---|
| Single zone failure | Multi-zone deployment (MIGs, regional GKE clusters) — handled automatically |
| Single region failure | Multi-region deployment with global load balancer |
| Cloud provider failure | Multi-cloud with Anthos / GKE Enterprise |
| Data corruption | Point-in-time recovery (PITR) for Cloud SQL; Cloud Storage versioning |

---

## 6.4 Backup and Restore Strategies

### Cloud SQL Backup and Recovery
| Feature | Description | Retention |
|---|---|---|
| **Automated backups** | Daily snapshots; stored in Cloud Storage (same region) | 1–365 days (default: 7) |
| **On-demand backups** | Manual snapshots at any time | Until manually deleted |
| **Point-in-time recovery (PITR)** | Restore to any second within retention window | Up to 7 days by default |
| **Cross-region replicas** | Async replica in another region; promote for failover | N/A (replica) |

```bash
# Create an on-demand backup
gcloud sql backups create --instance=my-database --async

# Restore to a specific point in time
gcloud sql instances restore-backup my-database \
  --backup-id=BACKUP_ID

# Enable PITR (requires binary logging)
gcloud sql instances patch my-database \
  --enable-bin-log \
  --retained-transaction-log-days=7
```

### Cloud Spanner Backup and Recovery
- **Managed backups:** Full database backups stored in Spanner's internal storage
- Backups are stored in the same instance configuration (multi-region or regional)
- **Cross-region copy:** Copy backups to a different region/configuration for DR

```bash
# Create a Spanner backup
gcloud spanner backups create my-backup \
  --instance=my-instance \
  --database=my-database \
  --expiration-date=2026-01-01T00:00:00Z \
  --async

# Copy backup to another region for DR
gcloud spanner backups copy my-backup-copy \
  --source-instance=my-instance \
  --source-backup=my-backup \
  --source-location=us-central1 \
  --destination-instance=dr-instance \
  --destination-location=us-east1
```

### Cloud Storage Backup Strategies
| Strategy | Implementation | Use Case |
|---|---|---|
| **Object versioning** | `gsutil versioning set on gs://bucket` | Protect against accidental deletion/overwrite |
| **Cross-region replication** | Dual-region or multi-region bucket | Geographic redundancy |
| **Lifecycle policies** | Transition to Archive; auto-delete after N days | Cost-effective long-term retention |
| **Retention locks** | WORM (Write Once Read Many) with retention policy | Compliance; prevent deletion |
| **Soft delete** | 7-day recovery window for deleted objects (default enabled) | Protection against accidental deletes |

### GKE Backup (Backup for GKE)
- Managed backup solution for GKE workloads
- Backs up: Kubernetes resource configuration AND persistent volume data
- Supports scheduled backups and manual backups
- Restore to same or different cluster (cross-region DR for GKE)

```bash
# Install Backup for GKE addon
gcloud container clusters update my-cluster \
  --update-addons BackupRestore=ENABLED \
  --region us-central1

# Create a backup plan
gcloud beta container backup-restore backup-plans create daily-backup \
  --project=my-project \
  --location=us-central1 \
  --cluster=projects/my-project/locations/us-central1/clusters/my-cluster \
  --selected-namespaces=production \
  --cron-schedule="0 2 * * *" \
  --backup-retain-days=30
```

---

## 6.5 Multi-Region Architecture Patterns

### Global Load Balancing
```
Internet
    ↓
Cloud HTTP(S) Load Balancer (Anycast IP — single global IP)
    ↓
URL Map → Backend Service
    ├── Backend: us-central1-mig (20% weight or geolocation-based)
    ├── Backend: europe-west1-mig (40% weight)
    └── Backend: asia-east1-mig (40% weight)

Features:
  - Automatic failover to healthy regions
  - Google's backbone network for low latency
  - SSL termination at the edge
  - Cloud CDN integration
```

### Traffic Distribution Options
| Option | Description | Use Case |
|---|---|---|
| **Geolocation-based routing** | Route to nearest region | Latency optimization |
| **Weighted routing** | Distribute by percentage | Canary deployments, capacity-based |
| **Failover** | Route to backup when primary unhealthy | Active-passive DR |
| **Session affinity** | Route same user to same backend | Stateful apps (avoid if possible) |

### Cloud CDN for Reliability
- Cache static content at **130+ edge Points of Presence (PoPs)** globally
- Reduces load on backend; backend failures less impactful
- **Cache modes:**
  - `CACHE_ALL_STATIC` — cache all static content by default
  - `USE_ORIGIN_HEADERS` — respect Cache-Control headers from origin
  - `FORCE_CACHE_ALL` — cache everything regardless of headers

```bash
# Enable Cloud CDN on a backend service
gcloud compute backend-services update my-backend \
  --enable-cdn \
  --cache-mode CACHE_ALL_STATIC \
  --global
```

### Multi-Region Databases
| Database | Multi-Region Config | SLA | RPO |
|---|---|---|---|
| **Cloud Spanner** | Multi-region instance configs (nam6, eur6, etc.) | 99.999% | Zero (synchronous) |
| **Bigtable** | Multi-cluster routing | 99.99% | Near-zero |
| **Cloud SQL** | Cross-region read replica | 99.95% | Near-zero (async) |
| **Firestore** | Multi-region location (US, EUR) | 99.999% | Zero (synchronous) |
| **Cloud Storage** | Dual-region or multi-region bucket | 99.99% | Zero |

---

## 6.6 GKE Autoscaling

### Three Layers of GKE Autoscaling
```
Layer 1: Pod-level (workload scaling)
  ├── HPA (Horizontal Pod Autoscaler): Scale pod count
  └── VPA (Vertical Pod Autoscaler): Adjust CPU/memory requests

Layer 2: Node-level (cluster scaling)
  └── Cluster Autoscaler: Add/remove nodes based on pending pods

Layer 3: Node pool-level (proactive scaling)
  └── Node Auto-Provisioning (NAP): Create new node pools automatically
```

### Horizontal Pod Autoscaler (HPA)
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-api-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-api
  minReplicas: 2
  maxReplicas: 50
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70  # Scale up when CPU > 70%
    - type: External
      external:
        metric:
          name: pubsub.googleapis.com|subscription|num_undelivered_messages
          selector:
            matchLabels:
              resource.labels.subscription_id: my-subscription
        target:
          type: AverageValue
          averageValue: "30"  # Scale to maintain ~30 messages per pod
```

### Vertical Pod Autoscaler (VPA)
```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: my-app-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  updatePolicy:
    updateMode: "Auto"  # Auto, Recreate, Initial, Off
  resourcePolicy:
    containerPolicies:
      - containerName: app
        minAllowed:
          cpu: 100m
          memory: 50Mi
        maxAllowed:
          cpu: 2
          memory: 2Gi
```

> **Exam Trap:** VPA in `Auto` mode can **restart pods** to apply new resource limits. Do not use VPA and HPA on the same metric — they conflict. HPA on CPU + VPA on memory is acceptable.

### Cluster Autoscaler
```bash
# Enable cluster autoscaler on a node pool
gcloud container node-pools update my-node-pool \
  --cluster my-cluster \
  --region us-central1 \
  --enable-autoscaling \
  --min-nodes 1 \
  --max-nodes 20

# Cluster autoscaler behavior:
#   Scale UP: Triggered when pods are in Pending state due to resource constraints
#   Scale DOWN: Triggered when node utilization < 50% for 10+ minutes AND pods can be rescheduled
```

### Cluster Autoscaler Scale-Down Protection
- Nodes with local storage are not scaled down by default
- Nodes with pods having PodDisruptionBudget violations are protected
- Pods with `cluster-autoscaler.kubernetes.io/safe-to-evict: "false"` annotation block scale-down
- Use **PodDisruptionBudgets (PDBs)** to ensure minimum availability during scale-down:

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: my-api-pdb
spec:
  minAvailable: 2  # Always keep at least 2 pods running
  selector:
    matchLabels:
      app: my-api
```

---

## 6.7 MIG Autoscaling

### MIG Autoscaling Configuration
```bash
# Configure autoscaling for a regional MIG
gcloud compute instance-groups managed set-autoscaling my-mig \
  --region us-central1 \
  --min-num-replicas 2 \
  --max-num-replicas 20 \
  --target-cpu-utilization 0.70 \
  --cool-down-period 90

# Autoscale based on load balancing serving capacity
gcloud compute instance-groups managed set-autoscaling my-mig \
  --region us-central1 \
  --min-num-replicas 2 \
  --max-num-replicas 50 \
  --target-load-balancing-utilization 0.80

# Autoscale based on Cloud Monitoring metric (e.g., Pub/Sub queue depth)
gcloud compute instance-groups managed set-autoscaling my-mig \
  --region us-central1 \
  --min-num-replicas 1 \
  --max-num-replicas 30 \
  --custom-metric-utilization metric=pubsub.googleapis.com/subscription/num_undelivered_messages,utilization-target=100,utilization-target-type=GAUGE
```

### MIG Autohealing
```bash
# Create a health check
gcloud compute health-checks create http my-health-check \
  --port 8080 \
  --request-path /health \
  --check-interval 10s \
  --timeout 5s \
  --healthy-threshold 2 \
  --unhealthy-threshold 3

# Apply autohealing to MIG
gcloud compute instance-groups managed update my-mig \
  --region us-central1 \
  --health-check my-health-check \
  --initial-delay 300  # Wait 300 seconds before first health check (startup time)
```

---

## 6.8 Incident Management

### Incident Response Lifecycle
```
1. DETECT
   ├── Alerting (Cloud Monitoring → PagerDuty/notification channel)
   ├── User reports
   └── Automated anomaly detection

2. TRIAGE
   ├── Assess severity (P1/P2/P3/P4)
   ├── Declare incident if P1/P2
   └── Assign incident commander

3. RESPOND
   ├── Mitigate impact (rollback, redirect traffic, increase capacity)
   ├── Communicate status (status page, stakeholder updates)
   └── Document in incident doc (real-time)

4. RESOLVE
   ├── Confirm service restored
   ├── Verify SLO metrics recovering
   └── Close the incident

5. POSTMORTEM
   ├── Root cause analysis (5 Whys)
   ├── Blameless culture — focus on systems, not people
   ├── Action items with owners and deadlines
   └── Share learnings across teams
```

### Severity Levels (Typical SRE Framework)
| Severity | Impact | Response Time | Example |
|---|---|---|---|
| **P1** | Complete service outage; SLO severely breached | < 5 minutes | Checkout service down |
| **P2** | Significant degradation; subset of users affected | < 15 minutes | Payments failing for 20% of users |
| **P3** | Minor degradation; most users unaffected | < 1 hour | Dashboard loading slowly |
| **P4** | Cosmetic issue; no functional impact | < 24 hours | Report export button misaligned |

### Blameless Postmortems
**Key elements of a good postmortem:**
1. **Timeline:** Exact sequence of events with timestamps
2. **Root cause(s):** What actually caused the incident (not who)
3. **Contributing factors:** What made the impact worse or detection slower
4. **What went well:** Practices that limited impact
5. **What went wrong:** Detection gaps, incorrect assumptions
6. **Action items:** Specific, assignable, time-bound improvements
7. **Lessons learned:** Shareable insights for other teams

**Rule:** Postmortems should be **blameless** — attribute failure to systems and processes, not individuals. This encourages transparency and honesty in analysis.

---

## 6.9 Cloud Monitoring Alerting

### Alerting Policy Structure
```
Alerting Policy
├── Condition(s)
│   ├── Metric threshold (CPU > 80% for 5 minutes)
│   ├── Metric absence (no requests for 10 minutes)
│   ├── Log-based metric (error rate > 10/min)
│   └── Uptime check failure (from 2+ locations)
├── Notification channels
│   ├── Email / SMS
│   ├── PagerDuty / OpsGenie
│   ├── Slack / Google Chat
│   └── Pub/Sub (for automated response)
└── Documentation
    ├── Runbook URL
    └── Alert description
```

### Alert Condition Types
| Type | Use Case |
|---|---|
| **Metric threshold** | CPU > 80%, disk > 90%, error rate > 1% |
| **Metric absence** | No heartbeat signal for 5+ minutes |
| **Log-based metric** | Count of CRITICAL log entries |
| **MQL-based** | Complex multi-metric conditions |
| **SLO burn rate** | Error budget consumed too quickly |
| **Uptime check failure** | External endpoint unavailable |

### SLO Burn Rate Alert Setup
```yaml
# Cloud Monitoring API - burn rate alert
alertPolicies:
  - displayName: "High error budget burn rate"
    conditions:
      - displayName: "Fast burn (14.4x)"
        conditionThreshold:
          filter: |
            select_slo_burn_rate("projects/my-project/services/my-api/serviceLevelObjectives/my-slo", 3600s)
          comparison: COMPARISON_GT
          thresholdValue: 14.4
          duration: 3600s  # Alert if 14.4x burn rate sustained for 1 hour
    notificationChannels:
      - projects/my-project/notificationChannels/pagerduty-oncall
    alertStrategy:
      autoClose: 3600s
```

---

## 6.10 Chaos Engineering

### Chaos Engineering Principles
- **Define steady state:** Establish baseline metrics (p99 latency, error rate, throughput)
- **Hypothesize:** What do you expect to happen when this component fails?
- **Inject failure:** Introduce the failure in a controlled way
- **Observe:** Does the system maintain steady state?
- **Learn:** If not, what needs to be fixed?

### Failure Injection Patterns
| Failure Type | GCP Implementation |
|---|---|
| Kill a pod | `kubectl delete pod [pod-name]` |
| Kill all pods in a deployment | `kubectl delete pods -l app=my-app` |
| Kill a node | Stop a GCE VM in a node pool |
| Introduce latency | Use Istio fault injection (delay) or tc (traffic control) |
| Simulate zone failure | Drain and cordon all nodes in one zone |
| Simulate region failure | Route all traffic away from one region |
| Exhaust disk space | Create large files to fill disk |
| Network partition | Use iptables rules to block traffic |

### Chaos Engineering in Practice
- Start in **non-production** environments
- Limit **blast radius**: target specific services, not the entire production system
- Automate **rollback** mechanisms before running experiments
- **Game Days:** Scheduled exercises where teams simulate disasters and practice response
- Tools: **Chaos Monkey** (Netflix), **Gremlin**, **Litmus** (open source, GKE-native)

---

## 6.11 Cloud Spanner Multi-Region Reliability

### Spanner Multi-Region Configurations
| Config | Regions | SLA | Use Case |
|---|---|---|---|
| `nam6` | N. America (6 regions) | 99.999% | N. American apps |
| `nam-eur-asia1` | 3 continents | 99.999% | Global apps |
| `eur6` | Europe (6 regions) | 99.999% | EU data residency |

### Spanner Replication
- **Paxos-based replication:** Writes committed to majority of replicas before acknowledging
- **Strongly consistent reads:** Can be served from any region
- **Staleness reads:** Optionally read with bounded staleness for lower latency

```python
# Strongly consistent read (any replica, full consistency)
with database.snapshot() as snapshot:
    results = snapshot.execute_sql("SELECT * FROM Orders WHERE id = @id",
                                   params={"id": "12345"},
                                   param_types={"id": spanner.param_types.STRING})

# Bounded staleness read (lower latency, may be slightly stale)
with database.snapshot(max_staleness=datetime.timedelta(seconds=15)) as snapshot:
    results = snapshot.execute_sql("SELECT * FROM Products")
```

---

## 6.12 Bigtable Multi-Cluster Routing

### Bigtable Replication
- Bigtable replicates data across **multiple clusters** (up to 8 clusters per instance)
- Clusters can be in different zones (same region) or different regions
- **Replication lag:** Near-zero (typically milliseconds)

### Routing Policies
| Policy | Description | Use Case |
|---|---|---|
| **Multi-cluster routing** | Automatically route to closest, healthiest cluster | HA and failover |
| **Single-cluster routing** | Always route to specific cluster | Strict consistency requirements |

### Bigtable Cluster Resizing
```bash
# Scale Bigtable cluster nodes (no downtime)
gcloud bigtable clusters update my-cluster \
  --instance my-instance \
  --num-nodes 10

# Bigtable auto-scaling (2025 feature)
gcloud bigtable clusters update my-cluster \
  --instance my-instance \
  --autoscaling-min-nodes 3 \
  --autoscaling-max-nodes 30 \
  --autoscaling-cpu-target 60
```

---

## 6.13 PagerDuty and Incident Tooling Integration

### GCP → PagerDuty Integration
```
Cloud Monitoring Alert
    ↓
Notification Channel: PagerDuty (via webhook or Events API v2)
    ↓
PagerDuty Incident Created
    ├── On-call rotation notified (phone/SMS/push)
    ├── Escalation policy triggered if no ack in X minutes
    └── Incident timeline tracked

Resolution:
  Cloud Monitoring alert resolves
    ↓
  PagerDuty auto-resolves incident
```

### Alert Routing Best Practices
- Route **P1 alerts** to PagerDuty (24/7 on-call)
- Route **P2 alerts** to Slack channel + email
- Route **P3/P4 alerts** to team email or JIRA ticket creation (no on-call wake)
- Use **alert grouping** (Cloud Monitoring or PagerDuty) to reduce noise during major incidents

---

## 6.14 Common Exam Question Patterns and Traps

### Pattern 1: DR Strategy Selection
**Q:** "A financial system requires RPO of 0 and RTO of < 1 minute globally..."
**A:** Cloud Spanner multi-region (synchronous replication, automatic failover) + active-active global load balancer

### Pattern 2: Autoscaling Conflict
**Q:** "GKE pods are not scaling down despite low CPU..."
**A:** Check PodDisruptionBudget settings; check if `safe-to-evict` is false; check if VPA conflicts with HPA

### Pattern 3: Backup for Compliance
**Q:** "Retain audit logs for 7 years in a tamper-proof manner..."
**A:** Log sink to Cloud Storage → enable **retention lock** (WORM) on the bucket → use Archive storage class

### Pattern 4: Zero RPO Requirement
**Q:** "Database must support synchronous replication across two US regions..."
**A:** Cloud Spanner with `nam6` multi-region config (or `nam4` dual-region)

### Pattern 5: Cold Start During Traffic Spike
**Q:** "Cloud Run service has latency spikes when traffic suddenly increases..."
**A:** Set `--min-instances` to keep warm instances; enable CPU Boost for faster cold starts

### Common Traps
- **RTO vs RPO:** RTO = how quickly you recover; RPO = how much data you can lose — don't confuse them
- **Cloud SQL cross-region replica ≠ automatic failover:** Cross-region replicas require manual promotion
- **Cloud Spanner automatic failover:** Spanner DOES automatically failover within a multi-region config
- **HPA + VPA conflict:** Using both on the same resource metric (CPU) causes conflicts — use different metrics
- **Cluster Autoscaler vs Node Auto-Provisioning:** Cluster Autoscaler scales existing pools; NAP creates new pools
- **Backup for GKE is separate from PV backups:** Must explicitly include volume data in backup scope
- **Cold standby cost:** Cheapest DR but RTO is measured in hours — never appropriate for critical systems

---

## 6.15 Summary Table

| Topic | Key Concept | Exam Signal Words |
|---|---|---|
| Error budget | Budget = 1 - SLO; freeze deploys when exhausted | "reliability target", "feature freeze", "SLO breach" |
| RTO/RPO | RTO = recovery time; RPO = data loss tolerance | "acceptable downtime", "data loss", "failover time" |
| Active-active DR | Multi-region + global LB + Spanner | "zero downtime", "RPO = 0", "automatic failover" |
| Cold standby | Backups + restore scripts | "lowest cost DR", "hours to recover" |
| Cloud SQL backup | PITR + cross-region replica | "restore to point in time", "database DR" |
| Spanner multi-region | Paxos replication, 99.999% SLA | "global consistent DB", "zero RPO" |
| GKE HPA | Scale pods on CPU/custom metrics | "horizontal scaling", "Pub/Sub queue depth" |
| GKE VPA | Adjust resource requests automatically | "rightsize pods", "memory OOM" |
| Cluster Autoscaler | Scale nodes on pending pods | "scale down unused nodes", "cost efficiency" |
| PDB | Maintain minimum available pods | "safe scale-down", "disruption budget" |
| Cloud CDN | Cache at edge PoPs | "global performance", "reduce origin load" |
| Incident management | Detect → triage → respond → resolve → postmortem | "blameless", "RCA", "action items" |
| Chaos engineering | Inject failures to test resilience | "Game Days", "failure injection", "test DR" |
| Bigtable autoscaling | Automatic node scaling | "Bigtable scale", "auto-scaling" |
| Log retention | WORM + Archive storage | "tamper-proof", "compliance retention", "7 years" |
