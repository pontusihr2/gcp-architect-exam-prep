# Domain 2: Managing and Provisioning a Solution Infrastructure
> 10 Practice Questions — PCA Difficulty

---

**Q1: Your team manages 200 GCP projects across 5 business units. Each business unit should only be able to deploy resources in specific geographic regions to meet data sovereignty requirements. The security team wants to enforce this without relying on individual project owners to configure it correctly. What is the MOST scalable approach?**

- A) Send each project owner a configuration checklist; audit quarterly
- B) Create an Org Policy constraint `constraints/gcp.resourceLocations` and apply it at the Folder level for each business unit
- C) Use Cloud Asset Inventory to detect out-of-region resources and trigger automated remediation
- D) Create a custom IAM role that removes `compute.regions.use` permission for non-approved regions and apply it to all users

**Answer: B**

**Explanation:** Org Policies applied at the Folder level enforce constraints on all projects within that folder hierarchy — even if a project owner tries to create resources in an unauthorized region, the policy prevents it. This is preventive (not reactive) and requires no action from individual project owners. Cloud Asset Inventory with remediation (C) is reactive — resources are created before being detected and removed. Custom IAM roles (D) cannot block resource creation at the infrastructure level; IAM permissions and org policies are separate mechanisms, and IAM doesn't have granular enough compute region controls. Manual checklists (A) are not scalable for 200 projects.

---

**Q2: A company uses Terraform to manage all GCP infrastructure. The team wants to ensure that no infrastructure changes are made manually through the GCP Console or gcloud CLI. What combination of controls enforces this? (Select TWO)**

- A) Configure Terraform Cloud with Sentinel policies to block non-Terraform changes
- B) Enable GCP Audit Logs (Admin Activity) and set up a log-based alert when console-originated changes are detected
- C) Restrict IAM permissions so that only the Terraform service account has write access to GCP resources; developers have viewer-only access
- D) Use VPC Service Controls to block the GCP Console from accessing project APIs
- E) Enable Binary Authorization to verify all deployments come from Terraform

**Answer: B, C**

**Explanation:** Restricting human IAM permissions to viewer-only (C) ensures developers cannot make changes directly — only the Terraform service account (used by CI/CD pipelines) has write access, enforcing infrastructure-as-code. Audit log alerts (B) provide a detective control — any manual changes generate alerts for immediate investigation. Together they create preventive + detective layers. Terraform Cloud Sentinel (A) is a valid enterprise Terraform control but only covers Terraform Cloud — not direct GCP API access. VPC Service Controls (D) restricts network-level API access, not console user access. Binary Authorization (E) is for container image signing, not infrastructure management.

---

**Q3: You need to deploy a new version of a Cloud Run service with zero downtime. The new version has a database schema change that is backward-compatible. You want to validate the new version handles 5% of production traffic before full rollout, with the ability to instantly roll back if error rates increase. What deployment approach should you use?**

- A) Redeploy the Cloud Run service with the new image; Cloud Run handles zero-downtime automatically
- B) Deploy the new version as a separate Cloud Run service, update the load balancer to send 5% of traffic to the new service
- C) Deploy the new revision with `--no-traffic`; use traffic splitting to send 5% to the new revision; monitor error rates; if healthy, shift 100% traffic
- D) Use a blue/green approach: deploy a second Cloud Run service, test it separately, then switch DNS

**Answer: C**

**Explanation:** Cloud Run natively supports traffic splitting between revisions. Deploying with `--no-traffic` prevents the new revision from receiving requests immediately. Traffic splitting (5% canary) validates the new version in production with limited blast radius. Rollback is instant — redirect 100% back to the previous revision with a single command (`gcloud run services update-traffic`). Option A would immediately route all traffic to the new version (not a canary). Option B overcomplicates with a separate service and load balancer change. Option D (blue/green) introduces a DNS change latency for rollback and doesn't support granular percentage splitting.

---

**Q4: A development team needs to provision identical environments (dev, staging, production) for a microservices application on GKE. The environments must be reproducible and any configuration drift must be detectable. What is the recommended approach?**

