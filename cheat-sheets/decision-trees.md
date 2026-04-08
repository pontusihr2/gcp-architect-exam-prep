# GCP Architecture Decision Trees
> Google Professional Cloud Architect — 2025 Edition

Use these trees to rapidly eliminate wrong answers on the exam. Start at the root question and follow the branches.

---

## 1. Compute Service Selection

```mermaid
flowchart TD
    c_start("START: What is your workload?")
    c_q1{"Is it a legacy VM / requires specific OS /<br/>custom kernel / stateful VM?"}
    c_gce["Compute Engine (GCE)"]
    c_spot["Spot / Preemptible VMs"]
    c_cud["Committed Use Discounts (CUDs)"]
    c_mig["MIG with stateful config + Regional PD"]
    c_q2{"Is it containerized?"}
    c_q3{"Is it stateless + HTTP?<br/>(serve web traffic, APIs)"}
    c_q4{"Minimal ops overhead?<br/>No cluster to manage?"}
    c_cloudrun["Cloud Run (fully managed)"]
    c_crzero["Cloud Run (default) — scale to zero"]
    c_crwarm["Cloud Run with min-instances — always warm"]
    c_q5{"Complex orchestration? Multi-container?<br/>Service mesh? Stateful pods? Custom scheduling?"}
    c_gke["GKE"]
    c_gkeauto["GKE Autopilot — fully managed nodes"]
    c_gkestd["GKE Standard — node customization / GPUs"]
    c_q6{"Is it a background job / batch container?"}
    c_crjobs["Cloud Run Jobs (for containers)<br/>or GKE Jobs / CronJobs"]
    c_q7{"Is it event-driven?<br/>Short execution? Single function?"}
    c_q8{"HTTP trigger / Pub/Sub / GCS event / Firestore event?"}
    c_cf["Cloud Functions (Gen 2)"]
    c_cfpref["Gen 2 (preferred) — less than 60 min execution"]
    c_cfrun["Cloud Run instead — need more than 60 min"]
    c_q9{"Need complex workflow /<br/>multi-step orchestration?"}
    c_workflows["Cloud Workflows<br/>(+ Cloud Functions/Run for steps)"]
    c_q10{"Is it a web application?"}
    c_q11{"Standard language runtime<br/>(Python/Java/Node/Go/PHP/Ruby)?<br/>Moderate traffic? Rapid deployment?"}
    c_aes["App Engine Standard"]
    c_q12{"Custom runtime / Docker container /<br/>long-running requests?"}
    c_aef["App Engine Flexible<br/>(Note: Cloud Run usually preferred for new workloads)"]

    c_start --> c_q1
    c_q1 -->|YES| c_gce
    c_gce -->|short-lived batch| c_spot
    c_gce -->|steady-state| c_cud
    c_gce -->|stateful + HA| c_mig
    c_q1 -->|NO| c_q2
    c_q2 -->|YES| c_q3
    c_q3 -->|YES| c_q4
    c_q4 -->|YES| c_cloudrun
    c_cloudrun -->|scale to zero needed| c_crzero
    c_cloudrun -->|always warm| c_crwarm
    c_q4 -->|NO| c_q5
    c_q5 -->|YES| c_gke
    c_gke -->|fully managed nodes| c_gkeauto
    c_gke -->|node customization / GPUs| c_gkestd
    c_q3 -->|NO| c_q6
    c_q6 -->|YES| c_crjobs
    c_q2 -->|NO| c_q7
    c_q7 -->|YES| c_q8
    c_q8 -->|YES| c_cf
    c_cf -->|less than 60 min| c_cfpref
    c_cf -->|more than 60 min| c_cfrun
    c_q8 -->|NO| c_q9
    c_q9 -->|YES| c_workflows
    c_q7 -->|NO| c_q10
    c_q10 -->|YES| c_q11
    c_q11 -->|YES| c_aes
    c_q11 -->|NO| c_q12
    c_q12 -->|YES| c_aef
```

---

## 2. Database Service Selection

