# GCP Networking Services — Exam Reference (2025)

---

## VPC (Virtual Private Cloud)

### Core Concepts

- VPC is **global** — a single VPC spans all regions; subnets are regional
- **Auto mode VPC**: Automatically creates one subnet per region (default /20 ranges); easy to start but harder to control
- **Custom mode VPC**: Full control over subnets; required for production / enterprise; can convert from auto → custom (irreversible)
- No charge for VPC itself; charges for egress, load balancers, VPN, etc.

### Subnets & IP Ranges

- Each subnet has a **primary IP range** (for VMs) and optionally one or more **secondary IP ranges** (for GKE pods/services, alias IPs)
- Subnet ranges can be expanded (increase prefix length to smaller number) but **never shrunk**
- **Alias IP ranges**: Assign multiple IP addresses to a single VM NIC from the subnet's secondary range; used for containers on a VM

### Shared VPC

- **Host project**: Owns the VPC and subnets
- **Service projects**: Attach to the host project; use its subnets but cannot modify them
- Resources (VMs, GKE clusters) in service projects use host project network
- **Use case**: Centralized network management with distributed resource ownership; avoids VPC sprawl
- Requires `roles/compute.networkUser` on service project in host project

### VPC Peering

- Connects two VPCs (same or different org) at network layer; **not transitive**
- Both sides must create peering; traffic uses internal IPs
- **No overlapping CIDR ranges** allowed
- Does not support exporting/importing custom routes by default (configurable)
- **vs Shared VPC**: Peering = two separate VPCs connected; Shared VPC = single VPC shared across projects

> **Exam Tip**: VPC Peering is NOT transitive. If A↔B and B↔C, A cannot reach C. Use Shared VPC or a hub-and-spoke topology via Cloud Router/HA VPN for transitive routing.

> **Gotcha**: Changing from auto-mode to custom-mode VPC is irreversible.

---

## Cloud Load Balancing

### Load Balancer Types

| Type | Scope | Protocol | Backend | Use Case |
|------|-------|----------|---------|---------|
| **Global External HTTP(S) LB** | Global | HTTP/HTTPS/HTTP2/gRPC | Instance groups, NEGs, GCS, Cloud Run | Internet-facing web apps with global reach |
| **Regional External HTTP(S) LB** | Regional | HTTP/HTTPS | Instance groups, NEGs | Single-region; lower latency; simpler |
| **External TCP/SSL Proxy LB** | Global | TCP/SSL | Instance groups | Non-HTTP TCP (databases, gaming) |
| **External Network LB** | Regional | TCP/UDP | Instance groups | Ultra-low latency; direct server return; pass-through |
| **Internal HTTP(S) LB** | Regional | HTTP/HTTPS | Instance groups, NEGs | Internal microservices, east-west traffic |
| **Internal TCP/UDP LB** | Regional | TCP/UDP | Instance groups | Internal stateful apps (databases) |
| **Cross-region Internal HTTP(S) LB** | Global | HTTP/HTTPS | NEGs across regions | Global internal microservices |

### Backend Services & Health Checks

- **Backend Service**: Defines backends (instance groups/NEGs), load balancing algorithm, health check, session affinity, CDN policy
- **Load Balancing Algorithms**: Round robin (default), least connections, IP hash (session affinity)
- **Health Checks**: HTTP, HTTPS, HTTP/2, TCP, gRPC, SSL; specify port, path, thresholds (healthy/unhealthy count), interval
- **Session Affinity**: Client IP, generated cookie, header-based; for stateful applications

### Network Endpoint Groups (NEGs)

| NEG Type | Description |
|----------|-------------|
| **Zonal NEG** | GCE VMs or GKE pods; fine-grained pod-level routing |
| **Internet NEG** | External backend (IP:port or FQDN) |
| **Serverless NEG** | Cloud Run, App Engine, Cloud Functions as backends |
| **Private Service Connect NEG** | Reach services via PSC |
| **Hybrid NEG** | On-premises or other cloud endpoints via Cloud Interconnect/VPN |

> **Exam Tip**: Use Serverless NEGs to route HTTP(S) LB traffic to Cloud Run/App Engine. Use Zonal NEGs with GKE for pod-level load balancing (Container-native load balancing).

