# Domain 4: Analyzing and Optimizing Technical and Business Processes
> 10 Practice Questions — PCA Difficulty

---

**Q1: A company's data engineering team spends 40% of their time managing and patching their Hadoop cluster on GCE. The cluster runs nightly batch ETL jobs processing data from Cloud Storage, and the output is written to BigQuery. The CTO wants to reduce operational overhead. What change delivers the MOST operational improvement?**

- A) Upgrade to a newer Hadoop version on the same GCE cluster
- B) Migrate to Dataproc; use ephemeral clusters (create at job start, delete after job completes); store data in GCS (not HDFS)
- C) Containerize the Hadoop cluster and run it on GKE for easier management
- D) Add more automation scripts to handle patching and cluster maintenance

**Answer: B**

**Explanation:** Dataproc is a managed Hadoop/Spark service that eliminates OS patching, cluster configuration, and maintenance overhead. Using ephemeral clusters (create → process → delete) maximizes cost efficiency — you pay only for job runtime, not idle time. Storing data in GCS (instead of HDFS) decouples storage from compute — the cluster can be deleted without losing data. This pattern reduces operational burden from 40% to near-zero for infrastructure management. Running Hadoop on GKE (C) containerizes it but doesn't eliminate the patching/management overhead. Additional automation scripts (D) reduce manual effort but don't eliminate the root cause. Upgrading Hadoop (A) doesn't address operational overhead.

---

**Q2: A retail company's website runs on GKE and experiences 10x traffic spikes every Black Friday for 3 days. For the remaining 362 days, traffic is steady and predictable. The infrastructure team wants to optimize costs across both scenarios. Which pricing strategy BEST balances cost and reliability?**

- A) Use Spot VMs for all GKE nodes year-round; accept occasional disruptions
- B) Purchase 1-year Committed Use Discounts (CUDs) for the baseline node capacity; supplement with on-demand or Spot nodes during Black Friday peak
- C) Over-provision with enough on-demand capacity for Black Friday; run at that capacity year-round
- D) Use Spot VMs for baseline, and on-demand nodes for Black Friday only

**Answer: B**

**Explanation:** 1-year CUDs cover the predictable baseline capacity at 37–55% savings versus on-demand. During Black Friday, GKE cluster autoscaler adds burst capacity using on-demand nodes (reliability) or Spot nodes (savings, if disruption-tolerant). This two-tier approach optimizes cost for 362 days while handling the spike. Over-provisioning year-round (C) wastes ~90% of capacity for 362 days. All-Spot year-round (A) risks pod evictions during peak traffic — not suitable for production web serving. Option D (Spot baseline) provides unstable baseline capacity with no CUD savings.

---

**Q3: A company wants to measure the efficiency of their cloud migration. Before migration, their data center cost was $2M/year including hardware, power, cooling, and operations staff. Post-migration, their GCP bill is $1.4M/year. The operations team has been reduced from 8 to 3 people (average fully-loaded cost: $150K/person/year). What is the annual TCO savings?**

- A) $600K (just comparing cloud spend vs data center spend)
- B) $850K (cloud spend savings + reduced operations cost)
- C) $1.35M (cloud spend savings + full 8-person operations cost avoided)
- D) $600K minus the migration project cost

**Answer: B**

**Explanation:** TCO must include both infrastructure AND people costs. Data center cost: $2M (infrastructure) + $1.2M (8 × $150K staff) = $3.2M/year. Post-migration cost: $1.4M (GCP) + $450K (3 × $150K staff) = $1.85M/year. Annual savings: $3.2M - $1.85M = $1.35M. However, option B ($850K) only accounts for the $600K infrastructure savings + $750K staff reduction = correct $1.35M. Wait — recalculating: infrastructure savings = $600K; staff savings = 5 people × $150K = $750K; total = $1.35M. Option C is actually correct. The answer is **C** — full accurate TCO comparison shows $1.35M savings.

**Corrected Answer: C**

**Explanation:** Full TCO: on-prem = $2M infra + $1.2M staff (8 people) = $3.2M. Cloud = $1.4M GCP + $450K staff (3 people) = $1.85M. Annual savings = $3.2M - $1.85M = **$1.35M**. Option A only compares infrastructure. Option B miscalculates. Option D is incomplete without knowing migration cost. This question tests whether you include total staff costs in TCO — a common exam trap.

---

