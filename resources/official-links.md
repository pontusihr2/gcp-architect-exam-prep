# 🔗 Official & Community Resources — Professional Cloud Architect (2025)

> **Last reviewed:** October 2025 (aligned to October 2025 exam update)
> ⚠️ *Some resources require a free Google account or Google Cloud Skills Boost sign-in. Links marked **[Login]** require authentication.*

---

## 📋 Table of Contents

- [Official Google Certification Links](#-official-google-certification-links)
- [Online Practice Exams](#-online-practice-exams)
- [Google Cloud Documentation by Domain](#-google-cloud-documentation-by-domain)
  - [Compute](#compute)
  - [Networking](#networking)
  - [Storage & Databases](#storage--databases)
  - [Data Analytics & Pipelines](#data-analytics--pipelines)
  - [Security & Compliance](#security--compliance)
  - [AI / ML — Vertex AI](#ai--ml--vertex-ai)
  - [Operations & Observability](#operations--observability)
  - [Hybrid & Multi-Cloud](#hybrid--multi-cloud)
  - [Infrastructure as Code & DevOps](#infrastructure-as-code--devops)
- [Google Cloud Skills Boost (Qwiklabs) — Free Learning Paths](#-google-cloud-skills-boost-qwiklabs--free-learning-paths)
- [Official Whitepapers & Architecture Frameworks](#-official-whitepapers--architecture-frameworks)
- [Google Cloud YouTube Resources](#-google-cloud-youtube-resources)
- [Community Forums & Structured Courses](#-community-forums--structured-courses)
- [Recommended Books](#-recommended-books)

---

## 🎓 Official Google Certification Links

| Resource | URL | Notes |
|---|---|---|
| **Certification homepage** | <https://cloud.google.com/learn/certification/cloud-architect> | Start here — registration, exam guide, policies |
| **Exam guide (PDF)** | <https://cloud.google.com/learn/certification/guides/cloud-architect> | Download the Oct 2025 version — the definitive scope |
| **Official case studies** | <https://cloud.google.com/learn/certification/guides/cloud-architect#section-4> | EHR Healthcare, Altostrat, Cymbal Retail, KnightMotives |
| **Sample questions** | <https://docs.google.com/forms/d/e/1FAIpQLSf54f7FbtSJcXUY6-DUHfBG31jZ3pujgb8-a5io_9biJsNpqg/viewform> | ~30 official sample Qs from Google |
| **Certification FAQ** | <https://cloud.google.com/learn/certification/faqs> | Policies, reschedule, accessibility accommodations |
| **Kryterion scheduling** | <https://www.webassessor.com/googlecloud/> | Book/reschedule your exam slot here |
| **Google Cloud Skills Boost** | <https://www.cloudskillsboost.google/> | Official lab platform (Qwiklabs) — free tier available **[Login]** |
| **Cloud Architect learning path** | <https://www.skills.google/paths/12> | Official Google Skills learning path with labs and exam-prep material **[Login]** |
| **Practice exam announcement** | <https://cloud.google.com/blog/products/gcp/now-live-online-practice-exam-for-cloud> | Google Cloud blog post for the online practice exam rollout |
| **Google Cloud blog** | <https://cloud.google.com/blog> | New service announcements relevant to exam updates |
| **Google Cloud architecture center** | <https://cloud.google.com/architecture> | Reference architectures for all major patterns |
| **Solutions library** | <https://cloud.google.com/solutions> | Industry and use-case solution guides |

---

## 📝 Online Practice Exams

> Prioritize the official Google sample questions first, then use third-party practice exams to find weak areas. Avoid braindumps and always verify answers against official documentation.

| Provider | URL | Notes | Cost |
|---|---|---|---|
| **Official sample questions** | <https://docs.google.com/forms/d/e/1FAIpQLSf54f7FbtSJcXUY6-DUHfBG31jZ3pujgb8-a5io_9biJsNpqg/viewform> | Best source for question style and wording | Free |
| **Google Skills learning path** | <https://www.skills.google/paths/12> | Official Google path with labs and practice-oriented prep **[Login]** | Free / paid labs |
| **CloudJobs** | <https://cloudjobs.io/study/gcp-practice-tests> | Free browser-based GCP practice tests, including PCA coverage | Free |
| **Tutorials Dojo** | <https://portal.tutorialsdojo.com/courses/google-certified-professional-cloud-architect-practice-exams/> | Timed, review, and section-based exam modes with explanations | Paid |
| **Whizlabs** | <https://www.whizlabs.com/google-cloud-certified-professional-cloud-architect/> | Practice tests plus labs and exam-style scenario questions | Paid |
| **Certification Practice** | <https://certificationpractice.com/practice-exams/google-cloud-professional-cloud-architect> | Multiple timed practice sets and score tracking | Free |
| **MeasureUp** | <https://www.measureup.com/google-cloud.html> | Official practice test partner; closest to real-exam style | Paid |

---

## 📚 Google Cloud Documentation by Domain

### Compute

| Service | Documentation | Exam Relevance |
|---|---|---|
| **Compute Engine** | <https://cloud.google.com/compute/docs> | Instance types, MIGs, autoscaling, custom machine types |
| **GKE** | <https://cloud.google.com/kubernetes-engine/docs> | Cluster modes (Standard/Autopilot), Private Clusters, Workload Identity |
| **Cloud Run** | <https://cloud.google.com/run/docs> | Serverless containers, concurrency, min/max instances, VPC connector |
| **App Engine** | <https://cloud.google.com/appengine/docs> | Standard vs. Flexible, traffic splitting, versioning |
| **Cloud Functions** | <https://cloud.google.com/functions/docs> | Gen 1 vs. Gen 2 (backed by Cloud Run), event triggers |
| **Bare Metal Solution** | <https://cloud.google.com/bare-metal/docs> | On-prem SAP/Oracle workloads on dedicated hardware |
| **Compute Engine → Preemptible/Spot VMs** | <https://cloud.google.com/compute/docs/instances/spot> | Cost optimization — batch/fault-tolerant workloads |
| **Machine types reference** | <https://cloud.google.com/compute/docs/machine-resource> | General, compute-optimized, memory-optimized, GPU types |

### Networking

| Service | Documentation | Exam Relevance |
|---|---|---|
| **VPC overview** | <https://cloud.google.com/vpc/docs/vpc> | Auto vs. custom mode, subnets, secondary ranges |
| **Shared VPC** | <https://cloud.google.com/vpc/docs/shared-vpc> | Host/service project model, IAM for Shared VPC |
| **VPC Peering** | <https://cloud.google.com/vpc/docs/vpc-peering> | Non-transitive peering, route export/import |
| **Private Service Connect** | <https://cloud.google.com/vpc/docs/private-service-connect> | Accessing Google APIs and managed services privately |
| **Cloud Load Balancing** | <https://cloud.google.com/load-balancing/docs/load-balancing-overview> | **Critical** — full LB decision matrix, SSL offload, health checks |
| **Cloud CDN** | <https://cloud.google.com/cdn/docs> | Cache modes, TTL, signed URLs, cache invalidation |
| **Cloud Armor** | <https://cloud.google.com/armor/docs> | WAF rules, adaptive protection, DDoS, security policies |
| **Cloud VPN** | <https://cloud.google.com/network-connectivity/docs/vpn> | HA VPN (2 tunnels), Classic VPN, BGP routing |
| **Dedicated Interconnect** | <https://cloud.google.com/network-connectivity/docs/interconnect/concepts/dedicated-overview> | 10/100 Gbps, 99.99% SLA with redundancy |
| **Partner Interconnect** | <https://cloud.google.com/network-connectivity/docs/interconnect/concepts/partner-overview> | Through a service provider, 50 Mbps–50 Gbps |
| **Cross-Cloud Interconnect** | <https://cloud.google.com/network-connectivity/docs/interconnect/concepts/cci-overview> | GCP ↔ AWS/Azure dedicated connectivity (Oct 2025 emphasis) |
| **Cloud DNS** | <https://cloud.google.com/dns/docs> | Private zones, peering zones, forwarding zones, DNS policies |
| **Cloud NAT** | <https://cloud.google.com/nat/docs> | Outbound internet for private VMs, no inbound exposure |
| **Network Intelligence Center** | <https://cloud.google.com/network-intelligence-center/docs> | Connectivity tests, firewall insights, performance dashboard |
| **Hierarchical Firewall Policies** | <https://cloud.google.com/vpc/docs/firewall-policies> | Org/folder-level policies that override project rules |
| **Network Service Tiers** | <https://cloud.google.com/network-tiers/docs> | Premium (global routing) vs. Standard (regional routing) |

### Storage & Databases

| Service | Documentation | Exam Relevance |
|---|---|---|
| **Cloud Storage** | <https://cloud.google.com/storage/docs> | Storage classes, lifecycle rules, bucket policies, CORS, retention locks |
| **Cloud SQL** | <https://cloud.google.com/sql/docs> | MySQL/PostgreSQL/SQL Server, read replicas, HA (failover), PITR |
| **Cloud Spanner** | <https://cloud.google.com/spanner/docs> | Global horizontal scale, 99.999% SLA, external consistency |
| **AlloyDB for PostgreSQL** | <https://cloud.google.com/alloydb/docs> | HTAP — OLTP + analytics on PostgreSQL; 4x faster than Cloud SQL |
| **Firestore** | <https://cloud.google.com/firestore/docs> | Document DB, real-time sync, Native vs. Datastore mode |
| **Bigtable** | <https://cloud.google.com/bigtable/docs> | Wide-column, HBase API, time-series, IoT, petabyte scale |
| **Memorystore (Redis)** | <https://cloud.google.com/memorystore/docs/redis> | In-memory cache, session store, Pub/Sub fan-out |
| **Memorystore (Memcached)** | <https://cloud.google.com/memorystore/docs/memcached> | Simple distributed caching |
| **Database Migration Service** | <https://cloud.google.com/database-migration/docs> | Migrate MySQL/PostgreSQL → Cloud SQL/AlloyDB with CDC |
| **Filestore** | <https://cloud.google.com/filestore/docs> | Managed NFS for GKE/GCE workloads needing shared file storage |
| **Storage decision tree** | <https://cloud.google.com/products/databases> | Google's official product selector |

### Data Analytics & Pipelines

| Service | Documentation | Exam Relevance |
|---|---|---|
| **BigQuery** | <https://cloud.google.com/bigquery/docs> | **Highest-tested analytics service** — slots, reservations, partitioning, clustering, BI Engine |
| **BigQuery Omni** | <https://cloud.google.com/bigquery/docs/omni-introduction> | Analyze data in AWS S3 / Azure Blob without moving it |
| **Pub/Sub** | <https://cloud.google.com/pubsub/docs> | Topics, subscriptions, push vs. pull, dead-letter, ordering keys |
| **Dataflow** | <https://cloud.google.com/dataflow/docs> | Apache Beam managed, streaming + batch, Flex Templates, autoscaling |
| **Dataproc** | <https://cloud.google.com/dataproc/docs> | Managed Spark/Hadoop, autoscaling clusters, Dataproc Serverless |
| **Cloud Composer** | <https://cloud.google.com/composer/docs> | Managed Apache Airflow, DAG authoring, KubernetesPodOperator |
| **Datastream** | <https://cloud.google.com/datastream/docs> | CDC from Oracle/MySQL/PostgreSQL → BigQuery/Cloud Storage |
| **Dataplex** | <https://cloud.google.com/dataplex/docs> | Data mesh, data quality, cataloging, discovery, lineage |
| **Data Catalog** | <https://cloud.google.com/data-catalog/docs> | Metadata management, tagging, PII discovery (now part of Dataplex) |
| **Looker / Looker Studio** | <https://cloud.google.com/looker/docs> | BI and dashboards; Looker Studio = free, Looker = enterprise |
| **BigQuery Data Transfer Service** | <https://cloud.google.com/bigquery-transfer/docs> | Scheduled imports from SaaS sources (Ads, Salesforce, etc.) |
| **Analytics Hub** | <https://cloud.google.com/analytics-hub/docs> | Data exchange marketplace for BigQuery datasets |

### Security & Compliance

| Service | Documentation | Exam Relevance |
|---|---|---|
| **IAM overview** | <https://cloud.google.com/iam/docs/overview> | **Most-tested security topic** — roles, conditions, deny policies |
| **IAM deny policies** | <https://cloud.google.com/iam/docs/deny-overview> | New in 2023–2025: deny policies that override allows |
| **Workload Identity Federation** | <https://cloud.google.com/iam/docs/workload-identity-federation> | External IdP (AWS, Azure, GitHub Actions) → GCP access |
| **Service account best practices** | <https://cloud.google.com/iam/docs/best-practices-for-managing-service-account-keys> | Prefer Workload Identity over key files |
| **Cloud KMS** | <https://cloud.google.com/kms/docs> | Key hierarchy, CMEK, CSEK, EKM, Cloud HSM, rotation |
| **Secret Manager** | <https://cloud.google.com/secret-manager/docs> | Storing API keys, passwords, certificates — not KMS |
| **VPC Service Controls** | <https://cloud.google.com/vpc-service-controls/docs> | Service perimeters, access levels, bridge perimeters |
| **Cloud DLP / Sensitive Data Protection** | <https://cloud.google.com/dlp/docs> | Info types, de-identification, pseudonymization, inspection |
| **Security Command Center** | <https://cloud.google.com/security-command-center/docs> | Standard vs. Premium (CSPM, CIEM, threat detection) |
| **Binary Authorization** | <https://cloud.google.com/binary-authorization/docs> | Supply chain security — only signed images deployed to GKE |
| **Artifact Registry** | <https://cloud.google.com/artifact-registry/docs> | Container images + packages, SBOM, vulnerability scanning |
| **Organization Policy Service** | <https://cloud.google.com/resource-manager/docs/organization-policy/overview> | Guardrails: restrict resource locations, disable SA key creation |
| **Access Transparency / Access Approval** | <https://cloud.google.com/assured-workloads/access-transparency/docs> | Audit Google support access to your data |
| **Assured Workloads** | <https://cloud.google.com/assured-workloads/docs> | Compliance regimes: FedRAMP, HIPAA, IL4, CJIS |
| **Chronicle SIEM** | <https://cloud.google.com/chronicle/docs> | Cloud-native SIEM (appears in advanced security questions) |
| **reCAPTCHA Enterprise** | <https://cloud.google.com/recaptcha-enterprise/docs> | Bot/fraud protection for web apps (Cymbal Retail case study) |

### AI / ML — Vertex AI

| Service | Documentation | Exam Relevance |
|---|---|---|
| **Vertex AI overview** | <https://cloud.google.com/vertex-ai/docs/start/introduction-unified-platform> | Unified ML platform — training, serving, MLOps |
| **Vertex AI Pipelines** | <https://cloud.google.com/vertex-ai/docs/pipelines/introduction> | Kubeflow Pipelines / TFX managed — ML orchestration |
| **Vertex AI Feature Store** | <https://cloud.google.com/vertex-ai/docs/featurestore/overview> | Centralized feature management for training/serving |
| **Vertex AI Model Registry** | <https://cloud.google.com/vertex-ai/docs/model-registry/introduction> | Version models, track lineage, deploy to endpoints |
| **Vertex AI Model Garden** | <https://cloud.google.com/vertex-ai/docs/model-garden/explore-models> | Foundation models (Gemini, Llama, etc.) — RAG, fine-tuning |
| **AutoML** | <https://cloud.google.com/automl/docs> | No-code model training for tables, image, text, video |
| **BigQuery ML** | <https://cloud.google.com/bigquery/docs/bqml-introduction> | Train and serve ML models in BigQuery SQL |
| **Vertex AI Workbench** | <https://cloud.google.com/vertex-ai/docs/workbench/introduction> | Managed JupyterLab for data science |
| **Vertex AI Matching Engine** | <https://cloud.google.com/vertex-ai/docs/matching-engine/overview> | Vector similarity search — recommendation systems, semantic search |

### Operations & Observability

| Service | Documentation | Exam Relevance |
|---|---|---|
| **Cloud Monitoring** | <https://cloud.google.com/monitoring/docs> | Metrics, alerting policies, uptime checks, SLO monitoring |
| **Cloud Logging** | <https://cloud.google.com/logging/docs> | Log sinks, log-based metrics, retention, exclusions, CMEK logs |
| **Cloud Trace** | <https://cloud.google.com/trace/docs> | Distributed tracing — latency analysis for microservices |
| **Cloud Profiler** | <https://cloud.google.com/profiler/docs> | Continuous CPU/memory profiling in production |
| **Error Reporting** | <https://cloud.google.com/error-reporting/docs> | Aggregates and alerts on application exceptions |
| **Managed Service for Prometheus** | <https://cloud.google.com/stackdriver/docs/managed-prometheus> | Prometheus-compatible metrics at scale — GKE native |
| **Cloud Monitoring — SLO** | <https://cloud.google.com/stackdriver/docs/solutions/slo-monitoring> | Define SLOs, track error budgets, burn rate alerting |
| **Ops Agent** | <https://cloud.google.com/logging/docs/agent/ops-agent> | Unified logging + metrics agent for GCE VMs |

### Hybrid & Multi-Cloud

| Service | Documentation | Exam Relevance |
|---|---|---|
| **GKE Enterprise (Anthos)** | <https://cloud.google.com/anthos/docs/concepts/overview> | Fleet management across GCP, on-prem, other clouds |
| **Config Management** | <https://cloud.google.com/anthos-config-management/docs> | GitOps for GKE clusters — Policy Controller (OPA Gatekeeper) |
| **Cloud Service Mesh** | <https://cloud.google.com/service-mesh/docs> | Managed Istio — mTLS, traffic management, observability |
| **Google Distributed Cloud** | <https://cloud.google.com/distributed-cloud/docs> | GCP hardware on-prem/edge — air-gapped environments |
| **Migrate to Virtual Machines** | <https://cloud.google.com/migrate/virtual-machines/docs> | Lift-and-shift VMware/Hyper-V → GCE |
| **Migration Center** | <https://cloud.google.com/migration-center/docs> | Discovery, assessment, and migration planning |

### Infrastructure as Code & DevOps

| Service | Documentation | Exam Relevance |
|---|---|---|
| **Cloud Build** | <https://cloud.google.com/build/docs> | CI pipelines, build triggers, build steps, private pools |
| **Cloud Deploy** | <https://cloud.google.com/deploy/docs> | Managed CD — delivery pipelines, rollout strategies |
| **Terraform on GCP** | <https://cloud.google.com/docs/terraform> | Provider docs, Cloud Foundation Toolkit blueprints |
| **Config Connector** | <https://cloud.google.com/config-connector/docs/overview> | Kubernetes CRDs for GCP resources — GitOps for infra |
| **Artifact Registry** | <https://cloud.google.com/artifact-registry/docs> | Stores containers, Maven, npm, Python packages |
| **Cloud Source Repositories** | <https://cloud.google.com/source-repositories/docs> | Private Git hosting (legacy — most use GitHub/GitLab) |
| **Apigee API Management** | <https://cloud.google.com/apigee/docs> | Enterprise API gateway — rate limiting, monetization, analytics |
| **Cloud Endpoints** | <https://cloud.google.com/endpoints/docs> | Lightweight API management for Cloud Run/GKE/GCE |

---

## 🧪 Google Cloud Skills Boost (Qwiklabs) — Free Learning Paths

> **Sign in required** for all Skills Boost content. Free tier provides limited credits; paid subscription ($29/month) gives full access.
> 🔗 Portal: <https://www.cloudskillsboost.google/>

### Must-Complete Learning Paths for PCA

| Path | URL | Estimated Time | Cost |
|---|---|---|---|
| **Cloud Architecture: Design, Implement, and Manage** | <https://www.cloudskillsboost.google/paths/12> | ~40 hrs | Credits required **[Login]** |
| **Professional Cloud Architect Study Guide** (official path) | <https://www.cloudskillsboost.google/paths/78> | ~30 hrs | Credits required **[Login]** |
| **Google Cloud Fundamentals: Core Infrastructure** | <https://www.cloudskillsboost.google/courses/60> | ~8 hrs | Free preview available |
| **Networking in Google Cloud** | <https://www.cloudskillsboost.google/paths/14> | ~12 hrs | Credits required **[Login]** |
| **Security in Google Cloud** | <https://www.cloudskillsboost.google/paths/15> | ~16 hrs | Credits required **[Login]** |
| **Preparing for the Professional Cloud Architect Exam** | <https://www.cloudskillsboost.google/courses/879> | ~8 hrs | Credits required **[Login]** |
| **Hybrid and Multi-Cloud Architecture** | <https://www.cloudskillsboost.google/paths/109> | ~10 hrs | Credits required **[Login]** |

### High-Priority Individual Labs

| Lab | Skill Covered |
|---|---|
| [VPC Networking Fundamentals](https://www.cloudskillsboost.google/focuses/1229) | VPC, subnets, firewall rules |
| [Private Google Access and Cloud NAT](https://www.cloudskillsboost.google/focuses/1232) | Private networking patterns |
| [Cloud KMS: Qwik Start](https://www.cloudskillsboost.google/focuses/1713) | Encryption key management |
| [VPC Service Controls](https://www.cloudskillsboost.google/focuses/5593) | Service perimeters |
| [BigQuery: Qwik Start](https://www.cloudskillsboost.google/focuses/1892) | BigQuery fundamentals |
| [GKE Autopilot Cluster](https://www.cloudskillsboost.google/focuses/16943) | Managed Kubernetes |
| [Deploying to Cloud Run](https://www.cloudskillsboost.google/focuses/1148) | Serverless containers |
| [Pub/Sub: Qwik Start](https://www.cloudskillsboost.google/focuses/3719) | Async messaging |
| [Cloud Armor](https://www.cloudskillsboost.google/focuses/1232) | WAF / DDoS protection |
| [Terraform on Google Cloud](https://www.cloudskillsboost.google/paths/116) | IaC |

> 💡 **Tip:** Google Cloud Skills Boost frequently offers free "30-day challenge" or "Google Cloud Next" promotional access. Check the site before paying.

---

## 📄 Official Whitepapers & Architecture Frameworks

> All whitepapers are free from Google Cloud — no account required unless noted.

### Core Frameworks

| Document | URL | Why It Matters |
|---|---|---|
| **Google Cloud Architecture Framework** | <https://cloud.google.com/architecture/framework> | The primary reference — 6 pillars map directly to exam domains |
| **Google Cloud Well-Architected Framework (all pillars)** | <https://cloud.google.com/architecture/framework> | Operational excellence, reliability, security, cost, performance |
| **Reliability pillar** | <https://cloud.google.com/architecture/framework/reliability> | SRE, SLOs, disaster recovery, availability design |
| **Security pillar** | <https://cloud.google.com/architecture/framework/security> | Zero trust, defense in depth, IAM best practices |
| **Cost optimization pillar** | <https://cloud.google.com/architecture/framework/cost-optimization> | FinOps, right-sizing, pricing models |
| **Operational excellence pillar** | <https://cloud.google.com/architecture/framework/operational-excellence> | CI/CD, IaC, monitoring, incident management |
| **Performance optimization pillar** | <https://cloud.google.com/architecture/framework/performance-optimization> | Caching, CDN, autoscaling, database optimization |
| **System design on GCP** | <https://cloud.google.com/architecture/scalable-and-resilient-apps> | Scalable/resilient application design patterns |

### Networking Whitepapers

| Document | URL |
|---|---|
| **Best practices for enterprise organizations** | <https://cloud.google.com/docs/enterprise/best-practices-for-enterprise-organizations> |
| **VPC network design** | <https://cloud.google.com/solutions/best-practices-vpc-design> |
| **Hybrid connectivity decision tree** | <https://cloud.google.com/network-connectivity/docs/how-to/choose-product> |
| **Cloud Load Balancing deep dive** | <https://cloud.google.com/load-balancing/docs/choosing-load-balancer> |

### Security Whitepapers

| Document | URL |
|---|---|
| **Google infrastructure security design** | <https://cloud.google.com/security/infrastructure/design> |
| **Encryption at rest in Google Cloud** | <https://cloud.google.com/docs/security/encryption/default-encryption> |
| **Encryption in transit in Google Cloud** | <https://cloud.google.com/docs/security/encryption-in-transit> |
| **Zero trust: BeyondCorp Enterprise** | <https://cloud.google.com/beyondcorp-enterprise/docs/overview> |
| **HIPAA compliance on GCP** | <https://cloud.google.com/security/compliance/hipaa> |
| **PCI DSS on GCP** | <https://cloud.google.com/security/compliance/pci-dss> |
| **FedRAMP on GCP** | <https://cloud.google.com/security/compliance/fedramp> |
| **Data governance on Google Cloud** | <https://cloud.google.com/architecture/framework/system-design/data-governance> |

### Data & Migration Whitepapers

| Document | URL |
|---|---|
| **Migration to Google Cloud: overview** | <https://cloud.google.com/architecture/migration-to-gcp-overview> |
| **Database migration guide** | <https://cloud.google.com/architecture/database-migration-concepts-principles-part-1> |
| **BigQuery best practices** | <https://cloud.google.com/bigquery/docs/best-practices-performance-overview> |
| **Data lifecycle on GCP** | <https://cloud.google.com/solutions/data-lifecycle-cloud-patterns> |
| **Streaming analytics on GCP** | <https://cloud.google.com/architecture/streaming-data-from-cloud-storage-into-bigquery-using-dataflow> |

---

## 🎬 Google Cloud YouTube Resources

> All videos are free. Playlist links open in YouTube.

### Official Google Cloud Channels

| Channel | URL | Best For |
|---|---|---|
| **Google Cloud** (main channel) | <https://www.youtube.com/@googlecloud> | Product announcements, Cloud Next sessions |
| **Google Cloud Tech** | <https://www.youtube.com/@googlecloudtech> | Technical deep dives, architecture talks |
| **Learn GCP with Mahesh** | <https://www.youtube.com/@LearnGCPwithMahesh> | PCA exam-focused explanations |

### Key Playlists & Videos for PCA

| Video / Playlist | URL | Domain |
|---|---|---|
| **Cloud Next '24: Architecture keynotes** | <https://www.youtube.com/playlist?list=PLIivdWyY5sqLQ3m7X43KplkXkwcx2L56> | All domains |
| **Google Cloud Networking explained** | <https://www.youtube.com/watch?v=0hN-dyOV10c> | Domain 1, 2 |
| **GKE Deep Dive** | <https://www.youtube.com/watch?v=Wl7kH5HqBno> | Domain 1, 2 |
| **IAM best practices** | <https://www.youtube.com/watch?v=sRMIF2XCpjc> | Domain 3 |
| **VPC Service Controls deep dive** | <https://www.youtube.com/watch?v=HMolnZGbqkE> | Domain 3 |
| **Cloud KMS and encryption** | <https://www.youtube.com/watch?v=9Yt0TJOFSLY> | Domain 3 |
| **BigQuery architecture and best practices** | <https://www.youtube.com/watch?v=ZVgt1-LfWW4> | Domain 1, 4 |
| **Disaster recovery on GCP** | <https://www.youtube.com/watch?v=xsVHMpPGWbY> | Domain 6 |
| **SRE principles at Google** | <https://www.youtube.com/watch?v=d2wn_E1jxn8> | Domain 6 |
| **Cloud Armor and DDoS protection** | <https://www.youtube.com/watch?v=oXJ68Sa8jfU> | Domain 3 |
| **Vertex AI overview** | <https://www.youtube.com/watch?v=gT4qqHMiEpA> | Domain 1 |
| **FinOps on Google Cloud** | <https://www.youtube.com/watch?v=LuDmbRd5MUI> | Domain 4 |
| **Terraform on GCP** | <https://www.youtube.com/watch?v=GEcBkTm7UAU> | Domain 2, 5 |

---

## 👥 Community Forums & Structured Courses

> Use the online practice exam links above for testing. This section focuses on discussion spaces and structured courses that complement practice questions.

### Community Forums & Study Groups

| Community | URL | Notes |
|---|---|---|
| **Google Cloud Community** | <https://www.googlecloudcommunity.com/> | Official community — exam prep discussions, Q&A |
| **r/GoogleCloud** | <https://www.reddit.com/r/googlecloud/> | Reddit — exam experiences, resource recommendations |
| **r/gcpcloudcertification** | <https://www.reddit.com/r/gcpcloudcertification/> | GCP cert-specific subreddit |
| **GCP Study Hub (Discord)** | Search "GCP Study Hub Discord" on Reddit | Active Discord server for GCP exam prep (link changes) |
| **Cloud Certified (Slack)** | <https://cloudcertified.org/> | Slack community for multi-cloud cert prep |
| **LinkedIn Learning** | Search "Google Cloud Architect" on LinkedIn Learning | Video courses — often free with LinkedIn Premium |

### Video Courses (Structured Learning)

| Course | Platform | Instructor | Notes |
|---|---|---|---|
| **Google Cloud Professional Cloud Architect** | Coursera | Google Cloud Training | Official Google course, audit for free **[Login]** |
| **GCP Professional Cloud Architect — Masterclass** | Udemy | Dan Sullivan | Well-rated, updated for 2024–2025 |
| **Google Cloud Associate + Professional** | A Cloud Guru / Pluralsight | Multiple | Good for hands-on labs |
| **GCP: Google Cloud Platform — Complete Intro** | Udemy | Stephane Maarek / others | Context for non-GCP backgrounds |

---

## 📖 Recommended Books

| Book | Author | Notes |
|---|---|---|
| **Official Google Cloud Certified Professional Cloud Architect Study Guide** | Dan Sullivan | The de facto study guide — covers all domains with practice questions |
| **Google Cloud Platform in Action** | JJ Geewax | Practical GCP programming and architecture (supplementary) |
| **Site Reliability Engineering** | Google SRE team | Free online: <https://sre.google/sre-book/table-of-contents/> — Domain 6 critical reading |
| **The Site Reliability Workbook** | Google SRE team | Free online: <https://sre.google/workbook/table-of-contents/> — SLO implementation |
| **Cloud Native Patterns** | Cornelia Davis | Cloud architecture patterns (supplementary) |

---

## 🗂️ Quick Reference: Official Documentation Shortcuts

```
# Compute selection
https://cloud.google.com/products/compute

# Database selection guide
https://cloud.google.com/products/databases

# Storage selection guide
https://cloud.google.com/products/storage

# Load balancer selection
https://cloud.google.com/load-balancing/docs/choosing-load-balancer

# Hybrid connectivity selection
https://cloud.google.com/network-connectivity/docs/how-to/choose-product

# Pricing calculator
https://cloud.google.com/products/calculator

# SLA summary (all products)
https://cloud.google.com/terms/sla

# IAM predefined roles reference
https://cloud.google.com/iam/docs/understanding-roles
```

---

> 📅 **Exam: April 30, 2026** — Bookmark this file and revisit weekly as you progress through the study plan.
> 🔄 Check the [official exam guide](https://cloud.google.com/learn/certification/guides/cloud-architect) periodically — Google updates the PCA guide with service changes.
