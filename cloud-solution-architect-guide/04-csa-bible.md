# Cloud Solution Architect Bible
## The Complete Reference Guide

---

## 🟢 Plain English Summary — Read This First

### What is this document about?
This is the master reference guide — the "bible" for everything a Cloud Solution Architect needs to know. It covers AWS, Azure, Kubernetes, networking, security, and system design patterns all in one place. Think of it as your encyclopedia — you don't read it cover to cover, you refer to specific sections when you need them.

---

### Abbreviation Decoder

| Abbreviation | Full Name | Plain English |
|---|---|---|
| NFR | Non-Functional Requirement | How the system behaves (speed, reliability, security) vs what it does (features). |
| SLA | Service Level Agreement | A contract: "We guarantee 99.9% uptime or you get a refund." |
| RTO | Recovery Time Objective | Max acceptable downtime. "We can be down for 1 hour max." |
| RPO | Recovery Point Objective | Max acceptable data loss. "We can lose 5 minutes of data max." |
| MTTR | Mean Time to Recovery | Average time to fix something when it breaks. |
| MTBF | Mean Time Between Failures | Average time between breakdowns. Higher = more reliable. |
| VPC | Virtual Private Cloud | AWS's private network in the cloud. Your own isolated section of AWS. |
| VNet | Virtual Network | Azure's version of VPC. |
| IGW | Internet Gateway | The door that lets your VPC talk to the internet. |
| NAT | Network Address Translation | Lets private servers access the internet without being directly exposed to it. Like a receptionist who makes calls on your behalf. |
| NACL | Network Access Control List | A firewall at the subnet level in AWS. Stateless — you must explicitly allow both incoming AND outgoing traffic. |
| SG | Security Group | A firewall at the instance level in AWS. Stateful — if you allow incoming traffic, the response is automatically allowed out. |
| NSG | Network Security Group | Azure's version of Security Groups. |
| ALB | Application Load Balancer | AWS load balancer that understands HTTP/HTTPS. Can route based on URL path or hostname. |
| NLB | Network Load Balancer | AWS load balancer for TCP/UDP. Extremely fast, handles millions of requests per second. |
| CDN | Content Delivery Network | Servers around the world that cache your content close to users. |
| ASG | Auto Scaling Group | AWS feature that automatically adds or removes servers based on demand. |
| ECS | Elastic Container Service | AWS's service for running Docker containers. |
| EKS | Elastic Kubernetes Service | AWS's managed Kubernetes service. |
| ECR | Elastic Container Registry | AWS's private Docker image storage. |
| RDS | Relational Database Service | AWS's managed database service (MySQL, PostgreSQL, etc.). |
| DLQ | Dead Letter Queue | A queue where failed messages go after too many retry attempts. Like a "failed mail" bin. |
| FIFO | First In, First Out | Messages are processed in the exact order they were received. |
| KMS | Key Management Service | AWS's service for managing encryption keys. |
| CMK | Customer Managed Key | An encryption key that YOU control (not AWS). You can delete it, rotate it, restrict who uses it. |
| SSE | Server-Side Encryption | The cloud automatically encrypts your data before storing it. |
| IAM | Identity and Access Management | Controls who can log in and what they're allowed to do. |
| IRSA | IAM Roles for Service Accounts | A way for Kubernetes pods to securely access AWS services without storing passwords. |
| OIDC | OpenID Connect | A standard for identity verification. Used by Kubernetes to trust AWS IAM. |
| WAF | Web Application Firewall | Filters malicious web traffic (SQL injection, XSS attacks, etc.) before it reaches your app. |
| DDoS | Distributed Denial of Service | An attack where thousands of computers flood your service with fake requests to overwhelm it. |
| CSPM | Cloud Security Posture Management | A tool that continuously checks your cloud setup for security weaknesses. |
| SIEM | Security Information and Event Management | A system that collects and analyzes security logs from everywhere to detect threats. |
| SOAR | Security Orchestration, Automation and Response | Automatically responds to security threats (e.g., automatically blocks a suspicious IP). |
| PII | Personally Identifiable Information | Data that can identify a person (name, email, SSN, etc.). Must be protected by law. |
| TDE | Transparent Data Encryption | Automatically encrypts database files on disk. |
| mTLS | Mutual TLS | Both sides of a connection prove their identity with certificates. Not just the server — the client too. |
| SPIFFE | Secure Production Identity Framework For Everyone | A standard for giving services a cryptographic identity (like a passport for microservices). |
| CNI | Container Network Interface | A plugin system that handles networking for containers in Kubernetes. |
| PV | Persistent Volume | Storage in Kubernetes that survives pod restarts. |
| PVC | Persistent Volume Claim | A pod's request for storage. |
| HPA | Horizontal Pod Autoscaler | Automatically adds/removes pods based on CPU/memory usage. |
| VPA | Vertical Pod Autoscaler | Automatically adjusts how much CPU/memory a pod gets. |
| KEDA | Kubernetes Event-Driven Autoscaling | Scales pods based on external events (queue depth, etc.). |
| CRD | Custom Resource Definition | A way to extend Kubernetes with your own resource types. |
| OPA | Open Policy Agent | A policy engine used to enforce rules in Kubernetes (e.g., "all pods must have resource limits"). |
| FinOps | Financial Operations | The practice of managing and optimizing cloud costs. |
| TCO | Total Cost of Ownership | The full cost of something including hidden costs (maintenance, operations, etc.). |
| CQRS | Command Query Responsibility Segregation | Separate the "write" path from the "read" path for better performance. |
| CDC | Change Data Capture | Capturing every change made to a database and streaming those changes elsewhere. |
| ETL | Extract, Transform, Load | Moving data from one place to another, transforming it along the way. |
| ELT | Extract, Load, Transform | Load raw data first, then transform it in the destination. |
| OLTP | Online Transaction Processing | Databases optimized for many small, fast transactions (e.g., banking). |
| OLAP | Online Analytical Processing | Databases optimized for complex queries over large datasets (e.g., reporting). |
| ADR | Architecture Decision Record | A document that records WHY an architecture decision was made. |
| C4 | Context, Container, Component, Code | A model for drawing architecture diagrams at 4 levels of detail. |

