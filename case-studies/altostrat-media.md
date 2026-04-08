# Case Study 2: Altostrat Media

> **Exam Version:** Google Professional Cloud Architect (PCA) — 2025 (Updated October 2025)
> **Domain:** Media & Content Delivery | **Compliance:** GDPR (EU operations), COPPA (children's content)

---

## 1. Company Overview

**Altostrat Media** is a global media and content delivery company operating a video streaming platform, content management system, and programmatic advertising network. They serve hundreds of millions of users across North America, Europe, Southeast Asia, and Latin America.

**Products:**
- **AltostreamTV**: subscription video-on-demand (SVOD) platform — 50M+ monthly active users
- **AltoAds**: programmatic ad delivery network — serves real-time bidding (RTB) ads alongside content
- **AltoStudio**: content management and transcoding platform for content creators and studios
- **AltoDiscover**: AI-powered content discovery and recommendation engine

**Current challenges:**
- Uses 3 CDN providers (Akamai, Fastly, CloudFront) — complex multi-CDN management, fragmented analytics
- ML model training pipelines are slow and expensive on-premises GPU clusters
- Advertising analytics latency is 30+ minutes — ad campaign managers want near-real-time reporting
- Content tagging (metadata enrichment) is manual — thousands of new titles monthly
- Cost per streamed hour is increasing at scale; leadership has mandated 20% cost reduction
- No unified observability across CDN, application, and data layers

**Strategic goals:**
- Consolidate on GCP as primary cloud and CDN provider
- Use Vertex AI and Gemini models for content intelligence (tagging, search, recommendations)
- Reduce ad analytics latency to under 5 minutes
- Build a real-time content recommendation system personalized to each user
- Achieve 99.95% availability for AltostreamTV

---

## 2. Technical Requirements

1. **Global content delivery** with sub-100ms time-to-first-byte (TTFB) for video segments for 95th percentile of users worldwide.
2. **Video transcoding pipeline** — ingest raw video from studios (up to 4K), transcode to multiple formats (HLS, DASH), store and deliver efficiently.
3. **Real-time ad analytics** — process ad impression, click, and conversion events within 5 minutes of occurrence.
4. **Personalization at scale** — generate real-time content recommendations for 50M+ users with low latency (<200ms for recommendation API calls).
5. **AI-powered content tagging** — automatically enrich content metadata (genre, mood, characters, themes) using Gemini vision models.
6. **Unified observability** — single pane of glass for CDN performance, application health, and data pipeline health.
7. **ML training infrastructure** — train and retrain recommendation models on petabytes of behavioral data.
8. **Cost optimization** — storage lifecycle policies, compute autoscaling, and Spot VM usage for batch workloads.
9. **Global traffic management** — route users to the nearest healthy edge location; failover across regions automatically.
10. **Multi-region data replication** — content library available in all regions; user data and preferences synced globally.

---

## 3. Business Requirements

1. **Consolidate CDN vendors** — reduce operational complexity and enable unified analytics.
2. **Reduce cost per streamed hour by 20%** — through efficient storage, CDN optimization, and compute right-sizing.
3. **Improve content discovery** — AI-powered search and recommendations should increase average session length by 15%.
4. **Near-real-time ad reporting** — advertising partners require campaign performance data within 5 minutes.
5. **Enable generative AI features** — content summaries, smart search, and AI-generated content trailers using Gemini models.
6. **GDPR compliance** — EU user data must remain in EU regions; right-to-erasure must be implementable.
7. **COPPA compliance** — children's content section must prevent behavioral ad targeting and data collection for under-13 users.
8. **Reduce ML model training time** — current weekly retraining cycle must become daily or continuous.
9. **Developer velocity** — data science teams must be able to experiment with new models without infrastructure bottlenecks.

---

## 4. Existing Technical Infrastructure

### Content Infrastructure

| System | Technology | Notes |
|--------|-----------|-------|
| Origin servers | Bare metal, nginx | Located in 3 colocation facilities (US, EU, APAC) |
| Video storage | NetApp NAS + S3-compatible object store | ~5 PB total video library |
| Transcoding cluster | 200-node on-prem cluster, FFmpeg-based | Utilization: 40% average, 95% during prime time |
| Content CMS | Custom PHP/MySQL | Manages 500K+ titles |

### Ad Tech Stack

| System | Technology | Notes |
|--------|-----------|-------|
| Ad server | Custom Java app | Handles 2M+ RTB requests/second at peak |
| Ad event pipeline | Kafka + Spark on-prem | 30-min latency to reporting |
| Campaign reporting DB | PostgreSQL | Queried by ad campaign managers |

### Data & ML Infrastructure

| System | Technology | Notes |
|--------|-----------|-------|
| Data warehouse | On-prem Hadoop cluster (200 nodes) | 3PB of behavioral and content data |
| ML training | On-prem GPU cluster (50 x A100) | Shared by multiple teams; contention issues |
| Recommendation API | Python FastAPI, on-prem | Pre-computed recommendations, refreshed every 6 hours |

### CDN & Networking

- Multi-CDN: Akamai (US/EU), Fastly (APAC), CloudFront (Latin America)
- No unified CDN analytics or cache-hit visibility
- Peering agreements with major ISPs in some markets

---

## 5. Recommended GCP Architecture

### Project Structure

```
Organization: altostrat.com
  └── Folder: Production
  │     ├── Project: alto-media-core       (Cloud Run, GKE, Pub/Sub)
  │     ├── Project: alto-cdn              (Media CDN, Cloud Load Balancing)
  │     ├── Project: alto-data             (BigQuery, Dataflow, Pub/Sub)
  │     ├── Project: alto-ml              (Vertex AI, Cloud Storage ML datasets)
  │     └── Project: alto-adtech          (Ad server, ad analytics pipeline)
  └── Folder: Non-Production
        ├── Project: alto-dev
        └── Project: alto-staging
```

### Content Delivery

| Requirement | GCP Service | Details |
|-------------|------------|---------|
| Primary CDN | **Media CDN** | Google's media-optimized CDN, built on same infrastructure as YouTube; handles HLS/DASH natively |
| HTTP(S) CDN fallback | **Cloud CDN** | For non-video assets (thumbnails, web assets) |
| Global load balancing | **Cloud Load Balancing** (Global External) | Anycast routing; routes to nearest origin |
| DDoS protection | **Cloud Armor** | Adaptive protection + rate limiting for origin servers |
| Origin | **Cloud Storage** (nearline/coldline) | Video segments served directly from GCS via Media CDN |

**Media CDN configuration:**
- Route patterns: `/hls/**`, `/dash/**` → Media CDN (video segments)
- Route patterns: `/thumbnails/**`, `/metadata/**` → Cloud CDN
- Cache policies: HLS segment TTL 7 days; manifests TTL 60 seconds (dynamic)
- Edge caching nodes in 30+ global PoPs (leverages Google's edge network)

### Video Ingestion & Transcoding

```
Studio Upload → Cloud Storage (raw, us-central1)
                     │
                     ▼ (GCS event notification via Pub/Sub)
              Cloud Run Job (Transcoding Orchestrator)
                     │
                     ├─→ Transcoding API (managed service — replaces FFmpeg cluster)
                     │     └── Outputs: HLS (360p, 720p, 1080p, 4K), DASH, WebM
                     │
                     ▼
              Cloud Storage (transcoded segments — multi-region bucket)
                     │
                     ▼
              Media CDN (origin pull from GCS)
```

- **Transcoding API** (formerly Video Intelligence API for processing) — handles GPU-accelerated transcoding as a managed service; no infrastructure to manage
- **Cloud Storage multi-region bucket** (US, EU, ASIA): video segments replicated for low-latency origin pulls globally
- **Storage lifecycle policy**: raw uploads → delete after 90 days; cold archive to Coldline after 1 year

### Real-Time Ad Analytics Pipeline

```
Ad Impression/Click Events (2M events/sec)
        │
        ▼
  Pub/Sub (ad-events topic)
        │
        ▼
  Dataflow Streaming Pipeline
  (windowed aggregations: impressions, clicks, CTR per 1-min windows)
        │
        ├─→ BigQuery (streaming inserts) ← real-time reporting
        │     └── Looker Studio dashboards (auto-refresh: 1 min)
        │
        └─→ Bigtable (low-latency lookup for fraud detection)
              └── Real-time bidding fraud check (<5ms lookup)
```

- **Pub/Sub**: ingests ad events; guaranteed at-least-once delivery; auto-scales to 2M+ msgs/sec
- **Dataflow**: streaming pipeline with 1-minute tumbling windows; aggregates by campaign, ad unit, device type
- **BigQuery**: streaming inserts deliver data within seconds; dashboard queries hit materialized views for performance
- Achieves <5 minute latency from event to reporting dashboard (down from 30 minutes)

### AI/ML: Personalization & Content Intelligence

#### Recommendation System

```
User Behavior Events (clicks, watches, skips)
        │
        ▼
  Pub/Sub → Dataflow → BigQuery (behavioral data lake)
                              │
                              ▼
                    Vertex AI Pipelines
                    (daily retraining of recommendation model)
                              │
                              ▼
                    Vertex AI Model Registry
                              │
                              ▼
                    Vertex AI Endpoints (online serving)
                    └── Recommendation API: <200ms P95 latency
```

- **Vertex AI Feature Store**: stores pre-computed user features (watch history, preferences, demographics)
- **Vertex AI Pipelines**: orchestrates daily model retraining (Kubeflow Pipelines under the hood)
- **Vertex AI Training**: distributed training on TPU v4 pods for matrix factorization + transformer-based models
- **Vertex AI Endpoints**: autoscaling online serving with model versioning and A/B testing via traffic splitting

#### Content Intelligence with Gemini

```
New Content Ingested
        │
        ▼
  Cloud Run service (Content Enrichment)
        │
        ├─→ Vertex AI Gemini 1.5 Pro (vision) — generates:
        │     ├── Genre tags, mood tags, content descriptors
        │     ├── Scene-level summaries (used for smart search)
        │     └── Auto-generated episode descriptions
        │
        ├─→ Video Intelligence API — scene detection, explicit content flagging
        │
        └─→ Cloud Search / Vertex AI Search — indexes enriched metadata
              └── Powers semantic search on AltostreamTV
```

- **Gemini 1.5 Pro**: multimodal model; analyzes video frames + transcript to generate structured metadata
- **Vertex AI Search** (formerly Enterprise Search): semantic search across 500K+ titles using enriched metadata
- Results: content tagging time reduced from days (manual) to minutes (automated)

### Data Platform

| Layer | Service | Purpose |
|-------|---------|---------|
| Streaming ingest | **Pub/Sub** | All event streams (user, ad, content) |
| Stream processing | **Dataflow** | ETL, windowed analytics, enrichment |
| Analytics warehouse | **BigQuery** | Petabyte-scale analytics, ML feature engineering |
| ML feature store | **Vertex AI Feature Store** | Low-latency feature serving for recommendation API |
| Low-latency lookups | **Bigtable** | Ad fraud detection, session state |
| User preference store | **Firestore** | Per-user settings, watchlists, parental controls |

### Observability

- **Cloud Monitoring**: custom dashboards — CDN cache hit ratio, video start time, rebuffering rate
- **Cloud Logging**: centralized logs from all services including Media CDN access logs
- **Cloud Trace**: distributed tracing for recommendation API and ad serving latency
- **Network Intelligence Center**: CDN performance insights, topology visualization
- **BigQuery Log Analytics**: ad-hoc analysis of CDN access logs at petabyte scale

---

## 6. Migration Strategy

### Phase 1 — CDN Consolidation (Months 1–4)

- Provision Media CDN and Cloud CDN configurations
- Mirror content library to Cloud Storage multi-region buckets (via Transfer Service from S3-compatible on-prem store)
- Gradually shift traffic by geography: start with Latin America (lowest risk), then APAC, EU, US
- Run Media CDN alongside existing CDN providers using DNS-based traffic management
- Validate cache hit rates, TTFB, and rebuffering metrics before expanding
- Decommission Akamai/Fastly/CloudFront contracts as traffic shifts

### Phase 2 — Data Platform Migration (Months 3–6)

- Deploy Pub/Sub + Dataflow streaming pipeline for ad events in parallel with existing Kafka+Spark
- Shadow-mode comparison: validate Dataflow outputs match Spark outputs for 2 weeks
- Migrate Hadoop DW to BigQuery using BigQuery Data Transfer Service + custom Dataflow jobs
- Decommission Kafka/Spark once Dataflow pipeline is validated

### Phase 3 — ML Platform Migration (Months 5–9)

- Migrate ML training workloads to Vertex AI Training (GPU/TPU instances replace on-prem GPU cluster)
- Use Vertex AI Pipelines to orchestrate retraining workflows (was ad-hoc scripts)
- Deploy recommendation model to Vertex AI Endpoints; run A/B test with existing system
- Roll out Gemini-based content tagging pipeline for new content; backfill historical catalog

### Phase 4 — Origin & Transcoding Migration (Months 6–12)

- Deploy Transcoding API pipeline; run parallel with on-prem FFmpeg cluster
- Migrate on-prem CMS to Cloud-based CMS or integrate via APIs
- Shift all new content through GCP transcoding pipeline
- Decommission on-prem transcoding cluster and NAS storage
- Final decommission of colocation facilities

---

## 7. Security & Compliance Considerations

### GDPR (EU Operations)

| Requirement | Implementation |
|-------------|---------------|
| Data residency | Separate GCP project with **Assured Workloads** (EU region constraint); EU user data only stored in `europe-west1`, `europe-west4` |
| Right to erasure | Firestore user records deletable via API; BigQuery user events deleted via row-level TTL policies; purge pipeline for Cloud Storage |
| Consent management | User consent stored in Firestore; Dataflow pipeline enforces consent flags before behavioral data processing |
| Data processing audit | Cloud Audit Logs for all access to EU user data; DPA (Data Processing Agreement) with Google |

### COPPA (Children's Content)

- Users flagged as under-13 stored in separate Firestore collection with `coppa_restricted: true`
- Dataflow pipeline skips behavioral data collection for COPPA users
- Ad serving for COPPA users routes to contextual-only ad inventory (no behavioral targeting)
- Parental controls enforced via Firestore + Cloud Run policy service

### Platform Security

- **Cloud Armor**: WAF rules for ad server (OWASP Top 10), rate limiting on RTB endpoints (2M req/sec)
- **Cloud KMS**: CMEK for BigQuery datasets containing user PII; for ML training datasets
- **VPC Service Controls**: perimeter around `alto-data` and `alto-ml` projects
- **IAM**: data scientists access Vertex AI via Workbench; no direct access to production BigQuery tables with PII — use authorized views and column-level security
- **Artifact Registry + Binary Authorization**: all container images scanned before deployment

---

## 8. Key Design Decisions

### Decision 1: Media CDN vs. Cloud CDN for Video

**Chose Media CDN** for video streaming.
- *Rationale*: Media CDN is purpose-built for large media files and uses Google's edge infrastructure (the same network that powers YouTube). It natively handles HLS/DASH byte-range requests, supports large-file optimizations, and has better cache fill performance than Cloud CDN for media workloads. Cloud CDN is better suited for smaller objects (APIs, web assets).
- *Trade-off*: Media CDN has different pricing and configuration than Cloud CDN; requires separate setup.

### Decision 2: Bigtable for Ad Fraud Detection

**Chose Bigtable** for real-time fraud lookups.
- *Rationale*: RTB ad serving requires fraud lookups in <5ms. Bigtable provides single-digit millisecond latency at scale for key-value lookups. BigQuery and Cloud SQL are too slow for this use case. Firestore (~10ms) is borderline.
- *Trade-off*: Bigtable has a minimum cost (~3 nodes); not cost-effective for small workloads. At Altostrat's scale (2M req/sec) it is fully justified.

### Decision 3: Vertex AI Endpoints vs. Custom Serving (TensorFlow Serving on GKE)

**Chose Vertex AI Endpoints**.
- *Rationale*: Managed model serving with autoscaling, monitoring, and model versioning. Traffic splitting enables A/B testing between model versions. Integrates natively with Vertex AI Model Registry and Pipelines for end-to-end MLOps.
- *Trade-off*: Less flexibility in serving runtime customization compared to a custom TFServing deployment on GKE. Acceptable given Altostrat's goal of reducing operational burden.

### Decision 4: Multi-Region Cloud Storage vs. Dual-Region for Video Library

**Chose Multi-Region Cloud Storage** for video segments.
- *Rationale*: Altostrat serves users in US, EU, and APAC. Multi-region buckets (e.g., `us`, `eu`, `asia`) ensure that Media CDN origin pulls are served from nearby storage locations, minimizing cache fill latency. Dual-region (2 locations) would not cover APAC.
- *Trade-off*: Multi-region storage costs more than single-region. Cost is offset by reduced CDN origin pull bandwidth and improved cache fill performance.

### Decision 5: Gemini 1.5 Pro for Content Tagging

**Chose Gemini 1.5 Pro** (multimodal) over custom fine-tuned models.
- *Rationale*: Gemini 1.5 Pro's 1M token context window allows analysis of full video transcripts alongside sampled frames. Eliminates the need to build and maintain a custom computer vision pipeline. Prompt engineering with structured output (JSON schema) produces consistent metadata.
- *Trade-off*: Per-token cost is higher than a smaller custom model. Mitigated by processing new content only (not re-processing existing library); and cost is justified by eliminating manual tagging labor.

---

## 9. Sample Exam Questions

---

**Question 1**

Altostrat Media needs to deliver HLS video segments globally with sub-100ms TTFB for 95% of users. They want to use a single GCP CDN service. Which service should they choose?

- A) Cloud CDN with Cloud Storage origin
- B) Media CDN with Cloud Storage origin
- C) Cloud Load Balancing with multi-region backend
- D) Cloud Interconnect with on-premises origin

