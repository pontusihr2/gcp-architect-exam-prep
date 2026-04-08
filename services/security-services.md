# GCP Security Services — Exam Reference (2025)

---

## Cloud IAM

### Role Types

| Type | Description | Example |
|------|-------------|---------|
| **Primitive (Basic)** | Broad legacy roles; avoid for production | `roles/owner`, `roles/editor`, `roles/viewer` |
| **Predefined** | Fine-grained, service-specific, Google-maintained | `roles/storage.objectViewer`, `roles/compute.instanceAdmin` |
| **Custom** | User-defined; combine specific permissions | Useful when predefined grants too much access |

### IAM Concepts

- **Policy**: Binds members (principals) to roles on a resource; evaluated top-down in resource hierarchy (Org → Folder → Project → Resource)
- **Resource hierarchy inheritance**: Child resources inherit parent policies; you cannot revoke inherited policies at child level (only add)
- **Principals**: Google Account, service account, Google Group, Workspace domain, `allAuthenticatedUsers`, `allUsers`
- **Conditions**: Attribute-based access control on IAM bindings; conditions on resource name, request time, resource type, IP address

### Deny Policies (2023+)

- **IAM Deny**: Explicitly deny permissions regardless of allow policies; evaluated before allow policies
- Use to enforce organization-wide restrictions (e.g., deny all users ability to disable audit logging)
- `roles/iam.denyAdmin` required to manage deny policies

### Service Accounts

- Identity for workloads (VMs, Cloud Run, GKE pods, etc.)
- **Best practices**: Least privilege; use Workload Identity Federation (not keys); no `roles/owner` on service accounts
- **Workload Identity Federation**: Allow external workloads (GitHub Actions, AWS, Azure AD) to impersonate GCP service accounts without key files
- **Service Account Impersonation**: Principal A can impersonate SA B; useful for cross-project access; requires `roles/iam.serviceAccountTokenCreator`

> **Exam Tip**: Always prefer predefined roles over primitive roles. Use IAM Conditions for time-limited access or resource-specific restrictions. Deny policies override allows.

> **Gotcha**: IAM changes can take up to 60 seconds to propagate globally. `allAuthenticatedUsers` means any Google-authenticated user in the world — not just your org.

---

## Cloud KMS (Key Management Service)

### Hierarchy

```
Key Ring (regional/global)
  └── CryptoKey (purpose: ENCRYPT_DECRYPT, SIGN_VERIFY, MAC)
        └── CryptoKeyVersion (actual key material)
```

- **Key Rings**: Container for CryptoKeys; location is permanent; cannot be deleted
- **CryptoKeys**: Cannot be deleted (only disabled/destroyed); versions can be destroyed after 24-hour delay
- **Key Rotation**: Manual or automatic (rotation period); new version created; previous versions still usable to decrypt existing data
- **Key Locations**: Regional, multi-regional (`us`, `europe`, `global`); choose to match data location for latency and compliance

### Key Types

| Type | Use Case |
|------|---------|
| **CMEK (Customer-Managed Encryption Keys)** | Encrypt GCP service data (BigQuery, Cloud SQL, GCS, PD, etc.) |
| **Cloud HSM** | Keys stored in FIPS 140-2 Level 3 HSMs; for compliance requirements |
| **Cloud EKM (External Key Manager)** | Keys stored outside Google (Thales, Fortanix); Google never has key material |
| **CSEK (Customer-Supplied Encryption Keys)** | Customer provides key material with each API request (Cloud Storage, Compute Engine) |

- **CMEK Integration**: Supported by 40+ GCP services; if KMS key is disabled/destroyed, data becomes inaccessible
- **Envelope Encryption**: Data encrypted with Data Encryption Key (DEK); DEK encrypted with Key Encryption Key (KEK) in KMS

> **Exam Tip**: CMEK gives you control to revoke data access by disabling the KMS key. EKM keeps keys outside Google — used for highest compliance requirements.

> **Gotcha**: CryptoKey versions cannot be immediately destroyed — 24-hour scheduled destruction period. Key rings cannot be deleted even if empty.

