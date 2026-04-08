# Domain 5: Managing Implementation
> 8 Practice Questions — PCA Difficulty

---

**Q1: A company is deploying a new microservices application to GKE. The application consists of 12 services that communicate via gRPC. During testing, developers report that debugging failures is difficult because it's impossible to trace a request across multiple services. What should be implemented to address this?**

- A) Add detailed print statements to each service's request handling code
- B) Implement distributed tracing using Cloud Trace; instrument each service with the OpenTelemetry SDK to propagate trace context and send spans to Cloud Trace
- C) Enable VPC Flow Logs to capture network traffic between services
- D) Deploy a dedicated logging service that each microservice calls synchronously to record request details

**Answer: B**

**Explanation:** Distributed tracing with Cloud Trace enables end-to-end visibility across microservices. OpenTelemetry is the CNCF-standard instrumentation library that propagates trace context (trace ID, span ID) in gRPC headers, creating a trace tree showing exactly which services a request traversed and how long each hop took. This directly solves cross-service debugging. Print statements (A) generate unstructured logs with no cross-service correlation. VPC Flow Logs (C) capture network metadata (IP, port, bytes) — not application-level request tracing. A synchronous logging service (D) adds latency to every request and creates a bottleneck/SPOF; synchronous logging also doesn't provide trace correlation.

---

**Q2: A team is migrating a large PostgreSQL database (8 TB) from on-premises to Cloud SQL. The migration must minimize downtime — the business can only accept a maximum 2-hour maintenance window for cutover. The migration plan should handle the initial data transfer and keep the target database synchronized until cutover. What approach achieves this?**

- A) During the 2-hour maintenance window: pg_dump the 8 TB database, transfer via gsutil to GCS, import into Cloud SQL
- B) Use Database Migration Service (DMS): create a migration job in CDC mode; DMS performs the initial full load while the source remains online; DMS continuously replicates changes (CDC); at cutover time, stop writes to source, let DMS catch up, then switch applications to Cloud SQL
- C) Set up Cloud SQL read replicas pointing to the on-premises PostgreSQL via Dedicated Interconnect; promote the replica during the 2-hour window
- D) Export data weekly as CSV files to GCS; import into Cloud SQL; accept up to 1 week of data loss

**Answer: B**

**Explanation:** Database Migration Service (DMS) is GCP's managed database migration tool. It uses Change Data Capture (CDC) to continuously replicate changes from the source database to Cloud SQL in near-real-time. The initial load runs while the source database serves production traffic (online migration). By cutover time, the target is fully synchronized. The 2-hour window is used only for final CDC catch-up, application reconfiguration, and validation — not the full data transfer. Option A (pg_dump during window) would require the full dump + transfer + import time for 8 TB — likely 8–24 hours, far exceeding 2 hours. Cloud SQL read replicas (C) cannot replicate from external (on-prem) databases. Option D has unacceptable data loss.

---

**Q3: A mobile application backend runs on Cloud Run. During a recent incident, the team discovered that errors were not detected until users reported them, 45 minutes after they started. The team wants to implement proactive error detection and alerting with a target detection time of < 5 minutes. What should be implemented?**

- A) Ask users to report errors via an in-app feedback form; ops team reviews reports daily
- B) Configure Cloud Run to write structured logs with error severity to Cloud Logging; create log-based metrics for ERROR level logs; set up Cloud Monitoring alerting policies with a 5-minute evaluation window and PagerDuty/email notification
- C) Enable Cloud Run request logs; manually review them each morning
- D) Set up a Cloud Scheduler job that runs a synthetic check every 5 minutes and alerts if the response code is not 200

**Answer: B**

**Explanation:** Structured logging with error severity allows Cloud Logging to filter and count ERROR logs. Log-based metrics convert these into Cloud Monitoring metrics. Alerting policies with a short evaluation window (1–5 minutes) trigger notifications (PagerDuty, email, Slack via webhook) when error rates exceed a threshold — providing < 5-minute detection. Synthetic monitoring (D) only detects total outages (non-200 responses), not backend errors that still return 200. Daily manual review (C) has a 24-hour detection window. User reports (A) are reactive, not proactive. Option B provides the most granular, fastest error detection.

---

**Q4: A team is deploying a new feature behind a feature flag. The feature flag is stored as an environment variable in a Cloud Run service. To toggle the feature on, the team needs to update the environment variable and redeploy. They want to update the flag without redeployment and without modifying the application code. What should you recommend?**

- A) Store the feature flag value in a Cloud Storage file; the application reads the file on each request
- B) Store the feature flag in Secret Manager; application reads the secret at startup; update the secret value to toggle
- C) Use Firebase Remote Config to manage feature flags; the application reads the remote config value on each request — changes take effect without redeployment
- D) Use Cloud Memorystore (Redis) to store feature flags; application reads from Redis on each request; update Redis values to toggle

**Answer: C**

**Explanation:** Firebase Remote Config is purpose-built for dynamic configuration and feature flags — it's designed for exactly this use case. The application fetches config values from Remote Config on each request (or with caching/TTL), and changes take effect without redeployment. It supports A/B testing, gradual rollouts by percentage, and user segment targeting. Secret Manager (B) stores secrets, not dynamic feature flags — and updating a Secret Manager secret doesn't propagate automatically to running Cloud Run instances without a restart/new revision. GCS file read (A) works but is not purpose-built for feature flags and lacks A/B testing capabilities. Redis (D) works technically but requires infrastructure management and no SDK-level A/B support.

---

