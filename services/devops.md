# GCP DevOps & CI/CD Services — Exam Reference (2025)

---

## Cloud Build

### Overview

- **Serverless CI/CD** platform; executes build steps in Docker containers
- No infrastructure to manage; scales to zero; concurrent builds supported
- **Free tier**: 120 build-minutes/day on default pool

### Build Configuration

```yaml
# cloudbuild.yaml
steps:
  - name: 'gcr.io/cloud-builders/docker'
    args: ['build', '-t', '$_IMAGE_TAG', '.']
  - name: 'gcr.io/cloud-builders/docker'
    args: ['push', '$_IMAGE_TAG']
  - name: 'gcr.io/cloud-builders/gke-deploy'
    args: ['run', '--filename=k8s/', '--cluster=$_CLUSTER']
substitutions:
  _IMAGE_TAG: gcr.io/$PROJECT_ID/app:$COMMIT_SHA
  _CLUSTER: production-cluster
options:
  logging: CLOUD_LOGGING_ONLY
  machineType: 'E2_HIGHCPU_8'
timeout: '1200s'
```

### Triggers

| Trigger Type | Description |
|-------------|-------------|
| **Push to branch** | Build on every push to specific branch (e.g., `main`) |
| **Push new tag** | Build on git tag push; use regex pattern |
| **Pull request** | Build on PR open/update; requires Cloud Source Repos or GitHub App |
| **Manual** | Trigger manually via Console, CLI, or API |
| **Pub/Sub** | Trigger builds from Pub/Sub messages (event-driven) |
| **Webhook** | Trigger from any external system via HTTPS webhook |

### Substitutions

- Built-in: `$PROJECT_ID`, `$BUILD_ID`, `$COMMIT_SHA`, `$SHORT_SHA`, `$BRANCH_NAME`, `$TAG_NAME`, `$REPO_NAME`
- User-defined: Must start with `_` (e.g., `$_ENVIRONMENT`); set in trigger config or API call

### Worker Pools

- **Default pool**: Shared infrastructure; no VPC connectivity; internet access available
- **Private Pools**: Dedicated worker VMs in your VPC; access to private resources (Cloud SQL, Artifact Registry internal, on-prem via VPN/Interconnect); required for regulated environments
- Private pools support custom machine types and disk sizes

> **Exam Tip**: Use Private Pools when builds need to access resources not reachable from the public internet (VPC-only databases, internal package repos, on-prem systems).

> **Gotcha**: Cloud Build default pool workers have internet access but NO VPC access. Cloud Build generates an identity (service account) — grant it only necessary permissions (least privilege).

---

## Cloud Deploy

### Overview

- **Managed continuous delivery** service; automates deployment to sequences of environments
- Declarative pipeline definitions; GitOps-friendly; integrated with Cloud Build

### Core Concepts

```
Delivery Pipeline → Stages (targets in order)
  Stage 1: dev-cluster     → auto-promote
  Stage 2: staging-cluster → auto-promote
  Stage 3: prod-cluster    → requires approval
```

- **Delivery Pipeline**: Defines deployment sequence; references targets in order
- **Target**: Destination environment (GKE cluster, Cloud Run service, Anthos target, VM target via skaffold)
- **Release**: Immutable snapshot of application config + container image(s) at a point in time
- **Rollout**: Deployment of a Release to a specific Target; tracks progress and status

### Deployment Strategies

| Strategy | Description |
|----------|-------------|
| **Standard** | Replace all instances at once |
| **Canary** | Route % of traffic to new version; analyze metrics; promote or rollback |
| **Blue/Green** | Full new version deployed; cut over traffic; old version kept for rollback |

- **Canary**: Specify canary % (10%, 25%, 50%); supports metrics-based auto-promotion (Cloud Monitoring integration)
- **Deployment hooks**: Run verification jobs (smoke tests, integration tests) after each phase

### Approvals

- Required approvals before promotion to a target; configured on target definition
- Approvers notified via email; approve/reject in Cloud Deploy UI, CLI, or API
- Full audit trail in Cloud Audit Logs

### Rollbacks

- One-click rollback to any previous successful release
- Cloud Deploy handles traffic shifting and cleanup

> **Exam Tip**: Cloud Deploy is the GCP-native solution for progressive delivery. It integrates Cloud Build (CI) → Cloud Deploy (CD) for a complete CI/CD pipeline.