---

### The Well-Architected Framework in Plain English

Both AWS and Azure have a "Well-Architected Framework" — a checklist of best practices organized into pillars:

**AWS has 6 pillars:**
1. **Operational Excellence** — Can you run and monitor the system? Can you improve it over time?
2. **Security** — Is data protected? Is access controlled?
3. **Reliability** — Does it recover from failures? Can it handle demand spikes?
4. **Performance Efficiency** — Are you using the right resources for the job?
5. **Cost Optimization** — Are you wasting money anywhere?
6. **Sustainability** — Are you minimizing environmental impact?

**Azure has 5 pillars** (same ideas, slightly different grouping).

In an interview, if asked "how would you design X?", always mention which pillars your design addresses.

---

### Three-Tier Architecture in Plain English

Almost every web application follows this pattern:

```
User's browser
    ↓
CDN (serves cached content from nearby server)
    ↓
Load Balancer (distributes traffic across multiple servers)
    ↓
Application Servers (your actual code runs here)
    ↓
Cache (Redis — fast in-memory storage for frequent queries)
    ↓
Database (the permanent store of truth)
    ↓
Object Storage (S3/Blob — for files, images, backups)
```

Each layer can be scaled independently. If your app is slow, you add more application servers. If your database is slow, you add read replicas or caching.

---

> Single reference guide for AWS, Azure, Kubernetes, Networking, Security, Observability, and System Design. Use this periodically to reinforce and deepen your knowledge.

---

# PART 1: ARCHITECT FUNDAMENTALS

## 1.1 The Architect's Role

A Cloud Solution Architect:
- Translates business requirements into technical architecture
- Owns non-functional requirements (NFRs): scalability, reliability, security, performance, cost
- Makes and documents trade-off decisions
- Communicates across technical and non-technical stakeholders
- Does NOT write production code (but must understand it deeply)

## 1.2 Non-Functional Requirements (NFRs) — The Architect's Domain

Always clarify these before designing:

| NFR | Questions to Ask |
|-----|-----------------|
| Scalability | How many users? Peak vs average? Growth rate? |
| Availability | What's the SLA? 99.9% = 8.7h downtime/year, 99.99% = 52min/year |
| Latency | P50, P95, P99 requirements? Read vs write latency? |
| Durability | Can we lose data? RPO = 0 means no data loss |
| Security | Compliance requirements? PII data? Encryption needs? |
| Cost | Budget constraints? Cost per transaction target? |
| Maintainability | Team size? Deployment frequency? On-call burden? |