**Q5: Your team maintains a Cloud Composer (Apache Airflow) DAG that orchestrates a daily data pipeline. The DAG has been failing inconsistently for 3 days. The error message is `Database connection pool exhausted`. The pipeline connects to Cloud SQL, and the number of parallel tasks has not changed. What is the MOST likely cause, and what should you investigate?**

- A) Cloud SQL is undersized; upgrade to a larger instance type
- B) The Airflow scheduler has a bug; redeploy the Cloud Composer environment
- C) Investigate whether recent changes increased parallelism (more Airflow workers, more concurrent tasks), or whether connection leaks exist in the DAG code (connections not properly closed); also check Cloud SQL max_connections setting
- D) Cloud Composer has a memory issue; increase the Airflow worker memory allocation

**Answer: C**

**Explanation:** "Connection pool exhausted" means all available database connections are in use. Possible causes: (1) Increased parallelism — more Airflow workers or tasks running concurrently each opening connections; (2) Connection leaks — DAG code opens connections but doesn't close them (e.g., missing `finally` block or context manager); (3) Cloud SQL max_connections limit too low. The right investigation path: check if Airflow worker count changed, review DAG code for connection management, check Cloud SQL `max_connections` metric in Cloud Monitoring. Simply upgrading Cloud SQL (A) treats symptoms without diagnosing cause. Redeploying Composer (B) is a guess without investigation. Memory (D) is unrelated to connection pool exhaustion.

---

**Q6: A company uses Pub/Sub to process payment events. The consumer application (Cloud Run) processes each message and calls an external payment gateway API. Recently, messages are being processed multiple times, causing duplicate charges. What is the ROOT CAUSE, and how do you fix it?**

- A) Pub/Sub is delivering messages more than once; this is expected (at-least-once delivery); fix the payment service to be idempotent by checking if a transaction ID has already been processed before calling the payment gateway
- B) Increase the Pub/Sub message ack deadline so the consumer has more time to acknowledge
- C) Switch to Pub/Sub exactly-once delivery mode; this guarantees each message is processed exactly once
- D) Add more Cloud Run instances to process messages faster; the duplicate processing is caused by slow processing causing re-delivery

**Answer: A**

**Explanation:** Pub/Sub guarantees at-least-once delivery by design — messages may be redelivered if not acknowledged within the ack deadline. The correct fix is making the consumer idempotent: before calling the payment gateway, check if the transaction ID already exists in a database (e.g., Firestore, Cloud SQL). If it does, skip the payment call and acknowledge the message. This is a fundamental distributed systems pattern. Pub/Sub does offer "exactly-once" processing (B, C) but only within a single region and with ordering keys — it's not a substitute for idempotent business logic, especially when calling external APIs. Ack deadline extension (B) reduces redelivery frequency but doesn't eliminate duplicates. Scaling (D) doesn't solve the idempotency issue.

---

**Q7: A company is onboarding a new development team onto GCP. The team lead asks for "Developer" access — they should be able to deploy applications, manage Cloud Run services, read Cloud SQL data, but NOT be able to change IAM policies or delete projects. What combination of roles satisfies least privilege?**

- A) Grant `roles/editor` at the project level — this covers all development activities
- B) Grant `roles/run.admin` + `roles/cloudsql.client` + `roles/storage.objectAdmin` + `roles/logging.viewer` on the specific project; explicitly do NOT grant any IAM admin roles
- C) Grant `roles/owner` — developers need to be unblocked
- D) Grant `roles/run.developer` + `roles/cloudsql.viewer` + `roles/storage.objectViewer` — read-only for databases and storage

**Answer: B**

**Explanation:** The combination of targeted predefined roles (B) provides exactly what developers need: `run.admin` to manage Cloud Run services, `cloudsql.client` to connect to Cloud SQL (via proxy), `storage.objectAdmin` for application storage, `logging.viewer` for debugging. No IAM admin roles are included, so project/IAM cannot be modified. `roles/editor` (A) is overly broad — it allows modifying almost everything including potentially sensitive resources and IAM in some contexts. `roles/owner` (C) includes IAM admin. Option D is too restrictive — read-only SQL access doesn't allow application connectivity.

---

**Q8: A company wants to implement a secrets management strategy for their GKE workloads. Secrets include database passwords, API keys, and TLS certificates. Currently, secrets are stored as Kubernetes Secrets (base64 encoded, stored in etcd). The security team is concerned about the security of this approach. What improvement should you recommend?**

- A) Store secrets in environment variables in the pod spec — base64 encoded for security
- B) Integrate GKE with Secret Manager: store secrets in Secret Manager (with CMEK encryption); use the Secret Manager add-on for GKE (External Secrets Operator or Config Connector) to sync secrets to Kubernetes Secrets backed by Secret Manager; enable KMS encryption for etcd
- C) Store secrets in a ConfigMap — ConfigMaps are more secure than Secrets for sensitive data
- D) Use Vault (HashiCorp) deployed on GCE; GKE pods authenticate to Vault using a service account token and retrieve secrets at runtime

**Answer: B**

**Explanation:** This is the GCP-native best practice: Secret Manager stores secrets with CMEK encryption, access controls, versioning, and audit logging. The External Secrets Operator or GKE Config Connector syncs Secret Manager secrets to Kubernetes Secrets in the cluster. Additionally, enabling GKE's Application-layer secrets encryption (KMS-based etcd encryption) protects secrets at rest in etcd. This addresses the security team's concern about plain etcd storage. Environment variables (A) are less secure — visible in process lists. ConfigMaps (C) are not encrypted at rest. Vault (D) is a valid alternative but requires additional infrastructure management and is not the GCP-native recommended approach.