```mermaid
flowchart TD
    db_start("START: What kind of data model do you need?")
    db_q1{"SQL / Relational data?"}
    db_q2{"Need GLOBAL distribution + strong consistency?<br/>(multi-region writes, financial transactions globally)"}
    db_spanner["Cloud Spanner"]
    db_spannercost["Spanner — cost acceptable (most expensive option)"]
    db_spannerpg["Spanner PostgreSQL dialect — PostgreSQL-compatible"]
    db_q3{"High-performance PostgreSQL?<br/>AI/ML query acceleration?<br/>Migrating AlloyDB workloads?"}
    db_alloydb["AlloyDB for PostgreSQL"]
    db_q4{"Standard MySQL / PostgreSQL / SQL Server?<br/>Single-region OLTP? Managed service?"}
    db_cloudsql["Cloud SQL"]
    db_mysql["Cloud SQL for MySQL"]
    db_pg["Cloud SQL for PostgreSQL"]
    db_ss["Cloud SQL for SQL Server"]
    db_q5{"NoSQL data?"}
    db_q6{"Very high throughput — millions of ops/sec?<br/>Flat / wide-column schema?<br/>Time-series / IoT / ad-tech?"}
    db_bigtable["Cloud Bigtable"]
    db_btoverkill["Bigtable is OVERKILL for less than 1 TB — use Firestore instead"]
    db_bthbase["Bigtable (HBase-compatible API) — HBase migration"]
    db_btbq["Bigtable to BigQuery — analytics queries on top"]
    db_q7{"Document model? Mobile/web app?<br/>Real-time sync?<br/>User profiles, game state, product catalogs?"}
    db_firestore["Firestore"]
    db_fsmobile["Firestore with native iOS/Android SDK — mobile SDK needed"]
    db_fsds["Firestore in Datastore mode — Datastore legacy migration"]
    db_fsglobal["Firestore multi-region — global"]
    db_q8{"Key-value / cache / session store?"}
    db_memorystore["Memorystore"]
    db_redis["Memorystore for Redis"]
    db_memcached["Memorystore for Memcached"]

    db_start --> db_q1
    db_q1 -->|YES| db_q2
    db_q2 -->|YES| db_spanner
    db_spanner -->|cost acceptable| db_spannercost
    db_spanner -->|PostgreSQL-compatible| db_spannerpg
    db_q2 -->|NO| db_q3
    db_q3 -->|YES| db_alloydb
    db_q3 -->|NO| db_q4
    db_q4 -->|YES| db_cloudsql
    db_cloudsql -->|MySQL| db_mysql
    db_cloudsql -->|PostgreSQL| db_pg
    db_cloudsql -->|SQL Server| db_ss
    db_q1 -->|NO| db_q5
    db_q5 -->|YES| db_q6
    db_q6 -->|YES| db_bigtable
    db_bigtable -->|less than 1 TB| db_btoverkill
    db_bigtable -->|HBase migration| db_bthbase
    db_bigtable -->|analytics queries on top| db_btbq
    db_q6 -->|NO| db_q7
    db_q7 -->|YES| db_firestore
    db_firestore -->|mobile SDK needed| db_fsmobile
    db_firestore -->|Datastore legacy migration| db_fsds
    db_firestore -->|global| db_fsglobal
    db_q7 -->|NO| db_q8
    db_q8 -->|YES| db_memorystore
    db_memorystore -->|Redis| db_redis
    db_memorystore -->|Memcached| db_memcached
```

---

## 3. Storage Service Selection

