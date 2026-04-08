# Domain 2: Managing and Provisioning a Solution Infrastructure

> **Exam Weight: ~15%** — Expect 7–10 questions covering resource hierarchy, IAM, IaC, and infrastructure automation.

---

## 2.1 Domain Overview

This domain tests your ability to set up and manage GCP infrastructure correctly, including identity and access management, resource organization, billing, infrastructure-as-code tooling, and automation pipelines.

**Key skills tested:**
- Designing and managing the GCP resource hierarchy
- Configuring IAM roles, policies, and service accounts
- Implementing Infrastructure as Code (Terraform, Deployment Manager, Config Connector)
- Managing billing accounts, budgets, and quotas
- Provisioning GKE clusters and shared networking
- Implementing Workload Identity Federation for secure access

---

## 2.2 GCP Resource Hierarchy

### Hierarchy Levels (Top to Bottom)
```
Organization (e.g., example.com)
├── Folder: Division A
│   ├── Folder: Engineering
│   │   ├── Project: dev-project-001
│   │   ├── Project: staging-project-001
│   │   └── Project: prod-project-001
│   └── Folder: Data Science
│       └── Project: ml-project-001
└── Folder: Division B
    └── Project: finance-project-001
```

### Resource Hierarchy Key Principles
- **Policies are inherited downward:** A policy set at the Organization level applies to all child folders and projects
- **Deny policies override allow policies:** `Deny` policies take precedence at every level
- **Projects are the billing unit:** Resources are billed to the project; each project has exactly one billing account
- **Folders enable policy grouping:** Use folders to apply common policies to a group of projects (e.g., all production projects)
- **Organization node requires Cloud Identity or Google Workspace:** The org is tied to your domain

### Policy Inheritance Rules
| Scenario | Result |
|---|---|
| Role granted at Org level | Applies to ALL resources in the org |
| Role granted at Folder level | Applies to all projects in that folder |
| Role granted at Project level | Applies only to that project |
| Conflicting policies (parent allow, child deny) | Deny wins |

> **Exam Trap:** IAM policies are **additive** for allow roles — a user gets the **union** of all roles granted at all levels. But deny policies (IAM Deny) override allows.

---

## 2.3 IAM Roles and Policies

### IAM Role Types
| Type | Description | Example |
|---|---|---|
| **Basic (primitive)** | Coarse-grained, legacy roles | Owner, Editor, Viewer |
| **Predefined** | Google-managed, fine-grained | `roles/storage.objectViewer`, `roles/compute.admin` |
| **Custom** | User-defined, specific permissions | `myorg.customRole` |

### Best Practice: Use Predefined and Custom Roles
- **Never use Owner or Editor** in production — violates least privilege
- Create **custom roles** when predefined roles have too many permissions
- Use `roles/iam.securityReviewer` for read-only security auditing
- Use `roles/bigquery.dataViewer` instead of `roles/viewer` for BigQuery access

### IAM Policy Binding Structure
```json
{
  "bindings": [
    {
      "role": "roles/compute.instanceAdmin.v1",
      "members": [
        "user:alice@example.com",
        "serviceAccount:deploy-sa@project.iam.gserviceaccount.com",
        "group:devops@example.com"
      ],
      "condition": {
        "title": "Weekday access only",
        "expression": "request.time.getDayOfWeek('America/Chicago') >= 1 && request.time.getDayOfWeek('America/Chicago') <= 5"
      }
    }
  ]
}
```

### IAM Conditions
- Add time-based, resource-based, or tag-based conditions to role bindings
- Use **Resource Manager tags** in IAM conditions for attribute-based access control (ABAC)
- Example: Grant access to storage buckets tagged `env=prod` only during business hours

### IAM Deny Policies (Important for 2025 Exam)
- **IAM Deny** allows you to **explicitly deny** specific permissions, overriding any allow policies
- Applied at org, folder, or project level
- Use case: "Ensure no one (not even org admin) can delete production databases"
- Deny policies evaluate **before** allow policies

---

## 2.4 Service Accounts