**Answer: B — Media CDN with Cloud Storage origin**

*Explanation:* Media CDN is Google's media-optimized CDN built on the same infrastructure that powers YouTube. It is specifically designed for large media files, HLS/DASH streaming, and byte-range request handling. It provides sub-100ms TTFB by leveraging Google's global edge PoPs and pre-positioned caches. Cloud CDN (A) is suitable for general web content and APIs but not optimized for video streaming at this scale. Cloud Load Balancing (C) routes to regional backends but is not a CDN — it doesn't cache content at the edge. Interconnect (D) is a hybrid connectivity product, not a CDN.

---

**Question 2**

Altostrat Media currently processes ad events with 30-minute latency using Kafka + Spark. They want to reduce latency to under 5 minutes and integrate with BigQuery for dashboards. What is the recommended GCP architecture?

- A) Cloud Pub/Sub → Cloud Dataproc (Spark) → BigQuery
- B) Cloud Pub/Sub → Dataflow (streaming pipeline) → BigQuery streaming inserts
- C) Cloud Storage → BigQuery batch load → Looker Studio
- D) Cloud Pub/Sub → Cloud Functions → Cloud SQL

**Answer: B — Cloud Pub/Sub → Dataflow (streaming pipeline) → BigQuery streaming inserts**

*Explanation:* Pub/Sub handles high-throughput event ingestion at millions of messages per second. Dataflow's streaming mode (Apache Beam) processes events in near-real-time with windowed aggregations and writes to BigQuery via streaming inserts, making data available in dashboards within seconds-to-minutes. This achieves well under 5 minutes latency. Option A uses Dataproc (Spark) which introduces batch-oriented latency (minutes to hours for micro-batch). Option C is a batch pattern — latency is tied to batch frequency. Option D can't handle 2M events/sec with Cloud Functions and Cloud SQL lacks the streaming analytics capability required.