```mermaid
flowchart TD
    s_start("START: What type of storage access pattern?")
    s_q1{"OBJECT storage<br/>(files, blobs, unstructured data)?"}
    s_qfreq{"How frequently is data accessed?"}
    s_std["Cloud Storage STANDARD<br/>website assets, active datasets, streaming media"]
    s_nearline["Cloud Storage NEARLINE<br/>monthly backups, DR data, analytics archives"]
    s_coldline["Cloud Storage COLDLINE<br/>quarterly DR tests, compliance archives"]
    s_archive["Cloud Storage ARCHIVE<br/>regulatory retention, legal holds, 7+ year archives"]
    s_autoclass["Cloud Storage with AUTOCLASS enabled<br/>access pattern unpredictable / want auto-tiering"]
    s_q2{"BLOCK storage<br/>(VM disk, database volumes)?"}
    s_qtier{"Which performance tier?"}
    s_balanced["Balanced Persistent Disk (PD-Balanced)<br/>general-purpose VM workloads, OS disks, dev/test"]
    s_ssd["SSD Persistent Disk (PD-SSD)<br/>high IOPS databases — MySQL, PostgreSQL, SQL Server"]
    s_hdd["Standard Persistent Disk (PD-Standard / HDD)<br/>throughput-optimized — Hadoop, sequential reads/writes"]
    s_extreme["Hyperdisk Extreme<br/>highest IOPS — Oracle, SAP HANA"]
    s_multi["Hyperdisk ML — read-only multi-attach for ML inference<br/>Regional Persistent Disk — HA failover between zones"]
    s_q3{"FILE storage<br/>(NFS shared filesystem)?"}
    s_qfile{"What type of file storage need?"}
    s_filestore["Filestore<br/>lift-and-shift NAS workloads, shared app data,<br/>content management, genomics"]
    s_fshdd["Filestore Basic HDD — general workloads"]
    s_fsssd["Filestore Basic SSD — higher IOPS"]
    s_fsent["Filestore Enterprise — Enterprise + HA + Regional"]
    s_gcsfuse["GCS FUSE — mount GCS as filesystem<br/>cloud-native shared config / data<br/>(note latency tradeoffs)"]

    s_start --> s_q1
    s_q1 -->|YES| s_qfreq
    s_qfreq -->|multiple times per day / real-time serving| s_std
    s_qfreq -->|less than once per month| s_nearline
    s_qfreq -->|less than once per quarter| s_coldline
    s_qfreq -->|less than once per year| s_archive
    s_qfreq -->|access pattern unpredictable / auto-tiering| s_autoclass
    s_q1 -->|NO| s_q2
    s_q2 -->|YES| s_qtier
    s_qtier -->|general-purpose VM workloads, OS disks, dev/test| s_balanced
    s_qtier -->|high IOPS databases| s_ssd
    s_qtier -->|throughput-optimized| s_hdd
    s_qtier -->|highest IOPS available| s_extreme
    s_qtier -->|shared across multiple VMs| s_multi
    s_q2 -->|NO| s_q3
    s_q3 -->|YES| s_qfile
    s_qfile -->|lift-and-shift NAS / shared app data| s_filestore
    s_filestore -->|general workloads| s_fshdd
    s_filestore -->|higher IOPS| s_fsssd
    s_filestore -->|Enterprise + HA + Regional| s_fsent
    s_qfile -->|cloud-native shared config / data| s_gcsfuse
```

---

## 4. Network Connectivity Selection (On-Premises to GCP)

