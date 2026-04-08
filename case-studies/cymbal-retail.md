# Case Study 3: Cymbal Retail

> **Exam Version:** Google Professional Cloud Architect (PCA) — 2025 (Updated October 2025)
> **Domain:** Omnichannel Retail | **Compliance:** PCI-DSS (payment processing)

---

## 1. Company Overview

**Cymbal Retail** is a large omnichannel retailer operating physical stores, an e-commerce platform, and a mobile app across North America with expansion into Europe and Latin America underway. They sell consumer electronics, home goods, apparel, and sporting goods.

**Scale:**
- 1,200 physical store locations across North America
- E-commerce platform: 5M daily active users (normal), 50M peak (Black Friday/Cyber Monday)
- Mobile app: 12M registered users
- SKU catalog: 2M+ products across all categories
- Order volume: 500K orders/day (normal), 5M orders/day (peak)
- Annual revenue: $18B

**Current challenges:**
- E-commerce platform runs on-premises (Java microservices on VMware) — cannot scale fast enough for peak events; Black Friday 2023 resulted in 40 minutes of downtime
- Real-time inventory visibility is inconsistent: online shows "in stock" when stores are actually sold out
- Loyalty program processes points in nightly batch — customers don't see points for 24 hours
- Legacy ERP (SAP S/4HANA on-premises) is the system of record for inventory and orders
- Personalized recommendations engine is rule-based and static — no ML
- Payment processing is fragmented across 3 vendors with different compliance implementations
- International expansion is blocked by inability to scale infrastructure

**Strategic goals:**
- Move e-commerce platform to GCP — auto-scale to handle 10x traffic spikes without downtime
- Implement real-time inventory synchronization across all channels
- Modernize loyalty program with real-time points and rewards processing
- Deploy ML-powered personalization and demand forecasting
- Consolidate payment processing with unified PCI-DSS compliance posture
- Enable rapid international expansion via GCP's global infrastructure

---

## 2. Technical Requirements

1. **Auto-scaling** to handle 10x normal traffic (up to 50M daily active users) during peak events without manual intervention.
2. **Real-time inventory** updates synchronized across e-commerce, mobile, and in-store systems within 500ms of a transaction.
3. **Loyalty program** — real-time points calculation and redemption; customers see points within seconds of a transaction.
4. **ERP integration** — SAP S/4HANA on-premises must remain the system of record; GCP systems must sync with SAP via APIs.
5. **PCI-DSS compliance** — cardholder data must be isolated, encrypted, and subject to strict access controls.
6. **Personalization** — AI/ML-powered product recommendations based on browsing history, purchase history, and real-time session behavior.
7. **Demand forecasting** — predict inventory needs at SKU/store level 4–8 weeks out to optimize supply chain.
8. **Global availability** — platform must support EU and Latin America with data residency compliance.
9. **99.99% uptime** for checkout and payment flows (highest criticality).
10. **Mobile app backend** — low-latency API responses (<100ms P95) for product search, cart, and checkout.

---

## 3. Business Requirements

1. **Zero Black Friday downtime** — peak event traffic must be absorbed automatically; manual scaling is not acceptable.
2. **Reduce cart abandonment** — checkout latency over 2 seconds increases abandonment significantly; improve checkout P95 latency.
3. **Real-time loyalty engagement** — customers who see points immediately are 2x more likely to return; real-time processing is a business priority.
4. **Unified commerce** — inventory and order data must be consistent across all channels (no more "in stock online, out in store" issues).
5. **PCI-DSS compliance** — avoid PCI scope expansion; minimize which systems touch cardholder data.
6. **International expansion** — support EU (GDPR), Latin America (LGPD in Brazil) within 12 months.
7. **AI-driven revenue** — personalization and demand forecasting expected to increase revenue by 8–12% and reduce inventory overstock by 15%.
8. **Cost predictability** — finance requires forecasting of GCP costs; avoid surprise bills during peak events.

---

## 4. Existing Technical Infrastructure

### E-Commerce Platform (On-Premises)