---

## Artifact Registry

### Overview

- **Universal package manager** for containers and language artifacts; successor to Container Registry
- **Supports**: Docker, Maven, npm, Python (PyPI), Apt, Yum, Go, Helm, Kubeflow Pipelines, generic
- Multi-region and regional repositories; VPC Service Controls support
- **Integrated** with Cloud Build, Cloud Deploy, GKE, Cloud Run

### Key Features

| Feature | Details |
|---------|---------|
| **Vulnerability Scanning** | Auto-scan Docker images on push; powered by Artifact Analysis; CVE detection for OS + language deps |
| **SBOM (Software Bill of Materials)** | Auto-generate SBOM for container images; export to GCS |
| **Remote Repositories** | Proxy and cache upstream registries (Docker Hub, Maven Central, PyPI, npm); avoids rate limiting |
| **Virtual Repositories** | Federate multiple upstream repos behind single endpoint; search across multiple repos |
| **CMEK** | Encrypt repository contents with Cloud KMS |
| **Access Control** | Fine-grained IAM per repository; `roles/artifactregistry.reader`, `.writer`, `.admin` |

### Repository Formats and Use Cases

- **Docker**: Container images for GKE, Cloud Run, Compute Engine
- **Maven/Gradle**: Java artifacts; supports snapshot and release repositories
- **npm**: Node.js packages; private registry for internal packages
- **Python**: Private PyPI server; `pip install --extra-index-url`
- **Helm**: Helm charts for Kubernetes deployments
- **Generic**: Binary files, zips, custom artifacts

> **Exam Tip**: Use Remote Repositories to cache Docker Hub/npm/PyPI to avoid rate limits and ensure supply chain security. Virtual Repositories simplify the client configuration.

> **Gotcha**: Container Registry (`gcr.io`) is deprecated — migrate to Artifact Registry (`REGION-docker.pkg.dev`). Vulnerability scanning in Artifact Registry is more comprehensive than old Container Analysis.

---

## Cloud Source Repositories

- **Private Git repositories** on GCP; fully managed
- Free for up to 5 users, 50 GB storage, 50 GB egress/month
- Sync/mirror from GitHub or Bitbucket (one-way mirror)
- Integrated with Cloud Build triggers
- **Identity-based access**: IAM roles (`roles/source.reader`, `.writer`, `.admin`)
- **Use case**: Teams fully on GCP; existing GitHub/Bitbucket users should prefer native triggers from those platforms

---

## GKE Deployment Strategies

| Strategy | Description | Use Case |
|----------|-------------|---------|
| **Rolling Update** | Gradually replace old pods with new; configurable `maxSurge`/`maxUnavailable` | Default; zero-downtime for stateless apps |
| **Blue/Green** | Run two deployments; switch Service selector | Instant rollback; needs 2× resource capacity |
| **Canary** | Route % of traffic to new version via separate Deployment + Service/Ingress weights | A/B testing, gradual rollout |
| **Recreate** | Delete all old pods before creating new | Stateful apps that can't run two versions simultaneously |

- Use **Cloud Deploy** for automated canary/blue-green with approval gates on GKE
- **Argo Rollouts**: More advanced progressive delivery on GKE (not a GCP native service but common pattern)

---

## Config Management (ACM)

### Anthos Config Management (ACM)

- **GitOps-based config management** for GKE and Anthos fleets
- **Config Sync**: Syncs Kubernetes manifests from Git repo to clusters; supports hierarchical namespaces (namespace inheritance)
- **Policy Controller**: Kubernetes admission controller powered by OPA Gatekeeper; enforce guardrails (e.g., require resource limits, prohibit `latest` tag, enforce label naming)
- **Config Controller**: Manage GCP resources via Kubernetes CRDs (wraps Config Connector); GitOps for infrastructure

### Argo CD Pattern

- **Argo CD**: Popular open-source GitOps controller for GKE; declarative, Git-driven deployments
- Deploy via GKE Marketplace or Helm; not a GCP managed service but widely used
- Cloud Deploy can complement Argo CD for release lifecycle management

---

## Terraform on GCP

### State Management