```mermaid
flowchart TD
    net_start("START: Do you need private non-internet connectivity to GCP?")
    net_q1{"Need private non-internet<br/>connectivity to GCP?"}
    net_vpn["Cloud VPN (HA VPN) with encryption"]
    net_vpnsingle["Single HA VPN — BW less than 3 Gbps per tunnel"]
    net_vpnmulti["Multiple HA VPN tunnels bonded — more bandwidth needed"]
    net_q2{"Are you co-located in<br/>a Google PoP / data center?"}
    net_dedicated["Dedicated Interconnect"]
    net_ded10["10 Gbps links (up to 8 = 80 Gbps)"]
    net_ded100["100 Gbps links (up to 8 = 800 Gbps)"]
    net_dedha["4 circuits, 2 metros — need max SLA 99.99%"]
    net_q3{"Does a Partner Interconnect provider<br/>serve your location?"}
    net_partner["Partner Interconnect"]
    net_partbw["50 Mbps to 10 Gbps options"]
    net_partl23["Layer 2 direct VLAN or Layer 3 provider-managed BGP"]
    net_partsla["99.99% SLA with redundant connections"]
    net_peering["Direct Peering<br/>BGP peering — best-effort, no SLA, no GCP VPC access"]
    net_bwstart("Additional Decision:<br/>Does your BW requirement exceed current connection?")
    net_bw1["HA VPN is cost-effective — less than 1 Gbps"]
    net_bw2["Evaluate VPN vs Partner Interconnect — 1 to 10 Gbps (break-even ~3 Gbps)"]
    net_bw3["Dedicated Interconnect almost always wins on cost + latency — more than 10 Gbps"]

    net_start --> net_q1
    net_q1 -->|NO - public internet acceptable| net_vpn
    net_vpn -->|BW less than 3 Gbps per tunnel| net_vpnsingle
    net_vpn -->|more bandwidth needed| net_vpnmulti
    net_q1 -->|YES - private dedicated connectivity| net_q2
    net_q2 -->|YES - co-located at Google PoP| net_dedicated
    net_dedicated -->|10 Gbps links| net_ded10
    net_dedicated -->|100 Gbps links| net_ded100
    net_dedicated -->|need max SLA 99.99%| net_dedha
    net_q2 -->|NO - not co-located| net_q3
    net_q3 -->|YES| net_partner
    net_partner -->|50 Mbps to 10 Gbps options| net_partbw
    net_partner -->|Layer 2 or Layer 3 options| net_partl23
    net_partner -->|99.99% SLA| net_partsla
    net_q3 -->|NO - only need Google public API access| net_peering
    net_bwstart -->|less than 1 Gbps| net_bw1
    net_bwstart -->|1 to 10 Gbps| net_bw2
    net_bwstart -->|more than 10 Gbps| net_bw3
```

---

## 5. Data Processing Service Selection

```mermaid
flowchart TD
    dp_start("START: Is your processing batch, streaming, or both?")
    dp_q1{"BATCH only?"}
    dp_q1a{"Existing Hadoop/Spark/Hive/Pig jobs?"}
    dp_dataproc["Dataproc (managed Hadoop/Spark)"]
    dp_ephemeral["Ephemeral clusters (create/process/delete) — best practice"]
    dp_longrun["Long-running cluster — use autoscaling"]
    dp_q1b{"Data already in BigQuery / GCS?<br/>Need SQL-based transformations?"}
    dp_bqbatch["BigQuery (SQL queries, scheduled queries, BQML)"]
    dp_bqdml["Transform + load — BigQuery DML or dbt on BigQuery"]
    dp_dfbatch["Dataflow (Apache Beam) — batch mode<br/>custom complex pipeline, Java/Python preference"]
    dp_q2{"STREAMING only or<br/>STREAMING + BATCH (unified)?"}
    dp_q2a{"Low-latency, custom transformations,<br/>complex windowing?"}
    dp_dfstream["Dataflow (Apache Beam) — streaming mode<br/>exactly-once semantics, auto-scaling workers"]
    dp_q2b{"Existing Spark Streaming jobs to migrate?"}
    dp_dpstream["Dataproc with Spark Structured Streaming"]
    dp_pubsub["Pub/Sub to BigQuery direct subscription<br/>or Pub/Sub to Dataflow to BigQuery<br/>ingest + simple transforms, store, query"]
    dp_q3{"What is the latency requirement?"}
    dp_rt["Dataflow streaming — real-time less than 1 second"]
    dp_nrt["Dataflow or Pub/Sub to BigQuery — near-real-time 1 to 60 seconds"]
    dp_min["Dataflow batch or Dataproc — minutes"]
    dp_hrs["BigQuery scheduled queries or Dataproc batch — hours / overnight"]
    dp_q4{"Data warehouse / analytics queries<br/>(not pipeline)?"}
    dp_bqdw["BigQuery"]
    dp_bqadhoc["Ad hoc SQL analysis — on-demand pricing"]
    dp_bqslots["Predictable heavy workloads — slot reservations (flat-rate)"]
    dp_bqml["ML on data — BigQuery ML (BQML)"]

    dp_start --> dp_q1
    dp_q1 -->|YES| dp_q1a
    dp_q1a -->|YES| dp_dataproc
    dp_dataproc -->|ephemeral clusters| dp_ephemeral
    dp_dataproc -->|long-running cluster| dp_longrun
    dp_q1a -->|NO| dp_q1b
    dp_q1b -->|YES| dp_bqbatch
    dp_bqbatch -->|transform + load| dp_bqdml
    dp_q1b -->|NO| dp_dfbatch
    dp_q1 -->|NO| dp_q2
    dp_q2 -->|YES| dp_q2a
    dp_q2a -->|YES| dp_dfstream
    dp_q2a -->|NO| dp_q2b
    dp_q2b -->|YES| dp_dpstream
    dp_q2b -->|NO| dp_pubsub
    dp_q2 -->|NO| dp_q3
    dp_q3 -->|real-time less than 1 second| dp_rt
    dp_q3 -->|near-real-time 1 to 60 seconds| dp_nrt
    dp_q3 -->|minutes| dp_min
    dp_q3 -->|hours / overnight| dp_hrs
    dp_q3 -->|DW / analytics use case| dp_q4
    dp_q4 -->|YES| dp_bqdw
    dp_bqdw -->|ad hoc SQL analysis| dp_bqadhoc
    dp_bqdw -->|predictable heavy workloads| dp_bqslots
    dp_bqdw -->|ML on data| dp_bqml
```