---

## Cloud CDN

- Integrated with Global External HTTP(S) LB; caches responses at Google's global edge PoPs
- **Cache modes**: `USE_ORIGIN_HEADERS` (default), `CACHE_ALL_STATIC`, `FORCE_CACHE_ALL`
- **Cache keys**: URL, headers, cookies; customize to improve cache hit ratio
- **Signed URLs / Cookies**: Restrict CDN content to authorized users; time-limited
- **Negative caching**: Cache 404/301 responses; reduces origin load
- CDN Invalidation: `gcloud compute url-maps invalidate-cdn-cache`

> **Gotcha**: CDN only caches responses from healthy backends. If backend returns `Cache-Control: no-store`, CDN bypasses cache regardless of cache mode.

---

## Cloud DNS

- **Authoritative DNS**: Managed zones (public/private); 100% SLA uptime
- **Public Zones**: Accessible from internet; NS/SOA records auto-created
- **Private Zones**: Accessible from VPCs; attach to specific VPCs for resolution
- **Forwarding Zones**: Forward DNS queries for specific domains to on-premises resolvers (hybrid DNS)
- **Response Policy Zones (RPZ)**: Override DNS responses within VPC (DNS firewall)
- **DNS Peering**: Share private zone resolution across peered VPCs
- DNSSEC supported for public zones

> **Exam Tip**: Private zones + DNS peering is the recommended pattern for hybrid environments needing consistent DNS resolution.

---

## Cloud Armor

- DDoS protection and WAF (Web Application Firewall) for external HTTP(S) LBs
- **Security Policies**: Set of rules with priorities; action = allow/deny/redirect/throttle
- **Rule types**: IP/CIDR-based, geo-based, custom expression (CEL), rate-based, WAF preconfigured rules
- **WAF Preconfigured Rules**: OWASP Top 10 (SQLi, XSS, RCE, etc.); ModSecurity CRS 3.3 rule sets
- **Adaptive Protection**: ML-based; auto-detects L7 DDoS and suggests mitigation rules; included in Cloud Armor Enterprise
- **Tiers**:
  - *Standard*: Pay-per-use; basic DDoS, IP allow/deny lists
  - *Enterprise*: Advanced WAF, Adaptive Protection, named IP lists, DDoS response team access
- **Bot Management**: Integrate with reCAPTCHA Enterprise; challenge suspicious traffic
- Rules evaluated in **priority order** (lower number = higher priority); default rule at 2,147,483,647

> **Exam Tip**: Cloud Armor is ONLY for external HTTP(S) LBs. It does not protect TCP/UDP load balancers or internal LBs.

---

## Cloud NAT

- **Managed NAT gateway**: Allows VMs with only internal IPs to reach internet for outbound traffic
- Fully distributed; highly available; no single-point-of-failure
- Configure on a Cloud Router; specify subnet ranges that get NAT
- **NAT IP allocation**: Auto (dynamic) or manual (static IPs); manual required when external IP must be whitelisted
- No inbound connections allowed through Cloud NAT (egress-only)
- Supports GKE pods (if pod IP ranges are included)

---

## Private Google Access

- Allows VMs without external IPs to reach Google APIs and services using internal IPs
- Enabled per subnet (`privateIpGoogleAccess: true`)
- **Private Service Connect (PSC)**: Access Google APIs via private endpoint in your VPC with a specific internal IP; no need for internet routing
- **Restricted Google APIs VIP** (`restricted.googleapis.com`): Force all Google API traffic to stay within VPC Service Controls perimeter; use for data exfiltration prevention

---

## VPN

### HA VPN (Recommended)

- **99.99% SLA**; requires two tunnels per connection (active/active)
- Supports dynamic routing via Cloud Router (BGP)
- Each HA VPN gateway has 2 interfaces with 2 external IPs
- Up to 3 Gbps per tunnel; 6 Gbps combined per HA VPN gateway

### Classic VPN

- **99.9% SLA**; single tunnel; no HA without additional setup
- Supports static routing or dynamic (BGP with Cloud Router)
- Legacy; use HA VPN for new deployments

> **Exam Tip**: HA VPN always uses Cloud Router (BGP). Classic VPN can use static routes. For > 3 Gbps, use Cloud Interconnect.

