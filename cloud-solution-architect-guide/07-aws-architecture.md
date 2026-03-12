# AWS Architecture Patterns & Reference Architectures

---

## 🟢 Plain English Summary — Read This First

### What is this document about?
This document covers AWS-specific architecture patterns — how to structure your AWS accounts, how to design networks, how to build secure and scalable applications, and reference architectures for common use cases (e-commerce, data lakes, etc.).

Think of it as the "AWS playbook" — proven patterns that work at scale.

---

### Abbreviation Decoder

| Abbreviation | Full Name | Plain English |
|---|---|---|
| OU | Organizational Unit | A folder in AWS Organizations that groups accounts together. Rules applied to the OU apply to all accounts inside. |
| SCP | Service Control Policy | A hard limit on what any account in an OU can do. Even an admin can't bypass an SCP. |
| VPC | Virtual Private Cloud | Your private network in AWS. Isolated from other customers. |
| CIDR | Classless Inter-Domain Routing | A way to specify IP address ranges. "10.0.0.0/16" means 65,536 IP addresses. |
| IGW | Internet Gateway | The door that lets your VPC communicate with the internet. |
| NAT | Network Address Translation | Lets private servers make outbound internet requests without being directly exposed. |
| TGW | Transit Gateway | A hub that connects multiple VPCs and on-premises networks. Like a network router for your entire AWS environment. |
| ENI | Elastic Network Interface | A virtual network card. Can be moved between EC2 instances. |
| ALB | Application Load Balancer | HTTP/HTTPS load balancer. Routes based on URL path, hostname, headers. |
| NLB | Network Load Balancer | TCP/UDP load balancer. Extremely fast, gets a static IP. |
| ASG | Auto Scaling Group | Automatically adds/removes EC2 instances based on demand. |
| ECS | Elastic Container Service | AWS's service for running Docker containers. |
| EKS | Elastic Kubernetes Service | AWS's managed Kubernetes. |
| ECR | Elastic Container Registry | AWS's private Docker image storage. |
| Fargate | (not an acronym) | Serverless containers — you don't manage the underlying servers. |
| Lambda | (not an acronym) | Serverless functions. Run code without managing servers. Pay per invocation. |
| S3 | Simple Storage Service | Object storage. Store any file. 11 nines of durability. |
| EBS | Elastic Block Store | Block storage attached to EC2. Like a hard drive for your server. |
| EFS | Elastic File System | Shared file storage. Multiple EC2 instances can read/write the same files. |
| RDS | Relational Database Service | Managed relational databases (MySQL, PostgreSQL, etc.). |
| Aurora | (not an acronym) | AWS's own high-performance database. MySQL/PostgreSQL compatible but 5x faster. |
| DynamoDB | (not an acronym) | AWS's NoSQL database. Single-digit millisecond latency at any scale. |
| ElastiCache | (not an acronym) | Managed Redis or Memcached. In-memory caching. |
| SQS | Simple Queue Service | Message queue. Decouples services. Messages wait in the queue until processed. |
| SNS | Simple Notification Service | Pub/Sub. One message goes to many subscribers simultaneously. |
| EventBridge | (not an acronym) | Event bus. Routes events from AWS services, custom apps, and SaaS tools. |
| Kinesis | (not an acronym) | Real-time data streaming. Like Kafka but managed by AWS. |
| CloudFront | (not an acronym) | AWS's CDN. Caches content at 400+ edge locations worldwide. |
| Route 53 | (not an acronym) | AWS's DNS service. Also does health checks and traffic routing. |
| WAF | Web Application Firewall | Blocks malicious web traffic. |
| Shield | (not an acronym) | AWS's DDoS protection service. |
| GuardDuty | (not an acronym) | AWS's threat detection service. Uses ML to find suspicious activity. |
| Security Hub | (not an acronym) | Aggregates security findings from GuardDuty, Inspector, Macie into one dashboard. |
| Inspector | (not an acronym) | Scans EC2 instances and containers for vulnerabilities. |
| Macie | (not an acronym) | Finds PII (personal data) in S3 buckets. |
| CloudTrail | (not an acronym) | Records every API call made in your AWS account. The audit log. |
| Config | (not an acronym) | Continuously records resource configurations and checks compliance rules. |
| KMS | Key Management Service | Manages encryption keys. |
| CMK | Customer Managed Key | An encryption key you control. You can delete it, rotate it, restrict access. |
| Secrets Manager | (not an acronym) | Stores and automatically rotates secrets (passwords, API keys). |
| ACM | AWS Certificate Manager | Provides free TLS certificates for your domains. |
| IAM | Identity and Access Management | Controls who can do what in AWS. |
| IRSA | IAM Roles for Service Accounts | Lets Kubernetes pods securely access AWS services without storing credentials. |
| IMDSv2 | Instance Metadata Service v2 | A more secure way for EC2 instances to get their IAM credentials. Prevents SSRF attacks. |
| SSM | Systems Manager | Manage EC2 instances without SSH. Run commands, patch, access shell securely. |
| DMS | Database Migration Service | Migrates databases to AWS. Also does CDC (Change Data Capture). |
| Glue | (not an acronym) | Managed ETL service. Transforms data for analytics. |
| Athena | (not an acronym) | Query S3 data with SQL. No database to manage. Pay per query. |
| Redshift | (not an acronym) | AWS's data warehouse. For analytics on large datasets. |
| EMR | Elastic MapReduce | Managed Hadoop/Spark for big data processing. |
| SageMaker | (not an acronym) | AWS's machine learning platform. |
| QuickSight | (not an acronym) | AWS's business intelligence/dashboarding tool. |
| CRR | Cross-Region Replication | Automatically copies S3 objects to another region. |
| SRR | Same-Region Replication | Copies S3 objects within the same region. |
| WORM | Write Once Read Many | Data that can be written once but never modified or deleted. Used for compliance. |
| CDC | Change Data Capture | Capturing every database change and streaming it elsewhere. |
| RI | Reserved Instance | Pay upfront for 1-3 years to get up to 72% discount on EC2. |
| SP | Savings Plan | More flexible than Reserved Instances. Commit to a spend level, not specific instances. |