---

## Secret Manager

- **Centralized store** for API keys, passwords, certificates, and other sensitive data
- **Versions**: Each update creates a new version; previous versions remain accessible unless disabled/destroyed
- **Replication**:
  - *Automatic*: Google chooses multi-region locations
  - *User-managed*: Specify regions explicitly (for data residency compliance)
- **Access**: `roles/secretmanager.secretAccessor` to read latest/specific version; audit all access via Cloud Audit Logs
- **Rotation**: Built-in rotation support via Pub/Sub notifications to Cloud Functions/Cloud Run
- **CMEK**: Encrypt secret values with Cloud KMS keys
- **Secret Manager Add-on for GKE**: Mount secrets as Kubernetes secrets or environment variables directly in pods

> **Exam Tip**: Secret Manager is the correct answer for storing application secrets, not environment variables in Cloud Run or hard-coded in code. Integrate with GKE using the Secrets Store CSI Driver.

---

## Security Command Center (SCC)

### Tiers

| Feature | Standard (Free) | Premium | Enterprise |
|---------|----------------|---------|-----------|
| Asset inventory | Yes | Yes | Yes |
| Security health analytics | Basic | Full | Full |
| Web Security Scanner | Limited | Full | Full |
| Event Threat Detection | No | Yes | Yes |
| Container Threat Detection | No | Yes | Yes |
| Virtual Machine Threat Detection | No | Yes | Yes |
| Compliance reports | No | Yes | Yes |
| Posture management | No | Yes | Yes |

- **Findings**: Security issues, vulnerabilities, threats discovered by SCC detectors
- **Assets**: Inventory of all GCP resources (VMs, GCS buckets, IAM policies, etc.)
- **Sources**: SCC Standard, Event Threat Detection, Web Security Scanner, Partner integrations
- **Notifications**: Export findings to Pub/Sub → Cloud Functions → SIEM (Chronicle, Splunk)
- **Posture Management**: Define desired security state; detect deviations

---

## Cloud Armor

- WAF and DDoS protection for Global External HTTP(S) Load Balancers
- **Security Policies**: Ordered rules (priority 0–2,147,483,646); default rule at priority 2,147,483,647
- **Rule actions**: `allow`, `deny(403/404/429/502)`, `redirect`, `throttle`, `rate_based_ban`
- **Match conditions**: IP/CIDR, geo-match (`expression: origin.region_code == "CN"`), CEL expressions
- **WAF Preconfigured Rules**: ModSecurity CRS rules for OWASP Top 10; enable with `evaluatePreconfiguredExpr()`
- **Adaptive Protection**: ML-based real-time DDoS detection and mitigation rule suggestions
- **Rate Limiting**: Throttle per-IP or per-region request rates; return 429 on excess
- **Bot Management**: reCAPTCHA Enterprise integration; challenge bots, redirect suspicious traffic

---

## VPC Service Controls

- Establish **service perimeters** to prevent data exfiltration from GCP services
- Perimeter restricts which GCP services can be accessed from outside and which external identities can access inside
- **Access Levels**: Conditions for crossing perimeter (device trust, IP range, identity); defined in Access Context Manager
- **Ingress/Egress Rules**: Allow specific cross-perimeter flows (e.g., allow Cloud Build in project A to access GCS in project B)
- **Dry-Run Mode**: Log violations without enforcing; **critical** — always run dry-run before enforcing
- **Protected Services**: Cloud Storage, BigQuery, Pub/Sub, Cloud KMS, Vertex AI, and 100+ more
- Requires Uniform Bucket-Level Access on Cloud Storage buckets in perimeter

---

## Certificate Authority Service (CAS)

- **Managed private CA** for issuing TLS/mTLS certificates at scale
- Create private CA hierarchies (root CA + subordinate CAs) without managing PKI infrastructure
- Integrate with Certificate Manager for automated certificate provisioning on GCP LBs and services
- Billing: Per CA, per certificate issued

---

## BeyondCorp Enterprise / Identity-Aware Proxy (IAP)

