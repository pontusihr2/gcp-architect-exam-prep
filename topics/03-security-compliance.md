# Domain 3: Designing for Security and Compliance

> **Exam Weight: ~18%** — Expect 9–12 questions. Security is deeply integrated across all domains; many questions combine security with architecture.

---

## 3.1 Domain Overview

This domain covers how to design, implement, and enforce security controls across the entire GCP stack. It spans identity, network, data, and application security, as well as regulatory compliance frameworks.

**Key skills tested:**
- Implementing IAM least privilege and separation of duties
- Designing network security with VPC Service Controls and Cloud Armor
- Selecting and implementing encryption strategies (CMEK, CSEK, Google-managed)
- Protecting secrets and certificates
- Implementing Zero Trust / BeyondCorp
- Meeting compliance requirements (GDPR, HIPAA, PCI DSS)
- Using Security Command Center and audit logging

---

## 3.2 IAM Security Best Practices

### Principle of Least Privilege (PoLP)
- Grant only the **minimum permissions** required for a task
- Prefer **predefined roles** over basic roles (Owner/Editor/Viewer)
- Use **custom roles** when predefined roles grant too many permissions
- Apply **IAM Conditions** to restrict access by time, resource tag, or resource type
- Use **IAM Deny policies** to create absolute restrictions that cannot be overridden by allow policies

### Separation of Duties (SoD)
- No single identity should have the ability to perform a sensitive end-to-end operation alone
- Example: The identity that deploys code should NOT also have access to production data
- Implement via:
  - Separate service accounts per environment (dev-sa, staging-sa, prod-sa)
  - Separate folders/projects for dev/staging/prod with different IAM bindings
  - Require multiple approvals for sensitive operations (JIRA/Change Management integration)

### Privileged Access Management
- Use **IAM Conditions with time-bound access** for break-glass scenarios
- Implement **Access Approval** to require human approval before Google Support can access your data
- Use **Access Transparency** logs to see when Google accesses your data
- Enable **VPC Service Controls** to prevent data exfiltration even by authorized identities

### Service Account Security
```
DO:
  ✅ Create dedicated service accounts per application/service
  ✅ Grant minimum required roles
  ✅ Use Workload Identity Federation instead of keys
  ✅ Audit service account usage with Cloud Audit Logs
  ✅ Disable unused service accounts

DON'T:
  ❌ Use the Compute Engine default service account (has Editor role)
  ❌ Download and store service account key files
  ❌ Share service accounts between different applications
  ❌ Grant service accounts Editor or Owner roles
```

---

## 3.3 VPC Service Controls

### What VPC Service Controls Provide
- Creates a **security perimeter** around GCP resources (BigQuery, Cloud Storage, etc.)
- Prevents data exfiltration even from legitimate Google accounts
- Controls resource access based on: identity + network context + device context
- Works **independently of IAM** — adds an additional layer of control

### VPC Service Controls Architecture
```
Outside Perimeter (any GCP project)
    ↓ BLOCKED
Security Perimeter
├── Project A (BigQuery)
│   └── Dataset: sensitive-data  ← Only accessible from inside perimeter
├── Project B (Cloud Storage)
│   └── Bucket: confidential-docs ← Only accessible from inside perimeter
└── Access Policy
    ├── Access Level: corp-network (IP range 192.168.0.0/16)
    ├── Access Level: managed-device (device cert required)
    └── Ingress/Egress rules
```

### Access Levels
- **IP-based:** Restrict to corporate IP ranges or VPC networks
- **Device-based:** Require managed devices (Certificate Authority-issued certs)
- **Identity-based:** Restrict to specific users or service accounts

### Ingress and Egress Rules (2025 Exam Focus)
- **Ingress rules:** Allow specific sources (external projects, networks) to access resources inside the perimeter
- **Egress rules:** Allow resources inside the perimeter to access specific external destinations

> **Exam Tip:** VPC Service Controls blocks calls **even from the GCP Console** if you're outside the perimeter. This is a common trap — test this in staging first.

