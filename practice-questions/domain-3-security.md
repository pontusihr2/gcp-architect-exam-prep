# Domain 3: Designing for Security and Compliance
> 12 Practice Questions — PCA Difficulty

---

**Q1: A company processes credit card payments and must comply with PCI-DSS. Their payment processing application runs on GKE. The security team wants to ensure that the cardholder data environment (CDE) is isolated from the rest of GCP infrastructure, and that no cardholder data can be exfiltrated — even by a legitimate GCP user with IAM access. Which control BEST prevents data exfiltration?**

- A) Grant payment service account only `roles/storage.objectViewer` on the CDE bucket
- B) Implement VPC Service Controls to create a service perimeter around the CDE project, blocking data movement to projects outside the perimeter
- C) Enable Cloud Armor on the ingress to the payment service
- D) Use customer-managed encryption keys (CMEK) for all storage in the CDE project

**Answer: B**

**Explanation:** VPC Service Controls (VPC SC) creates a security perimeter around GCP projects that restricts which API calls can be made to/from the perimeter — even by authenticated, authorized identities. This prevents exfiltration scenarios like: "a compromised admin copies CDE data to their personal project." IAM alone (A) controls authorization but doesn't prevent an authorized user from copying data out. Cloud Armor (C) protects against web attacks (DDoS, OWASP), not data exfiltration by internal principals. CMEK (D) ensures you control encryption keys but doesn't prevent authorized access to the data.

---

**Q2: A company's security policy requires that all encryption keys for sensitive data be managed by the company, not Google. The key material should never leave the company's Hardware Security Module (HSM). Which GCP service satisfies this requirement?**

- A) Google-managed encryption keys (default) — Google handles key management
- B) Customer-managed encryption keys (CMEK) with Cloud KMS software keys
- C) Customer-managed encryption keys (CMEK) with Cloud HSM (Cloud KMS HSM-backed keys)
- D) Customer-supplied encryption keys (CSEK) — provide keys directly in API requests

**Answer: C**

**Explanation:** Cloud HSM (Cloud KMS with HSM-backed key versions) stores cryptographic keys in FIPS 140-2 Level 3 validated hardware security modules. The key material never leaves the HSM in plaintext — cryptographic operations occur within the HSM. This satisfies "key material never leaves company's HSM" using a cloud-hosted but hardware-isolated HSM. CMEK with software keys (B) stores keys in software — they can be exported. CSEK (D) requires the company to supply keys per request — complex key management; keys travel with API calls. Default Google-managed keys (A) gives Google key control.

---

**Q3: An application running on GCE needs to call the BigQuery API to read analytics data. A junior developer suggests using a service account JSON key file stored in the application's environment variables. What is the MOST secure alternative?**

- A) Store the JSON key file in Secret Manager and retrieve it at startup
- B) Attach a service account to the GCE VM instance; the application uses Application Default Credentials (ADC) to authenticate automatically
- C) Use the VM's metadata server to generate short-lived OAuth tokens using the default Compute Engine service account
- D) Use IAM Conditions to limit the JSON key file's validity to 24-hour windows

**Answer: B**

**Explanation:** Attaching a service account directly to a GCE VM is the recommended practice. The Application Default Credentials (ADC) library automatically uses the VM's attached service account via the metadata server — no key file required. The credentials are short-lived and automatically rotated by Google. This completely eliminates the risk of a leaked JSON key file. Option C is essentially the same mechanism (ADC uses the metadata server) but describes it at a lower level — both B and C are correct, B is the better answer because it describes the recommended pattern. Storing keys in Secret Manager (A) is better than env vars but still involves a long-lived credential. IAM Conditions (D) don't control key validity.

---

**Q4: A company stores sensitive customer PII in BigQuery. They need to allow their data analytics team to run queries on the data, but the PII fields (SSN, credit card numbers, email addresses) must not be visible in query results. The analytics team still needs to see non-PII fields. What solution should you implement?**

- A) Create a BigQuery view that excludes PII columns; grant analysts access only to the view
- B) Use the Cloud Data Loss Prevention (DLP) API to scan BigQuery tables and automatically redact PII fields; store redacted results in a separate dataset; grant analysts access to the redacted dataset
- C) Use BigQuery column-level access controls (policy tags) to classify PII columns; grant analysts `datacatalog.categoryFineGrainedReader` on non-PII tags only
- D) Enable BigQuery authorized views and implement row-level security policies to filter PII columns

**Answer: C**

**Explanation:** BigQuery column-level security using Policy Tags (Data Catalog taxonomy) is the native, scalable solution. Tag PII columns (SSN, CC, email) with a sensitive policy tag. Users without the `Fine-Grained Reader` role on that policy tag receive null/masked values when querying those columns — they can still query all other columns. This is enforced at the BigQuery engine level (cannot be bypassed). Option A (views) works but requires maintaining views for every table, and analysts could potentially access base tables if not careful. DLP (B) creates a copy of data with latency — analysts work on stale redacted copies. Row-level security (D) restricts rows, not columns.

