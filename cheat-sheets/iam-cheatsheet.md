# IAM Cheat Sheet
> Google Professional Cloud Architect — 2025 Edition

---

## 1. IAM Resource Hierarchy

```
Organization (root)
│   ├─ Org-level policies apply to ALL resources below
│   ├─ Org Admin role: resourcemanager.organizationAdmin
│   └─ Billing Account linked here or at project level
│
├─► Folder (optional, 1–10 levels deep)
│   ├─ Group projects logically (by team, env, BU)
│   ├─ Policies set here inherit to child folders + projects
│   └─ Folder Admin: resourcemanager.folderAdmin
│
│   ├─► Project
│   │   ├─ IAM policies set here apply to all resources in project
│   │   ├─ Billing tied here
│   │   └─ APIs enabled per project
│   │
│   │   └─► Resource (GCS Bucket, GCE VM, BigQuery Dataset, etc.)
│               └─ Most fine-grained IAM — override project-level
```

### Inheritance Rules
- **IAM policies are ADDITIVE** — you cannot deny access by setting a lower-level policy (unless using Deny policies or IAM Conditions)
- A principal with a role at the **Organization** level has that role for ALL projects under it
- A principal with a role at the **Project** level does NOT have it at org level
- To restrict access below an inherited role → use **IAM Conditions** or **Org Policies**
- **Deny policies** (IAM Deny, not Org Policies) can override allow policies — new in 2022+

---

## 2. Role Types

### Primitive Roles (Legacy — Avoid on Exam Unless Specifically Asked)
| Role | What It Does | Why Avoid |
|---|---|---|
| `roles/owner` | Full control + manage IAM + billing | Overly broad; violates least privilege |
| `roles/editor` | Read + write all resources (no IAM management) | Still too broad for most use cases |
| `roles/viewer` | Read-only all resources | Acceptable for read-only monitoring; prefer predefined |

> **Exam Rule:** If a question asks for "least privilege," eliminate any answer with Owner or Editor roles.

### Predefined Roles
- Google-managed collections of permissions for specific services
- Example: `roles/storage.objectViewer` — can read GCS objects, cannot delete or write
- Names follow pattern: `roles/[service].[Role]`

### Custom Roles
- User-defined collections of permissions
- Scope: **Project** or **Organization** level (not folder level)
- Use when predefined roles are too broad
- Lifecycle: `ALPHA`, `BETA`, `GA`, `DISABLED`
- Cannot include permissions marked as "not supported in custom roles"

---

## 3. Key Predefined Roles Reference

### Compute Engine
| Role | Permissions Summary |
|---|---|
| `roles/compute.admin` | Full GCE control |
| `roles/compute.instanceAdmin.v1` | Create/manage VMs; cannot manage firewall/network |
| `roles/compute.networkAdmin` | Manage VPC, firewall, LB, VPN — NOT instances |
| `roles/compute.securityAdmin` | Manage firewall rules and SSL certs |
| `roles/compute.viewer` | Read-only all GCE resources |
| `roles/compute.osLogin` | SSH without OS-level admin |
| `roles/compute.osAdminLogin` | SSH with sudo |

### Cloud Storage
| Role | Permissions Summary |
|---|---|
| `roles/storage.admin` | Full control of GCS (buckets + objects) |
| `roles/storage.objectAdmin` | Full control of objects; cannot manage bucket IAM |
| `roles/storage.objectCreator` | Upload objects only — cannot read or delete |
| `roles/storage.objectViewer` | Read objects + bucket metadata |
| `roles/storage.legacyBucketReader` | List bucket; do not use for new buckets |

### BigQuery
| Role | Permissions Summary |
|---|---|
| `roles/bigquery.admin` | Full BigQuery control |
| `roles/bigquery.dataOwner` | CRUD on datasets and tables |
| `roles/bigquery.dataEditor` | Read + write data; create tables |
| `roles/bigquery.dataViewer` | Read data and metadata |
| `roles/bigquery.jobUser` | Run jobs (queries) — needs dataViewer on datasets |
| `roles/bigquery.user` | Run jobs + create datasets; broad read access |

> **Exam Tip:** A BI analyst who only runs queries needs `bigquery.jobUser` + `bigquery.dataViewer` on the specific dataset — NOT `bigquery.user`.

### GKE
| Role | Permissions Summary |
|---|---|
| `roles/container.admin` | Full GKE + Kubernetes API |
| `roles/container.clusterAdmin` | Manage clusters; no Kubernetes objects |
| `roles/container.developer` | Access Kubernetes API within clusters |
| `roles/container.viewer` | Read-only on clusters |