---

**Question 3**

Altostrat's data science team wants to retrain their recommendation model daily on petabytes of behavioral data stored in BigQuery, deploy it to an endpoint, and run A/B tests between model versions. Which Vertex AI services should they use?

- A) Vertex AI Workbench + Vertex AI Training + Vertex AI Endpoints
- B) Vertex AI Pipelines + Vertex AI Training + Vertex AI Model Registry + Vertex AI Endpoints
- C) BigQuery ML + Cloud Run (custom serving)
- D) Cloud Dataproc + TensorFlow Serving on GKE

**Answer: B**

*Explanation:* The complete MLOps workflow requires: **Vertex AI Pipelines** to orchestrate the retraining workflow (trigger, preprocess, train, evaluate, register); **Vertex AI Training** to execute distributed model training on GPUs/TPUs with BigQuery as data source; **Vertex AI Model Registry** to version and manage models with metadata; **Vertex AI Endpoints** for online serving with traffic splitting (enables A/B testing between model versions). Workbench (A) is a Jupyter notebook environment — useful for development but not for production orchestration. BigQuery ML (C) is for simpler SQL-based models, not deep learning recommendation models at this scale. Dataproc + custom GKE (D) has no managed MLOps tooling.

---

**Question 4**

Altostrat needs to implement automatic content tagging for new videos using Gemini multimodal models. Videos are uploaded to Cloud Storage and should be processed asynchronously. Which architecture is correct?