| System | Technology | Notes |
|--------|-----------|-------|
| Web Frontend | React SPA | Served via on-prem nginx |
| API Gateway | Kong + Java microservices | 15 microservices (catalog, cart, checkout, etc.) |
| Product Catalog DB | PostgreSQL 13 | 2M+ SKUs, 500GB |
| Order Management DB | MySQL 8.0 | 10M+ orders, 2TB |
| Session Store | Redis (on-prem cluster) | Cart and session data |
| Search | Elasticsearch on-prem | Product search |

### Loyalty Program

| System | Technology | Notes |
|--------|-----------|-------|
| Loyalty DB | Oracle 12c | Customer accounts, points balances |
| Points Engine | Java batch job (nightly) | Calculates points, runs at 2 AM |
| Rewards Portal | PHP app | Customer-facing rewards catalog |

### In-Store Systems

| System | Technology | Notes |
|--------|-----------|-------|
| POS System | Custom Java app | 5 POS terminals per store × 1,200 stores = 6,000 terminals |
| Store Inventory | Oracle DB (per store, replicated to central Oracle) | Updated at store close (nightly sync) |
| Store Wi-Fi | Cisco Meraki | Managed centrally |

### ERP & Backend

| System | Technology | Notes |
|--------|-----------|-------|
| ERP | SAP S/4HANA (on-premises) | Inventory master, procurement, financials |
| Data Warehouse | On-prem Oracle DW | Reporting, sales analytics |
| ETL | Informatica (on-prem) | Nightly batch ETL from all systems to DW |

---

## 5. Recommended GCP Architecture

### Project Structure

```
Organization: cymbal-retail.com
  └── Folder: Production
  │     ├── Project: cymbal-ecommerce      (GKE, Cloud Run, APIs)
  │     ├── Project: cymbal-pci            (Payment processing — isolated PCI scope)
  │     ├── Project: cymbal-data           (BigQuery, Dataflow, Pub/Sub)
  │     ├── Project: cymbal-inventory      (Spanner, Pub/Sub — real-time inventory)
  │     └── Project: cymbal-security       (KMS, Secret Manager, SCC)
  └── Folder: Non-Production
        ├── Project: cymbal-dev
        └── Project: cymbal-staging
```

### E-Commerce Platform

| Component | GCP Service | Rationale |
|-----------|------------|-----------|
| API Gateway | **Apigee API Management** | Managed gateway with rate limiting, auth, analytics |
| Microservices | **GKE Standard** (us-central1, multi-zonal) | Stateful services needing fine-grained control |
| Stateless APIs | **Cloud Run** (autoscaling to 0 or 1000s) | Product catalog, recommendations API — scales fast |
| Static frontend | **Cloud CDN** + **Cloud Storage** | React SPA served from GCS via CDN |
| Product catalog DB | **Cloud Spanner** (multi-region) | Horizontally scalable, consistent, global |
| Order management | **Cloud Spanner** (multi-region) | Strong consistency needed for orders |
| Session store | **Memorystore for Redis** | Managed Redis, low-latency cart/session |
| Product search | **Vertex AI Search for Retail** | Managed search with ML ranking |

**Why Cloud Spanner for Product Catalog and Orders?**
- Handles 10x traffic spikes without sharding or manual scaling
- Multi-region configuration ensures 99.999% availability
- ACID transactions across distributed data — critical for inventory and orders
- No connection pool management needed at scale

### Real-Time Inventory System

```
Transaction Events (POS, e-commerce, mobile)
        │
        ▼
  Pub/Sub (inventory-events topic)
        │
        ├─→ Dataflow Streaming Pipeline
        │     └── Aggregates inventory changes per SKU/location
        │     └── Writes updates to Cloud Spanner (inventory table)
        │           └── Spanner CDC → Pub/Sub → E-commerce platform cache invalidation
        │
        └─→ BigQuery (inventory history — analytics, demand forecasting)

SAP S/4HANA (on-prem)
        │
        ├─→ Cloud Interconnect
        ├─→ SAP Integration Suite or Apigee → Pub/Sub (ERP events: PO, receipt, adjustment)
        └─→ Dataflow → Cloud Spanner (inventory master sync, near-real-time)
```

- **Cloud Spanner**: single source of truth for real-time inventory counts per SKU per location
- **Pub/Sub**: all inventory change events flow through a single topic — guaranteed delivery
- **Dataflow**: stateful streaming aggregation (handles out-of-order events, exactly-once semantics with Cloud Spanner writes)
- **Cloud Interconnect**: SAP on-prem → GCP connectivity for ERP integration
- Inventory update latency: <500ms from transaction to updated read