### Service Account Types
| Type | Description | Use Case |
|---|---|---|
| **User-managed** | You create and manage | Application identity for GCP services |
| **Default service accounts** | Auto-created (Compute Engine default SA, App Engine default SA) | Avoid using — too broad permissions |
| **Google-managed** | Managed by Google APIs | Internal GCP service-to-service calls |

### Service Account Best Practices
- Create **dedicated service accounts per application** (not one shared SA)
- Grant only the **minimum required roles** to each SA
- Do **not** create and download service account keys if avoidable
- Use **Workload Identity Federation** instead of keys for non-GCP workloads
- Use **short-lived tokens** (impersonation) instead of long-lived keys
- Rotate keys if they must be used; set org policy to **disable service account key creation** org-wide

### Service Account Key Risks
```
Risk: Downloaded .json key file can be leaked in:
  - Source code repositories (git history)
  - Container images
  - CI/CD environment variables
  - Log files

Mitigation:
  - org policy: constraints/iam.disableServiceAccountKeyCreation
  - Use Workload Identity Federation instead
  - Use Secret Manager to store keys if unavoidable
```

---

## 2.5 Workload Identity Federation

### What It Solves
Workload Identity Federation allows **external workloads** (GitHub Actions, AWS, Azure, on-premises) to access GCP resources **without service account keys**.

### How It Works
```
External Identity Provider (GitHub Actions OIDC)
    ↓ (OIDC token)
Workload Identity Pool (GCP)
    ↓ (token exchange for short-lived GCP credential)
Service Account Impersonation
    ↓
GCP Resource Access (Cloud Storage, BigQuery, etc.)
```

### Key Components
- **Workload Identity Pool:** Container for external identities
- **Workload Identity Pool Provider:** Configures the external IdP (AWS, Azure AD, GitHub OIDC, etc.)
- **Attribute Mapping:** Maps external token claims to Google attributes (e.g., `google.subject = assertion.sub`)
- **Attribute Conditions:** Restrict which external identities can authenticate

### GKE Workload Identity
- Allows Kubernetes service accounts (KSA) to act as GCP service accounts (GSA)
- **Setup:**
  1. Enable Workload Identity on GKE cluster
  2. Create GCP service account
  3. Bind KSA to GSA: `roles/iam.workloadIdentityUser`
  4. Annotate Kubernetes SA with GSA email

> **Exam Tip:** If a question asks "how to allow a GKE pod to access Cloud Storage without mounting service account keys," the answer is **GKE Workload Identity**.

---

## 2.6 Billing Accounts and Budget Management

### Billing Account Structure
```
Billing Account (credit card / invoicing)
├── Project A → $500/month
├── Project B → $200/month
└── Project C → $800/month
Total: $1,500/month
```

- One billing account can be linked to **multiple projects**
- One project can only be linked to **one billing account** at a time
- **Billing account admin** role: `roles/billing.admin`
- **Billing account viewer** role: `roles/billing.viewer`

### Budget Alerts
- Set budgets at **billing account or project level**
- Configure alerts at thresholds: 50%, 90%, 100% (actual or forecasted)
- Integrate with **Pub/Sub** for programmatic responses (e.g., auto-shutdown non-prod resources)
- Connect with **Cloud Monitoring** notification channels (email, PagerDuty, Slack)

### Budget Alert + Pub/Sub Automation Example
```
Budget Alert ($1,000 threshold reached)
    ↓
Pub/Sub Topic: billing-alerts
    ↓
Cloud Functions: shutdown-nonprod-vms
    ↓
Action: Stop all VMs with label env=dev
```

### Cost Allocation
- Use **resource labels** for cost attribution (e.g., `team: payments`, `env: prod`)
- Export billing data to **BigQuery** for custom analysis
- Use **Cloud Billing reports** and **cost breakdown** in the console
- **Committed Use Discounts (CUDs):** 1-year (20–30% discount) or 3-year (50–70%) for compute

---

## 2.7 Infrastructure as Code (IaC)

### Terraform on GCP

**Terraform** is the primary IaC tool for GCP (preferred over Deployment Manager in 2025).