---

## 6. Migration Strategy Selection

```mermaid
flowchart TD
    mg_start("START: What are your migration goals?")
    mg_q1{"Fastest migration to cloud? Minimal changes?<br/>(time-to-migrate is priority over optimization)"}
    mg_liftshift["LIFT AND SHIFT (Rehost)"]
    mg_ls_vms["VMs as-is — Migrate to Virtual Machines (formerly Velostrata)"]
    mg_ls_db["Databases as-is — Cloud SQL or GCE + database software"]
    mg_ls_con["Containers as-is — GKE or Cloud Run (if already containerized)"]
    mg_q2{"Modernize the platform but keep application logic?<br/>(medium complexity, medium risk, medium benefit)"}
    mg_replatform["RE-PLATFORM (Lift-and-Optimize)"]
    mg_rp_mysql["MySQL on GCE to Cloud SQL (managed DB)"]
    mg_rp_app["App on VMs to App Engine or Cloud Run (containerize app)"]
    mg_rp_hadoop["Hadoop on VMs to Dataproc (managed Hadoop)"]
    mg_rp_cache["Self-managed cache to Memorystore"]
    mg_q3{"Redesign the architecture for cloud-native benefits?<br/>(high complexity, high risk, maximum benefit)"}
    mg_rearchitect["RE-ARCHITECT (Refactor / Cloud-native)"]
    mg_ra_mono["Monolith to Microservices on GKE/Cloud Run"]
    mg_ra_etl["Batch ETL to Streaming with Pub/Sub + Dataflow"]
    mg_ra_oracle["On-prem Oracle to Cloud Spanner (global)"]
    mg_ra_file["File-based workflow to GCS + Cloud Functions/Eventarc"]
    mg_factors("Decision Factors")
    mg_f1["Timeline pressure — Lift and Shift"]
    mg_f2["Budget constraints — Re-platform (targeted savings)"]
    mg_f3["Technical debt — Re-architect (long-term cost savings)"]
    mg_f4["Compliance requirements — Evaluate all options for data residency"]
    mg_f5["Vendor lock-in concerns — Re-architect with open standards (GKE, Dataflow/Beam)"]
    mg_risk("Risk Assessment")
    mg_r1["Business criticality HIGH — Start with Re-platform (lower risk than Re-architect)"]
    mg_r2["Application well-understood — Re-architect for max cloud benefit"]
    mg_r3["Application legacy / poor documentation — Lift and Shift first, then modernize"]

    mg_start --> mg_q1
    mg_q1 -->|YES| mg_liftshift
    mg_liftshift -->|VMs as-is| mg_ls_vms
    mg_liftshift -->|Databases as-is| mg_ls_db
    mg_liftshift -->|Containers as-is| mg_ls_con
    mg_q1 -->|NO| mg_q2
    mg_q2 -->|YES| mg_replatform
    mg_replatform -->|MySQL on GCE| mg_rp_mysql
    mg_replatform -->|App on VMs| mg_rp_app
    mg_replatform -->|Hadoop on VMs| mg_rp_hadoop
    mg_replatform -->|self-managed cache| mg_rp_cache
    mg_q2 -->|NO| mg_q3
    mg_q3 -->|YES| mg_rearchitect
    mg_rearchitect -->|Monolith| mg_ra_mono
    mg_rearchitect -->|Batch ETL| mg_ra_etl
    mg_rearchitect -->|On-prem Oracle| mg_ra_oracle
    mg_rearchitect -->|file-based workflow| mg_ra_file
    mg_factors -->|timeline pressure| mg_f1
    mg_factors -->|budget constraints| mg_f2
    mg_factors -->|technical debt| mg_f3
    mg_factors -->|compliance requirements| mg_f4
    mg_factors -->|vendor lock-in concerns| mg_f5
    mg_risk -->|business criticality HIGH| mg_r1
    mg_risk -->|application well-understood| mg_r2
    mg_risk -->|application legacy / poor documentation| mg_r3
```