### Loyalty Program — Real-Time Architecture

```
Customer Transaction (POS, e-commerce, mobile)
        │
        ▼
  Pub/Sub (loyalty-events topic)
        │
        ▼
  Cloud Run (Loyalty Engine — event-driven)
  - Calculates points earned (complex rules engine)
  - Validates and applies redemptions
  - Updates customer balance
        │
        ▼
  Firestore (loyalty-accounts collection)
  - Customer loyalty balance, tier, history
  - Low-latency reads for app/portal
        │
        ▼
  Mobile Push / Email (via Firebase Cloud Messaging + Pub/Sub)
  "You just earned 150 points! Your new balance: 3,420 points"
```

- Real-time points: customer sees balance update within seconds (not 24 hours)
- **Firestore** chosen for per-customer document store — low-latency reads (<10ms) for mobile app
- **Cloud Run** Loyalty Engine scales automatically; no infrastructure to manage
- **Pub/Sub** ensures no transaction events are lost even during load spikes

### Payment Processing (PCI-DSS)

```
┌────────────────────────────────────────────────────────┐
│  PCI Scope Boundary (cymbal-pci project)               │
│                                                         │
│  Checkout API (Cloud Run, private VPC)                 │
│       │                                                 │
│       ├─→ Payment Token Service                        │
│       │     └── Tokenizes card data via Vault/HSM      │
│       │                                                 │
│       └─→ Payment Processor API (external PCI vendor)  │
│             └── Stripe / Adyen / Braintree             │
│                                                         │
│  Cloud KMS (HSM-backed) — encrypts cardholder tokens   │
│  VPC Service Controls — perimeter around PCI project   │
│  Cloud Audit Logs — all access to cardholder data      │
│  No storage of raw PANs — tokenization only            │
└────────────────────────────────────────────────────────┘
```

**PCI-DSS scope minimization:**
- Raw card numbers never stored in Cymbal systems — tokenization at point of capture
- All tokenized data encrypted with CMEK (Cloud KMS HSM keys)
- Dedicated GCP project (`cymbal-pci`) isolates PCI-scope systems from the rest of the platform
- Network: no direct peering from PCI project to e-commerce project; communication via API Gateway only

### AI/ML — Personalization & Demand Forecasting

#### Personalization

```
User Behavior Events (clicks, views, add-to-cart, purchases)
        │
  Pub/Sub → Dataflow → BigQuery (behavioral data lake)
                              │
                    Vertex AI Feature Store
                    (user features: category affinity, brand preferences)
                              │
                    Vertex AI Endpoints
                    (real-time recommendation model)
                              │
                    Vertex AI Search for Retail
                    (product search with personalized ranking)
```

- **Vertex AI Search for Retail**: managed search product purpose-built for retail; uses user event data to personalize search results and recommendations; integrates directly with product catalog
- **Vertex AI Feature Store**: serves real-time user features for the recommendation model
- Output: product recommendations on homepage, PDP, cart, and email

#### Demand Forecasting

- **BigQuery ML** with **ARIMA_PLUS** time-series model — trained on 3 years of sales history per SKU/location
- Forecasts inventory needs 4–8 weeks out for supply chain planning
- Output: BigQuery table → Looker dashboards for merchandising team
- Alternative: **Vertex AI Forecast** for more complex multi-variate models

### Black Friday Scaling

| Component | Normal Capacity | Peak Strategy |
|-----------|----------------|---------------|
| Cloud Run (Catalog, Recs) | ~100 instances | Auto-scales to 5,000+ instances in minutes |
| GKE (Core microservices) | 20 nodes | Cluster Autoscaler + Vertical Pod Autoscaler → 200 nodes; pre-provisioned node pools |
| Cloud Spanner | 10 processing units | Scales automatically (no action needed) |
| Memorystore (Redis) | Replicated instance | Reads from read replicas; pre-scaled 1 week before Black Friday |
| Cloud Load Balancing | Unlimited | Anycast global; no capacity planning needed |