### Common Exam Scenario
**Q:** "A regulated company needs to ensure their BigQuery data cannot be accessed from unauthorized Google accounts, even if IAM is misconfigured..."
**A:** Implement VPC Service Controls perimeter around the BigQuery project

---

## 3.4 Private Google Access

### Types of Private Access
| Feature | What It Does | Use Case |
|---|---|---|
| **Private Google Access** | Allows VMs without public IPs to reach Google APIs | VMs in private subnets |
| **Private Service Connect** | Creates private endpoint for Google APIs in your VPC | Zero-trust API access |
| **Private Google Access for on-premises** | Extends Private Google Access over Cloud VPN/Interconnect | Hybrid on-premises access |
| **Serverless VPC Access** | Allows Cloud Run/Functions to access VPC resources | Serverless to private resources |

### Configuration
```bash
# Enable Private Google Access on a subnet
gcloud compute networks subnets update my-subnet \
  --region us-central1 \
  --enable-private-ip-google-access

# Create a Private Service Connect endpoint
gcloud compute addresses create google-apis-endpoint \
  --global \
  --purpose PRIVATE_SERVICE_CONNECT \
  --addresses 10.0.0.100 \
  --network my-vpc
```

---

## 3.5 Cloud Armor

### What Cloud Armor Does
- **Web Application Firewall (WAF)** and **DDoS protection** for Google Cloud Load Balancers
- Inspect HTTP/HTTPS traffic; allow/deny based on rules
- Apply pre-configured rule sets (OWASP Top 10, SQL injection, XSS)
- Rate limiting to prevent abuse

### Cloud Armor Security Policy Types
| Type | Capacity | Features |
|---|---|---|
| **Standard** | Rule count limit | Basic allow/deny, geo-restriction |
| **Plus (Managed Protection)** | Higher limits | Adaptive protection (ML-based DDoS), threat intelligence |

### Common Cloud Armor Rules
```json
{
  "rules": [
    {
      "priority": 1000,
      "action": "allow",
      "match": {
        "versionedExpr": "SRC_IPS_V1",
        "config": {"srcIpRanges": ["203.0.113.0/24"]}
      },
      "description": "Allow corporate office IPs"
    },
    {
      "priority": 2000,
      "action": "deny(403)",
      "match": {
        "expr": {"expression": "origin.region_code == 'CN' || origin.region_code == 'RU'"}
      },
      "description": "Geo-block specific countries"
    },
    {
      "priority": 3000,
      "action": "deny(403)",
      "match": {
        "expr": {"expression": "evaluatePreconfiguredExpr('sqli-v33-stable')"}
      },
      "description": "Block SQL injection"
    }
  ]
}
```

### Adaptive Protection
- Uses **machine learning** to detect and alert on L7 DDoS attacks
- Automatically suggests rules to mitigate ongoing attacks
- Available in Cloud Armor Plus tier

> **Exam Tip:** Cloud Armor only works with **HTTP(S) Load Balancer** (global and regional). It does NOT work with Network Load Balancer (pass-through).

---

## 3.6 Encryption Strategies

### Encryption Hierarchy
```
Google Encryption Stack:
  Data → Encrypted at block level
    ↓
  Data Encryption Key (DEK) — per-chunk, per-file key
    ↓
  Key Encryption Key (KEK) — wraps DEKs
    ↓
  Key Management (KMS) — manages KEKs
```

### Encryption Option Comparison
| Option | Key Management | Control Level | Use Case |
|---|---|---|---|
| **Google-managed keys** | Google manages everything | None | Standard, low-risk data |
| **CMEK** (Customer-Managed Encryption Keys) | You manage keys in Cloud KMS | High | Regulated data, audit requirements |
| **CSEK** (Customer-Supplied Encryption Keys) | You supply the key material | Highest | Most sensitive data; you bear full responsibility |
| **Cloud HSM** | Keys in FIPS 140-2 Level 3 HSM | High + hardware assurance | Compliance requiring HSM |
| **Cloud EKM** (External Key Manager) | Keys in your external KMS (Thales, Fortanix) | Highest | Data sovereignty; GCP never sees your keys |