---

## 7. Disaster Recovery Strategy Selection

```mermaid
flowchart TD
    dr_start("START: What are your RTO and RPO requirements?")
    dr_q1{"RPO: Hours acceptable?<br/>RTO: Hours acceptable?"}
    dr_backup["BACKUP AND RESTORE"]
    dr_bu_gcs["GCS object storage for backups"]
    dr_bu_sql["Cloud SQL automated backups"]
    dr_bu_cost["Lowest cost, highest recovery time"]
    dr_q2{"RPO: Minutes/Hours?<br/>RTO: Hours/Day?"}
    dr_pilot["PILOT LIGHT"]
    dr_pl_rep["Core data replicated continuously (Cloud SQL replicas)"]
    dr_pl_comp["Minimal compute provisioned (start VMs on failover)"]
    dr_pl_cost["Medium cost, medium recovery time"]
    dr_q3{"RPO: Seconds/Minutes?<br/>RTO: Minutes?"}
    dr_warm["WARM STANDBY"]
    dr_ws_env["Reduced-capacity secondary environment always running"]
    dr_ws_disk["Regional Persistent Disk for data replication"]
    dr_ws_scale["Scale up secondary on failover event"]
    dr_ws_cost["Higher cost, faster recovery"]
    dr_q4{"RPO: Near-zero?<br/>RTO: Seconds?"}
    dr_active["ACTIVE-ACTIVE / HOT STANDBY"]
    dr_aa_spanner["Cloud Spanner (global, always-on)"]
    dr_aa_multi["Multi-region Cloud Run / GKE (Global LB auto-failover)"]
    dr_aa_glb["Global External Application LB (auto-routes to healthy backends)"]
    dr_aa_cost["Highest cost, near-zero downtime"]

    dr_start --> dr_q1
    dr_q1 -->|YES| dr_backup
    dr_backup --> dr_bu_gcs
    dr_backup --> dr_bu_sql
    dr_backup --> dr_bu_cost
    dr_q1 -->|NO| dr_q2
    dr_q2 -->|YES| dr_pilot
    dr_pilot --> dr_pl_rep
    dr_pilot --> dr_pl_comp
    dr_pilot --> dr_pl_cost
    dr_q2 -->|NO| dr_q3
    dr_q3 -->|YES| dr_warm
    dr_warm --> dr_ws_env
    dr_warm --> dr_ws_disk
    dr_warm --> dr_ws_scale
    dr_warm --> dr_ws_cost
    dr_q3 -->|NO| dr_q4
    dr_q4 -->|YES| dr_active
    dr_active --> dr_aa_spanner
    dr_active --> dr_aa_multi
    dr_active --> dr_aa_glb
    dr_active --> dr_aa_cost
```