**Black Friday Pre-Event Runbook:**
1. Scale Memorystore read replicas 1 week before
2. Pre-warm Cloud Run minimum instances to 500 (avoid cold-start spikes at traffic burst)
3. Review Cloud Spanner utilization projections; increase processing units if needed
4. Enable Cloud Armor rate limiting to prevent bot abuse during flash sales
5. Notify GCP account team for quota increases if estimated peak exceeds current quotas

### Data Platform

| Layer | Service | Purpose |
|-------|---------|---------|
| Event streaming | **Pub/Sub** | All transactional events (orders, inventory, loyalty) |
| Stream processing | **Dataflow** | Real-time ETL, inventory aggregation |
| Analytics DW | **BigQuery** | Sales analytics, forecasting, customer 360 |
| Operational DB | **Cloud Spanner** | Product catalog, inventory, orders (operational) |
| Customer data | **Firestore** | Loyalty, user profiles, preferences |
| Cache | **Memorystore (Redis)** | Sessions, cart, product cache |
| ERP integration | **Cloud Pub/Sub + Dataflow** | SAP S/4HANA change data capture → GCP |

---

## 6. Migration Strategy

### Phase 1 — Foundation & Connectivity (Months 1–3)

- Establish GCP organization, project structure, IAM, and VPC peering
- Deploy Cloud Interconnect from on-prem data center to GCP (for ERP integration and migration traffic)
- Set up Cloud KMS, Secret Manager, and VPC Service Controls
- Migrate product catalog and sessions to GCP: Cloud Spanner + Memorystore
- Do NOT migrate payment processing yet — establish PCI project structure first

### Phase 2 — E-Commerce Platform to GCP (Months 3–7)

- Containerize Java microservices; deploy to GKE (strangler fig)
- Deploy Cloud Run for stateless catalog and recommendation APIs
- Run parallel: GCP platform handles read traffic; on-prem handles writes → validate correctness
- Set up Dataflow pipeline for real-time inventory sync from SAP and POS systems
- Migrate product search to Vertex AI Search for Retail
- Conduct load test simulating 10x traffic before first Black Friday on GCP

### Phase 3 — Loyalty & PCI Migration (Months 6–10)

- Deploy real-time loyalty system (Cloud Run + Firestore + Pub/Sub)
- Parallel run: new loyalty system alongside nightly batch for 60 days; validate consistency
- Migrate payment processing to PCI-isolated project; update checkout service to tokenize cards
- Achieve PCI-DSS QSA assessment on new architecture before decommissioning old payment system

### Phase 4 — Analytics, ML & International (Months 10–18)

- Migrate Oracle DW to BigQuery; replace Informatica ETL with Dataflow
- Deploy Vertex AI personalization and demand forecasting models
- Set up EU GCP project (Assured Workloads) for international expansion
- Decommission on-prem e-commerce infrastructure
- Set up global Cloud Load Balancing with EU and LATAM backends

---

## 7. Security & Compliance Considerations

### PCI-DSS

| PCI-DSS Requirement | GCP Implementation |
|--------------------|--------------------|
| Req 1: Firewall protection | VPC firewall rules; no internet routes in PCI VPC except to payment processor |
| Req 2: No vendor defaults | Hardened GKE node images; OS Config Agent for patch compliance |
| Req 3: Protect stored data | Tokenization (no raw PAN storage); CMEK with Cloud KMS HSM for tokens |
| Req 4: Encrypt in transit | TLS 1.2+ enforced; mTLS between services via service mesh |
| Req 7: Restrict access | IAM with least privilege; VPC Service Controls perimeter on `cymbal-pci` |
| Req 8: Identify and authenticate | Workforce Identity Federation + MFA; service accounts with short-lived tokens |
| Req 10: Track and monitor | Cloud Audit Logs (Data Access enabled); export to locked GCS bucket |
| Req 11: Regular security testing | Security Command Center Premium; Cloud Vulnerability Scanning |
| Req 12: Information security policy | Google's PCI-DSS responsibility matrix + BAA |

**PCI Scope Reduction Strategy:**
- Tokenize card data at the browser/app layer using payment processor's JavaScript SDK — raw card data never hits Cymbal's servers
- Only the `cymbal-pci` project is in PCI scope — all other projects are out of scope
- Communication from e-commerce project to PCI project via Apigee API (no direct network peering)