- A) Cloud Storage trigger → Cloud Functions → Vertex AI Gemini API → Firestore (metadata)
- B) Cloud Storage notification → Pub/Sub → Cloud Run service → Vertex AI Gemini API → Firestore (metadata)
- C) Cloud Scheduler → Batch VM → Video Intelligence API → BigQuery
- D) Cloud Storage trigger → Dataflow → BERT model on Vertex AI

**Answer: B**

*Explanation:* Cloud Storage object notifications publish to Pub/Sub when a new video is uploaded. A Cloud Run service subscribes to the Pub/Sub topic, extracts video frames and transcript, calls the Vertex AI Gemini API for metadata generation (genre, mood, scene descriptions), and writes structured metadata to Firestore. This is asynchronous, scalable, and serverless. Cloud Functions (A) has a 60-minute maximum timeout — video analysis with Gemini can take longer for full-length films; Cloud Run supports longer timeouts (up to 60 minutes) and is more appropriate. Cloud Scheduler (C) is a polling/batch approach — not event-driven. BERT (D) is a text-only model — not suitable for video frame analysis.

---

**Question 5**

Altostrat Media serves EU users and must comply with GDPR. EU user data must remain within EU regions, and users must be able to exercise their right to erasure. How should Altostrat implement EU data residency in GCP?