- A) Manually configure each GKE cluster through the GCP Console; document steps in a runbook
- B) Use Terraform for cluster provisioning and Helm charts + GitOps (Argo CD or Config Sync) for Kubernetes workload deployment
- C) Use gcloud CLI scripts stored in a shared Google Drive folder; run them for each environment
- D) Use Deployment Manager for GKE clusters; use kubectl apply manually for each environment

**Answer: B**

**Explanation:** Terraform ensures reproducible, version-controlled cluster infrastructure (VPC, GKE config, node pools, IAM). Helm charts provide templated, parameterized Kubernetes manifests allowing environment-specific values. GitOps with Argo CD or GKE Config Sync continuously reconciles cluster state against Git — drift is detected and corrected automatically. This combination addresses both provisioning reproducibility and runtime configuration drift. Manual approaches (A, C) are error-prone and don't detect drift. Deployment Manager (D) is GCP-native IaC but less feature-rich than Terraform for GKE; kubectl manual apply doesn't detect drift.

---

**Q5: You manage a GKE cluster running production workloads. A critical CVE has been announced affecting the node OS. Nodes need to be patched as quickly as possible with zero downtime to running pods. What is the FASTEST approach that maintains availability?**

- A) Manually SSH into each node and run `apt-get upgrade`; restart the node after patching
- B) Delete the node pool and create a new one with the patched node image; Kubernetes will reschedule pods automatically
- C) Enable GKE node auto-upgrade and trigger a manual upgrade using `gcloud container clusters upgrade` with surge upgrade settings
- D) Cordon and drain each node manually, patch it, then uncordon

**Answer: C**

**Explanation:** GKE's managed node upgrade with surge upgrade settings provisions new (patched) nodes before draining old ones — maintaining capacity throughout the upgrade. GKE handles the cordon/drain/reschedule cycle automatically and in parallel per the surge settings. This is the fastest and safest zero-downtime approach. Manual SSH patching (A) is slow, error-prone, and doesn't guarantee nodes run the same image as new nodes going forward. Deleting/recreating the node pool (B) works but is disruptive — all pods are rescheduled simultaneously, potentially causing availability issues. Manual cordon/drain (D) is tedious and slow at scale.

---

**Q6: Your organization wants to establish a landing zone for 50+ GCP projects. New projects should automatically get: standard VPC configuration, approved firewall rules, organization policies applied, centralized log export, and billing alerts. Which GCP tool is designed for this use case?**

- A) Cloud Foundation Toolkit (CFT) with Terraform modules
- B) Cloud Deployment Manager with custom templates
- C) Manually create each project and apply settings via gcloud scripts
- D) Google Cloud Resource Manager API called from a Cloud Function on project creation

**Answer: A**

**Explanation:** The Cloud Foundation Toolkit (CFT) is Google's official set of Terraform modules specifically designed for establishing GCP landing zones and following Google's best practices. CFT modules handle org policies, VPC configuration, log exports, billing alerts, and IAM at scale — purpose-built for the described use case. Cloud Deployment Manager (B) is Google's older IaC tool, less feature-rich than Terraform for landing zone use cases. Manual scripts (C) are not scalable or maintainable. The Cloud Function approach (D) is custom code that duplicates what CFT provides out of the box.

---

**Q7: An application team accidentally deleted a Cloud SQL production database. The last successful backup was taken 6 hours ago. The database has Point-in-Time Recovery (PITR) enabled with binary logging. How can you recover data with minimum loss?**

- A) Restore the last backup from 6 hours ago; all data from the last 6 hours is unrecoverable
- B) Contact Google Support; they maintain copies of deleted databases for 30 days
- C) Use Cloud SQL PITR to restore to a point in time just before the deletion event, using the backup + binary logs to replay transactions
- D) Use Cloud Storage bucket where Cloud SQL backup exports are stored and import the last export file

**Answer: C**

