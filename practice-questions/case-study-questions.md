# Case Study Practice Questions
> 15 Questions Across 4 Official Case Studies — PCA Difficulty

**Reference:** Official case studies at https://cloud.google.com/certification/guides/professional-cloud-architect#case-studies

---

## Case Study 1: EHR Healthcare

**Context:** EHR Healthcare is a leading provider of electronic health record software. They operate on-premises data centers and want to migrate to GCP. Key requirements: HIPAA compliance, support for HL7 FHIR APIs, multi-region for business continuity, existing SQL Server and MongoDB workloads, real-time analytics on patient data, strict PII data handling, and a goal to reduce on-premises infrastructure.

---

**Q1: EHR Healthcare stores patient records in SQL Server on-premises. They want to migrate to a managed GCP database service with minimal application changes. The records contain PHI, require strong consistency, and the application team is only familiar with SQL. Which migration path is MOST appropriate?**

- A) Migrate to Bigtable — it handles large datasets at scale
- B) Migrate to Cloud SQL for SQL Server using Database Migration Service; ensure the instance has no public IP; use VPC Service Controls to protect the project
- C) Migrate to Cloud Spanner — it provides global SQL with strong consistency
- D) Migrate to BigQuery — it supports SQL queries on large healthcare datasets

**Answer: B**

**Explanation:** Cloud SQL for SQL Server provides a managed service with near-100% compatibility with SQL Server — minimal application changes needed. Database Migration Service handles the online migration with CDC for minimal downtime. No public IP + VPC Service Controls addresses HIPAA data protection requirements. Cloud Spanner (C) would require significant application re-architecture (different SQL dialect, no SQL Server compatibility). Bigtable (A) is NoSQL — doesn't support SQL Server workloads. BigQuery (D) is an analytics warehouse, not an OLTP database for transactional record management.

---

**Q2: EHR Healthcare needs to expose patient health records to third-party healthcare apps using the HL7 FHIR standard. The API must be secure, scalable, and comply with HIPAA. Which GCP architecture supports this?**

- A) Build a custom REST API on GCE VMs; use Cloud Armor to protect it
- B) Use the Google Cloud Healthcare API (FHIR store); expose it via Apigee API Gateway with OAuth 2.0 / SMART on FHIR authentication; VPC Service Controls on the Healthcare API project
- C) Store FHIR data in Firestore; expose via Cloud Functions with API key authentication
- D) Use BigQuery to store FHIR-formatted data; expose directly via BigQuery API to third parties

**Answer: B**

**Explanation:** The Google Cloud Healthcare API has a native FHIR R4 store — no custom implementation needed. Apigee provides enterprise API management with OAuth 2.0 and SMART on FHIR (the standard authentication protocol for healthcare apps). VPC Service Controls prevents data exfiltration. This is purpose-built for healthcare interoperability. A custom REST API (A) requires significant development and doesn't provide native FHIR semantics. Firestore (C) lacks FHIR-specific query capabilities and doesn't offer SMART on FHIR. BigQuery (D) is analytical — not designed for transactional FHIR API serving.

---

**Q3: EHR Healthcare wants to enable real-time analytics on patient data for clinical decision support. Clinicians need dashboards showing patient trends within seconds of new vitals being recorded. The analytics platform must be HIPAA-eligible. What architecture enables this?**

- A) Export patient data nightly to Cloud Storage; use Dataproc to process and load to BigQuery; connect Looker for dashboards
- B) Stream vitals data via Pub/Sub → Dataflow (real-time transform + deidentification via DLP API) → BigQuery streaming inserts; connect Looker Studio or Looker to BigQuery for real-time dashboards; ensure all services are HIPAA-eligible and covered by BAA
- C) Store vitals in Bigtable; run BigQuery analytics queries using a Bigtable external table
- D) Write vitals to Cloud SQL; run Looker directly against Cloud SQL for dashboards

**Answer: B**

**Explanation:** Pub/Sub ingests real-time vitals as events. Dataflow processes and transforms data within seconds, and the DLP API can deidentify PHI for analytics use (if needed for broader access). BigQuery streaming inserts make data available for queries within seconds of ingestion. Looker connects to BigQuery for real-time dashboard refreshes. All services (Pub/Sub, Dataflow, BigQuery) are HIPAA-eligible. Nightly batch (A) doesn't meet "within seconds" requirements. Bigtable + BigQuery external tables (C) adds complexity without enabling real-time streaming. Cloud SQL (D) is not designed for analytics workloads at clinical scale.