**Q4: A company's build and test pipeline takes 45 minutes to complete. Developers complain that slow feedback loops reduce productivity. Analysis shows: compile = 5 min, unit tests = 15 min, integration tests = 20 min, deploy to staging = 5 min. Integration tests can run in parallel as they test independent modules. What change provides the GREATEST pipeline acceleration?**

- A) Move the pipeline to faster GCE machines
- B) Parallelize the integration tests across multiple Cloud Build workers; each independent test module runs concurrently
- C) Cache build artifacts in Cloud Storage to skip re-compilation on unchanged code
- D) Move unit tests to run only on the main branch, not on feature branches

**Answer: B**

**Explanation:** The 20-minute integration test phase is the biggest bottleneck (44% of total time). Parallelizing independent test modules across Cloud Build workers can reduce this to near the time of the longest single module — potentially 3–5 minutes instead of 20. This alone could cut total pipeline time from 45 to 25–30 minutes — a ~40% improvement. Artifact caching (C) helps the 5-minute compile phase — smaller improvement. Faster machines (A) provides marginal improvement but doesn't address the structural bottleneck. Reducing test coverage (D) improves speed but increases risk — wrong tradeoff for quality-focused teams.

---

**Q5: A company wants to implement a GitOps-based deployment process. Currently, deployments involve manual `kubectl apply` commands run by senior engineers. They want: any commit to the `main` branch to trigger automated deployment to staging; production deployments require a pull request approval from two senior engineers; all deployment history should be auditable. Which architecture implements this?**

- A) Cloud Build trigger on `main` branch push → Cloud Build pipeline deploys to staging; separate Cloud Build trigger on Git tag → deploys to prod
- B) Cloud Build trigger on `main` push → deploys to staging; GitHub Actions workflow requiring 2 approvers → Cloud Build trigger on approval → deploys to prod; Deployment History tracked in Cloud Build history
- C) Developers manually deploy to staging for testing; senior engineers deploy to prod using gcloud CLI; Cloud Audit Logs track changes
- D) GKE Config Sync (ACM) configured with the Git repo; all changes merged to `main` auto-sync to staging cluster; prod cluster requires separate PR to `prod` branch with CODEOWNERS requiring 2 senior engineer approvals

**Answer: D**

**Explanation:** GKE Config Sync (Anthos Config Management) is the native GitOps tool — it continuously syncs a Git repository to GKE cluster state. Having separate branches (`main` → staging, `prod` → production) with branch protection rules (CODEOWNERS requiring 2 approvals for `prod`) perfectly implements the approval gate. Every deployment is a Git commit — fully auditable by design. Cloud Build triggers (A, B) work but are push-based, not reconciliation-based (Config Sync detects and corrects drift). Manual deployments (C) violate the GitOps automation requirement.

---

**Q6: Your company processes insurance claims in a batch system that runs nightly. The current process: claims arrive as CSV files via SFTP, a batch job reads the CSV, validates the data, applies business rules, and writes approved claims to a PostgreSQL database. The batch takes 6 hours; by morning, some claim statuses are stale. The business wants real-time claim processing. What is the BEST migration path?**

- A) Reduce the batch window by running the batch every hour instead of nightly
- B) Move the SFTP server to GCE; run the batch job every 15 minutes on Cloud Scheduler + Dataproc
- C) Replace SFTP with an API for claim submission → Pub/Sub for ingestion → Dataflow streaming pipeline for validation/rules → Cloud SQL (PostgreSQL) for storage; update claimants in real-time
- D) Move the existing batch job to Cloud Functions; trigger it every minute via Cloud Scheduler

**Answer: C**

**Explanation:** True real-time processing requires an event-driven streaming architecture. An API endpoint (Cloud Run or App Engine) accepts claims, publishes to Pub/Sub for reliable ingestion, Dataflow streaming processes each claim through validation and business rules as it arrives (seconds latency), and writes to Cloud SQL. This eliminates the batch paradigm entirely. Shorter batch windows (A, B) still have latency proportional to the window. Cloud Functions on a per-minute schedule (D) is micro-batching, not real-time; Cloud Functions also has an inappropriate execution model for stateful processing with complex business rules.

---

**Q7: A company has 300 GCP projects and is struggling to track cloud spend. Teams frequently exceed budgets, and finance cannot allocate costs to specific products. What practices should you recommend? (Select TWO)**

- A) Implement a mandatory resource labeling strategy: all resources must have `team`, `product`, `environment`, `cost-center` labels; enforce with Organization Policy
- B) Create one billing account per team; isolate costs naturally
- C) Set up Cloud Billing budgets with alerts at 50%, 80%, and 100% of budget per project or label group; notify both engineering leads and finance
- D) Move all projects to a flat on-demand pricing model to simplify billing
- E) Use BigQuery Billing Export to analyze spend by label in real-time dashboards

