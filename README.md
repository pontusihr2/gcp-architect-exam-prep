# ☁️ Google Professional Cloud Architect — Exam Prep (2025 Edition)

> **Target exam date:** April 30, 2026 &nbsp;|&nbsp; **Exam version:** October 2025 update &nbsp;|&nbsp; **Cert level:** Professional

[![Google Cloud](https://img.shields.io/badge/Google%20Cloud-Professional%20Cloud%20Architect-4285F4?logo=googlecloud&logoColor=white)](https://cloud.google.com/learn/certification/cloud-architect)
[![Status](https://img.shields.io/badge/Status-In%20Progress-yellow)]()
[![Exam Date](https://img.shields.io/badge/Exam%20Date-April%2030%2C%202026-brightgreen)]()

---

## 📋 Table of Contents

- [Exam Overview](#-exam-overview)
- [Exam Domains & Weights](#-exam-domains--weights)
- [Official Case Studies](#-official-case-studies)
- [Repo Structure](#-repo-structure)
- [Study Plan — 3-Week Sprint](#-study-plan--3-week-sprint)
- [Study Tips](#-study-tips)
- [Quick Links](#-quick-links)

---

## 📝 Exam Overview

| Attribute | Detail |
|---|---|
| **Exam name** | Professional Cloud Architect |
| **Exam version** | October 2025 update |
| **Question count** | 50–60 questions |
| **Duration** | 120 minutes |
| **Question types** | Multiple choice (single answer) + multiple select (scenario-based) |
| **Passing score** | ~70% (score scaled; Google does not publish exact threshold) |
| **Language** | English, Japanese, Spanish, Portuguese |
| **Delivery** | Remote proctored (online) or in-person at Kryterion test centers |
| **Cost** | $200 USD |
| **Validity** | 2 years from pass date |
| **Prerequisites** | None (3+ years industry experience recommended, 1+ year GCP) |

### 📌 Registration

Register at the official Google Cloud Certification portal:
**<https://cloud.google.com/learn/certification/cloud-architect>**

> **Tip:** Book your slot at least 2–3 weeks in advance on Kryterion. Remote proctored slots fill up quickly around popular dates.

### 🎯 What the Exam Tests

The Professional Cloud Architect exam assesses your ability to **design, develop, and manage** robust, secure, scalable, and highly available cloud solutions on Google Cloud. Scenarios are drawn from real enterprise situations; expect multi-paragraph case study questions requiring you to apply best practices across multiple services simultaneously. The October 2025 update places increased emphasis on:

- AI/ML workloads on Vertex AI and integration with existing architectures
- FinOps / cost optimization across large-scale deployments
- Hybrid and multi-cloud connectivity (Interconnect, Cross-Cloud Interconnect)
- Security posture management (Security Command Center Premium, CSPM)
- Modern data mesh and analytics architectures (BigQuery Omni, Dataplex)

---

## 🗂️ Exam Domains & Weights

> Source: [Official Exam Guide](https://cloud.google.com/learn/certification/guides/cloud-architect) — October 2025 version

| # | Domain | Weight |
|---|---|---|
| 1 | [Designing and Planning a Cloud Solution Architecture](#domain-1--designing-and-planning-a-cloud-solution-architecture) | **~24%** |
| 2 | [Managing and Provisioning Solution Infrastructure](#domain-2--managing-and-provisioning-solution-infrastructure) | **~15%** |
| 3 | [Designing for Security and Compliance](#domain-3--designing-for-security-and-compliance) | **~18%** |
| 4 | [Analyzing and Optimizing Technical and Business Processes](#domain-4--analyzing-and-optimizing-technical-and-business-processes) | **~18%** |
| 5 | [Managing Implementation](#domain-5--managing-implementation) | **~11%** |
| 6 | [Ensuring Solution and Operations Reliability](#domain-6--ensuring-solution-and-operations-reliability) | **~14%** |

---

### Domain 1 — Designing and Planning a Cloud Solution Architecture (~24%)

This is the **highest-weight domain** and the conceptual heart of the exam. Focus heavily here.

**Key sub-topics:**
- Designing a solution infrastructure that meets business and technical requirements
  - High availability, failover, disaster recovery (RPO/RTO targets)
  - Scalability: horizontal vs. vertical, autoscaling policies
  - Selecting appropriate compute: GKE, Cloud Run, Compute Engine, App Engine
- Designing network topologies
  - VPC design, Shared VPC, VPC Peering, Private Service Connect
  - Hybrid connectivity: Cloud VPN, Dedicated/Partner Interconnect, Cross-Cloud Interconnect
  - DNS strategy (Cloud DNS, private zones, forwarding)
- Designing a data storage strategy
  - Structured: Cloud SQL, Cloud Spanner, AlloyDB
  - Unstructured: Cloud Storage (storage classes, lifecycle rules)
  - Analytical: BigQuery, Bigtable, Firestore
  - Caching: Memorystore (Redis/Memcached)
- Designing for AI/ML workloads
  - Vertex AI Pipelines, Model Garden, feature stores
  - Data ingestion pipelines (Pub/Sub → Dataflow → BigQuery)
- Migration planning
  - Lift-and-shift vs. modernize vs. re-architect
  - Migration Center, Migrate for Compute Engine
  - Database Migration Service (DMS)

📁 Study notes: [`topics/01-designing-planning-architecture.md`](topics/01-designing-planning-architecture.md)

---

### Domain 2 — Managing and Provisioning Solution Infrastructure (~15%)

**Key sub-topics:**
- Infrastructure as Code (IaC)
  - Terraform on Google Cloud (Cloud Build + Terraform, workspaces)
  - Deployment Manager (legacy, still tested)
  - Config Connector for Kubernetes-native GCP resource management
- Configuring network services
  - Cloud Load Balancing tiers (Global HTTP(S), regional, internal, TCP/UDP)
  - Cloud CDN configuration and cache invalidation
  - Cloud Armor WAF policies and DDoS protection
- Deploying and implementing data solutions
  - Dataproc clusters, autoscaling, workflow templates
  - Dataflow streaming vs. batch, Flex Templates
  - Cloud Composer (Airflow) DAG management
- Deploying and implementing hybrid/multi-cloud environments
  - Anthos / GKE Enterprise fleet management
  - Config Management, Policy Controller
  - Service Mesh (Cloud Service Mesh / Istio)

📁 Study notes: [`topics/02-managing-provisioning-infrastructure.md`](topics/02-managing-provisioning-infrastructure.md)

---

### Domain 3 — Designing for Security and Compliance (~18%)

**Key sub-topics:**
- Identity and Access Management
  - IAM roles (primitive, predefined, custom), Conditions, deny policies
  - Workload Identity Federation (external IdP → GCP)
  - Service account best practices, impersonation vs. key files
  - Cloud Identity, Google Workspace federation
- Data protection
  - Cloud KMS (symmetric/asymmetric keys, HSM, CMEK, CSEK, EKM)
  - Secret Manager, Berglas
  - Data Loss Prevention (DLP) API — info types, de-identification
  - VPC Service Controls — service perimeters, access levels
- Network security
  - Private Google Access, Private Service Connect
  - Cloud Armor, Cloud NAT, firewall rules hierarchy (org → folder → project)
  - Hierarchical firewall policies
- Security posture and compliance
  - Security Command Center (Standard vs. Premium), findings, assets
  - Organization Policy Service constraints
  - Binary Authorization, Container Analysis, Artifact Registry
  - Compliance frameworks: PCI DSS, HIPAA BAA, ISO 27001, SOC 2 on GCP
  - Access Transparency, Access Approval

📁 Study notes: [`topics/03-security-compliance.md`](topics/03-security-compliance.md)

---

### Domain 4 — Analyzing and Optimizing Technical and Business Processes (~18%)

**Key sub-topics:**
- Stakeholder management and solution trade-offs
  - Translating business requirements into technical constraints
  - SLA vs. SLO vs. SLI definitions and design
  - Total cost of ownership (TCO) analysis
- Cost optimization (FinOps)
  - Committed Use Discounts (CUDs) vs. Sustained Use Discounts (SUDs)
  - Spot/Preemptible VMs strategy
  - BigQuery slot reservations vs. on-demand pricing
  - GCS intelligent tiering, storage class transitions
  - Billing export to BigQuery, billing budgets and alerts
  - Cloud FinOps Hub, Recommender API, Active Assist
- CI/CD and DevOps
  - Cloud Build triggers, build configs, artifact management
  - Cloud Deploy pipelines, deployment strategies (canary, blue/green, rolling)
  - Skaffold, Kustomize for Kubernetes delivery
- Testing and validation
  - Load testing strategies (Cloud Load Testing, Artillery)
  - Chaos engineering concepts on GCP
  - A/B testing with Traffic Director or Cloud Run traffic splitting

📁 Study notes: [`topics/04-optimizing-processes.md`](topics/04-optimizing-processes.md)

---

### Domain 5 — Managing Implementation (~11%)

**Key sub-topics:**
- Advising development/operations teams during implementation
  - API design and management: Cloud Endpoints, Apigee (X and hybrid)
  - gRPC vs. REST vs. GraphQL on GCP
  - Event-driven architecture: Pub/Sub, Eventarc, Cloud Tasks, Cloud Scheduler
- Interacting with Google Cloud programmatically
  - gcloud SDK, client libraries (Python, Java, Go)
  - REST and gRPC APIs, service account authentication patterns
  - Cloud Shell, Cloud Code (IDE plugin)
- Data migration execution
  - Storage Transfer Service (GCS ↔ S3 ↔ Azure Blob ↔ on-prem)
  - Transfer Appliance for large offline migrations
  - BigQuery Data Transfer Service (SaaS sources)

📁 Study notes: [`topics/05-managing-implementation.md`](topics/05-managing-implementation.md)

---

### Domain 6 — Ensuring Solution and Operations Reliability (~14%)

**Key sub-topics:**
- Monitoring and observability
  - Cloud Monitoring: metrics, alerting policies, uptime checks, dashboards
  - Cloud Logging: log sinks (BigQuery, GCS, Pub/Sub), log-based metrics, retention
  - Cloud Trace, Cloud Profiler, Error Reporting
  - Google Cloud Managed Service for Prometheus, Grafana integration
- Incident response
  - SRE principles: error budgets, toil reduction, postmortems
  - PagerDuty / OpsGenie integration via notification channels
  - Runbooks and alerting best practices
- Disaster recovery and business continuity
  - Backup and DR strategies: snapshot schedules, Cloud SQL backups, PITR
  - Multi-region active-active vs. active-passive architectures
  - RPO/RTO design patterns per tier (cold, warm, hot standby)
- Managing service mesh and distributed tracing
  - Cloud Service Mesh (Istio) telemetry
  - Distributed tracing with OpenTelemetry on GCP

📁 Study notes: [`topics/06-operations-reliability.md`](topics/06-operations-reliability.md)

---

## 🏢 Official Case Studies

The exam includes questions tied to **4 official case studies**. You **must** read and memorize these before exam day — they are available during the exam, but you won't have time to read them cold.

> 🔗 All case studies: <https://cloud.google.com/learn/certification/guides/cloud-architect>

---

### 🏥 Case Study 1: EHR Healthcare

| Attribute | Detail |
|---|---|
| **Industry** | Healthcare / Health IT |
| **Core challenge** | Migrate legacy on-premises EHR SaaS platform to GCP while meeting HIPAA/PHI requirements |
| **Scale** | Hundreds of healthcare providers, millions of patients |
| **Key requirements** | 99.9% uptime SLA, HIPAA BAA, data residency, HL7 FHIR API compliance |

**Critical GCP services to know for this case study:**
- **Compute:** GKE (containerized microservices), Cloud Run (APIs)
- **Database:** Cloud SQL (PostgreSQL — transactional EHR data), Cloud Spanner (global scale if needed)
- **Storage:** Cloud Storage (medical imaging, DICOM), Cloud Healthcare API (FHIR, DICOM, HL7v2)
- **Security:** Cloud KMS (CMEK for PHI at rest), VPC Service Controls (PHI perimeter), Cloud Armor, IAM Conditions
- **Connectivity:** Dedicated Interconnect (on-prem hospital systems), Private Google Access
- **Compliance:** HIPAA BAA, Assured Workloads (healthcare controls), DLP API (de-identification)
- **Observability:** Cloud Monitoring (uptime SLAs), Cloud Audit Logs (all admin/data access)

📁 Deep dive: [`case-studies/ehr-healthcare.md`](case-studies/ehr-healthcare.md)

---

### 🎬 Case Study 2: Altostrat Media (formerly Mountkirk Games variant for media)

| Attribute | Detail |
|---|---|
| **Industry** | Media & Entertainment / Online Games |
| **Core challenge** | Build globally scalable gaming backend; expand to new regions rapidly |
| **Scale** | Millions of concurrent users globally, spiky traffic patterns |
| **Key requirements** | Global low-latency, autoscaling, game state consistency, analytics pipeline |

**Critical GCP services to know for this case study:**
- **Compute:** GKE Autopilot (game server pods), Cloud Run (stateless APIs)
- **Networking:** Global HTTP(S) Load Balancer, Cloud CDN, Premium Network Tier
- **Database:** Cloud Spanner (global game state — strongly consistent), Bigtable (leaderboards, time-series), Memorystore Redis (session caching)
- **Analytics:** Pub/Sub → Dataflow → BigQuery (clickstream / game telemetry pipeline)
- **Storage:** Cloud Storage (game assets — multi-region bucket)
- **Scaling:** Managed Instance Groups with autoscaling, GKE HPA/VPA/KEDA
- **Reliability:** Multi-region active-active, chaos testing, SLO dashboards

📁 Deep dive: [`case-studies/altostrat-media.md`](case-studies/altostrat-media.md)

---

### 🛒 Case Study 3: Cymbal Retail

| Attribute | Detail |
|---|---|
| **Industry** | Retail / E-commerce |
| **Core challenge** | Modernize monolithic retail platform; enable omni-channel (online + in-store); improve data analytics |
| **Scale** | Large enterprise retailer, high seasonal traffic (Black Friday spikes) |
| **Key requirements** | Cost optimization, FinOps governance, real-time inventory, ML recommendations |

**Critical GCP services to know for this case study:**
- **Compute:** GKE (microservices migration), Cloud Run (event-driven functions)
- **Database:** Cloud SQL (order management), Firestore (product catalog — real-time sync), AlloyDB (analytics on transactional data)
- **Analytics:** BigQuery (retail analytics, demand forecasting), Looker (executive dashboards), Vertex AI (recommendation engine)
- **Integration:** Pub/Sub (inventory events), Eventarc (event-driven workflows), Apigee (partner APIs)
- **Cost:** CUDs for stable workloads, Spot VMs for batch ML training, BigQuery reservations, Recommender API
- **Networking:** Cloud Armor (e-commerce fraud/DDoS), reCAPTCHA Enterprise
- **Multi-cloud:** BigQuery Omni (data in AWS S3 from acquired company)

📁 Deep dive: [`case-studies/cymbal-retail.md`](case-studies/cymbal-retail.md)

---

### 🚗 Case Study 4: KnightMotives Automotive

| Attribute | Detail |
|---|---|
| **Industry** | Automotive / IoT / Connected Vehicles |
| **Core challenge** | Ingest and process high-volume IoT telemetry from connected vehicles; enable predictive maintenance and fleet analytics |
| **Scale** | Millions of vehicles, billions of events/day |
| **Key requirements** | Real-time streaming ingestion, edge + cloud hybrid, predictive ML models, data governance |

**Critical GCP services to know for this case study:**
- **IoT / Edge:** Cloud IoT Core (legacy — note: deprecated Aug 2023; exam may reference replacement patterns), Pub/Sub (device telemetry ingest), Google Distributed Cloud Edge
- **Streaming:** Pub/Sub → Dataflow (streaming pipelines) → Bigtable (low-latency lookups) + BigQuery (analytical store)
- **ML:** Vertex AI (predictive maintenance models), AutoML Tables, Model Registry
- **Storage:** Cloud Storage (raw telemetry archive, Avro/Parquet), BigQuery (data warehouse)
- **Governance:** Dataplex (data mesh, data quality, lineage), Data Catalog
- **Connectivity:** Dedicated Interconnect / Cross-Cloud Interconnect (manufacturing plant connectivity), Partner Interconnect
- **Security:** VPC Service Controls, CMEK, Workload Identity

📁 Deep dive: [`case-studies/knightmotives-automotive.md`](case-studies/knightmotives-automotive.md)

---

## 📁 Repo Structure

```
gcp-architect-exam-prep/
│
├── README.md                          ← You are here
│
├── topics/                            ← Domain-by-domain study notes
│   ├── 01-designing-planning-architecture.md
│   ├── 02-managing-provisioning-infrastructure.md
│   ├── 03-security-compliance.md
│   ├── 04-optimizing-processes.md
│   ├── 05-managing-implementation.md
│   └── 06-operations-reliability.md
│
├── services/                          ← Per-service deep dives & decision tables
│   ├── compute.md                     # GCE, GKE, Cloud Run, App Engine, Cloud Functions
│   ├── networking.md                  # VPC, LB, CDN, Interconnect, DNS, Cloud Armor
│   ├── storage.md                     # GCS, Persistent Disk, Filestore
│   ├── databases.md                   # Cloud SQL, Spanner, Bigtable, Firestore, AlloyDB
│   ├── bigdata-analytics.md           # BigQuery, Dataflow, Dataproc, Pub/Sub, Composer
│   ├── security-services.md           # IAM, KMS, SCC, VPC-SC, DLP, Secret Manager
│   ├── ai-ml.md                       # Vertex AI, AutoML, Gemini on Vertex
│   └── devops.md                      # Cloud Build, Cloud Deploy, Artifact Registry
│
├── case-studies/                      ← Official case study analysis
│   ├── ehr-healthcare.md
│   ├── altostrat-media.md
│   ├── cymbal-retail.md
│   └── knightmotives-automotive.md
│
├── cheat-sheets/                      ← Quick reference cards
│   ├── decision-trees.md              # Compute, DB, storage, network, data, migration, DR
│   ├── services-comparison.md         # Side-by-side service comparison tables
│   ├── iam-cheatsheet.md              # IAM hierarchy, roles, conditions, org policies
│   ├── networking-cheatsheet.md       # VPC, firewall, LB, hybrid connectivity
│   └── key-terms.md                   # RTO/RPO, SLI/SLO, CAP theorem, glossary
│
├── practice-questions/                ← Practice Q&A bank
│   ├── domain-1-architecture.md
│   ├── domain-2-infrastructure.md
│   ├── domain-3-security.md
│   ├── domain-4-processes.md
│   ├── domain-5-implementation.md
│   ├── domain-6-reliability.md
│   └── case-study-questions.md
│
└── resources/
    └── official-links.md              ← Curated links (official docs, labs, whitepapers)
```

---

## 📅 Study Plan — 3-Week Sprint

> **Assumes** ~2–3 hours/day on weekdays, ~4–5 hours/day on weekends.
> Adjust to your April 30, 2026 exam date — run multiple cycles if time allows.

### Week 1 — Foundation & High-Weight Domains

| Day | Focus | Activities |
|---|---|---|
| Mon | **Domain 1: Design & Architecture** | Read exam guide section 1 · Study VPC design patterns · Compute decision tree |
| Tue | **Domain 1 cont. + Storage** | Storage decision tree · Cloud SQL vs Spanner vs AlloyDB comparison · 20 practice Qs |
| Wed | **Domain 3: Security** | IAM deep dive · Cloud KMS CMEK/CSEK/EKM · VPC Service Controls hands-on lab |
| Thu | **Domain 3 cont.** | SCC, Binary Authorization, Organization Policy · DLP API · 20 practice Qs |
| Fri | **Case Studies: EHR Healthcare + Altostrat** | Read both case studies · Map each requirement to a GCP service · Annotate |
| Sat | **Hands-on Labs (4–5 hrs)** | GKE cluster + Cloud Armor lab · Cloud KMS lab · VPC peering lab |
| Sun | **Review + Mock Questions** | Redo any incorrect Qs · Flashcard review · Week 1 gap analysis |

### Week 2 — Operations, Analytics, and Remaining Domains

| Day | Focus | Activities |
|---|---|---|
| Mon | **Domain 4: Optimization & FinOps** | CUDs vs SUDs · BigQuery pricing · Cost optimization patterns |
| Tue | **Domain 4 cont. + CI/CD** | Cloud Build + Cloud Deploy · Blue/green canary patterns · 20 practice Qs |
| Wed | **Domain 2: Provisioning & IaC** | Terraform on GCP · Deployment patterns · Load Balancer selection guide |
| Thu | **Domain 5: Implementation** | Apigee vs Cloud Endpoints · Pub/Sub + Eventarc · Storage Transfer Service |
| Fri | **Case Studies: Cymbal Retail + KnightMotives** | Read both · Map requirements · Common trap answers |
| Sat | **Hands-on Labs (4–5 hrs)** | BigQuery + Dataflow streaming lab · Terraform deploy lab · Pub/Sub lab |
| Sun | **Domain 6: Reliability** | SRE principles · Cloud Monitoring alerting · DR patterns · Error budgets |

### Week 3 — Integration, Mock Exams, and Gap Closing

| Day | Focus | Activities |
|---|---|---|
| Mon | **Full Mock Exam #1 (timed, 120 min)** | Use Whizlabs or TutorialsDojo · Score and review every wrong answer |
| Tue | **Gap Analysis + Weak Domain Deep Dive** | Focus on bottom 2 scoring domains · Re-read cheat sheets |
| Wed | **Service Decision Trees** | Compute tree · Storage tree · Network tier selection · Complete the cheat sheets |
| Thu | **Full Mock Exam #2 (timed, 120 min)** | Different provider than #1 · Score ≥ 75% target before real exam |
| Fri | **Case Studies Final Review** | Re-read all 4 case studies aloud · Map every company requirement to exact GCP product/feature |
| Sat | **Light Review Only** | Cheat sheet read-through · Flashcards · 20 easy warm-up questions · Rest |
| Sun (Exam -1 day) | **Logistics + Rest** | Confirm exam booking · Test proctoring setup · Light review only · Sleep well |

---

## 💡 Study Tips

### Exam Strategy
- **Eliminate first:** GCP exams often have 2 obviously wrong answers. Narrow to the best 2, then apply the "most GCP-native / managed" principle.
- **"Managed > Self-managed":** Google's preferred answer almost always favors a managed service over a self-managed one (e.g., Cloud SQL > MySQL on GCE).
- **Cost questions:** Default to the option that minimizes cost while meeting requirements — not the option with maximum redundancy.
- **Security questions:** Apply least-privilege principle. Choose the most specific IAM role, not Owner or Editor.
- **Case study questions:** Map the company's constraints (compliance, SLA, budget) to the answer options. The "right" answer respects ALL stated constraints.

### High-Value Study Areas (October 2025 emphasis)
1. **VPC Service Controls** — tested heavily; know service perimeters, access levels, and bridge perimeters
2. **Cloud KMS hierarchy** — CMEK vs CSEK vs EKM vs Cloud HSM
3. **GKE networking** — know the difference between cluster-native (VPC-native), Shared VPC with GKE, and Private Clusters
4. **BigQuery pricing** — on-demand vs. capacity (slots), reservations, commitments
5. **DR patterns** — know exactly what RPO/RTO targets map to which GCP architecture (cold standby, warm, hot/active-active)
6. **Vertex AI architecture** — Feature Store, Model Registry, Pipelines, endpoint deployment
7. **Anthos/GKE Enterprise** — fleet management, Config Management, Policy Controller

### Memory Aids
- **Storage FABS:** Firestore (document), AlloyDB (analytical PostgreSQL), Bigtable (wide-column), Spanner (globally consistent relational)
- **Load Balancer matrix:** Global = HTTP(S) LB or TCP/SSL Proxy; Regional = Regional external/internal; Internal = passthrough L4 for VM-to-VM
- **Network tiers:** Premium = global Anycast (performance); Standard = regional (cost)
- **IAM hierarchy:** Org → Folder → Project → Resource (permissions are *additive* downward, *deny* policies override grants)

### Tools
- **Anki flashcards** — make a card for every service comparison (e.g., "Cloud SQL vs Cloud Spanner vs AlloyDB — when to use each")
- **Draw.io / Lucidchart** — draw reference architectures for each case study from memory
- **GCP Free Tier** — use free credits to build hands-on labs; muscle memory beats memorization

---

## 🔗 Quick Links

| Resource | Path |
|---|---|
| Official resources & links | [`resources/official-links.md`](resources/official-links.md) |
| Online practice exams | [`resources/official-links.md#-online-practice-exams`](resources/official-links.md#-online-practice-exams) |
| Domain 1 — Design & Planning | [`topics/01-designing-planning-architecture.md`](topics/01-designing-planning-architecture.md) |
| Domain 2 — Provisioning | [`topics/02-managing-provisioning-infrastructure.md`](topics/02-managing-provisioning-infrastructure.md) |
| Domain 3 — Security | [`topics/03-security-compliance.md`](topics/03-security-compliance.md) |
| Domain 4 — Optimization | [`topics/04-optimizing-processes.md`](topics/04-optimizing-processes.md) |
| Domain 5 — Implementation | [`topics/05-managing-implementation.md`](topics/05-managing-implementation.md) |
| Domain 6 — Reliability | [`topics/06-operations-reliability.md`](topics/06-operations-reliability.md) |
| EHR Healthcare case study | [`case-studies/ehr-healthcare.md`](case-studies/ehr-healthcare.md) |
| Altostrat Media case study | [`case-studies/altostrat-media.md`](case-studies/altostrat-media.md) |
| Cymbal Retail case study | [`case-studies/cymbal-retail.md`](case-studies/cymbal-retail.md) |
| KnightMotives Automotive case study | [`case-studies/knightmotives-automotive.md`](case-studies/knightmotives-automotive.md) |
| All decision trees (compute, DB, storage, network, data, migration, DR) | [`cheat-sheets/decision-trees.md`](cheat-sheets/decision-trees.md) |
| IAM cheat sheet | [`cheat-sheets/iam-cheatsheet.md`](cheat-sheets/iam-cheatsheet.md) |
| Networking cheat sheet | [`cheat-sheets/networking-cheatsheet.md`](cheat-sheets/networking-cheatsheet.md) |
| Services comparison | [`cheat-sheets/services-comparison.md`](cheat-sheets/services-comparison.md) |
| Key terms & glossary | [`cheat-sheets/key-terms.md`](cheat-sheets/key-terms.md) |
| Practice questions (all domains) | [`practice-questions/`](practice-questions/) |

---

> 📅 **Exam date: April 30, 2026** — You've got this. Consistent daily review beats cramming. Good luck! 🍀