---

**Q4: EHR Healthcare's existing on-premises identity provider (Active Directory) must be integrated with GCP. Employees should authenticate with their existing AD credentials to access GCP services. What is the recommended approach?**

- A) Create individual Google accounts for each employee manually
- B) Use Google Cloud Directory Sync (GCDS) to sync AD users to Cloud Identity; configure SAML 2.0 SSO between AD FS (or Azure AD) and Google Workspace / Cloud Identity; employees log in with AD credentials that federate to GCP
- C) Use a shared service account for all employee access to simplify credential management
- D) Create a Workforce Identity Federation pool connecting AD as the OIDC/SAML provider; no Google accounts required

**Answer: B or D (both are valid approaches)**

**Explanation:** Both B and D achieve the goal. GCDS + SAML SSO (B) is the classic approach: sync user/group identities to Cloud Identity, then use SSO so employees authenticate via AD. Workforce Identity Federation (D) is the newer approach that doesn't require provisioning Cloud Identity accounts — tokens are issued directly from the OIDC/SAML IdP. For the exam, B is the more traditionally expected answer. Shared service accounts (C) violate least privilege and auditability requirements. Manual account creation (A) doesn't scale and isn't integrated with AD lifecycle.

---

## Case Study 2: Altostrat Media

**Context:** Altostrat is an online media company that delivers video content globally. They have existing on-premises infrastructure and want to migrate to GCP. Key requirements: low-latency global video delivery to millions of users, support for live streaming and on-demand video, processing of raw video uploads into multiple formats/resolutions, real-time analytics on viewer behavior, and cost optimization for storage of large video archives.

---

**Q5: Altostrat receives raw video uploads from content creators (files ranging from 1 GB to 50 GB). These must be transcoded into 8 different resolutions/formats within 2 hours of upload. The transcoding workload is highly variable — sometimes 100 uploads per hour, sometimes 5. What architecture is MOST cost-effective?**

- A) Maintain a dedicated cluster of 50 high-CPU GCE VMs running 24/7 for transcoding
- B) GCS upload triggers Pub/Sub notification → Cloud Run Job (or Cloud Tasks) queues transcoding jobs → MIG of high-CPU GCE Spot VMs scales from 0 to N based on queue depth; transcoding VMs pull jobs from the queue
- C) Use Cloud Run (HTTP service) for transcoding — scale to zero when no uploads
- D) Use Dataproc with Spark for video transcoding; auto-scale the cluster based on job queue

**Answer: B**

**Explanation:** A queue-based architecture with Spot VMs provides optimal cost efficiency. GCS events trigger job queuing via Pub/Sub. A MIG autoscales from 0 (no cost at idle) to N instances based on queue depth. Spot VMs provide 60–91% cost savings for fault-tolerant batch transcoding (if a Spot VM is preempted, the job is retried). Cloud Run (C) has a 60-minute request timeout — insufficient for transcoding a 50 GB file in one request, and transcoding is CPU-intensive for long durations. Dedicated VMs (A) waste money during low periods. Dataproc (D) is designed for distributed data processing, not video transcoding.

---

**Q6: Altostrat wants to deliver video content with the lowest possible latency to users in North America, Europe, and Asia. Videos are stored in Cloud Storage in the US. What architecture minimizes delivery latency globally?**

- A) Use a single Cloud Storage bucket in us-central1; users download directly from GCS
- B) Use Cloud CDN with a Global External Application Load Balancer; videos are cached at Google's edge PoPs closest to users; first access populates the cache from the origin GCS bucket; subsequent views are served from the edge
- C) Replicate the GCS bucket to 3 regions and use GeoDNS to route users to the nearest region
- D) Use Dedicated Interconnect in each region to provide low-latency access to the GCS bucket

**Answer: B**

**Explanation:** Cloud CDN caches video content at Google's 100+ edge PoPs worldwide — users receive content from a PoP within milliseconds of their location rather than making a transatlantic round trip. The Global LB provides a single anycast IP that routes to the nearest healthy backend. CDN cache invalidation is available when content changes. Bucket replication with GeoDNS (C) requires you to manage replication, DNS health checks, and failover — Cloud CDN handles all of this automatically. Direct GCS serving (A) serves from the origin region for all global users. Dedicated Interconnect (D) is for private on-premises connectivity, not CDN-style global distribution.