---

### Multi-Account Strategy in Plain English

Why use multiple AWS accounts? Imagine a company where:
- The finance team's mistake can't accidentally delete the engineering team's servers
- The development environment can't accidentally affect production
- Each team gets their own bill

That's why companies use multiple accounts. AWS Organizations lets you manage them all from one place, with SCPs as guardrails.

**The standard structure:**
- **Management Account**: Only for billing and setting organization-wide rules. No actual workloads.
- **Security Accounts**: Centralized logging and security tools.
- **Infrastructure Accounts**: Shared networking (Transit Gateway, DNS).
- **Workload Accounts**: Separate accounts for production, staging, development.
- **Sandbox Accounts**: Developers can experiment freely without affecting anything.

---

### Defense in Depth in Plain English

Security in layers — like an onion. If one layer fails, the next one catches it:

1. **Edge**: CloudFront + WAF blocks malicious requests before they even reach your servers
2. **Network**: VPC with private subnets, Security Groups, NACLs
3. **Compute**: IAM roles (no passwords), Systems Manager (no SSH)
4. **Application**: Secrets Manager (rotate credentials), KMS (encrypt data)
5. **Data**: S3 Block Public Access, RDS encryption
6. **Detection**: CloudTrail + GuardDuty + Security Hub

---

> Deep dive into AWS-specific architecture patterns, services, and reference designs.

---

## 1. AWS Landing Zone & Multi-Account Strategy

### Why Multiple Accounts?
- Security isolation (blast radius containment)
- Billing separation
- Service limit isolation
- Compliance boundaries

