# Domain 6: Ensuring Solution and Operations Reliability
> 10 Practice Questions — PCA Difficulty

---

**Q1: A company's SLO for their checkout API is 99.9% availability (measured over a rolling 30-day window). Last month, the service had two incidents: one 25-minute outage and one 15-minute degradation (50% error rate). How much of their error budget has been consumed, and what should the team do?**

- A) The error budget is 43.8 minutes/month; 25 minutes consumed = 57% remaining; continue deploying
- B) The error budget is 43.8 minutes/month; the 25-minute full outage = 25 min; the 15-minute degradation at 50% error rate = 7.5 min equivalent; total ≈ 32.5 min consumed = ~74% budget consumed; reduce deployment frequency and focus on reliability
- C) The error budget is 0 — any outage violates the 99.9% SLO
- D) Error budgets only count full outages, not degradations; 25 minutes consumed = 57% remaining; the degradation doesn't count

**Answer: B**

**Explanation:** Error budget for 99.9% over 30 days = 0.1% × 43,200 min = 43.2 minutes. The 25-minute full outage consumes 25 minutes directly. The 15-minute degradation at 50% error rate consumes proportional budget: 15 min × 50% = 7.5 minutes equivalent. Total consumed ≈ 32.5 min / 43.2 min = ~75% consumed. With only ~25% remaining for the rest of the month, the team should reduce deployment risk (feature freeze) and focus on reliability improvements. Error budgets account for partial degradations proportionally — not just binary outages (D is wrong). The error budget is not zero (C) — 99.9% means 0.1% is allowed to fail.

---

**Q2: A Cloud Run service intermittently returns 503 errors during traffic spikes. Cloud Monitoring shows the service is hitting its maximum instance limit (100 instances). The p99 latency is 8 seconds during spikes. What changes address the root cause? (Select TWO)**

- A) Increase the Cloud Run max-instances limit beyond 100 to allow more horizontal scaling
- B) Enable Cloud Run minimum instances to keep warm instances ready during anticipated spikes
- C) Increase the Cloud Run CPU and memory limits — the service is CPU-bound
- D) Optimize the service code to reduce per-request latency (e.g., connection pooling, caching); profile the service to identify bottlenecks
- E) Switch from Cloud Run to GKE for better scaling control

**Answer: A, D**

**Explanation:** Two separate issues exist: (1) The service is hitting the instance cap, causing 503s — increasing max-instances (A) allows more horizontal scaling to handle the spike. (2) p99 latency of 8 seconds is high — even with more instances, slow requests hold connections longer, requiring more instances for the same throughput. Code optimization (D) — connection pooling (reduce Cloud SQL connection overhead), caching (reduce repeated computation), and profiling — directly reduces per-request latency, lowering instance requirements. Minimum instances (B) reduce cold starts but don't help when max-instances is the binding constraint. CPU/memory (C) might help if CPU-bound but isn't indicated. Migrating to GKE (E) adds operational overhead without addressing the root latency problem.

---

**Q3: Your team manages a GKE cluster. The business requires that any node failure should not cause application downtime. An entire zone in your region failed. The application currently runs with 3 replicas all scheduled to nodes in zone A. What configuration changes prevent this from happening again?**

- A) Increase replicas to 10; with more pods, losing zone A is less impactful
- B) Configure pod anti-affinity rules to spread pods across zones; deploy a regional GKE cluster (3 zones); ensure PodDisruptionBudget (PDB) allows maximum 1 unavailable replica; set replicas ≥ 3 (one per zone)
- C) Use GKE Autopilot — it automatically handles zone distribution
- D) Enable Cloud Armor on the GKE ingress; this prevents zone-level failures

**Answer: B**

**Explanation:** Zone-level failure resilience requires pods to be distributed across multiple zones. Pod anti-affinity rules with `topologyKey: topology.kubernetes.io/zone` force Kubernetes to schedule pods on different zone nodes. A regional GKE cluster has node pools spanning 3 zones. With ≥3 replicas spread across 3 zones, losing 1 zone loses at most 1 replica — the remaining 2 continue serving traffic. PodDisruptionBudget ensures rolling updates don't simultaneously remove too many pods. Just increasing replicas (A) without anti-affinity doesn't guarantee zone distribution — Kubernetes might still schedule them in one zone. Cloud Armor (D) is a WAF, not related to zone failure. GKE Autopilot (C) helps with distribution but doesn't alone fix the existing workload configuration.

---