## 1.3 CAP Theorem

In a distributed system, you can only guarantee 2 of 3:
- **C**onsistency — Every read gets the most recent write
- **A**vailability — Every request gets a response (not necessarily latest data)
- **P**artition Tolerance — System works despite network partitions

**Real-world application:**
- Network partitions ALWAYS happen → you must choose CP or AP
- CP systems: HBase, Zookeeper, etcd (Kubernetes uses this)
- AP systems: Cassandra, DynamoDB (eventual consistency), CouchDB
- CA systems: Traditional RDBMS (only in single-node, no partition tolerance)

## 1.4 PACELC

Extends CAP: Even without partitions, there's a trade-off between Latency and Consistency.
- P → AC (partition: availability vs consistency)
- ELC → latency vs consistency (normal operation)

DynamoDB: PA/EL (available during partition, low latency normally)
Spanner: PC/EC (consistent during partition, consistent normally)

## 1.5 Consistency Models

From weakest to strongest:
1. **Eventual Consistency** — Will converge, but reads may be stale (DynamoDB default)
2. **Monotonic Read** — Once you read a value, you won't read older values
3. **Read-Your-Writes** — You always see your own writes
4. **Session Consistency** — Consistency within a session
5. **Bounded Staleness** — Data is at most X seconds old
6. **Strong Consistency** — Every read sees the latest write (DynamoDB with ConsistentRead=true)
7. **Linearizability** — Strongest: operations appear instantaneous

## 1.6 Well-Architected Framework

### AWS Well-Architected (6 Pillars)
1. **Operational Excellence** — Run and monitor systems, improve processes
2. **Security** — Protect data, systems, assets
3. **Reliability** — Recover from failures, meet demand
4. **Performance Efficiency** — Use resources efficiently
5. **Cost Optimization** — Avoid unnecessary costs
6. **Sustainability** — Minimize environmental impact

### Azure Well-Architected (5 Pillars)
1. **Reliability** — Resiliency, availability, recovery
2. **Security** — Confidentiality, integrity, availability
3. **Cost Optimization** — Manage and reduce costs
4. **Operational Excellence** — Operations processes
5. **Performance Efficiency** — Scaling and performance

---

# PART 2: AWS ARCHITECTURE

## 2.1 AWS Global Infrastructure

- **Regions**: Geographic areas (us-east-1, eu-west-1, ap-southeast-1)
- **Availability Zones (AZs)**: 2-6 isolated data centers per region
- **Edge Locations**: 400+ for CloudFront CDN
- **Local Zones**: Low-latency extension of regions
- **Wavelength Zones**: Ultra-low latency for 5G networks
- **Outposts**: AWS hardware in your data center

**Design principle**: Always deploy across minimum 2 AZs for HA, 3 AZs for production.

## 2.2 AWS Networking

### VPC (Virtual Private Cloud)
```
VPC (10.0.0.0/16)
├── Public Subnet AZ-a (10.0.1.0/24)  ← Internet Gateway attached
│   ├── NAT Gateway
│   └── Load Balancer
├── Public Subnet AZ-b (10.0.2.0/24)
├── Private Subnet AZ-a (10.0.3.0/24) ← No direct internet
│   └── Application servers
├── Private Subnet AZ-b (10.0.4.0/24)
├── Data Subnet AZ-a (10.0.5.0/24)    ← Most restricted
│   └── RDS, ElastiCache
└── Data Subnet AZ-b (10.0.6.0/24)
```

### Key Networking Components
- **Internet Gateway (IGW)**: Allows public internet access to/from VPC
- **NAT Gateway**: Allows private subnet resources to initiate outbound internet (not inbound)
- **Security Groups**: Stateful firewall at instance level (allow rules only)
- **NACLs**: Stateless firewall at subnet level (allow + deny rules)
- **VPC Peering**: Direct connection between 2 VPCs (non-transitive)
- **Transit Gateway**: Hub-and-spoke for connecting many VPCs and on-prem
- **PrivateLink**: Private connectivity to AWS services without internet
- **Direct Connect**: Dedicated physical connection to AWS (1Gbps, 10Gbps, 100Gbps)