### AWS Organizations Structure
```
Root
└── Management Account (billing, SCPs)
    ├── Security OU
    │   ├── Log Archive Account (CloudTrail, Config logs)
    │   └── Security Tooling Account (GuardDuty, Security Hub)
    ├── Infrastructure OU
    │   ├── Network Account (Transit Gateway, DNS)
    │   └── Shared Services Account (CI/CD, artifact registry)
    ├── Workloads OU
    │   ├── Production OU
    │   │   ├── Prod Account A
    │   │   └── Prod Account B
    │   └── Non-Production OU
    │       ├── Dev Account
    │       └── Staging Account
    └── Sandbox OU
        └── Developer Sandbox Accounts
```

### Service Control Policies (SCPs)
- Applied at OU or account level
- Restrict what IAM can do (even root user)
- Examples:
  - Deny leaving AWS Organizations
  - Require MFA for sensitive actions
  - Restrict to specific regions
  - Prevent disabling CloudTrail

### AWS Control Tower
- Automated landing zone setup
- Pre-built guardrails (preventive + detective)
- Account Factory: automated account provisioning
- Dashboard: compliance status across accounts

---

## 2. AWS Networking Reference Architecture

### Hub-and-Spoke with Transit Gateway
```
                    [Transit Gateway]
                    /       |        \
          [Network VPC]  [Prod VPC]  [Dev VPC]
          (hub)          (spoke)     (spoke)
              |
          [Direct Connect / VPN]
              |
          [On-Premises]
```

Transit Gateway features:
- Connects VPCs, VPNs, Direct Connect
- Route tables per attachment (control which VPCs can talk)
- Multicast support
- Inter-region peering
- Network Manager for visibility

### VPC Design Best Practices
```
CIDR Planning:
- Use /16 for VPC (65,536 IPs)
- Use /24 for subnets (256 IPs, 251 usable — AWS reserves 5)
- Leave room for growth
- Don't overlap with on-prem ranges

Subnet Layout (per AZ):
- Public: /24 (NAT GW, ALB, bastion)
- App: /22 (application servers, EKS nodes)
- Data: /24 (RDS, ElastiCache)
- Reserved: /24 (future use)
```

### AWS PrivateLink
```
Consumer VPC                    Provider VPC
[Application] → [Interface      [NLB] → [Service]
                 Endpoint]
                 (ENI with
                  private IP)
```
- Traffic stays on AWS network
- No VPC peering, no internet
- Use for: accessing AWS services, SaaS, cross-account services

---

## 3. AWS Compute Reference Architectures

### Auto Scaling Web Application
```
[Route 53] → [CloudFront] → [WAF]
                                ↓
                            [ALB]
                           /     \
                    [AZ-a]        [AZ-b]
                  [ASG: EC2]    [ASG: EC2]
                       ↓              ↓
                  [ElastiCache (Redis)]
                       ↓
                  [RDS Aurora Multi-AZ]
                  Primary ←→ Standby
                       ↓
                  [S3 (static assets, backups)]
```

### EKS Production Architecture
```
[Route 53] → [ALB] → [Ingress Controller]
                              ↓
                    [Istio Ingress Gateway]
                              ↓
                    [Service Mesh (Istio)]
                    /          |          \
            [Service A]  [Service B]  [Service C]
                ↓              ↓              ↓
           [RDS Aurora]  [DynamoDB]    [ElastiCache]
                              ↓
                    [SQS / EventBridge]
                              ↓
                    [Lambda / Worker pods]
```

EKS Node Groups:
```
System Node Group (t3.medium)
  - CoreDNS, kube-proxy, VPC CNI
  - Cluster Autoscaler
  - Monitoring agents

Application Node Group (m5.xlarge)
  - Application workloads
  - Spot instances for non-critical

GPU Node Group (p3.2xlarge)
  - ML inference workloads
  - Only when needed
```