**Q4: An organization wants to implement chaos engineering to improve reliability. They want to test whether their GKE application can survive a random pod failure. What is the RECOMMENDED approach?**

- A) Manually delete random pods in production to observe behavior
- B) Use a chaos engineering tool (e.g., Chaos Monkey, Litmus Chaos) in a staging environment that mirrors production topology; inject pod failures and observe whether the application recovers within SLO
- C) Analyze historical incident data to understand failure modes; no active failure injection needed
- D) Enable GKE node auto-repair — this randomly restarts nodes to test recovery

**Answer: B**

**Explanation:** Chaos engineering should be performed in a representative staging environment before production, using purpose-built tools that provide controlled injection, observability, and automatic recovery safeguards. Litmus Chaos and similar tools provide Kubernetes-native chaos experiments (pod kill, network delay, CPU stress) with configurable blast radius and automated rollback. This tests real resilience behavior. Manual deletion in production (A) is uncontrolled and risky. Historical analysis (C) doesn't test current resilience capabilities. GKE node auto-repair (D) repairs nodes when unhealthy — it's not a chaos engineering tool and doesn't "randomly restart" nodes as a test.

---

**Q5: A company runs a payment processing service with an SLO of 99.95% availability. The service is currently deployed in a single region (us-central1). The architecture team wants to design for the failure of an entire GCP region. What architecture changes are required?**

- A) Deploy redundant instances within us-central1 across all three zones; this provides sufficient redundancy
- B) Deploy the service in two regions (us-central1 and us-east1); use a Global External Application Load Balancer with health checks; configure Cloud Spanner for the database (multi-region); set up DNS failover
- C) Implement a warm standby in a second region; manually failover during a regional outage
- D) Enable Cloud Run's built-in multi-region feature to automatically replicate the service

**Answer: B**

**Explanation:** Region-level resilience requires active infrastructure in multiple regions. A Global External Application Load Balancer automatically routes traffic to the healthy region when one region fails (using backend health checks). Cloud Spanner multi-region provides database availability across regions with zero RPO. This is an active-active architecture with automatic failover — meeting the strict 99.95% SLO even through a regional outage. Multi-zone within one region (A) protects against zone failures, not full regional failures. Warm standby with manual failover (C) cannot meet 99.95% SLO — manual failover takes too long. Cloud Run doesn't have a built-in multi-region feature (D) — you must deploy to multiple regions and configure global LB.

---

**Q6: A team releases a new version of their API every 2 weeks. Post-release, they frequently detect bugs that were missed in testing. Rollbacks take 30 minutes. The MTTR is 2 hours. What practices should they implement to reduce MTTR and increase release confidence? (Select TWO)**

- A) Implement canary deployments: release to 5% of traffic first; monitor error rates for 30 minutes before full rollout
- B) Increase the release cadence to daily to catch bugs faster
- C) Add comprehensive integration tests to the CI/CD pipeline; use Cloud Build test steps and fail the pipeline on test failures
- D) Implement automated rollback: configure Cloud Deploy rollback triggers based on Cloud Monitoring error rate SLOs; auto-rollback if error rate exceeds threshold within 10 minutes of deployment
- E) Require manual QA sign-off for every release; this ensures no bugs reach production

**Answer: A, D**

**Explanation:** Canary deployments (A) expose bugs to only 5% of traffic — blast radius is minimal, and 30 minutes of monitoring catches issues before they affect all users. Automated rollback (D) reduces MTTR from 2 hours to 10 minutes — if the error rate exceeds the SLO threshold, Cloud Deploy automatically rolls back without human intervention. Together, these reduce both the probability of full-impact incidents (canary) and the recovery time (auto-rollback). More frequent releases (B) increases deployment risk without addressing quality. Integration tests in CI (C) are valuable but primarily reduce pre-production bugs, not post-release MTTR. Manual QA (E) slows release velocity without fully preventing production bugs.

---

**Q7: A company uses Cloud Monitoring to track their key services. An alert fires: "Cloud SQL CPU utilization > 80% for 15 minutes." The on-call engineer receives the page. What is the MOST appropriate immediate response?**

- A) Immediately upgrade the Cloud SQL instance to a larger machine type to resolve the alert
- B) Investigate: check Cloud SQL Query Insights for slow queries; check active connections; check if a scheduled job is running; if a specific query is causing load, kill it; consider temporary connection throttling; escalate if needed
- C) Reboot the Cloud SQL instance to clear the high CPU condition
- D) Do nothing — Cloud SQL autoscales CPU automatically