**Answer: A, C**

**Explanation:** Resource labels (A) are the foundation of cloud cost attribution — without labels, costs cannot be allocated to products/teams. Enforcing labels via Org Policy ensures no resource goes untracked. Budget alerts (C) provide proactive notification before overspending occurs — 50/80/100% thresholds catch issues early. Together, labels enable attribution and budgets enable control. Separate billing accounts per team (B) creates administrative overhead and prevents consolidated discounts. Flat on-demand pricing (D) eliminates CUD/SUD savings. BigQuery billing export (E) is valuable for analysis but doesn't solve the core labeling gap or provide proactive alerting.

---

**Q8: A data science team wants to run exploratory analysis on a 10 TB BigQuery dataset. They run many ad hoc queries, some of which accidentally scan the entire dataset and cost hundreds of dollars per query. How do you reduce cost while maintaining the team's query flexibility?**

- A) Grant the team read access to only 10% of the data
- B) Set a BigQuery custom quota (per-user daily data processed quota) on the data science team's project; partition the dataset tables and encourage using partition filters in queries
- C) Move the data to Cloud Storage and query it via BigQuery external tables — external tables have no query costs
- D) Require all queries to be reviewed and approved by a data engineer before execution

**Answer: B**

**Explanation:** Per-user daily data processed quotas cap how much data any single user can query per day, preventing runaway costs from accidental full-table scans. Table partitioning (by date, for example) combined with partition filter requirements (`requirePartitionFilter = true`) ensures queries must specify a partition — dramatically reducing data scanned per query. This maintains flexibility while controlling costs. Restricting data access (A) undermines the team's analytical capability. External tables in GCS (C) still incur query costs based on data scanned. Manual approval (D) creates a bottleneck and defeats the purpose of exploratory analysis.

---

**Q9: A company's website experiences poor performance. Users report slow page loads. The operations team analyzes Cloud Monitoring metrics and finds: web server CPU is 20% (not a bottleneck), database query latency is 800ms average (normal is 50ms), and network latency is normal. What is the MOST likely cause, and what tool should you use to investigate further?**

- A) The web server needs more CPU; upgrade to a larger GCE instance
- B) Database queries have performance issues; use Cloud SQL Query Insights to identify slow queries, missing indexes, or N+1 query patterns
- C) Network is the bottleneck; enable Premium Tier network routing
- D) The database has too many connections; increase Cloud SQL max_connections setting

**Answer: B**

**Explanation:** Database latency of 800ms (16x normal) is clearly the bottleneck. Cloud SQL Query Insights provides query performance analysis including: slow query identification, query execution plans, lock wait analysis, and historical trends. It can identify missing indexes, inefficient query patterns (like N+1 queries from ORMs), or long-running transactions. CPU at 20% (A) rules out a compute bottleneck. Network latency is stated as normal (C). While connection limits (D) could cause issues, the symptom is high latency (not connection errors), pointing to query execution problems rather than connection pool exhaustion.

---

**Q10: A company is considering whether to build or buy a data integration solution. They need to move data from 15 SaaS applications (Salesforce, SAP, Workday, etc.) into BigQuery daily. Their engineering team has capacity for one data engineer. What is the BEST recommendation?**

- A) Build a custom Python ETL pipeline using Cloud Functions + Cloud Composer (managed Airflow) for each source system
- B) Use a managed ELT/ETL SaaS tool like Fivetran or Stitch that provides pre-built connectors; configure connectors to sync to BigQuery
- C) Purchase Cloud Data Fusion and build custom pipelines for each source using the visual interface
- D) Use Dataflow with custom Apache Beam I/O connectors for each source system

**Answer: B**

**Explanation:** With one data engineer and 15 diverse SaaS sources, building and maintaining custom connectors is unsustainable. Managed ELT tools (Fivetran, Stitch, or now Google's BigQuery Data Transfer Service for some sources) provide pre-built, maintained connectors for popular SaaS applications. They handle schema changes, API rate limits, incremental sync, and error handling automatically. The data engineer's time is better spent on data modeling and analytics, not pipeline maintenance. Cloud Data Fusion (C) requires building each connector manually. Custom Beam/Dataflow (D) and custom Python (A) require significant engineering effort and ongoing maintenance per connector — not scalable for a 1-person team with 15 sources.