**Explanation:** Cloud SQL PITR (Point-in-Time Recovery) with binary logging enabled allows restoration to any second within the retention window. GCP uses the most recent backup as a base and replays binary log transactions to reach the target point in time — just before the deletion. This recovers nearly all data with minimal loss (seconds rather than 6 hours). Option A incorrectly assumes PITR isn't available. Google Support (B) does not maintain copies of deleted database contents. Manual GCS exports (D) are typically less frequent than automated backups and still wouldn't use binary logs for precise recovery.

---

**Q8: A company needs to deploy a Python web application that currently runs on a single server. They expect traffic spikes during business hours (10x normal traffic) and near-zero traffic at night. The team wants to minimize infrastructure management. The application is stateless and uses Cloud SQL for its database. What is the BEST deployment option?**

- A) Deploy on a single large Compute Engine VM with vertical scaling
- B) Deploy on a Managed Instance Group (MIG) with autoscaling; configure min=1, max=10 instances
- C) Deploy on Cloud Run; configure min-instances=0 and max-instances=auto; connect to Cloud SQL via Cloud SQL Auth Proxy
- D) Deploy on App Engine Standard with automatic scaling; connect to Cloud SQL via the built-in connector

**Answer: C or D (both are valid; prefer C for new workloads)**

**Explanation:** Both Cloud Run (C) and App Engine Standard (D) are correct. Cloud Run scales to zero during nighttime (eliminating idle cost), scales rapidly during traffic spikes, and has no infrastructure management. App Engine Standard also scales to zero and autoscales but is the older platform. Cloud Run is preferred for new Python workloads in 2024+ due to greater flexibility. A single GCE VM (A) cannot scale horizontally. A MIG (B) requires more operational management (instance templates, health checks, autoscaling policies) and has a minimum of 1 VM — not as cost-efficient as serverless for near-zero nighttime traffic.

---

**Q9: Your team wants to use Infrastructure as Code to manage GCP resources, but you also need to manage multi-cloud resources (AWS, Azure). You want a single workflow and state management system. Some team members are unfamiliar with cloud-specific tools. Which IaC tool should you choose?**

- A) Cloud Deployment Manager — GCP-native, best integration
- B) Terraform — multi-cloud provider support, large ecosystem, HCL syntax
- C) Pulumi — multi-cloud with general-purpose programming languages
- D) Ansible — declarative playbooks, agentless, multi-cloud

**Answer: B**

**Explanation:** Terraform is the industry standard for multi-cloud IaC. The GCP Terraform provider covers virtually all GCP resources. Terraform's HCL syntax has a lower learning curve than programming languages, and its large community provides extensive examples. State management in Terraform Cloud or GCS backend provides team collaboration. Cloud Deployment Manager (A) is GCP-only. Pulumi (C) supports multi-cloud but requires programming language knowledge (Python, TypeScript, etc.) — higher barrier for cloud-unfamiliar team members. Ansible (D) is better suited for configuration management than infrastructure provisioning; state management is weaker.

---

**Q10: An operations team wants to automate the deletion of idle Compute Engine VM instances to reduce costs. An "idle" VM is defined as CPU utilization < 5% for 7 consecutive days. What architecture implements this most reliably?**

- A) Use the GCP Console to manually review VM metrics weekly and delete idle instances
- B) Set up a Cloud Scheduler job (daily) that triggers a Cloud Function; the function queries Cloud Monitoring for CPU metrics, identifies idle VMs based on the criteria, and deletes them via the Compute Engine API
- C) Enable the Recommender API idle VM recommendations and set up an Org Policy to auto-enforce them
- D) Use Cloud Monitoring's alerting policy to send an email when a VM's CPU is < 5% for 7 days; ops team manually deletes

**Answer: B**

**Explanation:** Cloud Scheduler provides reliable time-based triggering. The Cloud Function queries Cloud Monitoring API for historical CPU utilization metrics (7-day window), identifies qualifying VMs, and calls the Compute Engine API to delete them. This is fully automated, auditable (Cloud Audit Logs capture the deletions), and reproducible. The Recommender API (C) does provide idle VM recommendations but does not auto-enforce deletions — it only surfaces recommendations for human review. Manual review (A, D) doesn't meet the automation requirement and is error-prone.