### Serverless Architecture
```
[API Gateway] → [Lambda Authorizer] → [Lambda Functions]
                                              ↓
                                    [DynamoDB / RDS Proxy]
                                              ↓
                                    [SQS / SNS / EventBridge]
                                              ↓
                                    [Lambda (async processing)]
                                              ↓
                                    [S3 / DynamoDB]
```

Lambda best practices:
- Keep functions small and focused
- Use environment variables for config
- Use Lambda Layers for shared code
- Set appropriate memory (CPU scales with memory)
- Use provisioned concurrency for latency-sensitive
- Use SQS for async invocation (handles retries)

---

## 4. AWS Data Architecture

### Data Lake Architecture
```
[Data Sources]
  - RDS (CDC via DMS)
  - Application logs (Kinesis)
  - S3 uploads
  - Third-party APIs
        ↓
[Ingestion Layer]
  - Kinesis Data Firehose (streaming)
  - AWS DMS (database migration)
  - AWS Glue (batch ETL)
        ↓
[Storage Layer — S3]
  - Raw zone (landing)
  - Processed zone (cleaned)
  - Curated zone (business-ready)
        ↓
[Catalog Layer]
  - AWS Glue Data Catalog
  - Lake Formation (governance)
        ↓
[Consumption Layer]
  - Athena (ad-hoc SQL)
  - Redshift (data warehouse)
  - EMR (Spark/Hadoop)
  - SageMaker (ML)
  - QuickSight (BI)
```

### Real-Time Analytics Pipeline
```
[IoT / App Events]
        ↓
[Kinesis Data Streams]
  (shards for parallelism)
        ↓
[Kinesis Data Analytics]
  (SQL or Apache Flink)
        ↓
[Lambda / Kinesis Firehose]
        ↓
[DynamoDB]  [S3]  [OpenSearch]
        ↓
[CloudWatch / QuickSight]
```

### CDC (Change Data Capture) Pattern
```
[RDS / Aurora]
  ↓ (binlog)
[AWS DMS]
  ↓
[Kinesis Data Streams]
  ↓
[Lambda / Glue]
  ↓
[Data Lake / Data Warehouse]
```

---

## 5. AWS Security Architecture

### Defense in Depth
```
Layer 1: Edge Security
  - CloudFront + WAF (SQL injection, XSS, rate limiting)
  - Shield Advanced (DDoS)
  - Route 53 DNSSEC

Layer 2: Network Security
  - VPC with private subnets
  - Security Groups (stateful, instance level)
  - NACLs (stateless, subnet level)
  - VPC Flow Logs

Layer 3: Compute Security
  - IAM roles (no long-term credentials)
  - IMDSv2 (prevent SSRF attacks)
  - Systems Manager (no SSH, no bastion)
  - Inspector (vulnerability scanning)

Layer 4: Application Security
  - Secrets Manager (rotate credentials)
  - KMS (encryption at rest)
  - ACM (TLS certificates)
  - Cognito (user authentication)

Layer 5: Data Security
  - S3 Block Public Access
  - S3 Object Lock (WORM)
  - RDS encryption
  - DynamoDB encryption

Layer 6: Detection & Response
  - CloudTrail (API audit)
  - GuardDuty (threat detection)
  - Security Hub (aggregation)
  - Config (compliance)
  - Macie (PII detection)
```

### IAM Best Practices
```
1. Root account: MFA, no access keys, only for billing
2. Use IAM roles, not users, for applications
3. Use instance profiles for EC2
4. Use IRSA (IAM Roles for Service Accounts) for EKS pods
5. Least privilege: start with deny, add specific allows
6. Use permission boundaries for delegated administration
7. Enable CloudTrail for all regions
8. Regular access reviews with IAM Access Analyzer
```

### IRSA (IAM Roles for Service Accounts) — EKS
```
Pod → ServiceAccount → IRSA annotation → OIDC Provider → IAM Role
                                                              ↓
                                                    AWS API (S3, DynamoDB, etc.)
```

```yaml
# ServiceAccount with IRSA annotation
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-app
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::123456789:role/my-app-role
```

