# Key Terms & Concepts Glossary
> Google Professional Cloud Architect — 2025 Edition

---

## 1. Availability & Recovery Terminology

### RTO, RPO, RTA

| Term | Full Name | Definition | Typical Target Values |
|---|---|---|---|
| **RTO** | Recovery Time Objective | Maximum acceptable time for a system to be down after a failure event. "How long can we be offline?" | Mission-critical: seconds–minutes; Standard: hours; Non-critical: days |
| **RPO** | Recovery Point Objective | Maximum acceptable data loss measured in time. "How much data can we afford to lose?" | Financial: ~0 (no data loss); Standard: 1–24 hours; Archival: 24+ hours |
| **RTA** | Recovery Time Actual | The actual measured time it took to restore service after an incident. Compared against RTO to evaluate DR effectiveness. | N/A (measured, not set) |
| **MTTR** | Mean Time to Repair | Average time to repair a system after failure (including detection + diagnosis + fix) | Target < 1 hour for high-availability systems |
| **MTBF** | Mean Time Between Failures | Average time between recoverable failures — a measure of reliability | Higher = more reliable |
| **MAO** | Maximum Acceptable Outage | Business-defined limit — equivalent to RTO in DR planning | Set by business, not technology |

### DR Strategy vs RTO/RPO
| Strategy | RTO | RPO | Relative Cost |
|---|---|---|---|
| Backup & Restore | Hours–Days | Hours | Low |
| Pilot Light | Hours | Minutes–Hours | Medium-Low |
| Warm Standby | Minutes–Hours | Seconds–Minutes | Medium-High |
| Active-Active (Hot) | Seconds | Near-zero | High |

---

## 2. SRE & Reliability Terminology

### SLI, SLO, SLA, Error Budget

| Term | Full Name | Definition | Example |
|---|---|---|---|
| **SLI** | Service Level Indicator | A quantitative measure of service behavior — the actual metric | "99.95% of requests return HTTP 200 in < 200ms over 30 days" |
| **SLO** | Service Level Objective | The target value for an SLI — your internal reliability goal | "SLO: 99.9% successful requests per rolling 30 days" |
| **SLA** | Service Level Agreement | A contractual commitment to customers with consequences for breach (e.g., credits) | "We guarantee 99.9% uptime; breach triggers billing credit" |
| **Error Budget** | — | Allowed amount of unreliability = 1 - SLO. Consumed by incidents/deployments | SLO 99.9% → 0.1% error budget = 43.8 minutes/month |

### Availability Nines Reference
| Availability | Annual Downtime | Monthly Downtime |
|---|---|---|
| 99% ("two nines") | 3.65 days | 7.3 hours |
| 99.9% ("three nines") | 8.76 hours | 43.8 minutes |
| 99.95% | 4.38 hours | 21.9 minutes |
| 99.99% ("four nines") | 52.6 minutes | 4.4 minutes |
| 99.999% ("five nines") | 5.26 minutes | 26.3 seconds |

### Error Budget Policy
- If error budget is **healthy** (budget remaining) → teams can deploy freely
- If error budget is **low** (< 10% remaining) → feature freeze; focus on reliability
- If error budget is **exhausted** → no deployments until next period; root cause analysis required

---

## 3. Distributed Systems Theory

### CAP Theorem
A distributed system can guarantee at most **two** of three properties simultaneously:

| Property | Description |
|---|---|
| **C — Consistency** | Every read receives the most recent write or an error (all nodes see same data at same time) |
| **A — Availability** | Every request receives a response (not necessarily the most recent data) |
| **P — Partition Tolerance** | System continues to operate despite network partitions (message drops between nodes) |

> **In practice**: Network partitions WILL happen. So real systems choose between **CP** (consistency + partition tolerance) or **AP** (availability + partition tolerance).

| GCP Service | CAP Classification | Notes |
|---|---|---|
| Cloud Spanner | CP (with external consistency) | Chooses consistency over availability during partitions |
| Firestore | CP (strong mode default) | Strong consistency by default |
| Bigtable | CP (row-level) | Strong within a row |
| Cloud Storage | AP (with strong for single objects) | Eventually consistent for list operations |

### ACID vs BASE

| Property | ACID | BASE |
|---|---|---|
| **Model** | Traditional relational DB transactions | NoSQL / distributed systems |
| **Stands for** | Atomicity, Consistency, Isolation, Durability | Basically Available, Soft state, Eventual consistency |
| **Atomicity** | All or nothing — transaction fully commits or fully rolls back | Not guaranteed; partial updates possible |
| **Consistency** | Data always in valid state per schema constraints | "Eventually" consistent — may read stale data temporarily |
| **Isolation** | Concurrent transactions appear sequential | No isolation guarantees by default |
| **Durability** | Committed data persists through failures | Durability varies by implementation |
| **GCP Examples** | Cloud SQL, Spanner, AlloyDB | Bigtable (row-level ACID only), Firestore (eventually consistent reads optional) |