- **Zero-trust access model**: Access based on identity + device context, NOT network location
- **IAP (Identity-Aware Proxy)**: Protects web apps and VMs; validates Google identity before allowing access; eliminates need for VPN for web apps
- **Context-aware access**: Add conditions based on device posture (cert-based, OS version, patch level) from Endpoint Verification
- **TCP Tunneling**: Use IAP to tunnel SSH/RDP traffic without bastion hosts or public IPs on VMs
- Requires App Engine, Cloud Run, or GCE backends behind an HTTPS Load Balancer

> **Exam Tip**: IAP is the recommended way to secure internal web apps for remote workers without a VPN. Use IAP TCP tunneling for SSH to VMs with no public IP.

---

## Cloud DLP (Data Loss Prevention)

- **Inspect**, **de-identify**, and **classify** sensitive data (PII, PHI, PCI, credentials)
- **Info Types**: SSN, credit cards, email addresses, medical record numbers; 100+ built-in; custom info types
- **Transformations**: Redaction, masking, tokenization, bucketing, date shifting, format-preserving encryption
- **Scanning targets**: BigQuery tables, Cloud Storage, Datastore/Firestore, images
- **Streaming inspection**: Inspect content inline (in data pipelines)
- Integration with Data Catalog for automatic data classification

---

## Audit Logging

### Log Types

| Log Type | What it Captures | Default |
|----------|-----------------|---------|
| **Admin Activity** | Resource configuration changes (create/delete/modify) | Always on; cannot disable |
| **Data Access** | Data read/write operations on resources | Off by default (can be costly); enable for sensitive data |
| **System Event** | GCP-generated events (live migration, maintenance) | Always on; cannot disable |
| **Policy Denied** | Denied requests due to security policy | Always on |

- Audit logs written to **Cloud Logging**; export to Cloud Storage/BigQuery/Pub/Sub via Log Sinks
- **Data Access audit logs** should be enabled for Cloud Storage, BigQuery, KMS in regulated environments
- **Log retention**: Admin Activity 400 days; Data Access 30 days in Logging; extend via export

---

## Binary Authorization

- **Deploy-time security policy** for container images on GKE, Cloud Run, GKE on Anthos
- Requires container images to be signed by trusted **attestors** before deployment
- **Attestors**: Entities that vouch for images (e.g., CI/CD pipeline, vulnerability scanner)
- **Attestations**: Cryptographic signatures stored in Artifact Analysis (Container Analysis)
- **Policies**: Require attestations for specific attestors; define what happens on policy violation (block, warn, allow with exemption)
- **Dry-run mode**: Log violations without blocking; transition to enforcement gradually
- Integration with Cloud Build: Auto-attest after successful build + scan

---

## Artifact Registry Vulnerability Scanning

- On-push scanning for Docker images stored in Artifact Registry
- Powered by **Container Analysis / Artifact Analysis** API
- Detects OS package vulnerabilities (CVEs) and language package vulnerabilities (Java, Python, Go, Node.js)
- SBOM (Software Bill of Materials) generation for supply chain security
- Integration with Binary Authorization: Automatically block deployment of images with critical vulnerabilities

---

## Common Exam Gotchas

1. **IAM deny policies** override allow policies — use carefully; can accidentally lock out users
2. **KMS keys cannot be deleted** — only disabled or scheduled for destruction (24h delay)
3. **VPC Service Controls dry-run mode** — ALWAYS enable dry-run before enforcing to avoid breaking access
4. **SCC Premium** is required for Event Threat Detection and compliance reports — Standard is free but limited
5. **IAP does NOT replace firewall rules** — it adds an authentication layer; firewall rules must also allow IAP health checks
6. **Secret Manager versions** persist until explicitly destroyed — clean up old versions to reduce cost and confusion
7. **Cloud Armor** only protects external HTTP(S) LBs — not TCP/UDP LBs or internal LBs
8. **Workload Identity** for GKE pods is preferred over mounting service account keys as secrets
9. **Audit logs for Data Access** are off by default — a common compliance finding
10. **CMEK** protects data at rest in GCP services; if you delete the KMS key, your data is permanently inaccessible