### GDPR/LGPD (International Expansion)

- EU project (Assured Workloads) for EU customer data
- Brazil project (Assured Workloads) for LGPD compliance
- Data erasure pipeline: delete Firestore records, BigQuery user events (TTL), Cloud Storage assets
- Consent management stored in Firestore; enforced in Dataflow pipelines

---

## 8. Key Design Decisions

### Decision 1: Cloud Spanner for Inventory and Orders

**Chose Cloud Spanner** (multi-region) over Cloud SQL.
- *Rationale*: Inventory and order data require strong consistency across thousands of concurrent transactions (especially during Black Friday). Cloud Spanner handles this without sharding or read replicas. The multi-region configuration provides 99.999% availability. Cloud SQL cannot scale horizontally without complex sharding, and managing connection pools at 50M DAU scale is operationally risky.
- *Trade-off*: Cloud Spanner costs more than Cloud SQL. Justified by the criticality of the workload and the avoidance of Black Friday downtime (40 minutes of downtime = ~$5M+ in lost revenue at $18B annual).

### Decision 2: Vertex AI Search for Retail vs. Self-Managed Elasticsearch

**Chose Vertex AI Search for Retail**.
- *Rationale*: Purpose-built for retail search — handles product catalog indexing, semantic search, personalized ranking, and "did you mean?" corrections out of the box. Processes user event data to improve ranking automatically. Eliminates the need to manage Elasticsearch clusters at scale.
- *Trade-off*: Less customization than a custom Elasticsearch deployment. For a retail search use case, the built-in features cover 95%+ of requirements.

### Decision 3: Pub/Sub for All Event Streaming

**Chose a single Pub/Sub backbone** for inventory, loyalty, and order events.
- *Rationale*: Pub/Sub provides guaranteed at-least-once delivery, fan-out to multiple subscribers, and autoscaling. Using a single messaging backbone (with separate topics) simplifies the architecture and reduces operational overhead compared to running Kafka on-prem or on GKE.
- *Trade-off*: Pub/Sub doesn't support consumer groups (like Kafka) — ordering is guaranteed within a message ordering key, but not globally. For order/inventory events, ordering within a customer or SKU key is sufficient.

### Decision 4: Firestore for Loyalty Customer Data

**Chose Firestore** over Cloud SQL for loyalty accounts.
- *Rationale*: Loyalty account lookups happen on every transaction and mobile app open — requiring <10ms latency at scale. Firestore's per-document reads are faster than Cloud SQL for simple lookups, scale horizontally, and require no connection pooling. The loyalty data model (per-customer document) maps naturally to Firestore's document model.
- *Trade-off*: Complex analytical queries on loyalty data are harder in Firestore. Solved by exporting loyalty events to BigQuery for analytics while keeping Firestore as the operational store.

---

## 9. Sample Exam Questions

---

**Question 1**

Cymbal Retail's e-commerce platform experienced 40 minutes of downtime during Black Friday due to inability to scale their on-premises infrastructure. After migrating to GCP, they need to handle 10x normal traffic automatically. Which services provide automatic scaling for stateful microservices and stateless APIs respectively?

- A) Cloud Run (stateful), GKE Autopilot (stateless)
- B) GKE Standard with Cluster Autoscaler (stateful microservices), Cloud Run (stateless APIs)
- C) Compute Engine Managed Instance Groups (stateful), App Engine (stateless)
- D) GKE Autopilot (both stateful and stateless)

**Answer: B**

*Explanation:* **GKE Standard with Cluster Autoscaler** automatically provisions new nodes when pods are pending, scaling to handle 10x traffic. It's appropriate for stateful microservices that need precise resource control, persistent storage, or specific node configurations. **Cloud Run** is ideal for stateless APIs — it scales to thousands of instances in seconds, including from zero, with no node management. GKE Autopilot (A, D) is suitable but provides less control for stateful workloads with complex scheduling requirements. Managed Instance Groups (C) are for VMs, not containers — higher operational overhead. App Engine has more limited scaling than Cloud Run for microservices.

---

**Question 2**