---

## 4. Cost & Financial Terminology

### TCO (Total Cost of Ownership)
All costs of owning and operating a system over its lifetime:
- **Direct costs**: Compute, storage, network, licenses
- **Indirect costs**: Operations staff, training, facilities, power (on-prem), DR infrastructure
- **Opportunity costs**: Engineering time spent on infrastructure vs. product features
- **Migration costs**: One-time costs to move to cloud

**Cloud TCO considerations:**
- CapEx (on-prem) → OpEx (cloud) shift — affects accounting and budgeting
- Include: Reserved/Committed Use Discounts, Sustained Use Discounts, egress costs
- Don't forget: Support contracts, IAM management overhead, data transfer costs

### Discount Types

| Discount Type | Full Name | How It Works | Applies To |
|---|---|---|---|
| **CUD** | Committed Use Discount | Commit to use specific resources for 1 or 3 years; up to 57% off | GCE, Cloud SQL, AlloyDB, GKE (node VMs) |
| **SUD** | Sustained Use Discount | Automatic discount for running VMs > 25% of month; up to 30% | GCE (N1/N2 general-purpose, not Spot) |
| **Spot VMs** | (formerly Preemptible) | Up to 91% discount; can be preempted with 30-sec notice | GCE Spot VMs, GKE spot node pools |
| **Flex CUD** | Flexible CUD | Commit to spend (not specific machine type); 1-year term | GCE flexible compute commitment |

**CUD vs SUD:**
- CUDs are explicit commitments; SUDs are automatic
- CUDs apply to specific machine families; SUDs apply to all qualifying GCE
- CUDs and SUDs cannot both apply to the same VM resource

### FinOps Terminology
| Term | Definition |
|---|---|
| **FinOps** | Operational framework for cloud financial management — collaboration between Finance, Engineering, and Business |
| **Cost Attribution** | Tagging resources with labels to assign costs to teams/products/environments |
| **Showback** | Reporting cloud costs to teams without charging them (awareness without accountability) |
| **Chargeback** | Actually charging internal teams for their cloud resource consumption |
| **Cloud Budget Alerts** | Alerting when spend exceeds % of budgeted amount (GCP: Budget & Alerts) |
| **Recommender** | GCP service providing right-sizing, idle resource, and IAM recommendations |
| **Active Assist** | Umbrella for GCP's ML-powered recommendations (including Recommender) |
| **Unit Economics** | Cost per business metric (e.g., cost per transaction, cost per user, cost per GB processed) |

---

## 5. Architecture Frameworks

### Google Cloud Well-Architected Framework — 6 Pillars

| Pillar | Focus | Key GCP Tools |
|---|---|---|
| **Operational Excellence** | Automate operations; monitor effectively; learn from incidents | Cloud Monitoring, Cloud Logging, Cloud Trace, Error Reporting, Deployment Manager, Terraform |
| **Security, Privacy & Compliance** | Protect data & systems; meet regulatory requirements | IAM, VPC SC, Cloud Armor, Secret Manager, Security Command Center, DLP API |
| **Reliability** | Build resilient, fault-tolerant systems | SLOs, multi-region deployments, Cloud Spanner, Cloud Run, global LBs |
| **Cost Optimization** | Use resources efficiently; avoid waste | CUDs, Spot VMs, Autoclass, Recommender, Budget Alerts, Committed Use |
| **Performance Optimization** | Achieve target performance; scale appropriately | Cloud CDN, Cloud Armor, Autoscaling, Memorystore, BigQuery slots |
| **Sustainability** | Reduce environmental impact | Google Carbon Footprint tool, efficient regions (low-carbon), right-sizing |

> **Note:** Some resources list 5 pillars (omitting Sustainability). The 2024/2025 framework includes Sustainability as the 6th pillar.

---

## 6. Compliance & Regulatory Concepts

### HIPAA (Health Insurance Portability and Accountability Act)
- US law protecting **Protected Health Information (PHI)**
- GCP is HIPAA-eligible (requires signing a Business Associate Agreement / BAA with Google)
- Key requirements: Access controls, audit logging, encryption at rest and in transit, PHI not stored in non-HIPAA eligible services
- **GCP HIPAA-eligible services**: GCE, GCS, BigQuery, Cloud SQL, GKE, Cloud Run, Pub/Sub (and others — check BAA)
- **Exam Tip**: If question involves healthcare data → look for answers with encryption, access logging, and BAA mention

