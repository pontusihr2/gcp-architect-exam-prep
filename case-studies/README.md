# GCP Professional Cloud Architect — Case Studies Guide

> **Exam Version:** 2025 (Updated October 2025)
> **Certification:** Google Professional Cloud Architect (PCA)

---

## How Case Studies Appear on the Exam

The PCA exam includes **real case study scenarios** that are provided as reference material. During the exam:

- Case studies are accessible via a **dedicated panel** — you can read them at any time during the exam without leaving the current question.
- Roughly **15–20% of exam questions** are tied to one of the official case studies.
- Questions will specify which case study they reference (e.g., *"In the context of EHR Healthcare..."*).
- Case study questions are often the **harder questions** on the exam — they require you to synthesize requirements, constraints, and trade-offs rather than recall isolated facts.

### The Four Official Case Studies (2025)

| File | Company | Domain | Key Themes |
|------|---------|--------|------------|
| `ehr-healthcare.md` | EHR Healthcare | Healthcare SaaS | HIPAA, lift-and-shift, containerization, HL7/FHIR |
| `altostrat-media.md` | Altostrat Media | Media & Streaming | CDN, AI/ML, Vertex AI, Gemini, large-scale data |
| `cymbal-retail.md` | Cymbal Retail | Omnichannel Retail | PCI-DSS, auto-scaling, real-time inventory, ERP integration |
| `knightmotives-automotive.md` | KnightMotives Automotive | Connected Vehicles / IoT | IoT telemetry, OTA updates, digital twins, UNECE WP.29 |

---

## How to Approach Case Study Questions

### Step 1: Read the Case Study Before the Exam

Google publishes the case studies publicly before the exam. **Read all four case studies thoroughly** before exam day. Know them well enough that you don't need to re-read them during the exam.

### Step 2: Identify the "Lenses" for Each Company

For each case study, always keep these lenses in mind:

1. **Business goals** — what is the company trying to achieve commercially?
2. **Technical requirements** — what must the architecture actually do?
3. **Constraints** — cost, compliance, existing investments, timeline
4. **Pain points** — what is currently broken or inefficient?

### Step 3: Eliminate Wrong Answers Using Requirements

Most wrong answers on case study questions violate one of:
- A stated compliance requirement (e.g., selecting a non-HIPAA-compliant service for EHR Healthcare)
- A scalability requirement (e.g., a solution that can't handle 10x traffic spikes for Cymbal Retail)
- A cost constraint (e.g., over-engineering a solution when cost optimization is explicitly required)
- A latency requirement (e.g., batch processing for a near-real-time use case)

### Step 4: Apply the "Most GCP-Native" Heuristic

Google's exam tends to favor:
- **Managed services** over self-managed (e.g., Cloud SQL over Compute Engine + MySQL)
- **Serverless** where appropriate (Cloud Run, Cloud Functions, BigQuery)
- **Google-proprietary services** that solve the stated problem directly (Pub/Sub for streaming, Dataflow for pipelines, Vertex AI for ML)

---

## Tips for Answering Case Study Questions

### ✅ Do

- **Match the answer to stated requirements** — if the company says "minimize operational overhead," choose managed/serverless services.
- **Respect compliance constraints** — HIPAA, PCI-DSS, and UNECE WP.29 significantly narrow your valid architecture options.
- **Consider multi-region vs. single-region** based on availability and latency requirements.
- **Think about cost at scale** — for companies like Altostrat Media and KnightMotives, cost matters; choose efficient storage tiers and autoscaling.
- **Use Cloud Armor, VPC Service Controls, and IAM correctly** — security is always in scope.

### ❌ Don't

- Don't over-engineer: if a question asks for a simple migration, don't propose a full microservices re-architecture.
- Don't ignore existing infrastructure: the case studies mention on-premises systems that need to integrate or migrate — honor those constraints.
- Don't confuse similar services: Cloud Dataflow vs. Cloud Dataproc, Bigtable vs. Spanner, Cloud Run vs. GKE — know when to use each.
- Don't forget DR and backup requirements: most case studies mention availability targets or DR objectives.

---

## Common Service Mapping to Case Study Themes

| Theme | Key GCP Services |
|-------|-----------------|
| Containerizing legacy apps | GKE, Cloud Run, Artifact Registry, Cloud Build |
| Relational DB migration | Cloud SQL, Database Migration Service, AlloyDB |
| Healthcare / HIPAA | VPC Service Controls, CMEK, Cloud Healthcare API, Assured Workloads |
| Real-time streaming | Pub/Sub, Dataflow, BigQuery |
| AI/ML workloads | Vertex AI, BigQuery ML, Gemini API, Model Garden |
| Global content delivery | Cloud CDN, Media CDN, Cloud Load Balancing |
| IoT ingestion | Pub/Sub, IoT Core (deprecated → direct Pub/Sub or MQTT bridge), Dataflow |
| Cost optimization | Committed Use Discounts, Spot VMs, Autoscaler, Cloud Storage lifecycle policies |
| Security & compliance | Cloud KMS, VPC Service Controls, Security Command Center, IAM, Audit Logs |
| Observability | Cloud Monitoring, Cloud Logging, Cloud Trace, Cloud Profiler, Error Reporting |
| Hybrid connectivity | Cloud VPN, Dedicated Interconnect, Partner Interconnect |
| Disaster recovery | Cloud Storage, Backup and DR, multi-region deployments, Cloud DNS failover |

---

## Study Recommendations

1. **Read each case study file top-to-bottom** at least twice before the exam.
2. **Answer all sample questions** in each file without looking at the answers, then review explanations.
3. **Cross-reference with the official exam guide** at [cloud.google.com/certification/cloud-architect](https://cloud.google.com/certification/cloud-architect).
4. For each wrong answer in practice, trace it back to a requirement in the case study you missed.
5. Time yourself: case study questions shouldn't take more than **2–3 minutes each**.

---

*Last updated: October 2025*
