# GCP Storage Services — Exam Reference (2025)

---

## Cloud Storage

### Storage Classes

| Class | Min Storage Duration | Access Frequency | Retrieval Cost | Use Case |
|-------|---------------------|-----------------|----------------|---------|
| **Standard** | None | Frequent | None | Hot data, web assets, ML datasets |
| **Nearline** | 30 days | ~Monthly | Per-GB retrieval | Backups accessed once/month |
| **Coldline** | 90 days | ~Quarterly | Per-GB retrieval | Disaster recovery, archived logs |
| **Archive** | 365 days | <Yearly | Per-GB retrieval (highest) | Long-term compliance archives |

- Storage class is set at bucket level but can be overridden per object
- **Autoclass**: Automatically moves objects between storage classes based on access patterns; no early deletion penalties when Autoclass manages transitions
- Cross-region/multi-region buckets: Standard class costs more but provides higher availability (99.95% SLA for multi-region)

### Access Control

- **Uniform Bucket-Level Access** (recommended): IAM-only; disables ACLs; enforces consistent permissions; required for VPC Service Controls
- **Fine-Grained Access**: Supports both IAM and per-object ACLs; legacy; avoid for new buckets
- **IAM Roles**: `roles/storage.objectViewer`, `roles/storage.objectCreator`, `roles/storage.admin`, `roles/storage.legacyBucketOwner`
- **Signed URLs**: Time-limited access for unauthenticated users; signed with service account private key or via `signBlob` API; max 7 days
- **Signed Policy Documents**: Control what can be uploaded via HTML form POST

### Data Protection

- **Object Versioning**: Retains overwritten/deleted objects; each version has a generation number; storage billed for all versions
- **Retention Policies**: Minimum retention period on objects; cannot delete before period expires; set on bucket
- **Bucket Lock**: Permanently locks a retention policy; used for WORM (Write Once Read Many) compliance (SEC Rule 17a-4, FINRA)
- **Soft Delete**: Default 7-day recovery window for deleted objects/buckets (replaces object versioning for simple recovery)

### Lifecycle Management

```yaml
# Example lifecycle rule
lifecycle:
  rule:
  - action:
      type: SetStorageClass
      storageClass: NEARLINE
    condition:
      age: 30
      matchesStorageClass: [STANDARD]
  - action:
      type: Delete
    condition:
      age: 365
      isLive: false   # deletes non-current versions
```

- Conditions: `age`, `createdBefore`, `isLive`, `matchesStorageClass`, `numNewerVersions`, `matchesPrefix/Suffix`
- Rules processed daily (not real-time)

### CORS Configuration

- Set via JSON/XML on bucket; allows web browsers to make cross-origin requests
- Specify allowed origins, methods, headers, and max age

### Transfer Options

| Option | Best For |
|--------|---------|
| `gsutil` / `gcloud storage` | CLI; single machine; < 1 TB |
| **Storage Transfer Service** | Large-scale transfers from S3, Azure, HTTP, on-prem; scheduled/recurring |
| **Transfer Appliance** | Offline; > 100 TB; high-latency networks |
| **BigQuery Data Transfer Service** | SaaS apps → BigQuery (not Cloud Storage transfers) |

> **Exam Tip**: Storage Transfer Service can schedule recurring transfers (e.g., daily S3 → GCS sync). Transfer Appliance is for petabyte-scale or very poor connectivity.

> **Gotcha**: Bucket Lock is **permanent and irreversible**. Deleting a bucket with a locked policy is impossible until all objects meet their retention period. Autoclass has a 30-day minimum before objects can move to Nearline.

---

## Persistent Disk

### Disk Types

| Type | IOPS/GB | Throughput | Use Case |
|------|---------|-----------|---------|
| **Standard HDD (pd-standard)** | 0.75 read/1.5 write | 180 MB/s | Sequential reads, low-cost storage |
| **Balanced SSD (pd-balanced)** | 6 IOPS/GB | 240 MB/s | General purpose; good price/performance |
| **SSD (pd-ssd)** | 30 IOPS/GB | 480 MB/s | Databases, latency-sensitive apps |
| **Extreme SSD (pd-extreme)** | Custom (up to 120K IOPS) | 2,400 MB/s | Highest-performance databases (SAP HANA) |
| **Hyperdisk Extreme** | Up to 350K IOPS | Custom | Next-gen; decoupled from disk size |
| **Hyperdisk Balanced** | Custom IOPS/throughput | Custom | Flexible performance per workload |

### Zonal vs Regional

- **Zonal PD**: Single zone; default; lower cost; data lost if zone fails
- **Regional PD**: Synchronously replicated across 2 zones; 2× cost; ~1ms additional write latency; supports failover for HA VMs
- Regional PD required for GKE Regional cluster high availability storage

### Snapshots

- **Incremental snapshots**: First snapshot is full; subsequent are incremental (only changed blocks)
- Stored in Cloud Storage (multi-regional by default); billed as Cloud Storage
- **Snapshot schedules**: Automate regular snapshots; retention policy; recommended for all production disks
- Snapshots can be used to create new disks in different zones/regions
- **Instant Snapshots**: Hyperdisk-only; near-instant, stored locally; cannot export

### Key Facts