### PCI-DSS (Payment Card Industry Data Security Standard)
- Standard for handling **cardholder data** (credit/debit card numbers, CVV, expiry)
- 12 requirements including: network segmentation, encryption, access control, vulnerability scanning
- GCP has many PCI-DSS compliant services (check Compliance Reports Manager)
- **Key architecture**: VPC with strict firewall rules, VPC Service Controls, Cloud Armor WAF, DLP API to detect/mask card data, Cloud Audit Logs
- **Scope reduction**: Use tokenization or encryption to reduce PCI scope (data outside cardholder data environment)

### GDPR (General Data Protection Regulation)
- EU regulation for **personal data** of EU residents
- Key principles: data minimization, purpose limitation, right to erasure, data portability, breach notification (72 hours)
- **GCP tools**: DLP API (detect PII), Cloud KMS (customer-managed encryption keys), org policy for data residency (`constraints/gcp.resourceLocations`), Access Transparency, deletion policies on GCS/BigQuery
- **Data Residency**: Use region-locked resource policies to ensure EU data stays in EU regions
- **Right to Erasure**: Design deletion workflows; Cloud Spanner / BigQuery point-in-time recovery complicates this — use TTL policies

### Shared Responsibility Model on GCP

```
Customer Responsibility:                Google Responsibility:
├─ Data (content/classification)        ├─ Physical security (data centers)
├─ Access policies (IAM)                ├─ Hardware
├─ Application-level security           ├─ Network infrastructure
├─ Guest OS (GCE only)                  ├─ Hypervisor
├─ Network config (firewall rules)      ├─ Managed service software
├─ Client-side encryption (optional)    ├─ Encryption at rest (default)
└─ Compliance configurations            └─ Encryption in transit (default)
```

| Service Type | Customer Manages | Google Manages |
|---|---|---|
| **IaaS (GCE)** | OS, patches, app, data, IAM | Hardware, hypervisor, physical |
| **PaaS (App Engine)** | App code, IAM, data | OS, runtime, middleware, hardware |
| **SaaS (BigQuery)** | Data, IAM, query security | Everything else |

---

## 7. Multi-Tenancy Concepts

| Concept | Definition | GCP Mechanism |
|---|---|---|
| **Multi-tenancy** | Multiple customers/teams share the same infrastructure | GKE namespaces, Shared VPC, org resource hierarchy |
| **Tenant isolation** | Ensuring one tenant cannot access another's data/resources | VPC SC, namespace policies in GKE, separate projects per tenant |
| **Noisy neighbor** | One tenant's resource usage degrading performance for others | GKE resource quotas, BigQuery slot reservations, committed use |
| **Soft multi-tenancy** | Tenants trust each other; lightweight isolation (namespaces) | GKE namespaces with RBAC |
| **Hard multi-tenancy** | Tenants don't trust each other; strong isolation required | Separate GKE clusters, separate projects, VPC SC |

---

## 8. Additional Key Architecture Concepts

### Idempotency
- An operation that produces the same result whether executed once or multiple times
- Critical for: Pub/Sub message processing (at-least-once delivery), Cloud Tasks, retry logic
- Design APIs and functions to be idempotent to safely retry failed operations

### Blue/Green vs Canary Deployments
| Strategy | Description | Risk | Rollback |
|---|---|---|---|
| **Blue/Green** | Two identical environments; switch traffic 0%→100% | Binary risk (all or nothing) | Instant (switch back) |
| **Canary** | Gradually shift traffic (1% → 5% → 10% → 100%) | Low (catch issues early) | Partial rollback easily |
| **Rolling** | Replace instances one-by-one; no idle environment | Medium | Slower rollback |
| **A/B Testing** | Split traffic by user attribute to compare versions | Low | Per-segment rollback |

### Event-Driven Architecture
- **Loose coupling**: Services communicate through events; no direct dependency
- **Choreography** (event-driven): Services react to events independently (Pub/Sub pattern)
- **Orchestration**: Central coordinator directs services (Cloud Workflows pattern)
- GCP pattern: Cloud Functions / Cloud Run triggered by Eventarc / Pub/Sub

### 12-Factor App Principles (Cloud-Native Apps)
Key factors relevant to GCP exam:
1. **Config in environment** (not code) → Secret Manager, Cloud Run env vars
2. **Stateless processes** → Cloud Run, App Engine Standard
3. **Port binding** → Container-based services
4. **Logs as event streams** → Cloud Logging integration
5. **Backing services as attached resources** → Cloud SQL, Memorystore via connection strings