---

## 6. AWS High Availability Patterns

### Multi-AZ Application
```
AZ-a                    AZ-b                    AZ-c
[ALB node]              [ALB node]              [ALB node]
[App EC2/Pod]           [App EC2/Pod]           [App EC2/Pod]
[RDS Primary]           [RDS Standby]           
[ElastiCache Primary]   [ElastiCache Replica]   
[NAT Gateway]           [NAT Gateway]           [NAT Gateway]
```

### Multi-Region Active-Active
```
[Route 53 Latency Routing]
  ↓                    ↓
[us-east-1]         [eu-west-1]
  ↓                    ↓
[ALB]               [ALB]
  ↓                    ↓
[EKS]               [EKS]
  ↓                    ↓
[Aurora Global DB Primary] ←→ [Aurora Global DB Secondary]
[DynamoDB Global Tables]  ←→ [DynamoDB Global Tables]
[S3 CRR]                  ←→ [S3 CRR]
```

### Circuit Breaker with ALB
```
ALB Target Group Health Checks:
- Unhealthy threshold: 2 consecutive failures
- Healthy threshold: 3 consecutive successes
- Interval: 30 seconds
- Timeout: 5 seconds

When target unhealthy:
- ALB stops routing to it
- Auto Scaling can replace it
- CloudWatch alarm triggers
```

---

## 7. AWS Cost Optimization Architecture

### Compute Savings
```
Production workloads:
  - Reserved Instances (1-3 year) → up to 72% savings
  - Savings Plans (more flexible) → up to 66% savings

Variable workloads:
  - Spot Instances → up to 90% savings
  - Use with ASG mixed instances policy
  - Spot interruption handling: drain, checkpoint

Development:
  - Scheduled scaling (off nights/weekends)
  - Smaller instance types
  - Spot for CI/CD runners
```

### Storage Savings
```
S3 Lifecycle Policy:
  Day 0: Standard (frequent access)
  Day 30: Standard-IA (infrequent access) → 40% cheaper
  Day 90: Glacier Instant Retrieval → 68% cheaper
  Day 180: Glacier Deep Archive → 95% cheaper
  Day 365: Delete (if not needed)

EBS:
  - Delete unattached volumes
  - Snapshot lifecycle policies
  - gp3 is cheaper than gp2 (same or better performance)
```

### Architecture for Cost
```
Serverless-first for variable workloads:
  Lambda: pay per invocation (no idle cost)
  Fargate: pay per task (no idle nodes)
  Aurora Serverless: pay per ACU (no idle DB)
  DynamoDB On-Demand: pay per request

Reserved for predictable:
  EC2 Reserved: steady-state workloads
  RDS Reserved: production databases
  ElastiCache Reserved: caching layer
```

---

## 8. AWS Reference Architecture: E-Commerce Platform

```
[CloudFront + WAF]
  ↓
[Route 53]
  ↓
[ALB]
  ↓
[EKS Cluster]
  ├── [Product Service] → [Aurora PostgreSQL]
  ├── [Order Service] → [Aurora PostgreSQL]
  │       ↓
  │   [SQS Order Queue]
  │       ↓
  │   [Lambda: Order Processor]
  │       ↓
  │   [SNS: Order Notifications]
  │       ↓
  │   [SES: Email] [SNS: SMS] [SQS: Push]
  ├── [User Service] → [DynamoDB]
  ├── [Cart Service] → [ElastiCache Redis]
  ├── [Search Service] → [OpenSearch]
  └── [Payment Service] → [RDS PostgreSQL]
          ↓
      [Stripe/PayPal API]

[S3: Product Images] ← [CloudFront]
[S3: Static Assets] ← [CloudFront]

[Monitoring]
  - CloudWatch (metrics, logs, alarms)
  - X-Ray (distributed tracing)
  - CloudTrail (audit)
  - GuardDuty (threat detection)
```
