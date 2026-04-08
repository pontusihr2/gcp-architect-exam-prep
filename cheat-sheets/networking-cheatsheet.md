# Networking Cheat Sheet
> Google Professional Cloud Architect — 2025 Edition

---

## 1. VPC Fundamentals

### VPC Types
| Feature | Default VPC | Custom VPC |
|---|---|---|
| **Creation** | Auto-created per project per region | Manually created |
| **Subnet mode** | Auto-mode | Custom-mode (recommended for production) |
| **Firewall rules** | Pre-configured allow rules (SSH 22, RDP 3389, ICMP, internal) | No pre-configured rules |
| **IP ranges** | Fixed predefined ranges per region | You define CIDR ranges |
| **Delete** | Can be deleted | Can be deleted |
| **Recommendation** | Dev/test only; disable in prod via org policy | Always use for production |

### Subnet Mode
| Mode | Behavior | When to Use |
|---|---|---|
| **Auto-mode** | One subnet auto-created per region using 10.128.x.0/20 | Simple, single-org setups; dev |
| **Custom-mode** | No auto-created subnets; full CIDR control | Production; multi-org; VPC peering (no CIDR overlap allowed) |

### Key VPC Properties
- **VPCs are global** — subnets are regional, but a single VPC spans all regions
- **Subnet secondary ranges**: Add secondary IP ranges for GKE Pod/Service IPs (alias IPs)
- **Private Google Access**: Subnets without external IPs can reach Google APIs via 199.36.153.x
- **VPC Flow Logs**: Capture network flow metadata (5-tuple) — sampled, stored in Cloud Logging
- **Shared VPC**: Central network project shares subnets to service projects (billing to service project; network managed centrally)

---

## 2. Firewall Rules

### Rule Properties
| Property | Values / Notes |
|---|---|
| **Direction** | `INGRESS` (incoming to VM) or `EGRESS` (outgoing from VM) |
| **Action** | `ALLOW` or `DENY` |
| **Priority** | 0 (highest) – 65535 (lowest); default rule = 65534 |
| **Target** | All instances, target tags, or target service account |
| **Source/Destination** | IP ranges, source tags, source service accounts |
| **Protocol/Port** | tcp, udp, icmp, esp, ah, sctp; or all |
| **Logging** | Optional; logs to Cloud Logging |

### Default Firewall Rules (Default VPC Only)
| Rule Name | Direction | Priority | Action | Description |
|---|---|---|---|---|
| `default-allow-internal` | INGRESS | 65534 | ALLOW | All traffic between instances in the VPC |
| `default-allow-ssh` | INGRESS | 65534 | ALLOW | TCP:22 from 0.0.0.0/0 |
| `default-allow-rdp` | INGRESS | 65534 | ALLOW | TCP:3389 from 0.0.0.0/0 |
| `default-allow-icmp` | INGRESS | 65534 | ALLOW | ICMP from 0.0.0.0/0 |
| *(implied)* | INGRESS | 65535 | DENY | Deny all ingress |
| *(implied)* | EGRESS | 65535 | ALLOW | Allow all egress |

> **Exam Rule:** Custom VPC has NO pre-configured rules (except the implied deny ingress / allow egress). You must add firewall rules. The implied deny/allow rules cannot be deleted but can be overridden with higher-priority rules.

### Firewall Best Practices
- Use **network tags** (not IP ranges) for dynamic VM targeting — IPs change, tags persist
- Use **service account-based rules** for stronger identity (tag can be spoofed by any instance in project)
- Enable **Firewall Insights** and **firewall logging** for security auditing
- **Hierarchical Firewall Policies** — set at org/folder level; override project-level rules (higher priority)

---

## 3. Load Balancer Quick Reference

| Type | Scope | Protocol | Notes |
|---|---|---|---|
| **Global External Application LB** | Global | HTTP/HTTPS/HTTP2/gRPC | CDN, Cloud Armor WAF, URL routing, serverless NEGs |
| **Regional External Application LB** | Regional | HTTP/HTTPS | Lower cost; no global anycast |
| **Internal Application LB** | Regional (internal) | HTTP/HTTPS/gRPC | Service mesh, microservices inside VPC |
| **External Proxy Network LB (TCP/SSL)** | Global | TCP/SSL | SSL termination, non-HTTP TCP globally |
| **Internal Proxy Network LB** | Regional (internal) | TCP | Internal L4 proxy |
| **External Passthrough Network LB** | Regional | TCP/UDP/ESP/ICMP | Preserves client IP; pass-through (no proxy) |
| **Internal Passthrough Network LB** | Regional (internal) | TCP/UDP | Classic 3-tier app internal LB; preserves client IP |