#### Terraform Provider Setup
```hcl
terraform {
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 5.0"
    }
  }
  backend "gcs" {
    bucket = "my-terraform-state"
    prefix = "prod/state"
  }
}

provider "google" {
  project = var.project_id
  region  = var.region
}
```

#### Remote State Best Practices
- Store Terraform state in **Cloud Storage** (enable versioning + locking via Cloud Storage object lock or Terraform Cloud)
- Use **workspaces** for environment separation (dev/staging/prod)
- Never store state locally in production

#### Terraform GCP Resource Example
```hcl
resource "google_compute_instance" "web_server" {
  name         = "web-server-01"
  machine_type = "e2-medium"
  zone         = "us-central1-a"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-12"
    }
  }

  network_interface {
    subnetwork = google_compute_subnetwork.subnet.id
    access_config {}
  }

  service_account {
    email  = google_service_account.app_sa.email
    scopes = ["cloud-platform"]
  }

  labels = {
    env  = "prod"
    team = "platform"
  }
}
```

### Cloud Deployment Manager

Google's native IaC tool using YAML, Python, or Jinja2 templates.

```yaml
# deployment.yaml
resources:
  - name: my-vm
    type: compute.v1.instance
    properties:
      zone: us-central1-a
      machineType: zones/us-central1-a/machineTypes/n1-standard-1
      disks:
        - deviceName: boot
          boot: true
          initializeParams:
            diskSizeGb: 20
            sourceImage: projects/debian-cloud/global/images/family/debian-12
```

> **Exam Tip:** Deployment Manager is a legacy tool — questions may contrast it with Terraform. Terraform is preferred for multi-cloud and complex deployments. Deployment Manager is GCP-native and requires no external tooling.

### Config Connector

- Kubernetes add-on that allows you to manage GCP resources using **Kubernetes manifests**
- Resources are defined as Kubernetes Custom Resource Definitions (CRDs)
- Enables GitOps workflows: manage GCP infra alongside K8s workloads

```yaml
# Create a Cloud Storage bucket via Config Connector
apiVersion: storage.cnrm.cloud.google.com/v1beta1
kind: StorageBucket
metadata:
  name: my-storage-bucket
  namespace: config-connector
spec:
  location: US-CENTRAL1
  storageClass: STANDARD
  versioning:
    enabled: true
```

---

## 2.8 Quota Management

### Quota Types
| Type | Description | Example |
|---|---|---|
| **Rate quota** | Limits API calls per minute | 1,000 BigQuery API requests/min |
| **Allocation quota** | Limits total resources | 8 CPUs per region |

### Quota Management Process
1. **View quotas:** Console → IAM & Admin → Quotas, or use `gcloud compute project-info describe`
2. **Request increases:** Submit quota request from the console (usually approved in 1–2 days for standard quotas)
3. **Monitor quota usage:** Cloud Monitoring metric `serviceruntime.googleapis.com/quota/rate/net_usage`
4. **Alert on quota exhaustion:** Set Cloud Monitoring alerts on quota utilization > 80%

> **Exam Tip:** "Quota exceeded" errors are a common interview scenario. Correct answer: request a quota increase, not architectural redesign unless explicitly stated.

---

## 2.9 Resource Labeling Strategy

### Label Best Practices
- Labels are **key-value pairs** attached to resources for organization and cost attribution
- Use a **consistent taxonomy** across all resources

### Recommended Label Schema
| Key | Values | Purpose |
|---|---|---|
| `env` | `prod`, `staging`, `dev` | Environment |
| `team` | `platform`, `data`, `frontend` | Owning team |
| `app` | `payments-api`, `user-service` | Application |
| `cost-center` | `cc-001`, `cc-002` | Finance attribution |
| `data-classification` | `public`, `internal`, `confidential` | Security |

### Labels vs Tags vs Network Tags
| Type | Scope | Use For |
|---|---|---|
| **Labels** | Almost all GCP resources | Cost allocation, filtering, IaC selection |
| **Resource Manager tags** | GCP resources (for IAM conditions, org policies) | ABAC, conditional access |
| **Network tags** | Compute Engine instances | Firewall rule targeting |