- PD is network-attached; independent of VM lifecycle (persists after VM deletion by default)
- Max disk size: 64 TB
- Read-only mode: Attach same disk to multiple VMs (useful for shared read-only data)
- Resize: Can expand online without downtime; cannot shrink

> **Exam Tip**: For SAP HANA or Oracle workloads requiring very high IOPS, use Extreme SSD or Hyperdisk Extreme. Regional PD is needed for synchronous replication and failover.

> **Gotcha**: Snapshot billing is for *compressed* stored data, not raw disk size. Resizing a disk online requires filesystem resize separately (e.g., `resize2fs` for Linux).

---

## Filestore

### Tiers

| Tier | Capacity Range | IOPS | Protocol | Use Case |
|------|---------------|------|----------|---------|
| **Basic HDD** | 1–63.9 TB | Up to 60K | NFSv3 | Dev/test, low-cost NFS |
| **Basic SSD** | 2.5–63.9 TB | Up to 480K | NFSv3 | Medium-performance NFS |
| **Zonal** | 1–100 TB | Up to 660K | NFSv3/NFSv4.1 | Low-latency, single-zone |
| **Regional** | 1–100 TB | Up to 660K | NFSv3/NFSv4.1 | Multi-zone HA NFS |
| **Enterprise** | 1–10 TB | Up to 480K | NFSv3/NFSv4.1 | HA, snapshots, replication |

- **NFS Protocol**: NFSv3 (default); Enterprise/Zonal/Regional support NFSv4.1
- **Use cases**: Lift-and-shift NFS workloads, shared filesystem for GKE (ReadWriteMany), genomics pipelines, EDA/HPC
- **Snapshots**: Enterprise/Zonal/Regional support snapshots; not available on Basic tiers
- Filestore instances are zonal (Basic/Zonal) or regional (Regional/Enterprise)

> **Exam Tip**: Filestore is the right choice when you need a shared POSIX filesystem (ReadWriteMany) for GKE. It's a managed NFS service.

---

## Cloud Storage FUSE

- Open-source FUSE adapter (`gcsfuse`) to mount Cloud Storage buckets as filesystems
- **Performance**: Lower throughput than Persistent Disk or Filestore; adds metadata overhead
- **Use cases**: Read-heavy ML training data access, sharing data between Cloud Run/Functions and Cloud Storage
- Supports random reads but sequential reads perform best
- Not POSIX-compliant (no atomic renames, limited symlink support)

> **Gotcha**: Cloud Storage FUSE does not provide the performance of a real filesystem. Use Filestore or PD for IOPS-sensitive workloads.

---

## Local SSD

- **Physically attached** to the host server; not network-attached
- Extremely high IOPS (up to 2.4M IOPS) and very low latency (~0.1ms)
- **Ephemeral**: Data lost if VM stops, crashes, or migrates; persists through live migration only
- 375 GB per partition; up to 24 partitions (9 TB) per VM
- Available on N2, N2D, C2, M1, M2, A2 machine types
- Not available on E2 or T2A

**Use cases**: Caching, temporary storage for Hadoop/Spark shuffle, high-performance databases where data is replicated elsewhere (e.g., Cassandra)

> **Exam Tip**: Local SSD is best for temporary data where performance is critical and durability is handled at the application layer (e.g., Bigtable, Cassandra, Redis AOF).

> **Gotcha**: If VM is stopped (not just restarted), local SSD data is permanently lost. Plan accordingly.

---

## Storage Decision Matrix

| Requirement | Recommended Service |
|-------------|-------------------|
| Object/blob storage, web assets | Cloud Storage (Standard) |
| Infrequently accessed backups (monthly) | Cloud Storage (Nearline) |
| Disaster recovery archives | Cloud Storage (Coldline/Archive) |
| WORM compliance archiving | Cloud Storage + Bucket Lock |
| Block storage for VM OS disk | Persistent Disk (Balanced SSD default) |
| Highest IOPS block storage (SAP, Oracle) | Persistent Disk Extreme / Hyperdisk |
| HA block storage across zones | Regional Persistent Disk |
| Shared NFS filesystem (GKE/VMs) | Filestore |
| High-performance shared NFS (HPC/EDA) | Filestore (Zonal or Enterprise) |
| Ephemeral ultra-high-performance storage | Local SSD |
| Mount GCS bucket as filesystem | Cloud Storage FUSE |
| Large offline data transfer (>100 TB) | Transfer Appliance |
| Recurring cloud-to-cloud sync | Storage Transfer Service |

---

## Common Exam Gotchas

1. **Nearline/Coldline/Archive** have early deletion fees — factor into TCO calculations
2. **Uniform bucket-level access** is required for VPC Service Controls; cannot be disabled after 90 days of enforcement
3. **Object Versioning + lifecycle rules** — use `isLive: false` condition to delete old versions
4. **Regional PD** has higher write latency (~1ms extra) and costs 2× zonal; only use when cross-zone HA is required
5. **Local SSD** data is lost on VM stop — not just for restarts
6. **Filestore Basic** does NOT support snapshots; use Enterprise or Zonal tier
7. **Signed URLs** can be generated without a service account key file using the `signBlob` IAM API
8. **Autoclass** has a 30-day observation period before moving objects; early deletion penalties do not apply when Autoclass triggers class changes