### CMEK Configuration
```bash
# Create a key ring and key
gcloud kms keyrings create my-keyring --location us-central1
gcloud kms keys create my-key \
  --location us-central1 \
  --keyring my-keyring \
  --purpose encryption

# Grant BigQuery SA permission to use the key
gcloud kms keys add-iam-policy-binding my-key \
  --location us-central1 \
  --keyring my-keyring \
  --member serviceAccount:bq-project-id@bigquery-encryption.iam.gserviceaccount.com \
  --role roles/cloudkms.cryptoKeyEncrypterDecrypter
```

### Key Rotation
- **Automatic rotation:** Cloud KMS can rotate keys on a schedule (e.g., every 90 days)
- Rotating a key creates a new **primary version** for encryption; old versions still decrypt existing data
- **Key disable/destroy:** Disabling makes data encrypted with that key inaccessible; destroying is permanent

> **Exam Trap:** With CSEK, **you must store the key** — if you lose it, your data is permanently inaccessible. Google cannot recover it.

---

## 3.7 Secret Manager

### What Secret Manager Provides
- Centralized, secure storage for API keys, passwords, certificates, connection strings
- Versioned secrets with automatic replication
- Fine-grained IAM access control per secret
- Audit logging for every access event
- Automatic rotation with Cloud Functions integration

### Secret Manager Usage Pattern
```python
from google.cloud import secretmanager

def get_secret(project_id: str, secret_id: str, version: str = "latest") -> str:
    client = secretmanager.SecretManagerServiceClient()
    name = f"projects/{project_id}/secrets/{secret_id}/versions/{version}"
    response = client.access_secret_version(request={"name": name})
    return response.payload.data.decode("UTF-8")

# Usage:
db_password = get_secret("my-project", "db-password")
```

### Secret Rotation
```
Cloud Scheduler (every 30 days)
    ↓
Pub/Sub Topic: secret-rotation
    ↓
Cloud Functions: rotate-db-password
    ↓
1. Generate new password
2. Update in database
3. Add new version to Secret Manager
4. Disable old version
5. Notify on-call team via PagerDuty
```

> **Exam Tip:** Always use Secret Manager (or HashiCorp Vault) instead of storing secrets in environment variables, config files, or Kubernetes ConfigMaps.

---

## 3.8 Certificate Authority Service

### What CAS Provides
- Managed **private Certificate Authority (CA)** service
- Issue TLS certificates for internal services, mTLS, VPN
- Integrate with Certificate Manager for automated rotation
- Supports ECDSA and RSA key types
- DevOps CA for short-lived certificates (workload identity)

### Certificate Manager (2025)
- Manages **public TLS certificates** for Cloud Load Balancers
- Supports **Google-managed certificates** (auto-provisioned, auto-renewed)
- Supports **bring-your-own certificates** (uploaded to Certificate Manager)

---

## 3.9 Binary Authorization

### What It Does
- **Policy enforcement for container images** — only signed, approved images can be deployed
- Prevents unauthorized or unscanned images from running in GKE or Cloud Run
- Creates a software supply chain security gate

### Binary Authorization Flow
```
Developer pushes code
    ↓
Cloud Build pipeline
    ├── Run tests → PASS
    ├── Vulnerability scan (Container Analysis) → PASS
    └── Sign image with Cloud KMS key (creates Attestation)
            ↓
        Artifact Registry (signed image)
            ↓
        GKE Deployment attempted
            ↓
        Binary Authorization Policy check
        ├── "Require attestation from vulnerability-scan-signer"
        └── Attestation found? → ALLOW deployment
```

### Binary Authorization Policy Modes
| Mode | Behavior |
|---|---|
| **Disabled** | No enforcement |
| **Dry run** | Log violations but allow deployment |
| **Enforce** | Block deployments that violate policy |

---

## 3.10 Artifact Registry Security

### Security Features
- **Vulnerability scanning:** Automatically scan container images and packages for CVEs
- **SBOM (Software Bill of Materials):** Generate and store SBOM for auditing
- **Remote repositories:** Proxy and cache public repositories (npm, PyPI, Maven, Docker Hub)
- **VPC Service Controls integration:** Lock down registry access to inside the perimeter
- **CMEK support:** Encrypt registry storage with customer-managed keys