> **Exam Trap:** Network tags only apply to firewall rules. Resource Manager tags are for IAM conditions and org policies. Labels are for billing and filtering — they do NOT directly affect security policies.

---

## 2.10 Shared VPC

### What Is Shared VPC?
- A **host project** owns the VPC and subnets
- **Service projects** use the shared subnets but deploy their own resources
- Centralized network management; decentralized resource deployment

### Shared VPC Setup Steps
1. Designate a project as the **Host Project** (enable Shared VPC hosting)
2. **Attach service projects** to the host project
3. Grant `roles/compute.networkUser` on specific subnets to service project accounts
4. Resources in service projects use host project subnets

### Shared VPC vs VPC Peering
| Aspect | Shared VPC | VPC Peering |
|---|---|---|
| Admin control | Centralized (host project) | Distributed (each project) |
| Routing | Centralized routing | Non-transitive routing |
| Use case | Enterprise, centralized networking | Partner/vendor connectivity |
| Firewall rules | Managed in host project | Managed per project |

---

## 2.11 GKE Cluster Provisioning

### Cluster Types
| Type | Node Management | Use Case |
|---|---|---|
| **Standard** | You manage node pools | Custom node configs, DaemonSets |
| **Autopilot** | Google manages nodes | Most production workloads |
| **Private cluster** | Nodes have no public IPs | Security-sensitive workloads |

### Private Cluster Configuration
```bash
gcloud container clusters create my-private-cluster \
  --region us-central1 \
  --enable-private-nodes \
  --enable-private-endpoint \
  --master-ipv4-cidr 172.16.0.0/28 \
  --enable-ip-alias \
  --subnetwork my-subnet
```

### Node Pool Best Practices
- Use **separate node pools** for different workloads (general, GPU, high-memory)
- Enable **node auto-provisioning** (part of cluster autoscaler) to create pools on demand
- Use **Spot node pools** for batch/stateless workloads
- Enable **Shielded nodes** for enhanced security (Secure Boot, vTPM)
- Use **Confidential GKE Nodes** for sensitive data processing

### GKE Cluster Autoscaler
- Configured per node pool with min/max node count
- Scales up when pods are pending due to insufficient resources
- Scales down when nodes are underutilized for 10+ minutes
- Works alongside HPA (pod-level) and VPA (resource-level)

---

## 2.12 Cloud APIs and API Enablement

### API Management
- Each GCP project must **explicitly enable** APIs before using services
- Enable via console, `gcloud services enable`, or Terraform
- APIs can be enabled/disabled via org policy: `constraints/serviceusage.restrictedServices`

```bash
# Enable required APIs for a GKE project
gcloud services enable \
  container.googleapis.com \
  compute.googleapis.com \
  cloudresourcemanager.googleapis.com \
  iam.googleapis.com
```

### Organization Policies
Common org policies for infrastructure governance:

| Constraint | Description |
|---|---|
| `constraints/compute.requireShieldedVm` | Require Shielded VM features |
| `constraints/compute.restrictCloudSQLInstances` | Restrict Cloud SQL to specific regions |
| `constraints/iam.disableServiceAccountKeyCreation` | Prevent SA key downloads |
| `constraints/compute.skipDefaultNetworkCreation` | Prevent auto-creation of default VPC |
| `constraints/gcp.resourceLocations` | Restrict resource creation to specific regions |
| `constraints/storage.uniformBucketLevelAccess` | Enforce uniform bucket-level access |
| `constraints/compute.restrictPublicIpAccess` | Prevent public IPs on VMs |

> **Exam Tip:** Org policies are **proactive controls** (prevent bad configurations before they happen). Security Command Center findings are **detective controls** (find bad configurations after they exist).

---

## 2.13 Cloud Build for Automation

### Cloud Build Overview
- **Serverless CI/CD platform** — no servers to manage
- Builds run in isolated Docker containers
- Integrates with GitHub, GitLab, Bitbucket via Cloud Build Triggers
- Outputs artifacts to Artifact Registry, Cloud Storage, or container registries