Cymbal Retail needs real-time inventory synchronization across e-commerce, mobile, and 1,200 physical stores. A transaction at any channel must update the inventory count for all channels within 500ms. Which architecture achieves this?

- A) Nightly batch ETL from all channels to Cloud SQL
- B) POS and e-commerce systems publish transaction events to Pub/Sub → Dataflow streaming → Cloud Spanner (inventory); Cloud Spanner change streams → Pub/Sub → channel cache invalidation
- C) Direct database writes from all channels to a shared Cloud SQL instance
- D) BigQuery streaming inserts from all channels → materialized views for inventory counts

**Answer: B**

*Explanation:* Pub/Sub receives transaction events from all channels (POS, e-commerce, mobile) with guaranteed delivery. Dataflow's streaming pipeline aggregates inventory changes (handling out-of-order events with watermarks) and writes to Cloud Spanner as the authoritative inventory store. Cloud Spanner's change streams publish changes back to Pub/Sub, triggering cache invalidation in Redis so the e-commerce frontend sees fresh data within 500ms. Option A is batch — 24-hour latency. Option C creates a bottleneck — a single Cloud SQL instance cannot handle 6,000 POS terminals + e-commerce writes simultaneously with sub-500ms consistency. BigQuery (D) is analytical, not operational — query latency is seconds-to-minutes.

---

**Question 3**

Cymbal Retail needs to ensure their payment processing environment meets PCI-DSS requirements. Which approach best minimizes PCI scope?

- A) Store encrypted credit card numbers in Cloud SQL within the main e-commerce project
- B) Use a separate GCP project (`cymbal-pci`) with VPC Service Controls; tokenize card data at the browser using the payment processor's SDK; store only tokens in Cymbal systems
- C) Store card data in a Cloud Storage bucket with CMEK encryption in the main project
- D) Use Cloud HSM to encrypt all card data stored in BigQuery

**Answer: B**

*Explanation:* PCI scope minimization requires that raw cardholder data (PAN) never touches Cymbal systems. By using the payment processor's JavaScript SDK (Stripe.js, Braintree's Drop-in UI, etc.), card data is sent directly to the payment processor — Cymbal's servers only receive a payment token. The `cymbal-pci` project with VPC Service Controls creates a network and API perimeter around the minimal in-scope systems. Options A, C, and D all store card data (even encrypted) in Cymbal's systems — this puts those systems in PCI scope, dramatically expanding the compliance burden and risk.

---

**Question 4**

Cymbal Retail's loyalty program currently processes points in nightly batches, causing 24-hour delays. They want customers to see loyalty points within seconds of a transaction. Which architecture should they implement?

- A) Increase batch frequency to every 30 minutes using Cloud Scheduler + Cloud SQL
- B) Transaction events → Pub/Sub → Cloud Run (points calculation engine) → Firestore (customer loyalty balance) → Firebase Cloud Messaging (push notification)
- C) Direct database update from POS system to Oracle loyalty DB via Cloud Interconnect
- D) Batch job on GKE running every 5 minutes reading from BigQuery

**Answer: B**

*Explanation:* The event-driven architecture using Pub/Sub + Cloud Run achieves real-time processing. When a transaction occurs, an event is published to Pub/Sub. Cloud Run (autoscaling, no cold-start issues with min instances) processes the event, applies the rules engine to calculate points, and updates the customer's Firestore document. Firestore's real-time listeners allow the mobile app to instantly reflect the new balance. Firebase Cloud Messaging sends the push notification. The total end-to-end latency is under 5 seconds. Increasing batch frequency (A) reduces latency but can't achieve "seconds" — it still has a polling delay. Direct Oracle writes (C) don't modernize the architecture. Polling BigQuery (D) is a batch approach, not event-driven.

---

**Question 5**

Cymbal Retail's e-commerce platform needs 99.99% availability for the checkout service. Their orders and inventory data are stored in Cloud Spanner. Which Cloud Spanner configuration best supports this availability target?

- A) Single-region Cloud Spanner with automated backups
- B) Dual-region Cloud Spanner configuration
- C) Multi-region Cloud Spanner configuration (e.g., `nam6`)
- D) Regional Cloud Spanner with a read replica in a second region

**Answer: C — Multi-region Cloud Spanner**