---

## Cloud Interconnect

### Dedicated Interconnect

- **Direct physical connection** between on-prem network and Google's network
- Speeds: 10 Gbps or 100 Gbps per circuit; multiple circuits for higher bandwidth
- Requires colocation in a Google-supported facility or partner colocation
- Lowest latency; traffic does not traverse public internet
- **99.9% SLA** (2 connections); **99.99% SLA** (4 connections in 2 metros)

### Partner Interconnect

- Connect via a **supported service provider** (Equinix, AT&T, etc.)
- Speeds: 50 Mbps to 50 Gbps
- For organizations that cannot meet dedicated requirements or need lower bandwidth
- **99.9% SLA** (2 partner connections); **99.99% SLA** with diverse providers

| | Dedicated | Partner | HA VPN |
|--|-----------|---------|--------|
| Bandwidth | 10/100 Gbps | 50 Mbps–50 Gbps | Up to 6 Gbps |
| Latency | Lowest | Low | Higher (internet) |
| SLA | 99.99% (4 circuits) | 99.99% (diverse) | 99.99% |
| Cost | Highest | Medium | Lowest |
| Setup | Complex | Easier via provider | Easiest |

---

## Network Intelligence Center

- Suite of network observability and diagnostics tools:
  - **Connectivity Tests**: Verify firewall/routing rules between two endpoints (simulation)
  - **Network Topology**: Visual graph of VPC topology
  - **Performance Dashboard**: Latency between zones/regions
  - **Firewall Insights**: Identify unused or overly permissive firewall rules
  - **Network Analyzer**: Automated detection of misconfigurations

---

## Firewall Rules

- Applied at **network level** (not VM level); stateful (return traffic allowed automatically)
- **Ingress** (inbound to VM) and **Egress** (outbound from VM)
- Lower priority number = evaluated first (range: 0–65535); default rules at 65534/65535
- **Target specification**: All instances in network, instances with specific **network tags**, or instances using a specific **service account**
- **Service account targets** (preferred over tags): Harder to spoof; tag can be self-assigned by any project member with rights

```
Priority | Direction | Action | Target | Source/Dest | Protocol/Port
```

- **Default firewall rules**: `default-allow-internal` (allow all within VPC), `default-allow-ssh`, `default-allow-rdp`, `default-allow-icmp`
- **Implied rules**: Deny all ingress, allow all egress (cannot delete but can override with higher-priority rules)
- **Firewall Policies**: Hierarchical (org/folder level); applied before VPC firewall rules; prefer for consistent enforcement

> **Exam Tip**: Use service account-based firewall rules in production (more secure than tags). Hierarchical firewall policies apply organization-wide.

---

## VPC Service Controls

- Define **service perimeters** around GCP projects to prevent data exfiltration
- Services inside perimeter can only communicate with other resources inside the perimeter
- Supports **access levels** (device state, IP range, identity) via Access Context Manager
- **Dry-run mode**: Log violations without enforcing; use before enabling enforcement
- **Ingress/Egress rules**: Allow controlled access across perimeter boundaries
- Works with: Cloud Storage, BigQuery, Pub/Sub, Cloud KMS, and many more APIs

> **Gotcha**: VPC Service Controls require Uniform Bucket-Level Access on Cloud Storage. IAM still applies inside a perimeter; Service Controls add an additional layer.

---

## Connectivity Decision Matrix

| Scenario | Recommended Option |
|----------|-------------------|
| On-prem to GCP, < 1 Gbps, internet-based | HA VPN |
| On-prem to GCP, 1–10 Gbps, dedicated link | Dedicated Interconnect |
| On-prem to GCP via telco partner | Partner Interconnect |
| Highest bandwidth + lowest latency | Dedicated Interconnect (100 Gbps) |
| Multi-region GCP to GCP routing | VPC (automatic via Google backbone) |
| Two separate GCP VPCs, same or different org | VPC Peering |
| Centralized network, multiple projects | Shared VPC |
| Internet-facing protection from DDoS/WAF | Cloud Armor + Global HTTPS LB |
| VMs need internet access, no public IP | Cloud NAT |
| Access Google APIs without public IP | Private Google Access / PSC |
| Data exfiltration prevention | VPC Service Controls |
