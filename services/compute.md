# GCP Compute Services — Exam Reference (2025)

---

## Compute Engine

### Machine Families

| Family | Purpose | Key Characteristics |
|--------|---------|---------------------|
| **E2** | Cost-optimized general purpose | Shared-core available; no local SSD; up to 32 vCPUs, 128 GB RAM |
| **N2** | Balanced general purpose | Intel Cascade Lake; up to 128 vCPUs, 864 GB RAM; local SSD supported |
| **N2D** | AMD general purpose | AMD EPYC; up to 224 vCPUs; cost ~10% lower than N2 |
| **N4** | Next-gen general purpose | Intel Emerald Rapids; higher performance/cost ratio |
| **C2** | Compute-optimized | Intel Cascade Lake; highest sustained clock; up to 60 vCPUs |
| **C2D** | Compute-optimized AMD | AMD EPYC Milan; up to 112 vCPUs |
| **M1/M2/M3** | Memory-optimized | M3: up to 30 TB RAM; SAP HANA certified |
| **A2/A3** | Accelerator-optimized | NVIDIA A100/H100 GPUs; ML training workloads |
| **T2D/T2A** | Scale-out optimized | AMD/Arm; best price-performance for scale-out |

### Pricing Models

- **On-demand**: Pay per second (minimum 1 minute); no commitment
- **Sustained Use Discounts (SUDs)**: Automatic discounts up to **30%** for N1/N2/N2D/E2; applied when instance runs >25% of the month. *NOT available for A2, C2D, or committed use*
- **Committed Use Discounts (CUDs)**: 1-year (up to 37% off) or 3-year (up to 55% off); resource-based or spend-based; applies to vCPU, RAM, GPU
- **Preemptible VMs**: Up to 80% discount; max 24-hour runtime; 30-second eviction notice; good for batch/fault-tolerant workloads
- **Spot VMs**: Successor to preemptible; no maximum runtime guarantee; same pricing as preemptible; preferred for new workloads
- **Custom Machine Types**: Specify exact vCPU/RAM ratios; premium of ~5% over predefined; useful when standard types over-provision

### Key Concepts

- **Sole-Tenant Nodes**: Physical servers dedicated to a single customer; required for bring-your-own-license (BYOL) Windows/SQL; compliance isolation
- **Instance Templates**: Immutable definitions for creating VMs; required for MIGs
- **Managed Instance Groups (MIGs)**: Autoscaling, autohealing, rolling updates; zonal or regional
- **Unmanaged Instance Groups**: No autoscaling/autohealing; legacy; avoid for new workloads
- **Live Migration**: Default for maintenance events; VMs migrate without restart
- **Shielded VMs**: Secure Boot + vTPM + Integrity Monitoring; protects against rootkits
- **Confidential VMs**: Memory encryption at rest and in use using AMD SEV

> **Exam Tip**: CUDs do NOT stack with SUDs on the same resource. CUDs give a flat discount; SUDs are automatic. Choose CUD when usage is predictable.

> **Gotcha**: Preemptible/Spot VMs cannot use sustained use discounts. Regional MIGs spread across 3 zones automatically.

---

## Google Kubernetes Engine (GKE)

### Standard vs Autopilot

| Feature | Standard | Autopilot |
|---------|---------|-----------|
| Node management | Customer manages nodes | Google manages nodes |
| Pricing | Per-node (VM cost) | Per-pod (vCPU/memory/storage) |
| Node pools | Full control | Abstract; managed |
| Cluster autoscaler | Optional | Always on |
| DaemonSets | Supported | Not supported |
| Custom node config | Full | Limited |
| Best for | Fine-grained control, special hardware | Simplicity, cost efficiency for variable load |

### Key GKE Features

- **Node Pools**: Groups of nodes with same config; upgrade independently; can mix machine types
- **Cluster Autoscaler**: Adds/removes nodes based on pending pods; set min/max per pool
- **Horizontal Pod Autoscaler (HPA)**: Scales pod replicas based on CPU/custom metrics
- **Vertical Pod Autoscaler (VPA)**: Adjusts pod resource requests/limits; don't use HPA + VPA CPU simultaneously
- **GKE Sandbox (gVisor)**: Kernel isolation via gVisor; for multi-tenant/untrusted workloads; adds ~20% overhead
- **Workload Identity**: Recommended way to authenticate pods to GCP APIs; maps K8s SA → GCP SA; eliminates key files in containers
- **Private Clusters**: Nodes have no public IPs; master accessible via private endpoint; requires Cloud NAT for egress
- **Binary Authorization**: Require signed container images before deployment
- **Config Connector**: Manage GCP resources via Kubernetes CRDs

### Release Channels

| Channel | Description | Use Case |
|---------|-------------|---------|
| Rapid | Latest K8s version quickly | Dev/test, early adopters |
| Regular | Balanced (default) | General production |
| Stable | Older, well-tested | Risk-averse production |
| No Channel | Manual version management | Special compliance requirements |

> **Exam Tip**: Workload Identity is always preferred over service account key files for pod-to-GCP authentication. Autopilot billing is per-pod, so idle clusters cost almost nothing.

> **Gotcha**: GKE Autopilot does NOT support privileged containers, DaemonSets, or host namespace access.

---

## Cloud Run

### Services vs Jobs

| | Cloud Run Services | Cloud Run Jobs |
|--|-------------------|----------------|
| Purpose | HTTP request handling | Batch/task execution |
| Trigger | HTTP/gRPC requests, Pub/Sub, events | On-demand, scheduled, Pub/Sub |
| Concurrency | Up to 1000 concurrent requests/instance | Parallelism via task count |
| Scaling | 0 → N (can set min instances) | N parallel tasks |

