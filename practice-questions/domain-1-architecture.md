# Domain 1: Designing and Planning a Cloud Solution Architecture
> 12 Practice Questions — PCA Difficulty

---

**Q1: A global e-commerce company needs to store product inventory data that is read by warehouses in North America, Europe, and Asia simultaneously. The inventory system must reflect real-time stock levels immediately after any purchase — overselling a product is not acceptable. The engineering team has 3 members and needs to minimize operational overhead. Which database solution best meets these requirements?**

- A) Deploy MySQL on Compute Engine in three regions with multi-master replication
- B) Use Cloud SQL with read replicas in each region and a primary in the US
- C) Use Cloud Spanner with a multi-region configuration
- D) Use Firestore in multi-region mode with strong consistency

**Answer: C**

**Explanation:** Cloud Spanner provides external consistency (the strongest consistency model available) and supports multi-region active-active writes — the only GCP-managed database that can guarantee no overselling with simultaneous global writes. Cloud SQL multi-region read replicas (B) only replicate asynchronously and cannot provide real-time globally consistent writes. Self-managed MySQL (A) increases operational burden for a 3-person team. Firestore (D) provides strong consistency but cannot handle the transactional inventory update requirements as well as Spanner for this relational use case.

---

**Q2: A media startup currently runs a video transcoding application on a cluster of 50 on-premises Linux VMs. Jobs arrive sporadically — sometimes hundreds per hour, sometimes none for days. Each transcoding job runs for 20–45 minutes. The startup wants to migrate to GCP with minimal code changes and reduce idle infrastructure costs. What is the MOST cost-effective architecture?**

- A) Lift and shift to 50 permanently-running Compute Engine VMs with Sustained Use Discounts
- B) Use Cloud Functions to trigger transcoding jobs, one function per video
- C) Deploy the transcoding application in a container and use Cloud Run Jobs with autoscaling, using Spot VMs for the underlying compute
- D) Create a Managed Instance Group (MIG) with autoscaling from 0 to 50 instances using Spot/Preemptible VMs

**Answer: D**

**Explanation:** A MIG with autoscaling from 0 instances eliminates idle costs during quiet periods, and Spot/Preemptible VMs provide up to 91% savings versus on-demand pricing — ideal for fault-tolerant batch transcoding. Spot VM interruptions are acceptable for jobs that can be retried. Cloud Run Jobs (C) would require containerization plus significant code changes (not "minimal changes"). Cloud Functions (B) has a 60-minute max execution time (Gen 2), which may be insufficient for 45-minute jobs, and would require significant refactoring. Option A wastes money running 50 VMs during idle periods.

---

**Q3: An enterprise runs a business-critical ERP application that requires a dedicated database. The database runs approximately 60% of each month consistently. The team wants to minimize database costs while maintaining performance. The application will run for at least 3 years. What pricing approach should they use for the Cloud SQL instance?**

- A) On-demand pricing — pay for what you use per second
- B) Sustained Use Discounts (SUDs) — automatic after 25% usage
- C) 3-year Committed Use Discount (CUD) for Cloud SQL
- D) Use a Spot VM and run the database on self-managed MySQL

**Answer: C**

**Explanation:** Cloud SQL supports Committed Use Discounts (CUDs) which provide up to 52% savings (1-year) or 70% savings (3-year) for predictable workloads. Since the team knows they'll use this for 3+ years, a 3-year CUD maximizes savings. SUDs (B) do apply to Cloud SQL but provide less savings than CUDs — and cannot be combined with CUDs on the same resource. Running a database on a Spot VM (D) is architecturally inappropriate — Spot VMs can be preempted at any time, causing database unavailability. Option A (on-demand) is always the most expensive for predictable workloads.

---

**Q4: A healthcare company is building a patient portal. The application has three components: a React frontend, a Node.js REST API, and a PostgreSQL database containing Protected Health Information (PHI). The team wants the database to never be exposed to the public internet. Which architecture satisfies these requirements?**

- A) Deploy all three components on GCE, use VPC firewall rules to restrict database port 5432 to the API server's IP
- B) Deploy the frontend on Cloud Storage (static hosting), API on Cloud Run with a VPC connector, and Cloud SQL (PostgreSQL) with private IP only (no public IP)
- C) Deploy the API on App Engine Standard, use Cloud SQL with a public IP and SSL certificate
- D) Deploy all components in a single Kubernetes pod on GKE, expose only the frontend via an ingress

**Answer: B**

**Explanation:** Cloud SQL with private IP only (public IP disabled) ensures the database is never accessible from the public internet. Cloud Run with a Serverless VPC Access connector allows the API to reach Cloud SQL's private IP while the API itself remains publicly accessible via Cloud Run's managed HTTPS endpoint. The static frontend on Cloud Storage is cost-efficient and scalable. Option A (firewall rules on GCE) keeps the DB port technically accessible from GCE but relies on firewall rules alone — not "never exposed to the public internet" by architecture. Option C with public IP on Cloud SQL violates the requirement. Option D packs all components into one pod, which is anti-pattern and doesn't address private DB access.

---