### Cloud Build Configuration (cloudbuild.yaml)
```yaml
steps:
  # Build the Docker image
  - name: 'gcr.io/cloud-builders/docker'
    args: ['build', '-t', 'us-central1-docker.pkg.dev/$PROJECT_ID/my-repo/my-app:$COMMIT_SHA', '.']

  # Run unit tests
  - name: 'gcr.io/cloud-builders/docker'
    args: ['run', 'us-central1-docker.pkg.dev/$PROJECT_ID/my-repo/my-app:$COMMIT_SHA', 'npm', 'test']

  # Push image to Artifact Registry
  - name: 'gcr.io/cloud-builders/docker'
    args: ['push', 'us-central1-docker.pkg.dev/$PROJECT_ID/my-repo/my-app:$COMMIT_SHA']

  # Deploy to Cloud Run
  - name: 'gcr.io/cloud-builders/gcloud'
    args:
      - 'run'
      - 'deploy'
      - 'my-app'
      - '--image=us-central1-docker.pkg.dev/$PROJECT_ID/my-repo/my-app:$COMMIT_SHA'
      - '--region=us-central1'

options:
  logging: CLOUD_LOGGING_ONLY

images:
  - 'us-central1-docker.pkg.dev/$PROJECT_ID/my-repo/my-app:$COMMIT_SHA'
```

### Cloud Build Triggers
- **Push to branch:** Trigger on every commit to `main` or release branches
- **Pull Request:** Trigger on PR creation/update for testing
- **Manual:** Trigger on demand
- **Pub/Sub:** Trigger based on events

---

## 2.14 Common Exam Question Patterns and Traps

### Pattern 1: Least Privilege
**Q:** "Developers need to deploy to Cloud Run but must not access production data..."
**A:** Create a custom IAM role with `run.services.update` only; bind to staging projects but not prod

### Pattern 2: Billing and Labels
**Q:** "Finance wants to see costs broken down by team..."
**A:** Apply `team` labels to all resources; export billing data to BigQuery; use Data Studio/Looker Studio dashboard

### Pattern 3: Workload Identity
**Q:** "A GitHub Actions pipeline needs to push images to Artifact Registry without storing credentials..."
**A:** Configure Workload Identity Federation with GitHub OIDC provider

### Pattern 4: Resource Hierarchy
**Q:** "Apply a consistent security policy across all production projects..."
**A:** Create a `Production` folder; place all prod projects in it; apply org policy/IAM at the folder level

### Common Traps
- **Primitive roles (Owner/Editor) in production:** Always wrong — use predefined or custom roles
- **Service account keys:** Avoid if possible; use Workload Identity Federation or metadata server
- **Default service accounts:** Do not use them — they have broad permissions; create dedicated SAs
- **VPC peering is non-transitive:** Cannot route traffic through an intermediate network
- **Labels are not security controls:** Cannot use labels in firewall rules; use network tags instead
- **Org policies are inherited but can be overridden:** Child resources can override parent org policies (if allowed)

---

## 2.15 Summary Table

| Topic | Key Concept | Exam Signal Words |
|---|---|---|
| Resource hierarchy | Org → Folder → Project → Resource | "apply policy to all projects", "inheritance" |
| IAM least privilege | Custom roles, predefined roles | "minimum permissions", "separation of duties" |
| Service accounts | Dedicated SA per app, no key downloads | "application identity", "GCE service account" |
| Workload Identity | Federation for external, GKE WI for pods | "GitHub Actions", "no keys", "GKE pod access" |
| Shared VPC | Host + service projects | "centralized networking", "multiple projects" |
| IaC | Terraform (primary), Deployment Manager | "automate provisioning", "reproducible infra" |
| Config Connector | K8s-native GCP resource management | "GitOps", "Kubernetes manifest", "GCP resource" |
| Org policies | Proactive guardrails | "prevent", "restrict", "enforce" |
| Billing | Labels + BigQuery export | "cost attribution", "team billing", "budget alert" |
| Quota management | Request increases, monitor usage | "quota exceeded", "scaling limit" |
| Cloud Build | Serverless CI/CD | "build pipeline", "automate deployment" |
| GKE provisioning | Standard vs Autopilot, private clusters | "node pools", "private nodes", "Workload Identity" |