### Artifact Registry IAM Roles
| Role | Permissions |
|---|---|
| `roles/artifactregistry.reader` | Pull images/packages |
| `roles/artifactregistry.writer` | Push images/packages |
| `roles/artifactregistry.repoAdmin` | Manage repository settings |

---

## 3.11 Security Command Center (SCC)

### Tiers
| Tier | Features |
|---|---|
| **Standard** | Asset inventory, security health analytics (free) |
| **Premium** | Threat detection, compliance reports, SIEM integration, web security scanner |
| **Enterprise** (2025) | SIEM + SOAR, multi-cloud (AWS, Azure), attack path simulation, risk scoring |

### SCC Finding Categories
- **Vulnerabilities:** Open firewall ports, public buckets, OS vulnerabilities, misconfigurations
- **Threats:** Malware, cryptomining, brute force attacks, data exfiltration
- **Compliance:** PCI DSS, HIPAA, CIS Benchmark, ISO 27001 violations

### SCC Integration
```
SCC Findings
    ↓
Pub/Sub notification
    ↓
Cloud Functions / Workflows
    ↓
Auto-remediation:
  - Close open firewall rule
  - Remove public bucket access
  - Create JIRA ticket
  - Page on-call security engineer
```

### Formerly Forseti Security
- Forseti was an open-source security monitoring tool; it has been superseded by SCC
- Exam questions may still reference "Forseti" — map this to SCC in modern context

---

## 3.12 Organization Policies for Security

### Key Security-Focused Org Policies
| Constraint | Security Benefit |
|---|---|
| `constraints/iam.disableServiceAccountKeyCreation` | Prevent SA key file leaks |
| `constraints/iam.disableServiceAccountKeyUpload` | Prevent uploading external keys |
| `constraints/compute.requireShieldedVm` | Require Secure Boot + vTPM |
| `constraints/compute.skipDefaultNetworkCreation` | Prevent auto-created default VPC |
| `constraints/compute.restrictPublicIpAccess` | Block public IPs on VMs |
| `constraints/storage.publicAccessPrevention` | Block public Cloud Storage buckets |
| `constraints/storage.uniformBucketLevelAccess` | Remove ACL-based bucket access |
| `constraints/gcp.resourceLocations` | Restrict to specific regions (data residency) |
| `constraints/cloudfunctions.allowedIngressSettings` | Restrict Cloud Functions to internal traffic |
| `constraints/run.allowedIngress` | Restrict Cloud Run to internal traffic |

---

## 3.13 Cloud DLP (Data Loss Prevention API)

### What DLP Does
- Discovers, classifies, and de-identifies sensitive data
- Supports 150+ built-in detectors: PII, credit cards, SSNs, health data, passwords
- Scans: BigQuery, Cloud Storage, Datastore

### DLP Transformation Methods
| Method | Description | Use Case |
|---|---|---|
| **Redaction** | Remove sensitive data entirely | Logs, debug output |
| **Masking** | Replace with fixed character (e.g., `****`) | UI display |
| **Tokenization** | Replace with reversible token | Analytics while preserving format |
| **Pseudonymization** | Deterministic encryption with HMAC | Join operations on anonymized data |
| **Generalization** | Reduce precision (age 34 → "30-40") | Statistical analysis |
| **Date shifting** | Randomly shift dates | Temporal analysis while preserving patterns |

### DLP Exam Scenario
**Q:** "Store customer data for analytics without exposing real SSNs or credit card numbers..."
**A:** Use Cloud DLP to de-identify data before loading into BigQuery; use tokenization to allow re-identification if needed

---

## 3.14 Compliance Frameworks

### GDPR (General Data Protection Regulation)
- **Applicability:** Data of EU residents (regardless of where company is located)
- GCP controls:
  - Data residency: use `constraints/gcp.resourceLocations` to keep data in EU
  - DLP API to discover and redact personal data
  - Audit logs for data access tracking
  - CMEK for encryption key control
  - Data Processing Addendum (DPA) with Google