**Q5: A company is designing a data pipeline that ingests clickstream events from their website (approximately 500,000 events per minute), enriches each event with user profile data from a database lookup, and stores enriched events for analysis. The pipeline must process events within 5 seconds of receipt. Which architecture is MOST appropriate?**

- A) Write events to Cloud Storage, trigger a Cloud Function on each upload, query Cloud SQL for enrichment, write to BigQuery
- B) Ingest events via Pub/Sub, process with Dataflow (streaming mode) with side inputs from Bigtable for user profiles, write to BigQuery
- C) Write events to BigQuery streaming inserts directly, use BigQuery scheduled queries for enrichment every 5 minutes
- D) Ingest events via Pub/Sub, batch with a 5-minute window, process with Dataproc Spark job, write to BigQuery

**Answer: B**

**Explanation:** Pub/Sub handles the high-throughput ingestion (500K events/min = ~8,333/sec) at scale. Dataflow in streaming mode processes events with sub-5-second latency. Bigtable is ideal for the low-latency user profile lookups (< 10ms) needed at this scale — Dataflow side inputs can query Bigtable per event. BigQuery receives enriched data for analysis. Option A won't achieve 5-second latency with GCS triggers. Option C (BigQuery scheduled queries every 5 min) violates the 5-second requirement. Option D (Dataproc batch with 5-minute window) also violates the latency requirement by design.

---

**Q6: Your company runs a microservices application on GKE. One service handles payment processing and requires strict isolation — it must not be able to make outbound calls to any service except the payment gateway (external HTTPS endpoint) and the internal ledger service. Which combination of controls best enforces this? (Select TWO)**

- A) Kubernetes NetworkPolicy restricting egress from the payment service pod to only the ledger service and payment gateway IP
- B) VPC firewall rules blocking all egress from the payment service VM, with exceptions for ledger service and the payment gateway
- C) Cloud Armor WAF rules on the ingress to the payment service
- D) VPC Service Controls perimeter around the payment project
- E) IAM deny policies on the payment service's service account

**Answer: A, B**

**Explanation:** Network isolation requires both levels of control. Kubernetes NetworkPolicy (A) enforces pod-to-pod network isolation at the Kubernetes layer, restricting which pods the payment service can communicate with. VPC firewall rules (B) enforce network-level egress restriction at the VM/node level, providing defense-in-depth. Together they implement a "defense in depth" network isolation strategy. Cloud Armor (C) protects inbound traffic, not outbound. VPC Service Controls (D) restricts GCP API access, not application-level network traffic. IAM deny policies (E) control GCP resource access, not network connectivity.

---

**Q7: An organization stores 200 TB of log files in Cloud Storage Standard class. Analysis shows 90% of objects are never accessed after 30 days, and objects older than 1 year are accessed approximately once per year for compliance audits. What lifecycle policy minimizes storage costs while maintaining access capability?**

- A) Set a lifecycle rule to delete all objects after 30 days
- B) After 30 days → move to Nearline; after 90 days → move to Coldline; after 365 days → move to Archive
- C) Enable Autoclass on the bucket and let GCP manage transitions automatically
- D) Move all data to Coldline immediately and accept the retrieval fee for the 10% that are accessed

**Answer: B**

**Explanation:** A structured lifecycle policy precisely matches the access patterns. Objects accessed within 30 days stay in Standard (no minimum duration penalty). Objects accessed monthly in the 30–90 day window fit Nearline (30-day minimum). Objects accessed quarterly in 90–365 days fit Coldline (90-day minimum). Objects accessed once per year fit Archive (365-day minimum, lowest storage cost). Option C (Autoclass) is also valid but only transitions objects to Standard or Nearline (not Coldline/Archive), so it's less cost-effective for the >1 year archive tier. Option A deletes compliance data. Option D moves hot data to Coldline, incurring unnecessary retrieval fees for the 10% frequently accessed.

---

**Q8: A retail company wants to build a recommendation engine for their e-commerce platform. The data science team has existing Python-based ML training code using TensorFlow, training jobs take 6–8 hours, and they need to train new models weekly. Trained models should serve predictions with < 100ms latency to millions of users. What architecture should you recommend?**

- A) Train on Cloud Functions (Gen 2) weekly; serve from Cloud Functions
- B) Train on Vertex AI Training with GPU-enabled VMs using a managed dataset; serve predictions via Vertex AI Endpoints (online prediction)
- C) Train on Dataproc Spark MLlib weekly; serve from BigQuery ML predictions
- D) Train on GKE with a Python pod; push model to Cloud Storage; serve from App Engine Standard

**Answer: B**

**Explanation:** Vertex AI Training is purpose-built for ML training jobs — it provisions GPU VMs on demand, handles the training lifecycle, and integrates with managed datasets. Vertex AI Online Prediction (Endpoints) provides managed, autoscaling serving infrastructure with < 100ms latency for real-time predictions. The existing TensorFlow code requires minimal modification. Cloud Functions (A) has a 60-minute execution limit — insufficient for 6–8 hour training. Dataproc + BigQuery ML (C) would require rewriting TensorFlow code in SparkML/BigQuery ML syntax. GKE + App Engine (D) requires significant operational overhead for managing the serving infrastructure and doesn't handle model versioning well.