### Route 53
- **Simple routing**: Single resource
- **Weighted routing**: A/B testing, gradual migration
- **Latency routing**: Route to lowest latency region
- **Failover routing**: Active-passive DR
- **Geolocation routing**: Route based on user location
- **Geoproximity routing**: Route based on geographic distance
- **Multi-value routing**: Multiple healthy resources

## 2.3 AWS Compute

### EC2
- Instance families: General (M), Compute (C), Memory (R), Storage (I/D), GPU (P/G)
- Purchasing: On-Demand, Reserved (1-3yr), Spot (up to 90% off, interruptible), Dedicated
- Auto Scaling Groups: min/max/desired, scaling policies (target tracking, step, scheduled)
- Launch Templates: versioned, supports mixed instance types

### ECS (Elastic Container Service)
- **EC2 launch type**: You manage the EC2 instances
- **Fargate launch type**: Serverless containers, AWS manages infrastructure
- Task Definition: Blueprint for containers (CPU, memory, networking, volumes)
- Service: Maintains desired count of tasks, integrates with ALB
- Use ECS when: Simple container workloads, team familiar with AWS-native tools

### EKS (Elastic Kubernetes Service)
- Managed Kubernetes control plane
- Node groups: managed (AWS handles patching) or self-managed
- Fargate profiles: serverless pods
- Networking: VPC CNI plugin (pods get VPC IPs), Calico for network policies
- Use EKS when: Kubernetes expertise, multi-cloud portability, complex workloads

### Lambda
- Event-driven, serverless functions
- Max 15 min execution, 10GB memory, 10GB ephemeral storage
- Cold start: 100ms-1s (mitigate with provisioned concurrency)
- Triggers: API Gateway, S3, DynamoDB Streams, SQS, SNS, EventBridge, Kinesis
- Use Lambda when: Event-driven, unpredictable traffic, short-lived tasks

### Decision Matrix: EC2 vs ECS vs EKS vs Lambda

| Factor | EC2 | ECS/Fargate | EKS | Lambda |
|--------|-----|-------------|-----|--------|
| Control | Full | Medium | High | None |
| Ops overhead | High | Low | Medium | None |
| Startup time | Minutes | Seconds | Seconds | Milliseconds |
| Cost model | Per hour | Per task | Per node | Per invocation |
| Best for | Legacy, full control | Simple containers | K8s workloads | Event-driven |

## 2.4 AWS Storage

### S3
- **Storage classes**: Standard, Intelligent-Tiering, Standard-IA, One Zone-IA, Glacier Instant, Glacier Flexible, Glacier Deep Archive
- **Durability**: 11 nines (99.999999999%)
- **Consistency**: Strong read-after-write consistency (since 2020)
- **Features**: Versioning, lifecycle policies, replication (CRR/SRR), event notifications, presigned URLs
- **Security**: Bucket policies, ACLs, Block Public Access, SSE-S3/SSE-KMS/SSE-C

### EBS (Elastic Block Store)
- Block storage attached to EC2
- Types: gp3 (general), io2 (high IOPS), st1 (throughput), sc1 (cold)
- Snapshots stored in S3, can copy cross-region
- Multi-attach: io1/io2 can attach to multiple EC2 (same AZ)

### EFS (Elastic File System)
- Managed NFS, shared across multiple EC2 instances
- Scales automatically, pay per GB used
- Performance modes: General Purpose, Max I/O
- Storage classes: Standard, Infrequent Access

### Storage Decision Matrix

| Use Case | Service |
|----------|---------|
| Object storage, static files, backups | S3 |
| Block storage for single EC2 | EBS |
| Shared file system across EC2 | EFS |
| High-performance shared storage | FSx for Lustre |
| Windows file shares | FSx for Windows |

## 2.5 AWS Databases

### RDS (Relational Database Service)
- Engines: MySQL, PostgreSQL, MariaDB, Oracle, SQL Server
- Multi-AZ: Synchronous standby replica (automatic failover ~1-2 min)
- Read Replicas: Asynchronous, up to 5 (15 for Aurora), can be cross-region
- Automated backups: 1-35 days retention, point-in-time recovery

### Aurora
- MySQL/PostgreSQL compatible, 5x faster than MySQL
- Storage: Auto-scales 10GB to 128TB, 6 copies across 3 AZs
- Aurora Serverless v2: Auto-scales compute
- Global Database: Cross-region replication <1 second lag
- Aurora Multi-Master: Multiple write nodes (same region)