- A) Store EU user data in a separate BigQuery dataset with region set to `EU`; use row-level security to restrict access
- B) Create a separate GCP project with Assured Workloads (EU regions policy); store EU user data only in `europe-west*` regions; implement a data erasure pipeline
- C) Enable VPC Service Controls on the main project with EU subnet restrictions
- D) Use Cloud Storage Regional buckets in `europe-west1` for all EU data

**Answer: B**

*Explanation:* **Assured Workloads** enforces organizational policies that restrict resource creation to EU regions, preventing accidental data storage outside the EU. A separate project provides a clean administrative and billing boundary for GDPR compliance. The data erasure pipeline deletes user records from Firestore, applies BigQuery row-level TTL, and purges Cloud Storage objects on right-to-erasure requests. Option A (BigQuery dataset region) helps for BigQuery but doesn't prevent EU data from being stored in other services in the same project. VPC Service Controls (C) controls API access, not data residency. Cloud Storage alone (D) doesn't cover all services where EU user data might land.

---

**Question 6**

Altostrat's recommendation API currently has P95 latency of 450ms due to pre-computed recommendations being refreshed only every 6 hours. They want to reduce P95 latency to under 200ms with more personalized, real-time recommendations. What change achieves this?

- A) Move pre-computed recommendations to Bigtable for faster reads
- B) Use Vertex AI Feature Store for real-time feature serving + Vertex AI Endpoints for online inference
- C) Increase the refresh frequency of pre-computed recommendations to every 30 minutes
- D) Deploy recommendation model to Cloud Run with Redis cache