---

**Q7: Altostrat has 5 PB of video archives. Most videos are accessed within 30 days of upload, then rarely accessed except for the top 1,000 most popular videos (which are accessed daily). They want to minimize storage costs without sacrificing performance for popular content. What storage strategy should they use?**

- A) Store all 5 PB in Cloud Storage Standard — accept the higher cost for instant access
- B) Store all 5 PB in Cloud Storage Coldline — accept higher retrieval latency for cost savings
- C) Store new and popular videos in Cloud Storage Standard; use lifecycle rules to move non-popular content to Nearline after 30 days and to Coldline after 1 year; use Cloud CDN caching to serve popular content without direct GCS egress
- D) Enable Autoclass on all buckets — it automatically optimizes between Standard and Nearline

**Answer: C**

**Explanation:** A tiered lifecycle strategy matches storage cost to access pattern: Standard for active content (first 30 days), Nearline for monthly-accessed archives (30+ days), Coldline for rarely accessed archives (1+ years). Cloud CDN caching ensures the top 1,000 popular videos are served from edge cache — they don't incur GCS egress costs even when in Standard tier. Autoclass (D) only transitions between Standard and Nearline — doesn't cover Coldline tier and has a per-object transition cost. All-Standard (A) is unnecessarily expensive for cold archives. All-Coldline (B) penalizes frequent access to popular videos with retrieval fees and higher latency.

---

## Case Study 3: Cymbal Retail

**Context:** Cymbal Retail is a large online retailer. They operate an e-commerce platform that runs on-premises and have begun a cloud migration. Key requirements: handle seasonal traffic spikes (10x on Black Friday/Cyber Monday), a microservices migration in progress from a monolithic application, real-time inventory management across warehouses, ML-based product recommendations, fraud detection on transactions, and integration with a legacy ERP system on-premises.

---

**Q8: Cymbal Retail's inventory system must reflect real-time stock levels across 500 warehouse locations worldwide. A purchase in Tokyo must immediately prevent overselling the last unit to a buyer in London. The system processes 50,000 inventory updates per minute. What database is MOST appropriate?**

- A) Cloud SQL with a primary in us-central1 and read replicas in each region
- B) Firestore in multi-region mode
- C) Cloud Spanner with a multi-region configuration (nam-eur-asia1)
- D) Bigtable with multi-cluster replication

**Answer: C**

**Explanation:** This is a textbook Cloud Spanner use case: global inventory with strong consistency (preventing overselling), high write throughput (50K updates/min), and multi-region active-active writes. Cloud Spanner's external consistency guarantees that a purchase in Tokyo immediately reflects for the London buyer. Cloud SQL multi-region (A) has asynchronous read replicas — a London buyer could see stale stock data and oversell. Firestore (B) provides strong consistency per document but cannot handle the transactional inventory locking semantics at this scale as effectively as Spanner. Bigtable (D) provides eventual consistency across clusters — unacceptable for inventory.

---

**Q9: Cymbal Retail needs to detect fraudulent transactions in real time. Each transaction should be scored within 200ms of payment submission. The data science team has trained an ML model using TensorFlow. What serving infrastructure meets the latency requirement?**

- A) Batch score transactions using BigQuery ML nightly; flag suspicious ones for manual review
- B) Deploy the TensorFlow model to Vertex AI Endpoints (Online Prediction); call the endpoint synchronously from the payment service; Vertex AI Endpoints autoscale to meet latency SLOs
- C) Run the TensorFlow model in a Cloud Function — it runs inline with the payment API
- D) Use Dataflow streaming pipeline to score transactions in near-real-time (1–5 second latency)

**Answer: B**

**Explanation:** Vertex AI Online Prediction endpoints serve ML model predictions with < 100ms latency (well within the 200ms requirement) and autoscale to handle traffic spikes. The payment service calls the endpoint synchronously and receives the fraud score before processing the payment. Batch scoring (A) cannot meet the 200ms real-time requirement — it's a nightly batch. Cloud Functions (C) may struggle with TensorFlow model loading (cold starts), memory limits, and consistent sub-200ms latency under load. Dataflow streaming (D) has 1–5 second latency — exceeds the 200ms requirement.

---