> **Source IP Preservation:** Only Passthrough LBs preserve the original client IP. Proxy LBs terminate the connection — use `X-Forwarded-For` header for original IP.

---

## 4. Hybrid Connectivity Quick Reference

| Option | BW | SLA | Encryption | Latency | Min Cost | Best For |
|---|---|---|---|---|---|---|
| **HA VPN** | Up to 3 Gbps/tunnel | 99.99% (2 tunnels) | IPsec (yes) | Variable (internet) | ~$0.05/hr/tunnel | < 3 Gbps; cost-sensitive; backup path |
| **Dedicated Interconnect** | 10 or 100 Gbps/link | 99.99% (redundant) | No (add own) | Predictable, low | ~$1,750/mo (10G) | > 1 Gbps; low latency SLA; collocated |
| **Partner Interconnect** | 50 Mbps–10 Gbps | 99.99% (redundant) | No | Low (private) | Provider pricing | Not at Google PoP; flexible BW |
| **Direct Peering** | Best-effort | None | No | Variable | Free | Google API access only; no VPC |
| **Cross-Cloud Interconnect** | 10 or 100 Gbps | 99.9% | No | Low | ~$1,750/mo | GCP ↔ AWS/Azure private connectivity |

---

## 5. DNS: Cloud DNS Reference

### DNS Zone Types
| Zone Type | Description | Use Case |
|---|---|---|
| **Public Zone** | Authoritative DNS for internet-facing domains | Public websites, external APIs |
| **Private Zone** | Internal DNS resolution within a VPC | Internal service discovery, internal hostnames |
| **Peering Zone** | Delegate DNS queries from one VPC to another VPC's private zone | Shared VPC, hub-and-spoke DNS |
| **Forwarding Zone** | Forward queries to external resolvers (on-prem DNS) | Hybrid environments — resolve on-prem names from GCP |
| **Managed Reverse Lookup Zone** | PTR records for VM internal IPs | Reverse DNS for logging/auditing |

### DNS Key Notes
- **Private zones** are VPC-scoped — bind to one or more VPCs
- **Inbound DNS forwarding** (Cloud DNS policy): Allow on-prem DNS servers to send queries to GCP private zones via an inbound forwarding IP in the VPC
- **Outbound DNS forwarding**: GCP VMs can forward queries to on-prem DNS (via forwarding zones)
- **GKE DNS**: CoreDNS (GKE Standard) or Cloud DNS for GKE (Autopilot) — recommended to enable Cloud DNS for GKE for scalability
- **Internal IP format**: `[vm-name].[zone].c.[project-id].internal`

---

## 6. Network Tiers

| Tier | Routing | Egress Cost | Latency | Use Case |
|---|---|---|---|---|
| **Premium Tier** | Google's private global backbone (lowest latency path) | Higher | Lowest | Production traffic; global apps; latency-sensitive |
| **Standard Tier** | Public internet routing from region | Lower (~20% savings) | Higher | Cost-sensitive; non-latency-sensitive; same-region |

> **Exam Tip:** Global LBs (anycast VIPs) require Premium Tier. Standard Tier is only available with regional LBs.
> Cloud NAT, Cloud VPN, and Dedicated Interconnect only support Premium Tier.

---

## 7. Private Access Options

| Feature | What It Does | When to Use |
|---|---|---|
| **Private Google Access** | VMs without external IPs can access Google APIs (storage.googleapis.com, etc.) via internal routes | Any VM that needs to call GCP APIs without a public IP |
| **Private Services Access** | Private IP connectivity between your VPC and Google-managed services (Cloud SQL, Memorystore, AlloyDB) via VPC peering with Google | Cloud SQL private IP; eliminates public IP for managed services |
| **Serverless VPC Access** | Connect Cloud Run / Cloud Functions / App Engine Standard to resources inside a VPC | Serverless workloads need to access VPC resources (Cloud SQL private IP, Memorystore) |
| **Private Service Connect (PSC)** | Privately expose GCP services or your own services with private endpoints inside your VPC — no VPC peering | Cleaner alternative to Private Services Access; avoids CIDR overlap issues; expose internal services to consumers |

> **Exam Rule:** Cloud Run needs to access Cloud SQL (private IP) → need **Serverless VPC Access** connector. VM needs to access Google APIs without public IP → enable **Private Google Access** on the subnet.

---