### DynamoDB
- Fully managed NoSQL, key-value and document
- Single-digit millisecond latency at any scale
- Capacity modes: Provisioned (predictable) or On-Demand (unpredictable)
- Global Tables: Multi-region, multi-active replication
- DynamoDB Streams: Change data capture
- DAX: In-memory cache, microsecond latency

### ElastiCache
- Redis: Rich data structures, pub/sub, persistence, clustering
- Memcached: Simple caching, multi-threaded
- Use Redis for: Session store, leaderboards, pub/sub, complex data
- Use Memcached for: Simple object caching, horizontal scaling

## 2.6 AWS Messaging

### SQS (Simple Queue Service)
- Standard: At-least-once delivery, best-effort ordering, unlimited throughput
- FIFO: Exactly-once processing, strict ordering, 3000 msg/sec with batching
- Visibility timeout: Message hidden while being processed
- Dead Letter Queue (DLQ): Failed messages after max retries
- Long polling: Reduces empty responses, more efficient

### SNS (Simple Notification Service)
- Pub/Sub: One message to many subscribers
- Subscribers: SQS, Lambda, HTTP/S, Email, SMS, Mobile Push
- Fan-out pattern: SNS → multiple SQS queues
- FIFO topics: Ordered, deduplication

### EventBridge
- Serverless event bus
- Event sources: AWS services, custom apps, SaaS (Salesforce, Zendesk)
- Rules: Pattern matching → target routing
- Schema registry: Discover and manage event schemas
- Use EventBridge for: Event-driven architectures, SaaS integrations

### Kinesis
- **Data Streams**: Real-time streaming, 1MB/sec per shard, 7-day retention
- **Data Firehose**: Load streaming data to S3, Redshift, OpenSearch (no code)
- **Data Analytics**: SQL on streaming data
- Use Kinesis when: Real-time analytics, log aggregation, IoT data

### SQS vs SNS vs EventBridge vs Kinesis

| Service | Pattern | Use Case |
|---------|---------|----------|
| SQS | Queue (pull) | Decoupling, work queues |
| SNS | Pub/Sub (push) | Fan-out notifications |
| EventBridge | Event bus | Event-driven, SaaS integration |
| Kinesis | Streaming | Real-time analytics, high throughput |

## 2.7 AWS Security

### IAM
- **Users**: Long-term credentials (avoid for applications)
- **Roles**: Temporary credentials, assumed by services/users
- **Policies**: JSON documents defining permissions
- **Groups**: Collection of users with shared policies
- Principle of least privilege: Grant minimum required permissions
- IAM Conditions: Restrict by IP, MFA, time, resource tags

### KMS (Key Management Service)
- Customer Managed Keys (CMK): You control rotation, deletion
- AWS Managed Keys: AWS manages, you can't delete
- Envelope encryption: Data encrypted with data key, data key encrypted with CMK
- Key policies + IAM policies both required for access

### Secrets Manager
- Store and rotate secrets (DB passwords, API keys)
- Automatic rotation with Lambda
- Cross-account access via resource policies
- vs Parameter Store: Secrets Manager has auto-rotation, costs more

### Security Services
- **GuardDuty**: Threat detection (ML-based, analyzes CloudTrail, VPC Flow Logs, DNS)
- **Security Hub**: Aggregates findings from GuardDuty, Inspector, Macie
- **Inspector**: Vulnerability scanning for EC2, ECR, Lambda
- **Macie**: PII detection in S3
- **WAF**: Web Application Firewall (SQL injection, XSS, rate limiting)
- **Shield**: DDoS protection (Standard: free, Advanced: $3000/month)
- **CloudTrail**: API audit log (who did what, when, from where)

---

# PART 3: AZURE ARCHITECTURE

## 3.1 Azure Global Infrastructure

- **Regions**: 60+ regions globally (paired regions for DR)
- **Availability Zones**: 3 physically separate zones per region
- **Region Pairs**: Each region paired with another (e.g., East US ↔ West US)
- **Sovereign Clouds**: Azure Government, Azure China, Azure Germany

**Design principle**: Use Availability Zones for HA, Region Pairs for DR.

## 3.2 Azure Resource Hierarchy

```
Azure AD Tenant
└── Management Groups (org-level governance)
    └── Subscriptions (billing + access boundary)
        └── Resource Groups (lifecycle management)
            └── Resources (VMs, databases, etc.)
```