*Explanation:* Cloud Spanner's **multi-region configurations** (e.g., `nam6` covering multiple US regions) provide 99.999% availability SLA by replicating data across multiple regions with automatic failover. A single region (A) has 99.99% SLA but is vulnerable to region-wide outages. Dual-region (B) improves on single-region but doesn't match the 99.999% SLA of multi-region. Cloud Spanner doesn't have "read replicas" in the traditional sense (D) — all nodes participate in consensus. For a checkout service at $18B annual revenue, multi-region is justified.

---

**Question 6**

Cymbal Retail wants to implement demand forecasting to predict inventory needs at the SKU/store level 4–8 weeks out. They have 3 years of historical sales data in BigQuery. What is the most straightforward approach using GCP-native services?

- A) Export BigQuery data to Cloud Storage → train a custom TensorFlow model on Vertex AI Training
- B) Use BigQuery ML with the ARIMA_PLUS model for time-series forecasting directly in BigQuery
- C) Use Dataflow to compute moving averages and export to Looker
- D) Deploy a custom forecasting model on Cloud Run that reads from BigQuery

**Answer: B — BigQuery ML with ARIMA_PLUS**

*Explanation:* BigQuery ML's `ARIMA_PLUS` model is purpose-built for time-series forecasting and runs entirely within BigQuery — no data movement, no infrastructure to manage. It automatically handles seasonality, holiday effects, and trend detection, which are critical for retail demand forecasting. The model can be trained with a simple SQL `CREATE MODEL` statement and produces forecasts per SKU/store. Custom TensorFlow (A) provides more flexibility but requires significantly more engineering effort. Moving averages (C) are not a forecasting model — they don't account for seasonality. Custom Cloud Run model (D) is unnecessarily complex when BigQuery ML solves the problem natively.

---

**Question 7**

During Black Friday, Cymbal Retail expects 50M daily active users (10x normal). Their product catalog API is deployed on Cloud Run. What should they configure to ensure the API handles the traffic spike without latency spikes from cold starts?

- A) Set Cloud Run maximum instances to 10,000
- B) Configure Cloud Run minimum instances to 500 and enable request concurrency of 80
- C) Deploy the API to GKE Standard with HPA configured to scale on CPU
- D) Use Cloud Run with no configuration changes — it handles scaling automatically

**Answer: B**

*Explanation:* Setting **minimum instances** (500) ensures that a base capacity of warm instances is always ready — preventing cold-start latency spikes when the sudden traffic burst hits at the moment Black Friday sales open. **Request concurrency of 80** allows each instance to handle 80 simultaneous requests, balancing throughput and latency. Maximum instances (A) alone doesn't prevent cold starts — new instances still incur startup latency during the burst. GKE with HPA (C) scales more slowly than Cloud Run (minutes vs. seconds) and requires node provisioning. Relying entirely on automatic scaling (D) works for gradual traffic growth but not an instantaneous 10x burst.

---

**Question 8**

Cymbal Retail's SAP S/4HANA ERP system on-premises is the system of record for inventory master data. GCP systems need near-real-time access to inventory updates from SAP. What is the recommended integration pattern?

- A) Nightly data export from SAP to Cloud Storage → BigQuery batch load
- B) SAP Integration Suite (or similar middleware) → Pub/Sub (change events) → Dataflow → Cloud Spanner (inventory mirror); Cloud Spanner is read-only for GCP services
- C) Direct JDBC connection from GKE services to on-premises SAP HANA DB via Cloud Interconnect
- D) Replicate entire SAP database to Cloud SQL using Database Migration Service

**Answer: B**