## 8. Cloud NAT

### What It Does
Allows VMs **without external IP addresses** to initiate outbound internet connections (NAT translation from private → public IP pool).

### Key Properties
- **Outbound only** — does NOT allow inbound connections (not a traditional two-way NAT)
- Configured per **Cloud Router** per **region**
- Can share NAT IPs across multiple VMs or use static reserved IPs
- **Port allocation**: Static (reserved) or Dynamic (auto-allocates ports)
- Works with GCE instances, GKE nodes (via node pool setting), Cloud Run (via Serverless VPC Access + NAT)

### When to Use Cloud NAT
- VMs in private subnets need to download packages / reach internet APIs
- Security requirement: no public IPs on VMs
- GKE nodes in private clusters need to pull container images from Docker Hub

### Limitations
- Does NOT work for inbound traffic (use LB or IAP for that)
- Does NOT replace VPN/Interconnect for on-prem connectivity
- Does NOT mask inter-VPC traffic (only internet-bound traffic)

---

## 9. Important IP Ranges and Reserved Addresses

### Special GCP Ranges
| Range | Purpose |
|---|---|
| `169.254.169.254/32` | GCE Metadata Server (VM metadata endpoint) |
| `199.36.153.4/30` | Private Google Access — restricted VIP range (googleapis.com) |
| `199.36.153.8/30` | Private Google Access — private VIP range (Cloud Run, etc.) |
| `35.199.192.0/19` | Google's health check source IPs — must allow in firewall for LB health checks |
| `130.211.0.0/22` | Legacy health check source IPs (allow both with external LBs) |
| `10.0.0.0/8` | Private RFC 1918 (common VPC range) |
| `172.16.0.0/12` | Private RFC 1918 |
| `192.168.0.0/16` | Private RFC 1918 |

### Reserved IPs per Subnet
Every subnet reserves 4 IPs:
- `.0` — Network address
- `.1` — Default gateway
- Second-to-last — Reserved by Google
- Last (`.255` for /24) — Broadcast

---

## 10. Common Networking Exam Gotchas

### 1. Health Check Firewall Rule
> ❗ Forgetting to allow `35.199.192.0/19` and `130.211.0.0/22` from Google health checkers is a common mistake. Without this rule, LB backends are marked unhealthy even if your app is running fine.

### 2. VPC Peering — No Transitive Routing
> ❗ VPC Peering is NOT transitive. If VPC-A peers with VPC-B, and VPC-B peers with VPC-C, VPC-A cannot reach VPC-C. Use **Shared VPC** or a **hub VPC with NVAs** for transitive routing.

### 3. Private Google Access ≠ VPC Service Controls
> Private Google Access lets VMs reach Google APIs. VPC Service Controls restricts which projects can access which APIs (data perimeter). They serve different purposes.

### 4. GKE Private Cluster — Control Plane Access
> GKE private clusters have private nodes (no external IPs on nodes). But the control plane endpoint can be public or private. For fully private GKE: configure **authorized networks** or use **private control plane endpoint** + Cloud NAT for node internet access.

### 5. Cloud VPN and BGP
> HA VPN requires **Cloud Router** with **BGP** for dynamic routing. Classic VPN used static routes. Always prefer HA VPN (99.99% SLA requires 2 tunnels between 2 VPN gateways).

### 6. Shared VPC — Who Pays for What
> In Shared VPC: Network resources (firewall, subnets, IPs) are billed to the **host project**. Compute resources (VMs) are billed to the **service project**. IAM grants needed: `roles/compute.networkUser` on the subnet to service project's SA.

### 7. Firewall Rules and Load Balancers
> External passthrough LB: firewall rules must allow traffic from `0.0.0.0/0` (LB is pass-through). For internal LB: allow source IP ranges of clients, not the LB IP.

### 8. DNS — On-Prem Resolution
> For bidirectional hybrid DNS: configure **Cloud DNS inbound forwarding policy** (on-prem → GCP zone) AND **Cloud DNS forwarding zone** (GCP → on-prem DNS server). Without both, only one direction works.

### 9. Premium vs Standard Tier for Global LB
> Cannot use Standard Tier with Global External LB. If a question gives you a cost-saving option with Standard Tier + Global LB — that's an incorrect option; global LB requires Premium Tier.

### 10. VPC CIDR and Peering Overlap
> VPC Peering fails if the two VPCs have **overlapping CIDR ranges**. Plan IP addressing carefully. This is why auto-mode VPC (uses same 10.128.x ranges) is problematic for organizations needing peering.