- **Management Groups**: Apply policies and RBAC across subscriptions
- **Subscriptions**: Billing unit, service limits apply per subscription
- **Resource Groups**: Logical container, resources share lifecycle
- **Azure Policy**: Enforce standards (e.g., "all resources must have tags")
- **Azure Blueprints**: Package of policies, RBAC, ARM templates for governance

## 3.3 Azure Networking

### VNet (Virtual Network)
```
VNet (10.0.0.0/16)
├── Public Subnet (10.0.1.0/24)
│   └── Application Gateway / Azure Firewall
├── App Subnet (10.0.2.0/24)
│   └── App Service / AKS nodes
└── Data Subnet (10.0.3.0/24)
    └── Azure SQL, Cosmos DB
```

### Key Networking Components
- **NSG (Network Security Group)**: Stateful firewall at subnet or NIC level
- **Azure Firewall**: Managed, stateful firewall with threat intelligence
- **VNet Peering**: Connect VNets (same or cross-region)
- **Hub-Spoke Topology**: Central hub VNet with shared services, spoke VNets for workloads
- **Azure Virtual WAN**: Managed hub-spoke for large enterprises
- **Private Link**: Private connectivity to Azure PaaS services
- **Private Endpoints**: Network interface for Private Link
- **ExpressRoute**: Dedicated private connection to Azure (50Mbps to 100Gbps)
- **VPN Gateway**: Site-to-site or point-to-site VPN

### Azure DNS
- Public DNS zones: Manage DNS records for internet-facing domains
- Private DNS zones: Name resolution within VNets
- Azure DNS Private Resolver: Conditional forwarding for hybrid DNS

### Load Balancing in Azure

| Service | Layer | Scope | Use Case |
|---------|-------|-------|----------|
| Azure Load Balancer | L4 | Regional | TCP/UDP load balancing |
| Application Gateway | L7 | Regional | HTTP/S, WAF, SSL termination |
| Azure Front Door | L7 | Global | Global HTTP/S, CDN, WAF |
| Traffic Manager | DNS | Global | DNS-based routing, failover |

**Request flow for a global web app:**
```
User → Azure Front Door (global routing + WAF + CDN)
     → Application Gateway (regional L7 LB + WAF)
     → App Service / AKS
     → Azure SQL / Cosmos DB
```

## 3.4 Azure Compute

### Azure Virtual Machines
- Series: B (burstable), D (general), E (memory), F (compute), N (GPU), L (storage)
- Availability: Availability Sets (fault/update domains) or Availability Zones
- VMSS (Virtual Machine Scale Sets): Auto-scaling VMs
- Spot VMs: Up to 90% discount, evictable

### Azure App Service
- PaaS for web apps, APIs, mobile backends
- Tiers: Free, Shared, Basic, Standard, Premium, Isolated
- Deployment slots: Blue-green deployments
- Auto-scaling, custom domains, SSL, VNet integration

### Azure Functions
- Serverless compute, event-driven
- Triggers: HTTP, Timer, Blob, Queue, Service Bus, Event Hub, Cosmos DB
- Durable Functions: Stateful workflows (orchestrator + activity functions)
- Consumption plan (serverless) vs Premium plan (pre-warmed) vs Dedicated

### AKS (Azure Kubernetes Service)
- Managed Kubernetes control plane (free)
- Node pools: System pool (critical pods) + User pools (workloads)
- Networking: kubenet (basic) or Azure CNI (pods get VNet IPs)
- Azure CNI Overlay: Pods get overlay IPs, more scalable
- Cluster autoscaler + KEDA (event-driven autoscaling)
- Azure AD integration for RBAC
- Azure Policy for Kubernetes (OPA Gatekeeper)

## 3.5 Azure Storage

### Azure Blob Storage
- Tiers: Hot (frequent), Cool (infrequent, 30-day min), Archive (rare, 180-day min)
- Types: Block blobs (files), Append blobs (logs), Page blobs (VHDs)
- Redundancy: LRS, ZRS, GRS, GZRS, RA-GRS, RA-GZRS
- Lifecycle management: Auto-tier or delete based on rules
- Soft delete, versioning, immutable storage (WORM)

### Azure Files
- Managed SMB/NFS file shares
- Azure File Sync: Sync on-prem file servers with Azure Files

