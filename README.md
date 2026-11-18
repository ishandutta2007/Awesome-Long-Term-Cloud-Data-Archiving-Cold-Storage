# Awesome-Long-Term-Cloud-Data-Archiving-Cold-Storage

# Awesome-Long-Term-Cloud-Data-Archiving-Cold-Storage

# Awesome-Long-Term-Cloud-Data-Archiving-Cold-Storage

# Awesome Long-Term Cloud Data Archiving & Cold Storage

> A curated list of top Long-Term Cloud Data Archiving and Cold Storage platforms — both SaaS/Hosted commercial solutions and open-source alternatives for cost-efficient retention of inactive data at scale.
> 🌟 Star this repo if you find it useful! 🌟

---

## 📋 Table of Contents

- [SaaS / Hosted Platforms](#-saas--hosted-platforms)
  - [AWS S3 Glacier Deep Archive](#aws-s3-glacier-deep-archive)
  - [Azure Archive Storage](#azure-archive-storage)
  - [Google Cloud Archive Storage](#google-cloud-archive-storage)
  - [Wasabi Hot Cloud Storage](#wasabi-hot-cloud-storage)
  - [Backblaze B2](#backblaze-b2)
  - [Iron Mountain Cloud Archive](#iron-mountain-cloud-archive)
  - [Seagate Lyve Cloud](#seagate-lyve-cloud)
  - [Quantum ActiveScale](#quantum-activescale)
- [Open-Source Software](#-open-source-software)
  - [HuskHoard](#huskhoard)
  - [S3QL](#s3ql)
  - [Barry](#barry)
  - [Deepfreeze](#deepfreeze)

---

## 🟦 SaaS / Hosted Platforms

Commercial, cloud-hosted Long-Term Archiving and Cold Storage platforms. These are not open-source but are included as reference points for feature comparison.

### AWS S3 Glacier Deep Archive
**Type:** Lowest-cost cloud archive tier
AWS S3 Glacier Deep Archive is the cheapest storage class in the AWS ecosystem, designed for data that is accessed rarely and can tolerate retrieval times of up to 12 hours. It is the canonical cold storage tier against which other providers are compared. **Trade-off:** Retrieval costs and minimum storage durations apply, and egress fees are charged for data leaving AWS.

### Azure Archive Storage
**Type:** Low-cost archive tier within Azure Blob Storage
Azure Archive Storage provides a cold storage tier for long-term retention with retrieval times measured in hours. It integrates with the broader Azure data management ecosystem, including lifecycle policies for automatic tiering. **Trade-off:** Early deletion fees apply if data is removed before the minimum retention period.

### Google Cloud Archive Storage
**Type:** Cold storage tier within Google Cloud Storage
Google Cloud Archive Storage is the lowest-cost tier in GCS, designed for data accessed less than once a year. It supports Object Lifecycle Management for automatic transitions between storage classes. **Trade-off:** Minimum storage duration of 365 days applies, with retrieval fees for early access.

### Wasabi Hot Cloud Storage
**Type:** Flat-rate S3-compatible storage with no egress fees
Wasabi provides S3-compatible object storage at approximately **$0.0069/GB/month** with **no egress fees**, making it attractive for archive workloads where frequent restores or media streaming are expected [citation:1][citation:10]. **Critical caveat:** Wasabi enforces a **90-day minimum storage duration** per object and a **1 TB minimum billing**—objects deleted before 90 days still incur the remaining storage charge, and frequent data churn makes it expensive [citation:6][citation:10][citation:15]. Egress is free within reasonable-use limits, with alerting provided for anomalous usage [citation:1].  
**Best for:** Media delivery, video streaming archives, and large active datasets with frequent downloads where predictable flat-rate pricing is valuable.

### Backblaze B2
**Type:** Low-cost S3-compatible storage with metered egress
Backblaze B2 offers storage at approximately **$0.006/GB/month** with **no minimum storage duration** and **no minimum object age**, making it the most flexible option for variable workloads [citation:10][citation:15]. Egress is free up to **3x your total storage** per month, then **$0.01/GB**; free unlimited egress is available to Cloudflare and select CDN partners [citation:1][citation:15]. It has a native B2 API that Restic supports directly (preferred over its S3-compatibility layer for new repos) [citation:10].  
**Best for:** Variable backup workloads, personal archives, and teams wanting predictable per-GB billing without minimum-commitment surprises.

### Iron Mountain Cloud Archive
**Type:** Fully managed offline cold storage with cloud retrieval
Iron Mountain's Iron Cloud Secure Offline Storage (SOS) combines cloud object storage with **physical tape vaulting** for inactive data, with data retrieved from the climate-controlled vault within **24 hours** of request [citation:7]. Iron Mountain **does not charge egress fees**, and the service is positioned for regulatory and compliance-driven retention where data is rarely accessed but must be recoverable [citation:7]. The service is technology-agnostic, working with the customer's existing cloud providers.  
**Best for:** Organizations with strict compliance retention requirements (financial services, healthcare, legal) needing ultra-low-cost offline storage with managed retrieval SLAs.

### Seagate Lyve Cloud
**Type:** Mass-capacity cloud storage for active archives
Seagate Lyve Cloud provides S3-compatible object storage designed for mass-capacity active archives, leveraging Seagate's hardware heritage. It is positioned for media and entertainment, surveillance, and other data-intensive workloads requiring large-scale retention with predictable pricing.

### Quantum ActiveScale
**Type:** On-premises scale-out object storage with cold tier
Quantum ActiveScale is a scale-out object storage system with an **Active class** (on NVMe or HDD) and an optional **Cold class on tape** that is S3 Glacier-compatible [citation:11]. Lifecycle policies move objects between classes at **no storage, access, or restore charge** [citation:11]. It supports up to 19 nines of durability with erasure coding, and in a three-site deployment, data remains fully accessible even after a data center outage [citation:11]. ActiveScale Cold Storage combines object storage with tape for long-term retention, available as a service for a cloud-like experience on-premises [citation:2].  
**Best for:** Organizations that must keep archived data on their own premises for sovereignty, latency, or egress reasons, while still wanting S3-compatible access and cloud-like lifecycle management.

### Druva Archival Storage
**Type:** Automated cloud tiering for backup data
Druva automatically tiers aged backup data to lower-cost storage, including **Amazon S3 Infrequent Access and Amazon Glacier**, as part of its Hybrid Workloads platform [citation:3]. Its **Long Term Retention (LTR)** feature stores cold copies of backup data in **Amazon Glacier Deep Archive or Azure Cold Storage**, consuming fewer credits than active backups [citation:12]. **Important constraint:** LTR is limited to Enterprise and Elite editions, requires backup sets with retention of one year or more, and early deletion from the cold tier before 12 months incurs a fee of 0.35 × (12 - months in cold tier) × TB deleted [citation:12]. Druva bills on deduplicated data stored per TB/month, with dedupe efficiency directly affecting cost [citation:8].  
**Best for:** Existing Druva customers wanting automated, integrated archival tiering without managing separate cold storage workflows.

---

## 🟩 Open-Source Software

Self-hosted, community-driven Long-Term Archiving and Cold Storage tools. The open-source ecosystem focuses on **data tiering and stubbing**, **filesystem-based cloud archiving**, and **backup-to-cold-storage pipelines**.

### HuskHoard
**Stack:** Rust / Linux (fanotify) / SQLite | **License:** Open Source (GitHub)
HuskHoard is an open-source **data tiering archive for Linux** that transparently offloads cold data to **S3 buckets, HDDs, or LTO tape** while keeping files fully visible and accessible to the OS as searchable local stubs [citation:13]. Key characteristics:
- **Zero-overhead transparent stubbing**: Uses the Linux `fanotify` kernel API (no FUSE) to block and resume processes in real-time when a stubbed file is requested 
- **StreamGate HTTP Gateway**: Watch 4K video directly from tape or S3 via a local HTTP bridge, with indexed direct access via a jump table for instant seeking in 10TB files without downloading the whole thing 
- **Native SCSI Tape Driver**: Professional-grade support for LTO-5 through LTO-9 drives, handling hardware positioning and block alignment to prevent "shoe-shining" 
- **Open formats**: Payload data stored in standard Zstd streams verified by BLAKE3; catalog metadata exportable to Apache Parquet for DuckDB, Python, or Spark analysis 
- **No vendor lock-in**: "Easy Exit" promise with standard formats 

**Trade-off:** Requires Linux kernel 5.1+ and `fanotify` support; configuration and operation require Linux systems administration expertise.  
**Best for:** Linux administrators and data engineers wanting transparent tiering of cold data from expensive NVMe to cheap S3, HDD, or LTO tape without proprietary enterprise software.

### S3QL
**Stack:** Python / FUSE / S3 | **License:** GPL-3.0 (Debian package)
S3QL is a **full-featured UNIX filesystem** that stores all data online using storage services like **Amazon S3, Google Storage, or OpenStack**, providing a dynamically sized, infinite-capacity disk accessible from any computer with internet access [citation:9][citation:14]. Key characteristics for archival:
- **Standard conforming**: Conceptually indistinguishable from a local filesystem 
- **Compression, encryption, deduplication**: Features that make it especially suitable for online backup and archival 
- **Immutable trees and snapshotting**: Supports archival workflows 
- **Flexible backends**: Can store data underneath a regular mount point, enabling use with NFS, CIFS, or SFTP servers 

**Trade-off:** S3QL is a FUSE-based filesystem, which introduces performance overhead compared to native filesystems, and it is a **mountable archive** rather than a lifecycle-managed cold storage service.  
**Best for:** Users wanting a POSIX-compatible filesystem backed by cloud object storage with built-in encryption and deduplication, accepting FUSE overhead.

### Barry
**Stack:** Go | **License:** Open Source (Go package)
Barry is a client-server tool that **watches a local directory for new backups and uploads them to a cold-storage Swift service** (OVH-flavored) [citation:4]. It is a focused pipeline for moving backup artifacts into cold object storage. **Important status:** The project describes itself as a **work-in-progress** with a TODO file, indicating early-stage maturity [citation:4].  
**Best for:** Developers wanting a simple Go-based client-server pipeline for pushing local backups to OpenStack Swift-compatible cold storage.

### Deepfreeze
**Stack:** Python / Elasticsearch | **License:** Open Source (Elastic)
Deepfreeze is an open-source tool from Elastic for **archiving and restoring Elasticsearch indices to cloud object storage** (AWS S3, Azure Blob, Google Cloud Storage) [citation:5]. It manages the full lifecycle: setting up ILM policies and index templates, triggering index snapshots to cold storage, and restoring ("thawing") indices when needed. It supports multiple cloud providers with credential configuration via environment variables or YAML config [citation:5].  
**Best for:** Elasticsearch operators wanting to move aged indices to cheap cloud storage while retaining the ability to restore them on demand, accepting the Elasticsearch-specific scope.

---

## 📊 Comparison at a Glance

| Software | License | Stack | Cold Storage Type | Egress Fees | Self-Hosted | Best For |
|----------|---------|-------|-------------------|-------------|-------------|----------|
| **AWS Glacier Deep Archive** | Proprietary | Cloud | Cloud archive tier | ✅ Charged | ❌ | AWS-native lowest-cost archive |
| **Azure Archive** | Proprietary | Cloud | Cloud archive tier | ✅ Charged | ❌ | Azure-native archive |
| **Google Cloud Archive** | Proprietary | Cloud | Cloud archive tier | ✅ Charged | ❌ | GCP-native archive |
| **Wasabi** | Proprietary | Cloud | S3-compatible hot | ✅ None (fair use) | ❌ | Frequent restores, media |
| **Backblaze B2** | Proprietary | Cloud | S3-compatible | ✅ Metered (3x free) | ❌ | Variable backup workloads |
| **Iron Mountain** | Proprietary | Cloud + Tape | Offline vault | ✅ None | ❌ | Compliance retention |
| **Quantum ActiveScale** | Proprietary | On-prem | Tape cold class | N/A (on-prem) | ✅ (Appliance) | Sovereignty + S3 on-prem |
| **Druva LTR** | Proprietary | Cloud | Glacier Deep Archive | (Integrated) | ❌ | Druva customers automated tiering |
| **HuskHoard** | Open Source | Rust/Linux | S3, HDD, LTO tape | (S3 dependent) | ✅ | Transparent tiering without vendor lock-in |
| **S3QL** | GPL-3.0 | Python/FUSE | S3-compatible | (Provider dependent) | ✅ | POSIX filesystem on S3 |
| **Deepfreeze** | Open Source | Python | AWS/Azure/GCP | (Provider dependent) | ✅ | Elasticsearch index archiving |

---

## 🤝 Contributing

Contributions are welcome! If you know of an open-source Long-Term Archiving or Cold Storage tool that should be listed here, please open a pull request with:

- Project name and URL
- Brief description (1–2 sentences)
- Stack and license
- Why it stands out

Please ensure the project is actively maintained (releases within the last 12 months) and genuinely open-source.

**Note on the open-source ecosystem:** The open-source landscape for **long-term cloud archiving** is **focused on infrastructure and pipeline layers** rather than managed cold storage services. **HuskHoard** is the most ambitious open-source effort, providing transparent data tiering from expensive local storage to S3, HDD, or LTO tape with indexed direct access and open formats—explicitly designed as a free alternative to enterprise tiering software [citation:13]. **S3QL** provides a POSIX filesystem abstraction over cloud object storage with built-in encryption and deduplication for archival use cases [citation:9][citation:14]. **Barry** and **Deepfreeze** address specific pipelines: pushing backups to Swift-compatible cold storage and archiving Elasticsearch indices to cloud buckets respectively [citation:4][citation:5]. No open-source project replicates the fully managed cold storage services of AWS Glacier or Iron Mountain, but these tools cover the core archiving workflows for self-hosted deployments.

---

## 📄 License

This list is released under CC0 1.0 Universal (Public Domain). The software projects listed retain their respective licenses.

---

⭐ **If this list helped you discover a useful tool, please star the repo!** ⭐