**Q10: Cymbal Retail is migrating from a monolith to microservices on GKE. During the migration (expected to take 18 months), both the monolith and new microservices need to co-exist and communicate. The monolith runs on GCE. What pattern enables gradual migration?**

- A) Shut down the monolith immediately; all traffic goes to incomplete microservices
- B) Use the Strangler Fig pattern: place a Global Load Balancer in front of both systems; route specific API paths to new microservices as they're ready; remaining paths route to the monolith; decommission monolith paths as microservices are completed
- C) Run the monolith on GCE and microservices on GKE in completely separate VPCs with no communication between them
- D) Deploy the monolith inside GKE as a single large pod; gradually break it apart within the pod

**Answer: B**

**Explanation:** The Strangler Fig pattern is the industry standard for monolith-to-microservices migration. A load balancer (or API gateway like Apigee or Cloud Endpoints) acts as a traffic router: new feature paths route to finished microservices while legacy paths continue to the monolith. This allows incremental migration with zero downtime and zero risk from incomplete services. Users see no change — both backends share the same domain. Immediate shutdown (A) is too risky. Separate VPCs with no communication (C) prevents the microservices from accessing existing monolith data/logic needed during migration. A single GKE pod (D) doesn't achieve the microservices architecture goal.

---

**Q11: Cymbal Retail's legacy ERP system runs on-premises and cannot be migrated to GCP. The new GCP microservices need to read order data from the ERP system. The ERP system exposes a REST API on the corporate network. What connectivity and integration approach should you use?**

- A) Expose the ERP REST API on the public internet with an API key for authentication
- B) Set up Dedicated Interconnect (or HA VPN) between on-premises and GCP; deploy Apigee Hybrid or Cloud Endpoints as an API gateway in the VPC to proxy requests to the ERP API; GCP microservices call the API gateway via private VPC networking
- C) Copy ERP data to BigQuery daily using a CSV export; microservices query BigQuery instead of the ERP API
- D) Require GKE pods to have public IPs to reach the ERP API over the internet

**Answer: B**

**Explanation:** Private connectivity (Dedicated Interconnect or HA VPN) creates a secure, private network path between GCP and on-premises. An API gateway in the GCP VPC provides a stable, managed interface for microservices to call. The gateway proxies requests to the ERP API over the private link. This is secure, performant, and doesn't expose the ERP system to the internet. Exposing ERP on the public internet (A) is a security risk for a legacy system. Daily CSV export (C) doesn't provide real-time order data access needed by operational microservices. Public IPs on pods (D) routes traffic over the internet — insecure and potentially violates corporate security policy.

---

## Case Study 4: KnightMotives Automotive

**Context:** KnightMotives is an automotive manufacturer expanding into connected vehicles. Requirements: ingest telemetry data from millions of connected vehicles (GPS, engine diagnostics, sensor data), real-time vehicle monitoring and alerting for safety events, a mobile app for vehicle owners, OTA (over-the-air) software updates to vehicle ECUs, ML models for predictive maintenance, and strict GDPR compliance as they sell vehicles in the EU (vehicle location data = personal data).

---

**Q12: KnightMotives needs to ingest telemetry data from 2 million connected vehicles, each sending data every 10 seconds. The system must handle the full fleet sending data simultaneously (200,000 messages per second). What ingestion architecture handles this scale?**

- A) Each vehicle connects directly to a Cloud SQL database via the internet
- B) Vehicles publish telemetry via MQTT to Cloud IoT Core (or a managed MQTT broker); data flows to Pub/Sub for downstream processing; Pub/Sub scales to millions of messages per second
- C) Each vehicle sends data to a Cloud Function via HTTP; the function stores it in Firestore
- D) Vehicles write data to Cloud Storage files; a batch job processes them every hour

**Answer: B**

**Explanation:** Cloud IoT Core (or Pub/Sub with an MQTT bridge) is designed exactly for this IoT scale — 200K messages/second. MQTT is the standard IoT telemetry protocol (lightweight, designed for constrained devices and unreliable networks). Pub/Sub is a fully managed, globally scalable message bus that can handle millions of messages per second. Cloud SQL (A) cannot handle 200K concurrent connections and simultaneous writes — it's not designed for IoT-scale ingestion. Cloud Functions (C) at 200K/second would require massive fan-out and incur very high costs. Batch file uploads (D) don't support real-time safety alerting.

---