### Azure Disks
- Types: Ultra (highest IOPS), Premium SSD, Standard SSD, Standard HDD
- Managed disks: Azure manages storage account
- Shared disks: Attach to multiple VMs (clustering)

## 3.6 Azure Databases

### Azure SQL Database
- Fully managed SQL Server
- Service tiers: DTU-based (Basic/Standard/Premium) or vCore-based
- Elastic pools: Share resources across multiple databases
- Hyperscale: Up to 100TB, fast backups
- Business Critical: In-memory OLTP, read replicas

### Azure Cosmos DB
- Globally distributed, multi-model NoSQL
- APIs: SQL (Core), MongoDB, Cassandra, Gremlin (graph), Table
- Consistency levels: Strong, Bounded Staleness, Session, Consistent Prefix, Eventual
- Multi-region writes: Any region can accept writes
- Serverless or provisioned throughput (RU/s)

### Azure Cache for Redis
- Managed Redis
- Tiers: Basic, Standard (replication), Premium (clustering, persistence)
- Use cases: Session cache, data cache, message broker

## 3.7 Azure Messaging

### Azure Service Bus
- Enterprise messaging: queues and topics (pub/sub)
- Features: FIFO, sessions, dead-letter queue, scheduled messages, transactions
- Use for: Reliable message delivery, decoupling services

### Azure Event Hubs
- Big data streaming platform (like Kafka)
- Partitions: Parallel processing
- Consumer groups: Multiple independent readers
- Kafka-compatible endpoint
- Use for: Telemetry, log aggregation, real-time analytics

### Azure Event Grid
- Event routing service (like EventBridge)
- Sources: Azure services, custom topics
- Handlers: Functions, Logic Apps, Service Bus, Webhooks
- Use for: Event-driven architectures, reactive systems

### Azure Service Bus vs Event Hubs vs Event Grid

| Service | Pattern | Use Case |
|---------|---------|----------|
| Service Bus | Queue/Topic | Reliable messaging, ordering |
| Event Hubs | Streaming | High-throughput telemetry |
| Event Grid | Event routing | Reactive, event-driven |

## 3.8 Azure Security

### Azure AD (Entra ID)
- Identity platform: users, groups, service principals, managed identities
- Managed Identities: System-assigned or user-assigned (no credentials needed)
- Conditional Access: MFA, device compliance, location-based policies
- PIM (Privileged Identity Management): Just-in-time privileged access

### Azure Key Vault
- Secrets, keys, certificates management
- HSM-backed keys (Premium tier)
- Soft delete + purge protection
- Access policies or RBAC for access control

### Azure Security Services
- **Microsoft Defender for Cloud**: CSPM + workload protection
- **Microsoft Sentinel**: Cloud-native SIEM/SOAR
- **Azure DDoS Protection**: Standard tier for advanced protection
- **Azure WAF**: On Application Gateway or Front Door
- **Azure Firewall Premium**: IDPS, TLS inspection, URL filtering

---

# PART 4: COMMON PATTERNS (AWS & AZURE)

## 4.1 Three-Tier Architecture

```
Internet
  ↓
[CDN / WAF]                    ← CloudFront / Azure Front Door
  ↓
[Load Balancer]                ← ALB / Application Gateway
  ↓
[Application Tier]             ← EC2/ECS/EKS / App Service/AKS
  ↓
[Cache]                        ← ElastiCache / Azure Cache for Redis
  ↓
[Database Tier]                ← RDS/Aurora / Azure SQL
  ↓
[Object Storage]               ← S3 / Azure Blob
```

## 4.2 Microservices Architecture

```
[API Gateway]                  ← API GW / Azure APIM
  ↓
[Service Mesh]                 ← Istio / Linkerd
  ↓
[Service A] [Service B] [Service C]
  ↓           ↓           ↓
[DB A]      [DB B]      [DB C]   ← Database per service
  ↓
[Message Bus]                  ← SQS/SNS/EventBridge / Service Bus/Event Grid
  ↓
[Async Services]
```

## 4.3 Event-Driven Architecture

```
[Producer Services]
  ↓
[Event Bus/Stream]             ← EventBridge/Kinesis / Event Grid/Event Hubs
  ↓
[Event Router/Filter]
  ↓
[Consumer A] [Consumer B] [Consumer C]
  ↓
[Dead Letter Queue]            ← For failed events
```