**Answer: B**

**Explanation:** The right response to an alert is to investigate before taking action. High CPU could be caused by: a bad query (kill it), a scheduled batch job (expected, monitor), a traffic spike (scale if needed), or a runaway process (identify and address). Cloud SQL Query Insights shows the top CPU-consuming queries with execution details. Understanding the root cause prevents taking disruptive actions unnecessarily. Immediately scaling up (A) may be the right action but should follow diagnosis. Rebooting (C) would cause downtime and doesn't fix the root cause. Cloud SQL does NOT autoscale CPU (D) — it requires manual action or a scheduled maintenance window for resizing.

---

**Q8: A company's SRE team wants to implement postmortem culture after incidents. The last incident was a 90-minute database outage caused by a failed schema migration. What should a blameless postmortem include?**

- A) Identify the engineer who ran the migration and document it as human error; add a warning to their performance review
- B) Timeline of events; root cause analysis (the schema migration ran without a dry-run); contributing factors (no staging migration test, no rollback script prepared); action items: add migration dry-run step to deploy pipeline, test all migrations in staging first, add automated rollback scripts to migration procedures; assign owners and due dates
- C) Change the deployment process to require manager approval for all database changes; this prevents future incidents
- D) Document only the customer impact and resolution time; don't investigate root cause to avoid blame

**Answer: B**

**Explanation:** A blameless postmortem focuses on systemic failures, not individual blame. It includes a factual timeline, multiple contributing factors (not just "human error"), and concrete action items with owners and due dates. In this case: missing dry-run capability, no staging testing, no rollback script — all systemic gaps. These produce actionable improvements that prevent recurrence. Blaming the engineer (A) discourages reporting and doesn't fix systems. Manager approval for all DB changes (C) creates bottlenecks without addressing the root cause. Documenting only customer impact (D) fails to enable organizational learning.

---

**Q9: A company's BigQuery pipeline runs nightly and has been gradually getting slower over 3 months. Initial runtime was 20 minutes; it now takes 75 minutes. The data volume has only increased 15%. What should you investigate first?**

- A) The network bandwidth between BigQuery and GCS is degraded
- B) Investigate query execution plans in BigQuery (INFORMATION_SCHEMA.JOBS), check for data skew in shuffle operations, verify partition pruning is occurring, check if new columns or JOINs were added to queries in the past 3 months, and review table partitioning and clustering configuration
- C) BigQuery slot capacity has been reduced; purchase more flat-rate slots
- D) The Cloud Billing account has budget caps that throttle BigQuery performance

**Answer: B**

**Explanation:** A gradual slowdown with minimal data growth typically indicates query inefficiency rather than capacity problems. Key investigation areas: INFORMATION_SCHEMA.JOBS shows historical query execution stats — look for increased bytes processed, shuffle data, or step times. Data skew occurs when partition sizes become uneven over time. Missing partition pruning means queries scan more data as the table grows. Query plan changes (new JOINs, missing clustering) compound over time. BigQuery and GCS are in the same infrastructure — network is not a bottleneck (A). Slot exhaustion (C) would cause consistent slowdowns from the start, not gradual degradation. Billing budget caps (D) don't throttle query performance.

---

**Q10: A company operates a global application and wants to understand the end-user experience in different geographic regions. They suspect users in Asia-Pacific are experiencing higher latency than users in North America, but their server-side metrics look healthy. What should they implement to measure and diagnose this?**

- A) Check Cloud Monitoring server-side latency metrics from the load balancer
- B) Deploy Synthetic Monitors (Uptime Checks) from multiple Google Cloud regions; instrument the frontend with the Google Cloud Trace JavaScript SDK to capture real user browser traces; use Cloud Monitoring dashboards to compare regional latency breakdowns
- C) Ask users in Asia-Pacific to report their experience manually
- D) Enable VPC Flow Logs in Asia-Pacific regions to measure network latency

**Answer: B**

**Explanation:** Server-side metrics (A) measure latency at the load balancer — they miss DNS resolution time, TLS handshake, network transit from user to Google edge, and browser rendering time. Synthetic monitors (Cloud Monitoring Uptime Checks) from GCP's globally distributed check locations simulate user requests from various regions, revealing geographic latency differences. Real User Monitoring (RUM) with browser-side tracing captures actual user-experienced latency including all frontend components. Together, synthetic + RUM provides a complete picture of the user experience vs. server-side health. VPC Flow Logs (D) capture server-to-server network flows within GCP, not user-to-server latency.