---

**Q5: A financial company requires all administrative actions on GCP infrastructure (creating VMs, modifying firewall rules, IAM changes) to be logged and retained for 7 years for regulatory audit purposes. By default, which logs are captured, and what additional configuration is needed?**

- A) All logs including Data Access logs are captured by default for 7 years; no additional configuration needed
- B) Admin Activity audit logs are enabled by default (30-day retention); configure a log sink to export to Cloud Storage with a 7-year lifecycle retention policy
- C) No logs are captured by default; enable all audit log types in each project and export to BigQuery
- D) Admin Activity logs are enabled by default and retained for 400 days; extend by enabling VPC Flow Logs

**Answer: B**

**Explanation:** Admin Activity audit logs (which capture all administrative actions: create, delete, modify on GCP resources, IAM changes) are enabled by default and **cannot be disabled**. However, Cloud Logging retains logs for only **30 days** by default (_log buckets have configurable retention_). To meet the 7-year regulatory requirement: create a log sink (aggregated export) from Cloud Logging → Cloud Storage, configure a Cloud Storage lifecycle policy to retain objects for 7 years, and enable WORM retention on the bucket. Data Access logs (C) are NOT enabled by default — they must be explicitly enabled per service. Default retention (D) is 30 days, not 400 days.

---

**Q6: A startup is building a multi-tenant SaaS application. Each tenant's data is stored in a separate BigQuery dataset. The application uses a single service account for all database operations. A new security audit recommends that a compromise of one tenant's workload should not expose other tenants' data. What change achieves this?**

- A) Encrypt each tenant's dataset with a separate CMEK key
- B) Create a separate service account per tenant; grant each SA access only to its tenant's dataset; use Workload Identity to bind each SA to the specific application component handling that tenant
- C) Implement VPC Service Controls with one perimeter per tenant project
- D) Use BigQuery authorized views to expose only each tenant's data through a single query interface

**Answer: B**

**Explanation:** Per-tenant service accounts with scoped BigQuery dataset permissions implement least privilege. If one tenant's service account or workload is compromised, the attacker can only access that tenant's dataset. Workload Identity ensures each application component runs as its tenant-specific SA without key files. Separate CMEK keys (A) protect confidentiality at rest but don't prevent an authorized SA (with access to all datasets) from reading across tenants. VPC Service Controls (C) is project-level granularity — one perimeter per project is costly for multi-tenancy. BigQuery authorized views (D) centralize data access but still use a single SA underneath.

---

**Q7: An organization wants to ensure that only approved container images can run in their GKE production cluster. Images must have been signed by the company's security team using a verified cryptographic key before they can be deployed. Which GCP feature implements this?**

- A) Container Registry vulnerability scanning — scan all images before deployment
- B) Binary Authorization — enforce attestation policies requiring images to be signed by an approved attestor before GKE will admit them
- C) Cloud Armor — WAF rules to block requests from containers running unapproved images
- D) GKE Workload Identity — ensures all pod workloads have valid GCP identities

**Answer: B**

**Explanation:** Binary Authorization is a deploy-time security control for GKE (and Cloud Run). It enforces that container images have been signed (attested) by a specified attestor (e.g., the security team's Cloud KMS key) before the GKE admission controller allows the pod to be created. This prevents deploying unsigned or compromised images. Artifact Registry vulnerability scanning (A) detects vulnerabilities but doesn't enforce policy — a vulnerable image could still be deployed. Cloud Armor (C) is a WAF for web traffic, not container image control. Workload Identity (D) provides GCP identity to pods, not image signing enforcement.

---

**Q8: A company must comply with GDPR. A user exercises their "right to erasure" and requests that all their data be deleted. The user's data exists in: a Cloud Firestore document, a BigQuery table, archived data in Cloud Storage (Coldline), and Pub/Sub message archives. What is the MOST complete approach?**

- A) Delete the Firestore document; the rest will expire eventually
- B) Delete the Firestore document; run BigQuery DML DELETE; delete GCS objects; note that Pub/Sub messages expire after 7 days maximum retention
- C) Use the Cloud DLP API to scan and delete the user's PII across all services automatically
- D) Pseudonymize the user's data by replacing their ID with a random token — this satisfies GDPR erasure

**Answer: B**

**Explanation:** Right to Erasure requires actively deleting the data. For each service: Firestore — delete the document directly. BigQuery — use DML `DELETE WHERE user_id = X`. Cloud Storage — delete the specific objects containing the user's data (retrieve object list by user metadata/prefix). Pub/Sub — messages auto-expire within the retention window (max 31 days); if already expired or if no subscription, data is gone. If messages still exist, deleting the subscription/topic removes them. Cloud DLP (C) can scan and identify PII but doesn't have a native "delete across all services" capability. Pseudonymization (D) is a valid GDPR technique but courts and regulators generally expect actual deletion for erasure requests.

---

**Q9: A company's security team wants to detect when a GCS bucket is made publicly accessible (the bucket ACL is changed to `allUsers`). They want automated remediation within 5 minutes of detection. Which architecture achieves this?**