- **Backend in Cloud Storage**: Store Terraform state in GCS bucket; enables team collaboration; enable versioning on bucket for state history
```hcl
terraform {
  backend "gcs" {
    bucket = "my-tfstate-bucket"
    prefix = "terraform/state"
  }
}
```

### Authentication

| Method | Use Case |
|--------|---------|
| **Application Default Credentials (ADC)** | Local development; `gcloud auth application-default login` |
| **Service Account Key** | CI/CD (legacy); avoid where possible |
| **Workload Identity Federation** | GitHub Actions, GitLab CI, Terraform Cloud; keyless auth to GCP |

### Workload Identity Federation for CI/CD

```hcl
# GitHub Actions → GCP (keyless)
resource "google_iam_workload_identity_pool" "github" { ... }
resource "google_iam_workload_identity_pool_provider" "github" {
  attribute_mapping = {
    "google.subject" = "assertion.sub"
    "attribute.repository" = "assertion.repository"
  }
  oidc { issuer_uri = "https://token.actions.githubusercontent.com" }
}
```

### GCP-Specific Terraform Providers

- `hashicorp/google`: Standard GCP provider; use `google_project_iam_binding` (authoritative) vs `google_project_iam_member` (additive)
- `hashicorp/google-beta`: Beta features/APIs
- **Warning**: `google_project_iam_binding` replaces ALL bindings for a role — can remove existing members unintentionally; prefer `google_project_iam_member` in shared environments

> **Exam Tip**: Use Workload Identity Federation for GitHub Actions/GitLab CI to authenticate to GCP without service account keys. Always store state in GCS with versioning.

---

## Monitoring & Alerting in CI/CD

### Cloud Monitoring Integration

- **SLOs in Cloud Deploy**: Define SLOs (error rate, latency); auto-rollback canary if SLO breached
- **Cloud Monitoring metrics**: Track deployment frequency, change failure rate, MTTR (DORA metrics)
- **Uptime Checks**: HTTP checks from multiple GCP regions; alert on failure

### Notifications & Incident Response

- **Alerting Policies**: Trigger on metric threshold, log-based metrics, uptime check failure
- **Notification Channels**: Email, SMS, PagerDuty, OpsGenie, Slack, Webhook, Pub/Sub
- **Error Reporting**: Auto-aggregate errors from application logs; notify on new error classes
- **Cloud Trace + Cloud Profiler**: Distributed tracing and performance profiling in production

### CI/CD DORA Metrics

| Metric | How to Measure in GCP |
|--------|----------------------|
| Deployment Frequency | Cloud Deploy rollout count over time |
| Lead Time for Changes | Trigger timestamp → rollout complete timestamp |
| Change Failure Rate | Failed rollouts / total rollouts |
| Mean Time to Recover | Time from failure detection (SCC/Monitoring alert) to rollback complete |

---

## CI/CD Architecture Pattern (Complete GCP Pipeline)

```
Developer Push
    ↓
GitHub / Cloud Source Repos
    ↓
Cloud Build Trigger
    ├── Run unit tests
    ├── Build Docker image
    ├── Push to Artifact Registry
    ├── Run vulnerability scan
    └── Create Binary Authorization attestation
    ↓
Cloud Deploy (Release creation)
    ├── Deploy to dev (auto)
    ├── Run smoke tests (deployment hook)
    ├── Deploy to staging (auto-promote)
    ├── Run integration tests
    └── Deploy to production (approval required)
         ├── Canary: 10% traffic
         ├── Monitor SLOs (5 min)
         └── Full rollout or rollback
```

---

## Common Exam Gotchas

1. **Cloud Build default pool** has no VPC access — use Private Pools for builds needing internal network access
2. **Cloud Deploy approvals** are on the Target, not the pipeline — configure approval on the production target
3. **Artifact Registry** replaces Container Registry — `gcr.io` is deprecated
4. **Binary Authorization** requires an attestation for every image before GKE deployment — missing attestors block deployment
5. **`google_project_iam_binding`** is authoritative and destructive for a role — use `google_project_iam_member` in shared environments
6. **ACM Policy Controller** violations block admission to the cluster — test policies in dry-run mode first
7. **Cloud Deploy canary rollbacks** are automatic when SLOs breach — configure SLOs and monitoring properly
8. **Workload Identity Federation** eliminates service account key rotation burden — always prefer over key-based auth for CI/CD systems