*Explanation:* SAP Integration Suite (SAP's iFlows or a Kafka SAP connector) captures change events from SAP and publishes them to Pub/Sub. Dataflow processes these events and maintains a near-real-time mirror of inventory in Cloud Spanner. GCP e-commerce services read from Cloud Spanner (not SAP directly), keeping SAP as the system of record while providing cloud-native performance. This decouples GCP services from SAP availability and network latency. Nightly batch (A) doesn't meet near-real-time requirements. Direct JDBC (C) creates tight coupling and a potential bottleneck on Cloud Interconnect; SAP HANA is not designed for thousands of concurrent JDBC connections from cloud microservices. DMS (D) replicates the entire database — SAP HANA has a proprietary schema not compatible with Cloud SQL.

---

## 10. Architecture Diagram (Text)

```
┌────────────────────────────────────────────────────────────────────────────────┐
│                        Cymbal Retail — GCP Architecture                        │
├────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  CHANNELS                                                                       │
│  ┌──────────┐    ┌──────────┐    ┌──────────────────────────────┐             │
│  │  Web/SPA │    │ Mobile   │    │  1,200 Physical Stores (POS) │             │
│  │  (CDN)   │    │  App     │    │  (Cloud Interconnect → GCP)  │             │
│  └────┬─────┘    └────┬─────┘    └──────────────┬───────────────┘             │
│       │               │                          │                              │
│  ┌────▼───────────────▼──────────────────────────▼───────────────────────┐    │
│  │  Cloud Armor (WAF) → Cloud Load Balancing (Global Anycast)            │    │
│  │                              │                                         │    │
│  │           Apigee API Management (Gateway, rate limiting, auth)         │    │
│  └──────────────────────────────┬──────────────────────────────────────  ┘    │
│                                  │                                              │
│  ┌───────────────────────────────▼────────────────────────────────────────┐   │
│  │  APPLICATION LAYER (cymbal-ecommerce project)                          │   │
│  │                                                                         │   │
│  │  ┌─────────────────────┐  ┌─────────────────────┐  ┌────────────────┐ │   │
│  │  │  GKE Standard       │  │  Cloud Run           │  │  Loyalty Svc   │ │   │
│  │  │  (Order, Cart,      │  │  (Catalog, Search,   │  │  Cloud Run     │ │   │
│  │  │   User microsvcs)   │  │   Recommendations)   │  │  (Event-driven)│ │   │
│  │  └──────────┬──────────┘  └──────────┬───────────┘  └───────┬────────┘ │   │
│  └─────────────┼───────────────────────  ┼──────────────────────┼──────────┘   │
│                │                          │                       │              │
│  ┌─────────────▼──────────────────────────▼───────────────────────▼──────────┐ │
│  │  DATA LAYER                                                                │ │
│  │  ┌──────────────────┐  ┌───────────────────┐  ┌────────────────────────┐ │ │
│  │  │  Cloud Spanner   │  │  Memorystore Redis │  │  Firestore             │ │ │
│  │  │  (Inventory,     │  │  (Sessions, Cart,  │  │  (Loyalty accounts,    │ │ │
│  │  │   Orders, SKUs)  │  │   Product cache)   │  │   User preferences)    │ │ │
│  │  └──────────────────┘  └───────────────────┘  └────────────────────────┘ │ │
│  └────────────────────────────────────────────────────────────────────────────┘ │
│                                                                                 │
│  EVENTS & REAL-TIME                                                             │
│  Pub/Sub (inventory, loyalty, order events)                                     │
│    → Dataflow (streaming ETL, inventory aggregation)                           │
│    → BigQuery (analytics, demand forecasting via BigQuery ML ARIMA_PLUS)       │
│                                                                                 │
│  ┌──────────────────────────────────────────────────────────────────────────┐  │
│  │  PCI SCOPE (cymbal-pci project — isolated)                               │  │
│  │  Cloud Run (Checkout/Tokenization) → Payment Processor API               │  │
│  │  Cloud KMS (HSM) │ VPC Service Controls │ Cloud Audit Logs               │  │
│  └──────────────────────────────────────────────────────────────────────────┘  │
│                                                                                 │
│  AI/ML                                                                          │
│  Vertex AI Search for Retail (product search + personalization)                 │
│  Vertex AI Endpoints (recommendation model) ← Vertex AI Feature Store          │
│  Vertex AI Pipelines (model retraining)                                         │
│                                                                                 │
│  ERP INTEGRATION                                                                │
│  SAP S/4HANA (on-prem) → Cloud Interconnect → Pub/Sub → Dataflow → Spanner    │
│                                                                                 │
│  OBSERVABILITY: Cloud Monitoring │ Cloud Logging │ Cloud Trace │ SCC Premium   │
└────────────────────────────────────────────────────────────────────────────────┘
```

---

*Last updated: October 2025 | Google PCA Exam Prep*