### HIPAA (Health Insurance Portability and Accountability Act)
- **Applicability:** Protected Health Information (PHI) in the US
- GCP controls:
  - Sign BAA (Business Associate Agreement) with Google
  - Enable HIPAA-eligible GCP services only
  - Cloud Audit Logs for all PHI access
  - CMEK for PHI encryption
  - VPC Service Controls to isolate PHI data
  - Access controls: least privilege + MFA

### PCI DSS (Payment Card Industry Data Security Standard)
- **Applicability:** Any system that stores, processes, or transmits cardholder data
- GCP controls:
  - Cloud Armor WAF to protect cardholder data environment
  - CMEK for CHD encryption
  - Network segmentation with VPC Service Controls
  - Cloud Audit Logs + SCC for monitoring
  - Dedicated projects for PCI scope (isolate from non-PCI workloads)
  - Regular vulnerability scanning (SCC + Container Analysis)

### Compliance Summary
| Framework | GCP Key Controls |
|---|---|
| GDPR | DLP, data residency org policy, CMEK, audit logs, DPA |
| HIPAA | BAA, CMEK, audit logs, VPC SC, least privilege |
| PCI DSS | Cloud Armor, CMEK, network segmentation, vulnerability scanning |
| SOC 2 | Audit logs, SCC, org policies, CMEK |
| ISO 27001 | SCC Enterprise, IAM, CMEK, audit logs |
| FedRAMP | Only use FedRAMP-authorized services, separate GCP org |

---

## 3.15 BeyondCorp / Zero Trust Architecture

### Zero Trust Principles
- **"Never trust, always verify"** — no implicit trust based on network location
- Verify **identity + device + context** for every access request
- Move security perimeter from network boundary to resource level

### BeyondCorp Enterprise (Identity-Aware Proxy)
- **Identity-Aware Proxy (IAP):** Enforce identity and context checks before granting access to web apps and VMs
- No VPN required — access is granted based on user identity and device posture
- Supports MFA enforcement, device certificate verification, and context-aware access

### Context-Aware Access with IAP
```
User Request → IAP → Check:
  1. Is user authenticated? (Google Identity/SAML/OIDC)
  2. Is device managed? (endpoint verification)
  3. Is access level met? (IP range, device cert, OS version)
  4. Does IAM allow this user?
  ↓ ALL PASS
  Application (GCE, GKE, App Engine, Cloud Run)
```

### SSH/RDP to VMs without Public IPs
- Use **IAP TCP Tunneling** to SSH/RDP to VMs through IAP without opening firewall ports
```bash
gcloud compute ssh my-vm --tunnel-through-iap --zone us-central1-a
```

---

## 3.16 Shielded VMs and Confidential Computing

### Shielded VMs
Protect against rootkits, bootkits, and malicious firmware:
| Feature | What It Does |
|---|---|
| **Secure Boot** | Verifies bootloader signature; blocks unsigned boot components |
| **vTPM (Virtual TPM)** | Provides hardware-based key storage; measures boot integrity |
| **Integrity Monitoring** | Baseline measurement of boot sequence; alerts on deviation |

### Confidential Computing
- **Confidential VMs:** Encrypts VM memory during processing using AMD SEV (Secure Encrypted Virtualization)
- **Confidential GKE Nodes:** GKE nodes running on Confidential VMs
- **Confidential Space:** Multi-party computation — process sensitive data without exposing it to the operator
- Use case: Healthcare analytics where patient data must not be visible to the cloud provider

> **Exam Tip:** Confidential VMs protect **data in use** (memory encryption). CMEK/CSEK protect **data at rest**. TLS protects **data in transit**.

---

## 3.17 Audit Logging

### Cloud Audit Log Types
| Type | What It Logs | Default? |
|---|---|---|
| **Admin Activity** | API calls that modify resources (create, delete, modify) | Always on, cannot disable |
| **Data Access** | API calls that read or modify data (e.g., BigQuery SELECT) | Disabled by default (enable per service) |
| **System Event** | GCP system-generated actions (e.g., live migration) | Always on |
| **Policy Denied** | Denials due to org policies or VPC SC | Always on |