**Q13: KnightMotives needs to detect and alert on safety-critical vehicle events (airbag deployment, brake failure warning) within 500ms of the event occurring. The vehicle fleet generates 200,000 telemetry messages per second. What processing architecture meets the latency requirement?**

- A) Write all telemetry to BigQuery; use BigQuery scheduled queries every minute to detect safety events
- B) Pub/Sub ingestion → Dataflow streaming pipeline with event pattern matching and session windows → real-time alert if safety event detected → Pub/Sub notification topic → Cloud Functions / Cloud Run to send push notification to mobile app and emergency services
- C) Pub/Sub → Dataproc batch job every 5 minutes → alert generation
- D) Store raw data in Bigtable; run analytics queries against Bigtable every 30 seconds

**Answer: B**

**Explanation:** Dataflow streaming with Apache Beam processes each telemetry message within 100–300ms of Pub/Sub ingestion — well within 500ms. Pattern matching on the stream (e.g., filter for `event_type=AIRBAG_DEPLOYMENT`) triggers an immediate alert path. Pub/Sub fan-out distributes alerts to downstream notification services. BigQuery scheduled queries (A) have 1-minute minimum intervals — 60x too slow. Dataproc batch (C) at 5-minute intervals is 600x too slow. Bigtable query polling (D) introduces polling latency and is not event-driven.

---

**Q14: KnightMotives collects GPS location data from vehicles in the EU. Under GDPR, this constitutes personal data. The company needs to store historical routes for trip analysis but must comply with GDPR data subject rights (access, erasure). What data architecture satisfies both analytics and GDPR requirements?**

- A) Store raw GPS data indefinitely in BigQuery — GDPR doesn't apply to anonymized aggregate analytics
- B) Store raw GPS with vehicle ID in BigQuery with row-level security; implement a deletion policy: when a user exercises erasure rights, run a BigQuery DML DELETE for their vehicle ID; use partition expiry aligned with data retention policy; use DLP API to detect and handle any direct identifiers; document data flows in a Data Processing Agreement
- C) Don't store GPS data — only process in-stream and discard; this simplifies GDPR compliance
- D) Store GPS data only in the vehicle on-board computer; never transmit to the cloud

**Answer: B**

**Explanation:** A well-designed data architecture can support both analytics and GDPR compliance. Vehicle ID is a pseudonymous identifier (links to a person via the customer database). BigQuery stores routes with vehicle ID; row-level security ensures only authorized processes can query personal data. When a user requests erasure, BigQuery DML DELETE removes their rows. Partition expiry automates retention period enforcement. DLP API scans for direct identifiers. GDPR doesn't prohibit storing personal data (A is incorrect) — it requires proper controls. Discarding all location data (C) prevents the entire use case. On-device only (D) prevents fleet analytics and safety monitoring.

---

**Q15: KnightMotives delivers OTA (over-the-air) software updates to vehicle ECUs. Updates range from 500 MB to 2 GB. The delivery system must: handle millions of vehicles downloading simultaneously, support resumable downloads (vehicles may lose connectivity), track which vehicles have successfully applied which update versions, and allow rollback of a bad update. What GCP architecture supports this?**

- A) Email update packages to vehicle owners; owners manually install via USB
- B) Store update packages in Cloud Storage (multi-region); use signed URLs for authenticated downloads (time-limited per vehicle); implement resumable upload protocol (GCS supports resumable downloads natively); track vehicle update status in Firestore (vehicle_id → update_version, status); use Pub/Sub to push update notifications to vehicles; implement rollback via previous version re-delivery
- C) Serve updates from a single Compute Engine VM with nginx; use a CDN in front
- D) Use Cloud Run to stream update packages directly from the database to vehicles

**Answer: B**

**Explanation:** Cloud Storage is purpose-built for this: it natively supports resumable downloads (HTTP Range requests), can serve millions of concurrent downloads, provides signed URLs for authentication (time-limited, vehicle-specific — no permanent credentials), and integrates with Cloud CDN for global distribution of large objects. Firestore tracks per-vehicle update state with real-time sync. Pub/Sub notifies vehicles of available updates. Rolling back is simply pointing vehicles to the previous version's signed URL. A single GCE VM (C) cannot handle millions of concurrent downloads. Cloud Run (D) is not designed for large binary streaming from databases. Manual USB (A) doesn't meet OTA requirements.