## 4.4 CQRS Pattern

```
[Client]
  ↓
[Command API]    [Query API]
  ↓                  ↓
[Write DB]       [Read DB]     ← Separate optimized stores
  ↓
[Event Stream]
  ↓
[Projections]    → Update Read DB
```

## 4.5 Saga Pattern

### Choreography (event-based)
```
Order Service → OrderCreated event
  → Payment Service listens → PaymentProcessed event
  → Inventory Service listens → InventoryReserved event
  → Shipping Service listens → ShipmentCreated event
```

### Orchestration (central coordinator)
```
Saga Orchestrator
  → Call Payment Service
  → Call Inventory Service
  → Call Shipping Service
  → Handle compensating transactions on failure
```

## 4.6 Strangler Fig Pattern (Monolith Migration)

```
Phase 1: [Monolith] ← all traffic
Phase 2: [Proxy/API GW] → [Monolith] + [New Service A]
Phase 3: [Proxy/API GW] → [New Service A] + [New Service B] + [Remaining Monolith]
Phase 4: [Proxy/API GW] → [Microservices] (monolith retired)
```

---

# PART 5: MULTI-REGION & DISASTER RECOVERY

## 5.1 Key Metrics

- **RTO (Recovery Time Objective)**: How long can the system be down?
- **RPO (Recovery Point Objective)**: How much data can we lose?
- **MTTR (Mean Time to Recovery)**: Average time to restore service
- **MTBF (Mean Time Between Failures)**: Average time between failures

## 5.2 DR Strategies

| Strategy | RTO | RPO | Cost | Description |
|----------|-----|-----|------|-------------|
| Backup & Restore | Hours | Hours | $ | Restore from backup |
| Pilot Light | 10-30 min | Minutes | $$ | Minimal standby, scale on failover |
| Warm Standby | Minutes | Seconds | $$$ | Scaled-down running copy |
| Multi-Site Active-Active | Seconds | Near-zero | $$$$ | Full capacity in multiple regions |

## 5.3 Active-Active Multi-Region

```
[Global Load Balancer / Route 53 / Traffic Manager]
  ↓                    ↓
[Region A]          [Region B]
  ↓                    ↓
[App Tier A]        [App Tier B]
  ↓                    ↓
[DB Primary A] ←→ [DB Primary B]   ← Bi-directional replication
```

Challenges:
- Data consistency across regions
- Conflict resolution for concurrent writes
- Latency for cross-region reads
- Cost of data transfer

## 5.4 Data Replication Strategies

| Service | Replication | RPO |
|---------|-------------|-----|
| Aurora Global | Async, <1s lag | ~1 second |
| DynamoDB Global Tables | Multi-active | Near-zero |
| S3 CRR | Async | Minutes |
| Azure Cosmos DB | Multi-region writes | Near-zero |
| Azure SQL Geo-replication | Async | Seconds |
| Azure Blob GRS | Async | ~15 min |

---

# PART 6: COST OPTIMIZATION

## 6.1 FinOps Principles

1. Teams need visibility into their cloud spend
2. Everyone is responsible for their cloud usage
3. Reports should be accessible and timely
4. Optimize for business value, not just cost reduction

## 6.2 Cost Optimization Strategies

### Compute
- Right-size instances (use AWS Compute Optimizer / Azure Advisor)
- Use Spot/Preemptible for fault-tolerant workloads (up to 90% savings)
- Reserved Instances / Savings Plans for predictable workloads (up to 72% savings)
- Auto-scaling to match demand
- Serverless for variable workloads

### Storage
- S3 Intelligent-Tiering for unknown access patterns
- Lifecycle policies to move data to cheaper tiers
- Delete unattached EBS volumes, old snapshots
- Compress data before storing

### Networking
- Use VPC endpoints to avoid NAT Gateway costs
- CloudFront/CDN to reduce origin requests
- Data transfer costs: same AZ is free, cross-AZ costs, cross-region costs more

### Database
- Aurora Serverless for variable workloads
- DynamoDB On-Demand for unpredictable traffic
- Read replicas to offload read traffic
- Connection pooling (RDS Proxy / PgBouncer)

## 6.3 Cost Allocation

- Tag everything: environment, team, project, cost-center
- Use AWS Cost Explorer / Azure Cost Management
- Set budgets and alerts
- Chargeback vs showback models