### Log Retention
| Audit Log Type | Default Retention |
|---|---|
| Admin Activity | 400 days |
| Data Access | 30 days |
| System Event | 400 days |

To extend retention: create a **log sink** to Cloud Storage (Coldline for long-term archival).

### Log Sink Architecture
```
Cloud Audit Logs
    ↓
Log Router (with inclusion/exclusion filters)
    ↓
Destinations:
  ├── Cloud Storage (long-term archive)
  ├── BigQuery (analysis + dashboards)
  ├── Pub/Sub (real-time SIEM forwarding)
  └── Cloud Logging bucket (another project)
```

> **Exam Tip:** For compliance, you often need to export logs to a **separate security project** that developers cannot access. Use log sinks with a destination in a security/audit project.

---

## 3.18 Common Exam Question Patterns and Traps

### Pattern 1: Encryption Key Control
**Q:** "Company policy requires that if the company revokes access, GCP cannot decrypt data..."
**A:** Use **Cloud EKM** (External Key Manager) — Google never has access to keys stored externally

### Pattern 2: Data Exfiltration Prevention
**Q:** "Prevent authorized users from copying sensitive BigQuery data to personal GCS buckets..."
**A:** **VPC Service Controls** — creates a perimeter that prevents data movement to outside projects

### Pattern 3: Zero Trust Access
**Q:** "Allow employees to access internal web apps without a VPN, enforcing device checks..."
**A:** **BeyondCorp / Identity-Aware Proxy (IAP)** with Context-Aware Access

### Pattern 4: Supply Chain Security
**Q:** "Ensure only vulnerability-scanned and approved images run in production GKE..."
**A:** **Binary Authorization** with attestations from the vulnerability scanning step

### Common Traps
- **Cloud Armor ≠ VPC firewall:** Cloud Armor is for HTTP(S) LB; VPC firewall rules are for all traffic
- **CSEK key loss = permanent data loss:** Google cannot recover data if you lose CSEK keys
- **Data Access audit logs are NOT enabled by default:** Must explicitly enable per service
- **VPC Service Controls ≠ VPC firewall:** VPC SC controls API-level access; firewalls control network-level access
- **IAP protects apps, not databases:** Cannot use IAP for Cloud SQL; use Private Service Connect or Cloud SQL Auth Proxy

---

## 3.19 Summary Table

| Topic | Key Concept | Exam Signal Words |
|---|---|---|
| Data exfiltration prevention | VPC Service Controls | "prevent exfiltration", "security perimeter", "authorized users" |
| Encryption: customer control | CMEK (Cloud KMS) | "customer-managed keys", "audit key usage", "rotate keys" |
| Encryption: external key | Cloud EKM | "external key manager", "GCP cannot decrypt", "data sovereignty" |
| Encryption: data in use | Confidential VMs | "memory encryption", "process sensitive data", "AMD SEV" |
| WAF / DDoS | Cloud Armor | "SQL injection", "XSS", "DDoS", "rate limiting" |
| Secrets storage | Secret Manager | "API keys", "database passwords", "rotate secrets" |
| Container signing | Binary Authorization | "only approved images", "supply chain security" |
| Zero Trust access | IAP / BeyondCorp | "no VPN", "device check", "context-aware" |
| Finding misconfigs | Security Command Center | "detect vulnerabilities", "compliance violations" |
| Sensitive data discovery | Cloud DLP | "PII", "redact", "tokenize", "de-identify" |
| Compliance | CMEK + Audit Logs + Org Policies | "GDPR", "HIPAA", "PCI DSS" |
| Supply chain | Artifact Registry + Binary Auth | "image scanning", "CVE", "SBOM" |
| VM security | Shielded VMs | "rootkit", "Secure Boot", "vTPM", "integrity" |
| Least privilege | Custom IAM roles + Deny policies | "minimum permissions", "cannot delete", "locked down" |