### Cloud SQL
| Role | Permissions Summary |
|---|---|
| `roles/cloudsql.admin` | Full Cloud SQL |
| `roles/cloudsql.editor` | Manage instances; cannot set IAM |
| `roles/cloudsql.viewer` | Read-only |
| `roles/cloudsql.instanceUser` | Connect to instances using Cloud SQL Auth Proxy |
| `roles/cloudsql.client` | Connect to instances (use with Cloud SQL Auth Proxy) |

### IAM & Admin
| Role | Permissions Summary |
|---|---|
| `roles/iam.securityAdmin` | Get/set IAM policies on all resources |
| `roles/iam.serviceAccountAdmin` | Create/manage service accounts |
| `roles/iam.serviceAccountUser` | Attach service accounts to resources (act as SA) |
| `roles/iam.serviceAccountTokenCreator` | Create OAuth tokens for a SA (impersonation) |
| `roles/iam.workloadIdentityUser` | Allow K8s SA to impersonate GCP SA |

---

## 4. Service Accounts

### Key Concepts
- A service account is both a **principal** (can be granted roles) and an **identity** (resources can run as it)
- Email format: `[sa-name]@[project-id].iam.gserviceaccount.com`
- Default service accounts are created automatically — **restrict their permissions** (they get Editor by default — remove this!)

### Service Account Best Practices
1. **One SA per application/service** — not shared across workloads
2. **Principle of least privilege** — grant only necessary roles
3. **Disable default SA or restrict permissions** — default SA has Editor role (dangerous)
4. **Prefer Workload Identity** over SA keys for GKE workloads
5. **Avoid SA key files** — they're credentials that can leak; use short-lived tokens instead
6. **Rotate SA keys** if you must use them (90 day max recommended; use `iam.serviceAccountKeyExpiryHours` org policy)
7. **Audit SA usage** with Cloud Audit Logs

### SA Key Types
| Key Type | Description | When to Use |
|---|---|---|
| Google-managed | Rotated automatically by Google | Always preferred — used internally |
| User-managed (JSON key) | Downloaded JSON file; you manage rotation | Last resort for non-GCP external systems |

### Workload Identity Federation (WIF)
- Allow workloads **outside GCP** (GitHub Actions, AWS, Azure, on-prem) to impersonate a GCP SA **without SA key files**
- Works via OIDC or SAML2 token exchange
- **Workload Identity for GKE**: Map K8s SA → GCP SA so pods get GCP credentials automatically
  ```
  K8s ServiceAccount → [Workload Identity binding] → GCP Service Account
  ```
- **WIF for external**: GitHub Actions → GCP SA (no key needed for CI/CD pipelines)

```
# WIF for GitHub Actions — example IAM binding
gcloud iam service-accounts add-iam-policy-binding \
  my-sa@project.iam.gserviceaccount.com \
  --role="roles/iam.workloadIdentityUser" \
  --member="principalSet://iam.googleapis.com/projects/PROJECT_NUM/locations/global/workloadIdentityPools/POOL/attribute.repository/ORG/REPO"
```

---

## 5. IAM Conditions

IAM Conditions allow **attribute-based access control** — grant a role only when certain conditions are met.

### Syntax (CEL — Common Expression Language)
```
# Allow only during business hours
request.time.getHours("America/New_York") >= 9 &&
request.time.getHours("America/New_York") <= 17 &&
request.time.getDayOfWeek("America/New_York") >= 1 &&
request.time.getDayOfWeek("America/New_York") <= 5

# Allow only on specific resources by path
resource.name.startsWith("projects/_/buckets/my-bucket/objects/prod/")

# Allow only from specific IP (use with VPC SC or Access Context Manager)
request.auth.access_levels.exists(x, x == "accessPolicies/POLICY/accessLevels/LEVEL")
```

### Common Condition Use Cases
- **Temporary access**: Grant role with expiry date (`request.time < timestamp("2025-12-31T00:00:00Z")`)
- **Resource path restriction**: Limit access to specific GCS prefix
- **Environment separation**: Allow dev SA to access only dev-prefixed resources
- **Emergency break-glass**: Conditional break-glass role active only within time window

---

## 6. Org Policies vs IAM Policies

| Aspect | IAM Policies | Org Policies |
|---|---|---|
| **Purpose** | Control WHO can do WHAT on which resource | Control WHAT can be done at all (regardless of who) |
| **Granularity** | Per principal (user, SA, group) | Per resource hierarchy node (org, folder, project) |
| **Inheritance** | Additive (parent + child = combined permissions) | Enforced top-down; can be overridden if `inheritFromParent: false` |
| **Example** | "Alice can read GCS objects in project X" | "No public GCS buckets allowed in this org" |
| **Managed by** | IAM Admin / Project Owner | Org Policy Admin |
| **Bypass** | Not bypassable by lower hierarchy | Cannot be overridden by IAM grants |
| **Audit** | Cloud Audit Logs (IAM events) | Org policy violation logs |