**Answer: B**

*Explanation:* **Vertex AI Feature Store** serves pre-computed user features with single-digit millisecond latency. **Vertex AI Endpoints** serves the recommendation model for online inference, typically achieving <100ms latency for well-optimized models. Together, they enable real-time personalized recommendations far better than batch pre-computation. Moving to Bigtable (A) would speed up reads but doesn't address the staleness problem — recommendations are still only as fresh as the last batch. More frequent refreshes (C) reduces staleness but doesn't improve latency and increases compute cost significantly. Cloud Run + Redis (D) requires building and maintaining a custom serving stack — and P95 latency improvement is not guaranteed without the managed endpoint infrastructure.

---

**Question 7**

Altostrat Media wants to minimize the cost of storing 5PB of video content on GCP. Videos are actively streamed by users, but older content (>2 years) is rarely accessed (less than 1 access per month). Which Cloud Storage configuration is most cost-effective?

- A) All video in Standard storage class
- B) All video in Nearline storage class
- C) Recent content (<2 years) in Standard; older content (>2 years) in Coldline; lifecycle policy to automate transitions
- D) All video in Coldline storage class

**Answer: C**

*Explanation:* Cloud Storage lifecycle policies automate object transitions between storage classes. Standard storage is appropriate for actively streamed content where retrieval latency and retrieval costs must be minimized. Coldline is appropriate for content accessed less than once per quarter — the lower storage cost outweighs the retrieval cost for rarely-accessed archives. A lifecycle rule transitions objects to Coldline after 730 days (2 years). All-Standard (A) is unnecessarily expensive for cold content. All-Nearline (B) is not optimal — Nearline has retrieval fees that add up for frequently accessed content. All-Coldline (D) would make streaming active content prohibitively expensive due to per-retrieval charges.

