# Backup & Recovery Strategy HashiCorp Vault with Integrated Raft Storage on AWS

This repository provides a complete HA and Disaster Recovery strategy for HashiCorp Vault on AWS. It includes multi-AZ and multi-region Raft-based architectures, automated backup and restore procedures, AWS integration (KMS, S3, EC2), and failover workflow

Vault, which is used by many applications, infrastructure components, and services to generate tokens, encrypt and decrypt data, and store sensitive information, it is a critical service. If Vault fails, it can impact all dependent services and infrastructure.
To reduce risk, we recommend separating Vault clusters by use case — for example, one Vault cluster dedicated to your EKS deployments, and another for managing infrastructure secrets.
Given its importance, RTO and RPO should be zero or close to zero, ensuring no downtime or data loss. Therefore, we recommend a Multi-Site Active-Active architecture, with High Availability (HA) deployment across multiple AWS Availability Zones


## about Backup & Recovery 

#### Backup:
Backup is designed to protect against host failures and malicious attacks (such as ransomware that encrypts data). It captures data at a specific point in time — for example, through snapshots stored in a remote or secure location.
#### Disaster Recovery (DR):
Disaster recovery focuses on critical production applications and protects against major failures, such as a full AWS region outage where your production environment is hosted. The DR site typically remains in a standby state, ready to take over.
Disaster recovery involves continuous data replication (streaming) and aims for minimal downtime — especially for applications that are critical.

#### Key Factors to Identify:
-	RTO (Recovery Time Objective): How quickly must you recover?
-	RPO (Recovery Point Objective): How much data can you afford to lose?
-	Data criticality & size
-	Compliance requirements
-	Budget & complexity tolerance


## Evaluate AWS Disaster Recovery Strategies

| **Strategy**                 | **RTO/RPO**     | **Use Cases**                             | **Cost**   |
|-----------------------------|-----------------|-------------------------------------------|------------|
| Backup & Restore            | Hours           | Non-critical data, archival               | 💲         |
| Pilot Light                 | 10–30 minutes   | Core databases, moderate RTO/RPO          | 💲💲        |
| Warm Standby                | Minutes         | Critical apps, low downtime               | 💲💲💲       |
| Multi-Site (Active-Active)  | Near-zero       | Mission-critical, always-on workloads     | 💲💲💲💲      |


Photo 1 

Photo 2 


#### Multi-AZ Cluster with Raft Storage

- Deploy **5 Vault nodes across 3 AWS Availability Zones (AZs)** to tolerate the simultaneous failure of **2 nodes or an entire AZ**.
- Use **integrated Raft storage** for synchronous data replication between nodes, eliminating the dependency on **Consul**.
- Set up a **Network Load Balancer** in front of the cluster to distribute traffic and handle node failures.

#### Multi-Region Replication

- *Performance Replication (Enterprise):*  
  Synchronize secrets between AWS clusters in different regions (e.g., `eu-west-1` and `us-east-1`) to reduce latency and balance the load.

- *Disaster Recovery Replication:*  
  Maintain a **standby cluster** in another region, capable of taking over in **under 1 minute** using **pre-generated DR tokens**.




#### Key AWS Integration

##### Auto-Scaling and Instance Management
- Use **EC2 Auto Scaling Groups** with **preconfigured AMIs** to automatically replace failed nodes.

##### Encryption and Key Management
- Integrate **AWS KMS** for **automatic cluster unsealing** using **encrypted unseal keys**.

##### Immutable Backups
- Store **Raft snapshots** in **S3** with **versioning enabled**.
- Archive snapshots in **Glacier** for **long-term retention**.

##### Best Practices
- ✅ Run **regular DR drills** to test snapshot restoration and the unseal process.
- 📊 Monitor **Vault health** (`vault status`, metrics) using **Amazon CloudWatch** or **Prometheus**.


## Automated Disaster Recovery Workflow

### Failure Detection
- Monitor clusters with **Amazon CloudWatch** (CPU/RAM metrics, node status).
- Trigger **Lambda Functions** if health checks fail for **more than 2 minutes**.

### Traffic Redirection
- Update **Route 53 records** with **weighted routing policies** to switch to the DR cluster.
- Use **Vault Agent** on application instances to **dynamically reload Vault endpoints**.


## Technical Points: Backup & Recovery Strategy

### 1. Raft Snapshot-Based Backup

- Schedule **periodic snapshots** of the Raft data store using Vault’s built-in commands.
- Store these snapshots in a **secure location**, such as **Amazon S3**, with **encryption enabled via AWS KMS**.
- This ensures the snapshots are protected from **unauthorized access and tampering**.

**Example Commands:**

```bash
vault operator raft snapshot save /path/to/snapshot.snap
vault operator raft snapshot restore /path/to/snapshot.snap

## References

- [Vault Raft Cluster High Availability Test in AWS | TeKanAid](https://www.tekanaid.com/posts/vault-raft-cluster-high-availability-test-in-aws)
- [Disaster Recovery (DR) Architecture on AWS, Part IV: Multi-site Active/Active | AWS Architecture Blog](https://aws.amazon.com/blogs/architecture/disaster-recovery-dr-architecture-on-aws-part-iv-multi-site-activeactive/)