- A) Run a daily Cloud Scheduler job to audit all bucket ACLs using the GCS API
- B) Enable the Security Command Center (SCC) and configure a Pub/Sub notification for the "PUBLIC_BUCKET_ACL" finding; trigger a Cloud Function to remove the allUsers ACL automatically
- C) Use Cloud Asset Inventory to monitor bucket IAM policy changes; trigger a Cloud Function for remediation
- D) Enable VPC Service Controls on all projects — this prevents buckets from being made public

**Answer: B**

**Explanation:** Security Command Center detects "PUBLIC_BUCKET_ACL" findings in near-real-time (within minutes of the change). SCC can publish findings to Pub/Sub, triggering a Cloud Function that calls the GCS API to remove the `allUsers` ACL. This achieves the < 5-minute remediation requirement. Cloud Asset Inventory (C) can also detect IAM changes but has slightly higher latency for policy changes compared to SCC findings. Daily scheduler jobs (A) don't meet the 5-minute requirement. VPC Service Controls (D) restricts API access from outside a perimeter but doesn't prevent bucket-level ACL changes made from within the project.

---

**Q10: A company is building an internal API that should only be accessible from their on-premises network and their GCP VPCs. The API must never be accessible from the public internet. Which combination achieves this? (Select TWO)**

- A) Deploy the API on Cloud Run with `--ingress=internal-and-cloud-load-balancing`
- B) Deploy the API on Cloud Run with `--ingress=internal`; connect on-prem via Cloud VPN or Dedicated Interconnect; access the Cloud Run service via a Private Service Connect endpoint
- C) Deploy the API on Cloud Run with a public HTTPS endpoint; use Cloud Armor to block all non-RFC1918 source IPs
- D) Deploy the API behind an Internal Application Load Balancer (Internal ALB); configure on-prem access via Dedicated Interconnect
- E) Use VPC Service Controls to block all external access to the Cloud Run API

**Answer: B, D**

**Explanation:** Cloud Run with `--ingress=internal` (B) ensures the service is only reachable from within the VPC and Cloud Load Balancing — not directly from the internet. Combined with Private Service Connect and Cloud VPN/Interconnect for on-prem access, this creates a fully private path. An Internal Application Load Balancer (D) is only accessible within the VPC and connected networks (via Interconnect/VPN) — this is the standard pattern for internal-only APIs. Cloud Armor (C) blocks at the edge but the service still technically has a public endpoint — not "never accessible from internet." VPC Service Controls (E) control GCP service API access, not custom application endpoints.

---

**Q11: A DevOps engineer wants to grant a contractor temporary read access to Cloud Logging for the next 30 days. After 30 days, the access must automatically expire with no manual intervention required. What is the recommended approach?**

- A) Create the IAM binding and set a calendar reminder to remove it in 30 days
- B) Use IAM Conditions with a time-based expression: `request.time < timestamp("2025-XX-XXT00:00:00Z")`; bind `roles/logging.viewer` with this condition
- C) Create a temporary service account, grant it `roles/logging.viewer`, and provide the SA key to the contractor; delete the SA after 30 days
- D) Add the contractor to a Google Group with logging access; remove them from the group after 30 days

**Answer: B**

**Explanation:** IAM Conditions with a date-expiry expression automatically expire the IAM binding at the specified timestamp — no manual intervention required. After the date passes, the IAM binding is still present but the condition evaluates to false, effectively revoking access. This is the native GCP mechanism for temporary access grants. Calendar reminders (A) rely on human action and can be forgotten. A service account with a key (C) creates a long-lived credential that could be used after the 30 days if not properly deleted. Google Group management (D) still requires manual removal from the group.

---

**Q12: A healthcare company must ensure that PHI (Protected Health Information) stored in Cloud Storage is encrypted with keys that the company fully controls. If the company revokes the encryption key, access to the PHI must be immediately impossible — even for Google engineers. What configuration achieves this?**

- A) Enable Google-managed encryption with customer-supplied encryption keys (CSEK)
- B) Use CMEK with Cloud KMS; the company manages key versions and can disable/destroy key versions to revoke access
- C) Use CMEK with Cloud HSM and enable key destruction — once the key is destroyed, data is permanently inaccessible
- D) Use client-side encryption before uploading to GCS; store the encryption key in an on-premises HSM

**Answer: B**

**Explanation:** CMEK with Cloud KMS gives the company control over the encryption keys. Disabling a key version in Cloud KMS immediately prevents GCP from using that key to decrypt data — all subsequent read operations fail even for Google. The key is not destroyed (so data could be recovered by re-enabling), but access is revoked. Destroying the key (C) permanently prevents access (correct for permanent revocation), but B is the better answer for "if we want to revoke access with potential recovery." CSEK (A) provides customer key control but keys must be provided with each API request — this doesn't prevent Google internal access. Client-side encryption (D) is the most extreme isolation but makes GCP data services (like BigQuery or Dataflow integration) unusable.