### Key Configuration

- **Concurrency**: Requests handled per instance (default: 80, max: 1000); higher = better CPU utilization for I/O-bound services
- **Min Instances**: Avoid cold starts; incur idle costs; set for latency-sensitive services
- **Max Instances**: Prevent runaway scaling / cost spikes; default 100
- **CPU Allocation**:
  - *Request-based* (default): CPU only allocated during request handling
  - *Always-on*: CPU allocated even between requests; required for background tasks/timers
- **Timeout**: Max 3600 seconds (1 hour) per request
- **VPC Connector / Direct VPC Egress**: Connect Cloud Run to VPC resources (e.g., Cloud SQL, Memorystore)
- **Cloud Run for Anthos**: Deploy Cloud Run workloads on GKE on-premises or other clouds

> **Exam Tip**: For websockets or streaming responses, set concurrency to 1. Use "Always-on CPU" for services with background processing or cron-like tasks within the service.

> **Gotcha**: Cloud Run containers must listen on the port defined by `$PORT` env var (default 8080). Services with min-instances > 0 are billed for idle time.

---

## App Engine

### Standard vs Flexible

| Feature | Standard | Flexible |
|---------|---------|---------|
| Language runtimes | Python, Java, Go, Node.js, PHP, Ruby (specific versions) | Any via custom Docker |
| Scaling | Automatic, basic, manual | Automatic, manual |
| Scale to zero | Yes | No (min 1 instance) |
| Startup time | Seconds | Minutes |
| Pricing | Per instance-hour (F/B classes) | Per vCPU/memory/hour |
| Local disk | None (read-only filesystem) | Ephemeral writable disk |
| SSH access | No | Yes |
| Background threads | No | Yes |

### Key Features

- **Traffic Splitting**: Split traffic between versions by IP, cookie, or random; use for A/B testing and canary deployments
- **Versions**: Each deploy creates a new version; can roll back instantly; multiple versions can receive traffic simultaneously
- **Services (Modules)**: Microservices within an App Engine app; each has its own versions and scaling
- **Cron Jobs**: app.yaml-based scheduled tasks; routed to specific handlers
- **Dispatch Rules**: Route requests to specific services based on URL patterns

> **Exam Tip**: App Engine Standard is best for web apps needing fast scale-to-zero. App Engine Flexible is essentially a managed VM and closer to Cloud Run. Prefer Cloud Run for new workloads.

---

## Cloud Functions

### Gen1 vs Gen2

| Feature | Gen1 | Gen2 |
|---------|------|------|
| Max timeout | 9 min (HTTP), 10 min (background) | 60 min (HTTP/event) |
| Max concurrency | 1 request/instance | Up to 1000/instance (like Cloud Run) |
| Min instances | No | Yes |
| Underlying | App Engine Flex (custom) | Cloud Run + Eventarc |
| Traffic splitting | No | Yes |
| Max memory | 8 GB | 32 GB |
| Max vCPUs | 4 | 8 |

### Triggers

- **HTTP**: HTTPS endpoint; synchronous
- **Pub/Sub**: Asynchronous; push subscription
- **Cloud Storage**: Object finalize/delete/archive/metadata update
- **Firestore**: Document create/update/delete/write (Gen1 only native; Gen2 via Eventarc)
- **Eventarc**: Gen2 preferred; supports 90+ GCP event sources

> **Exam Tip**: Gen2 is the recommended choice for all new functions. Gen2 supports concurrency — important for cost optimization vs Gen1 (one request = one instance).

> **Gotcha**: Cold starts are more severe in Gen1. Use minimum instances to mitigate. Cloud Functions Gen2 is essentially Cloud Run with a function-as-a-service interface.

---

## Bare Metal Solution

- Physical servers in Google-managed colocation facilities
- Connected to GCP via low-latency dedicated network
- **Use cases**: Oracle Database (licensing), legacy apps requiring physical hardware, FIPS compliance
- Customer manages OS and software; Google manages hardware/power/cooling
- Not subject to hypervisor overhead

---

## Compute Decision Matrix

| Use Case | Recommended Service |
|----------|-------------------|
| Full OS control, custom kernel, BYOL licensing | Compute Engine |
| Containerized microservices, fine-grained control | GKE Standard |
| Containers without infrastructure management | GKE Autopilot or Cloud Run |
| Stateless HTTP microservices, event-driven | Cloud Run |
| Simple web apps, scale-to-zero required | Cloud Functions or Cloud Run |
| Legacy web apps needing managed platform | App Engine |
| Batch/embarrassingly parallel jobs | Compute Engine MIG or Cloud Run Jobs |
| ML training with GPUs | Compute Engine A2/A3 or Vertex AI |
| Oracle / SAP HANA / legacy bare metal | Bare Metal Solution |
| Multi-cloud / on-premises containers | GKE on Anthos / Cloud Run for Anthos |
| Short burst tasks < 9 min | Cloud Functions |

---

## Common Exam Gotchas

1. **SUDs + CUDs don't stack** — CUD takes priority
2. **Spot VMs** replace preemptible for new workloads; same price, no 24h limit but still preemptible
3. **GKE Autopilot** charges per pod, not per node — idle clusters are nearly free
4. **Cloud Run min-instances** costs money even with zero traffic
5. **App Engine Flexible** always has at least 1 running instance — no scale-to-zero
6. **Custom machine types** allow odd CPU/RAM combos but add ~5% premium
7. **Sole-tenant nodes** are required for certain BYOL Windows Server scenarios
8. **E2 machines** do NOT support local SSD or GPUs