---

**Question 8**

Altostrat's ad server handles 2M real-time bidding (RTB) requests per second. Each request requires a fraud check (key-value lookup) that must complete in under 5ms. Which database service should back the fraud detection lookup?

- A) Cloud SQL (PostgreSQL)
- B) Firestore
- C) BigQuery
- D) Cloud Bigtable

**Answer: D — Cloud Bigtable**

*Explanation:* Cloud Bigtable delivers single-digit millisecond latency for key-value reads at massive throughput — it is designed for exactly this use case (high-throughput, low-latency lookups). At 2M req/sec, Bigtable's horizontal scaling ensures consistent performance. Cloud SQL (A) has latency of 1–10ms but cannot scale to 2M req/sec without significant sharding complexity. Firestore (B) typical latency is 10–50ms — too slow for 5ms RTB requirement. BigQuery (C) is an analytical warehouse with query latency of hundreds of milliseconds to minutes — completely unsuitable for RTB.

---

## 10. Architecture Diagram (Text)

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                      Altostrat Media — GCP Architecture                      │
├──────────────────────────────────────────────────────────────────────────────┤
│                                                                               │
│  CONTENT DELIVERY                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │  End Users (Global)                                                   │   │
│  │       │                                                               │   │
│  │  ┌────▼──────────────────────────────────────────────────────────┐  │   │
│  │  │  Media CDN (30+ global PoPs)         Cloud CDN (web assets)   │  │   │
│  │  │  HLS/DASH video segments             Thumbnails, metadata     │  │   │
│  │  └────────────────────────┬──────────────────────────────────────┘  │   │
│  │                           │ (cache miss: origin pull)                │   │
│  │  ┌────────────────────────▼──────────────────────────────────────┐  │   │
│  │  │  Cloud Storage (Multi-Region: US, EU, ASIA)                    │  │   │
│  │  │  /video/hls/**, /video/dash/** — transcoded segments           │  │   │
│  │  └───────────────────────────────────────────────────────────────┘  │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│                                                                               │
│  CONTENT INGESTION & TRANSCODING                                             │
│  Studio Upload → GCS (raw) → Pub/Sub → Cloud Run (Orchestrator)             │
│                                               │                               │
│                                    Transcoding API (managed)                 │
│                                               │                               │
│                              GCS (transcoded, multi-region)                  │
│                                                                               │
│  AI/ML LAYER                                                                 │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │  Content Intelligence                 Recommendation Engine          │   │
│  │  ┌──────────────────────┐            ┌───────────────────────────┐  │   │
│  │  │  Cloud Run (Enrichmt)│            │  Vertex AI Feature Store  │  │   │
│  │  │       │              │            │  (user features)          │  │   │
│  │  │  Gemini 1.5 Pro      │            └─────────────┬─────────────┘  │   │
│  │  │  (vision, tagging)   │                          │                 │   │
│  │  │       │              │            ┌─────────────▼─────────────┐  │   │
│  │  │  Vertex AI Search    │            │  Vertex AI Endpoints      │  │   │
│  │  │  (semantic search)   │            │  (rec. model, <200ms P95) │  │   │
│  │  └──────────────────────┘            └───────────────────────────┘  │   │
│  │                                                                       │   │
│  │  Vertex AI Pipelines (daily retraining) → Vertex AI Model Registry   │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│                                                                               │
│  AD TECH PIPELINE                                                            │
│  Ad Events (2M/sec) → Pub/Sub → Dataflow (streaming, 1-min windows)         │
│                                      │                                        │
│                         ┌────────────┴───────────────┐                       │
│                    BigQuery                        Bigtable                   │
│                    (analytics DW,              (fraud detection,              │
│                     <5min latency)              <5ms lookup)                  │
│                         │                                                     │
│                    Looker Studio (dashboards)                                 │
│                                                                               │
│  OBSERVABILITY                                                                │
│  Cloud Monitoring + Cloud Logging + Cloud Trace + Network Intelligence Center │
│                                                                               │
│  SECURITY                                                                     │
│  Cloud Armor │ Cloud KMS (CMEK) │ VPC Service Controls │ IAM │ Audit Logs    │
│                                                                               │
│  EU GDPR: Separate project (Assured Workloads, EU-only regions)              │
└──────────────────────────────────────────────────────────────────────────────┘
```

---

*Last updated: October 2025 | Google PCA Exam Prep*