---

**Q9: A company is migrating a 3-tier web application from on-premises. The application consists of a web tier (50 Apache servers), an application tier (20 Java application servers), and a MySQL database. The migration must complete within 3 months and the application must remain unchanged. What migration approach should you recommend?**

- A) Re-architect the application as microservices on GKE before migrating
- B) Re-platform: containerize the app tier with Cloud Run, migrate MySQL to Cloud SQL
- C) Lift and shift: migrate VMs to GCE using Migrate to Virtual Machines; migrate MySQL to Cloud SQL for MySQL
- D) Lift and shift: migrate all three tiers to GCE VMs using Migrate to Virtual Machines

**Answer: C**

**Explanation:** The 3-month timeline and "application must remain unchanged" requirement indicate a lift-and-shift approach. Migrating web and app tiers to GCE using Migrate to Virtual Machines (formerly Velostrata) minimizes code changes. MySQL to Cloud SQL is a minimal-change re-platform that eliminates database operational overhead while keeping the same MySQL engine and query compatibility — a well-justified optimization. Full lift-and-shift to GCE (D) is correct for VMs but misses the opportunity to eliminate database operational burden with Cloud SQL. Re-architecting (A) or re-platforming entirely (B) violates the "application must remain unchanged" and 3-month timeline constraints.

---

**Q10: A financial services firm needs to store transaction records that must be retained for exactly 7 years and cannot be modified or deleted during that period (SEC regulatory requirement). After 7 years, records must be automatically deleted. What GCP storage solution and configuration meets these requirements?**

- A) Cloud Storage with Versioning enabled and lifecycle policy to delete after 7 years
- B) Cloud Storage with a Retention Policy (7-year duration) and Object Holds; lifecycle rule to delete after 7 years; enable Archive storage class
- C) BigQuery with table-level access controls and a scheduled query to delete rows after 7 years
- D) Cloud SQL with pg_partman extension for time-based table partitioning and automated deletion

**Answer: B**

**Explanation:** A Cloud Storage Retention Policy creates a WORM (Write Once Read Many) lock — objects cannot be deleted or modified before the retention period expires, even by project owners (when the bucket lock is applied). Archive storage class minimizes cost for the 7-year retention period. The lifecycle delete rule ensures automatic deletion after 7 years. This directly satisfies the SEC WORM requirement. Option A (versioning) doesn't prevent deletion — it just keeps versions; a user can still delete permanently. BigQuery (C) and Cloud SQL (D) don't have built-in WORM retention controls equivalent to GCS retention policies.

---

**Q11: You are designing the network architecture for a new GCP deployment. The organization has 3 teams: frontend, backend, and data. Each team should have administrative control over their own resources but cannot access other teams' resources. Additionally, network traffic between the frontend and backend tiers must be possible, but the data team's VPC should be completely isolated from the frontend. What architecture achieves this?**

- A) Single VPC with subnet-level firewall rules, one subnet per team
- B) Three separate VPCs; VPC Peering between frontend and backend VPCs; no peering to the data VPC
- C) Shared VPC with three service projects; hierarchical firewall policies to restrict cross-project traffic
- D) Three separate VPCs; a Cloud VPN tunnel between frontend and backend; data VPC completely isolated

**Answer: B**

**Explanation:** Three separate VPCs provide natural administrative isolation — each team manages their own VPC. VPC Peering between frontend and backend enables required traffic. The data VPC remains isolated (no peering). Shared VPC (C) gives a central team control over all networking — teams lose administrative control, violating the requirement. A single VPC with subnets (A) shares the same administrative boundary — the VPC admin can see all subnets. Cloud VPN (D) works but adds unnecessary cost and complexity compared to VPC Peering for same-Google-network connectivity.

---

**Q12: A startup is launching a new mobile application. The development team consists of 4 engineers. The app needs: user authentication, a NoSQL database for user profiles, push notifications, real-time data sync to mobile clients, and file storage for user-uploaded images. The team wants maximum velocity with minimal infrastructure management. Which combination of services forms the best foundation?**

- A) GCE VMs for backend, Cloud SQL for database, Cloud Storage for images, custom push notification server
- B) Firebase Authentication + Firestore (real-time sync + offline) + Firebase Cloud Messaging (FCM) + Cloud Storage; backend logic on Cloud Functions
- C) App Engine Standard for backend, Bigtable for user profiles, Cloud Storage for images, Pub/Sub for notifications
- D) GKE Autopilot for backend, Datastore for profiles, Cloud Storage for images, Cloud Tasks for notifications

**Answer: B**

**Explanation:** Firebase provides a purpose-built, fully integrated mobile backend-as-a-service. Firebase Auth handles authentication with multiple providers. Firestore provides real-time sync and offline support natively — a key mobile requirement. Firebase Cloud Messaging (FCM) is Google's push notification service. Cloud Storage integrates natively with Firebase. Cloud Functions handle any server-side logic. The entire stack is serverless with no infrastructure management — ideal for a 4-person team wanting maximum development velocity. Other options (A, C, D) require significant operational overhead for a small team and don't provide native real-time sync capabilities.