### Key Org Policy Constraints (Exam Favorites)
| Constraint | Effect |
|---|---|
| `constraints/iam.disableServiceAccountKeyCreation` | Prevent SA key downloads |
| `constraints/iam.allowedPolicyMemberDomains` | Restrict IAM grants to specific domains |
| `constraints/compute.disableSerialPortAccess` | Block serial port (security hardening) |
| `constraints/compute.vmExternalIpAccess` | Prevent external IPs on VMs |
| `constraints/compute.restrictCloudSQLInstances` | Restrict Cloud SQL to specific regions |
| `constraints/gcp.resourceLocations` | Restrict resource creation to specific regions (data residency) |
| `constraints/storage.uniformBucketLevelAccess` | Enforce uniform bucket IAM (disable legacy ACLs) |
| `constraints/compute.skipDefaultNetworkCreation` | Don't create default VPC in new projects |

---

## 7. VPC Service Controls vs IAM

| Aspect | IAM | VPC Service Controls |
|---|---|---|
| **Protects against** | Unauthorized identity access | Data exfiltration (even by authorized identities) |
| **Mechanism** | Identity-based allow/deny | Network perimeter around services (service perimeter) |
| **Use case** | "Only Alice can access this bucket" | "Data in this project cannot leave to external GCS, even via legitimate API" |
| **Complements** | Works alongside VPC SC | Works alongside IAM |
| **Granularity** | Per identity / resource | Per GCP project / service API |
| **Bypass scenario** | Cannot bypass if VPC SC also restricts | Cannot bypass VPC SC by having IAM permission |
| **When to use** | Always (default security layer) | Sensitive data (PII/PCI/HIPAA), prevent insider threats |

> **Exam Rule:** The question mentions "prevent data exfiltration" or "insider threat" → VPC Service Controls. The question mentions "least privilege access" → IAM predefined/custom roles.

---

## 8. Common Exam IAM Scenarios

### Scenario 1: Cross-Project Access
**Problem**: Cloud Run service in Project A needs to read from BigQuery in Project B.

**Solution**:
1. Cloud Run in Project A uses a dedicated Service Account (SA-A)
2. Grant `roles/bigquery.dataViewer` to SA-A on the **specific dataset** in Project B
3. Grant `roles/bigquery.jobUser` to SA-A on **Project B** (jobs run in the project being queried)

> Granting at dataset level = least privilege. Granting `bigquery.user` on project = too broad.

---

### Scenario 2: Least Privilege for ETL Data Pipeline
**Problem**: Dataflow pipeline reads from GCS (raw bucket), writes to BigQuery (processed dataset).

**Solution**:
```
Dataflow Worker SA:
  ├─ roles/storage.objectViewer  → raw-data GCS bucket (read only)
  ├─ roles/bigquery.dataEditor   → processed-dataset in BigQuery (write)
  ├─ roles/bigquery.jobUser      → Dataflow project (run jobs)
  └─ roles/dataflow.worker       → Dataflow project (worker permissions)
```

---

### Scenario 3: Developer Needs SSH Without Full Admin
**Problem**: Developer needs SSH access to GCE instances without IAM admin rights.

**Solution**:
- Enable **OS Login** on the project (`enable-oslogin=true` metadata)
- Grant developer `roles/compute.osLogin` (regular SSH) or `roles/compute.osAdminLogin` (sudo)
- Remove project-level Editor/Owner from developers

---

### Scenario 4: Federated Identity for External Users
**Problem**: Company uses Azure AD / LDAP. Employees need GCP access without individual Google accounts.

**Solution Options**:
- **Google Cloud Identity** with **SAML 2.0 SSO** (sync from Azure AD / Okta via GCDS)
- **Workforce Identity Federation** (newer) — federate from any OIDC/SAML provider, no Google account required
- Grant IAM roles to groups (not individuals) — sync group membership from IdP

---

### Scenario 5: Temporary Elevated Access (Break-Glass)
**Problem**: On-call engineer needs temporary access to production during incidents.

**Solution**:
- **IAM Conditions with time-based expiry**
- Or: Privileged Access Manager (PAM) — request/approve temporary role grants with audit log
- Role granted with `request.time < timestamp("EXPIRY")` condition — auto-expires

---

### Scenario 6: Audit Who Accessed What
**Problem**: Compliance requires audit trail of all BigQuery data access.

**Solution**:
- Enable **Cloud Audit Logs** (Data Access audit logs — not enabled by default!)
  - `DATA_READ`, `DATA_WRITE` audit logs for BigQuery
- Export to **Cloud Logging** → **BigQuery** (audit dataset) or **GCS** for long-term retention
- Use **Access Transparency Logs** for Google admin access to your data
