# Azure Architecture Patterns & Reference Architectures

---

## 🟢 Plain English Summary — Read This First

### What is this document about?
This document covers Azure-specific architecture patterns — how to structure your Azure environment, how to design networks, how to build secure and scalable applications on Azure, and reference architectures for common use cases.

Think of it as the "Azure playbook" — the same as the AWS playbook but for Microsoft's cloud.

---

### Abbreviation Decoder

| Abbreviation | Full Name | Plain English |
|---|---|---|
| MG | Management Group | A folder that holds subscriptions. Rules applied here flow down to all subscriptions inside. |
| RG | Resource Group | A logical container for related Azure resources. Delete the RG = delete everything in it. |
| VNet | Virtual Network | Azure's private cloud network. Like AWS VPC. |
| NSG | Network Security Group | Firewall rules for subnets or network interfaces. |
| AGIC | Application Gateway Ingress Controller | Uses Azure Application Gateway as the Kubernetes Ingress Controller. |
| AKS | Azure Kubernetes Service | Azure's managed Kubernetes. |
| ACR | Azure Container Registry | Azure's private Docker image storage. Like AWS ECR. |
| VMSS | Virtual Machine Scale Sets | Automatically scales VMs up/down. Like AWS Auto Scaling Groups. |
| APIM | API Management | Azure's API Gateway. Handles auth, rate limiting, versioning, developer portal. |
| ARM | Azure Resource Manager | The deployment and management layer for Azure. All Azure operations go through ARM. |
| Bicep | (not an acronym) | Azure's infrastructure-as-code language. Simpler than ARM templates. |
| AAD / Entra ID | Azure Active Directory / Microsoft Entra ID | Azure's identity platform. Manages users, groups, service principals. |
| SP | Service Principal | An identity for an application (not a human). Like an IAM role in AWS. |
| MI | Managed Identity | An identity for an Azure resource (VM, AKS pod, etc.) that Azure manages automatically. No passwords needed. |
| PIM | Privileged Identity Management | Just-in-time access. Request elevated permissions for a limited time. |
| RBAC | Role-Based Access Control | Your role determines what you can do. Owner > Contributor > Reader. |
| CMK | Customer-Managed Key | You control the encryption key in Key Vault. Not Microsoft. |
| TDE | Transparent Data Encryption | Automatically encrypts Azure SQL database files on disk. |
| CSPM | Cloud Security Posture Management | Defender for Cloud's feature that checks your security posture and gives a score. |
| CWP | Cloud Workload Protection | Defender for Cloud's feature that detects threats in running workloads. |
| SIEM | Security Information and Event Management | Microsoft Sentinel — collects and analyzes security logs to detect threats. |
| SOAR | Security Orchestration, Automation and Response | Automatically responds to security incidents. |
| ExpressRoute | (not an acronym) | A dedicated private connection from your office/data center to Azure. Like AWS Direct Connect. |
| VPN Gateway | (not an acronym) | Encrypted VPN tunnel from your office to Azure over the internet. |
| Private Link | (not an acronym) | Access Azure PaaS services (SQL, Storage, etc.) over a private connection, not the internet. |
| Private Endpoint | (not an acronym) | A network interface that gives a PaaS service a private IP in your VNet. |
| Front Door | (not an acronym) | Azure's global HTTP load balancer + CDN + WAF. Like AWS CloudFront + Route 53 combined. |
| Traffic Manager | (not an acronym) | DNS-based global load balancer. Routes users to the nearest/healthiest region. |
| Application Gateway | (not an acronym) | Regional HTTP/HTTPS load balancer with WAF. Like AWS ALB. |
| Azure Load Balancer | (not an acronym) | Regional TCP/UDP load balancer. Like AWS NLB. |
| Azure Firewall | (not an acronym) | Managed, stateful firewall for your VNet. Centralized egress control. |
| Azure Bastion | (not an acronym) | Secure browser-based SSH/RDP to VMs without exposing them to the internet. |
| Azure DNS | (not an acronym) | Azure's DNS service. Manages public and private DNS zones. |
| Cosmos DB | (not an acronym) | Azure's globally distributed NoSQL database. Like AWS DynamoDB but with more APIs. |
| Azure SQL | (not an acronym) | Azure's managed SQL Server database. Like AWS RDS. |
| Service Bus | (not an acronym) | Enterprise message queue and pub/sub. Like AWS SQS + SNS. |
| Event Hubs | (not an acronym) | Big data streaming platform. Like AWS Kinesis or Apache Kafka. |
| Event Grid | (not an acronym) | Event routing service. Like AWS EventBridge. |
| Data Factory | (not an acronym) | Managed ETL/ELT service. Like AWS Glue. |
| Synapse Analytics | (not an acronym) | Data warehouse + big data analytics. Like AWS Redshift + EMR. |
| Databricks | (not an acronym) | Managed Apache Spark. Available on both Azure and AWS. |
| ADLS | Azure Data Lake Storage | Scalable storage for big data analytics. Built on top of Blob Storage. |
| ASR | Azure Site Recovery | Disaster recovery service. Replicates VMs to another region. |
| GRS | Geo-Redundant Storage | Blob storage replicated to a paired region. |
| GZRS | Geo-Zone-Redundant Storage | Replicated across zones AND to a paired region. |
| LRS | Locally Redundant Storage | 3 copies within one data center. Cheapest option. |
| ZRS | Zone-Redundant Storage | 3 copies across 3 availability zones. |
| RA-GRS | Read-Access Geo-Redundant Storage | GRS but you can also read from the secondary region. |
| WORM | Write Once Read Many | Immutable storage. Data can't be modified or deleted. For compliance. |
| DTU | Database Transaction Unit | Azure SQL's old pricing model. A bundle of CPU, memory, and I/O. |
| vCore | Virtual Core | Azure SQL's newer pricing model. More transparent — you pick CPU cores directly. |
| RU | Request Unit | Cosmos DB's unit of throughput. Every operation costs a certain number of RUs. |
| KEDA | Kubernetes Event-Driven Autoscaling | Scales AKS pods based on external events (queue depth, etc.). |
| OPA | Open Policy Agent | Policy engine used with Azure Policy for Kubernetes. |
| kubenet | (not an acronym) | Basic AKS networking. Pods get overlay IPs, not VNet IPs. |
| Azure CNI | Azure Container Networking Interface | Advanced AKS networking. Pods get real VNet IPs. |
| Workload Identity | (not an acronym) | Lets AKS pods access Azure services using Managed Identity. No passwords in pods. |

---

### Hub-Spoke Network Topology in Plain English

Imagine a wheel: the hub is in the center, spokes radiate outward.

**Hub VNet** (shared services):
- Azure Firewall (all traffic goes through here)
- VPN Gateway (connection to your office)
- ExpressRoute Gateway (dedicated connection to your office)
- Azure Bastion (secure VM access)
- DNS

**Spoke VNets** (workloads):
- Production workloads
- Development workloads
- Shared services

Traffic between spokes goes through the hub firewall. This means centralized security inspection and logging.

---

### Azure vs AWS — Key Differences in Plain English

| Concept | AWS | Azure |
|---|---|---|
| Private network | VPC | VNet |
| Account grouping | Organizations + OUs | Management Groups |
| Identity | IAM | Azure AD (Entra ID) |
| Managed K8s | EKS | AKS |
| Serverless | Lambda | Azure Functions |
| NoSQL DB | DynamoDB | Cosmos DB |
| Object storage | S3 | Blob Storage |
| Message queue | SQS | Service Bus |
| Event streaming | Kinesis | Event Hubs |
| Global LB | CloudFront + Route 53 | Front Door + Traffic Manager |
| Dedicated connection | Direct Connect | ExpressRoute |
| Secrets | Secrets Manager | Key Vault |
| Encryption keys | KMS | Key Vault |
| Audit log | CloudTrail | Activity Log |
| Compliance | Config + Security Hub | Azure Policy + Defender for Cloud |

---

> Deep dive into Azure-specific architecture patterns, services, and reference designs.

---

## 1. Azure Landing Zone & Governance

### Azure Management Hierarchy
```
Azure AD Tenant (identity boundary)
└── Root Management Group
    ├── Platform Management Group
    │   ├── Identity Subscription (Azure AD DS, AD Connect)
    │   ├── Management Subscription (Log Analytics, Automation)
    │   └── Connectivity Subscription (Hub VNet, ExpressRoute, DNS)
    ├── Landing Zones Management Group
    │   ├── Corp Management Group
    │   │   ├── Production Subscription
    │   │   └── Non-Production Subscription
    │   └── Online Management Group
    │       └── Public-facing Subscription
    ├── Sandbox Management Group
    │   └── Developer Sandbox Subscriptions
    └── Decommissioned Management Group
```

### Azure Policy
```
Policy Definition → Policy Assignment → Compliance Evaluation
                         ↓
                   Scope: MG / Subscription / RG
                         ↓
                   Effect: Audit / Deny / Append / Modify / DeployIfNotExists
```

Common policies:
- Require tags on resources
- Allowed locations (data residency)
- Allowed VM SKUs
- Require HTTPS on storage accounts
- Deploy Log Analytics agent (DeployIfNotExists)

### Azure Blueprints
Package of:
- Policy assignments
- Role assignments
- ARM templates
- Resource groups

Use for: Repeatable, compliant environment setup

### Azure RBAC
```
Security Principal (User/Group/SP/Managed Identity)
  ↓ assigned
Role Definition (Owner/Contributor/Reader/Custom)
  ↓ at
Scope (MG/Subscription/RG/Resource)
```

Custom role example:
```json
{
  "Name": "VM Operator",
  "Actions": [
    "Microsoft.Compute/virtualMachines/start/action",
    "Microsoft.Compute/virtualMachines/restart/action",
    "Microsoft.Compute/virtualMachines/read"
  ],
  "NotActions": [],
  "AssignableScopes": ["/subscriptions/{id}"]
}
```

---

## 2. Azure Networking Reference Architecture

### Hub-Spoke Topology
```
                    [Hub VNet]
                    /    |    \
          [Firewall] [VPN GW] [ExpressRoute GW]
                    \    |    /
                     [Peering]
                    /    |    \
          [Spoke 1]  [Spoke 2]  [Spoke 3]
          (Prod)     (Dev)      (Shared Svcs)
```

Hub VNet contains:
- Azure Firewall (centralized egress)
- VPN Gateway (site-to-site VPN)
- ExpressRoute Gateway (dedicated connection)
- Azure Bastion (secure VM access)
- DNS Private Resolver

Spoke VNets:
- Peered to hub (not to each other)
- Traffic between spokes goes through hub firewall
- Each spoke = one subscription/workload

### Azure Virtual WAN (for large enterprises)
```
[Virtual WAN Hub - Region A]  ←→  [Virtual WAN Hub - Region B]
  /        |        \                /        |        \
[VNet A] [VPN]  [ExpressRoute]    [VNet B] [VPN]  [ExpressRoute]
```

Managed hub-spoke with:
- Automated routing
- Any-to-any connectivity
- Integrated security (Azure Firewall in hub)

### Azure Private Link Architecture
```
Consumer VNet                    Provider
[App] → [Private Endpoint] → [Private Link Service] → [Azure PaaS / Custom Service]
         (NIC with private IP)
```

Use for:
- Azure SQL, Storage, Key Vault, etc. (no public internet)
- Cross-subscription access
- SaaS provider services

### AKS Networking Options

#### kubenet (basic)
```
Node IP: VNet IP (10.0.1.4)
Pod IPs: Overlay (172.16.0.0/16) — NOT in VNet
Service IPs: Overlay (10.96.0.0/12)

Pros: Smaller VNet CIDR needed
Cons: Extra hop for pod traffic, no direct pod access from VNet
```

#### Azure CNI (advanced)
```
Node IP: VNet IP (10.0.1.4)
Pod IPs: VNet IPs (10.0.2.0/24) — IN VNet
Service IPs: Overlay (10.96.0.0/12)

Pros: Direct pod access, NSG on pods, Private Link to pods
Cons: Requires more VNet IPs
```

#### Azure CNI Overlay (recommended for new clusters)
```
Node IP: VNet IP (10.0.1.4)
Pod IPs: Overlay (192.168.0.0/16) — NOT in VNet
Service IPs: Overlay (10.96.0.0/12)

Pros: Scalable (no VNet IP exhaustion), better than kubenet
Cons: Slightly more complex
```

---

## 3. Azure Compute Reference Architectures

### AKS Production Architecture
```
[Azure Front Door + WAF]
  ↓
[Application Gateway + WAF]
  ↓
[AKS Cluster]
  ├── System Node Pool (Standard_D2s_v3)
  │   - CoreDNS, kube-proxy
  │   - KEDA, Cluster Autoscaler
  │   - Azure Monitor agent
  │
  ├── User Node Pool (Standard_D4s_v3)
  │   - Application workloads
  │   - Spot instances for non-critical
  │
  └── GPU Node Pool (Standard_NC6s_v3)
      - ML inference (when needed)

[AGIC] → [Application Gateway]
[Azure AD Workload Identity] → [Azure Key Vault, Storage, SQL]
[Azure Monitor + Container Insights]
[Azure Policy for Kubernetes]
```

### App Service Architecture
```
[Azure Front Door]
  ↓
[App Service (Premium v3)]
  ├── Deployment Slots: Production, Staging
  ├── VNet Integration (outbound)
  ├── Private Endpoints (inbound)
  ├── Auto-scaling (scale out/in)
  └── Custom domains + Managed certificates
  ↓
[Azure SQL (Business Critical)]
  ├── Primary replica (read/write)
  ├── Read replicas (3 included)
  └── Geo-replication to secondary region
  ↓
[Azure Cache for Redis (Premium)]
  ├── Clustering
  ├── Persistence
  └── Geo-replication
```

### Azure Functions Architecture
```
[Event Hub / Service Bus / HTTP]
  ↓
[Azure Functions (Premium Plan)]
  ├── Pre-warmed instances (no cold start)
  ├── VNet integration
  ├── Durable Functions for workflows
  └── KEDA for event-driven scaling
  ↓
[Cosmos DB / Azure SQL / Blob Storage]
```

Durable Functions patterns:
```
Function Chaining: A → B → C → D (sequential)
Fan-out/Fan-in: A → [B, C, D] → E (parallel then aggregate)
Async HTTP: Start → Poll status → Complete
Monitor: Periodic check until condition met
Human Interaction: Wait for approval (with timeout)
```

---

## 4. Azure Data Architecture

### Modern Data Platform on Azure
```
[Data Sources]
  - Azure SQL (CDC via Change Feed)
  - Cosmos DB (Change Feed)
  - Event Hubs (streaming)
  - Blob Storage (files)
  - On-prem (via Data Factory)
        ↓
[Ingestion]
  - Azure Data Factory (batch ETL/ELT)
  - Event Hubs (streaming)
  - Azure Data Factory (CDC)
        ↓
[Storage — Azure Data Lake Storage Gen2]
  - Bronze (raw)
  - Silver (cleaned)
  - Gold (business-ready)
        ↓
[Processing]
  - Azure Databricks (Spark)
  - Azure Synapse Analytics (SQL + Spark)
  - Stream Analytics (real-time SQL)
        ↓
[Serving]
  - Synapse SQL Pool (data warehouse)
  - Azure Analysis Services (semantic model)
  - Cosmos DB (operational analytics)
        ↓
[Consumption]
  - Power BI (dashboards)
  - Azure ML (machine learning)
  - Custom APIs
```

### Real-Time Analytics on Azure
```
[IoT Hub / Event Hubs]
  ↓
[Azure Stream Analytics]
  (SQL queries on streams)
  ↓
[Azure Functions] (alerts, actions)
[Power BI] (real-time dashboards)
[Azure Data Lake] (archival)
[Cosmos DB] (operational store)
```

### Cosmos DB Multi-Region Architecture
```
[Application — Region A] → [Cosmos DB — Region A] (write)
[Application — Region B] → [Cosmos DB — Region B] (write)
                                    ↕ (replication)
[Application — Region C] → [Cosmos DB — Region C] (write)
```

Consistency levels (choose per operation):
- Strong: Linearizable reads (highest latency)
- Bounded Staleness: At most K versions or T seconds behind
- Session: Consistent within a session (default, recommended)
- Consistent Prefix: Reads never see out-of-order writes
- Eventual: Lowest latency, highest availability

---

## 5. Azure Security Architecture

### Defense in Depth
```
Layer 1: Perimeter
  - Azure DDoS Protection Standard
  - Azure Front Door WAF
  - Application Gateway WAF

Layer 2: Network
  - Azure Firewall (centralized, stateful)
  - NSGs (subnet + NIC level)
  - Azure Bastion (no public IP on VMs)
  - Just-in-time VM access

Layer 3: Compute
  - Managed Identities (no credentials)
  - Azure AD Workload Identity (AKS)
  - Defender for Servers
  - Update Management

Layer 4: Application
  - Azure API Management (auth, rate limiting)
  - Azure AD (authentication)
  - Azure Key Vault (secrets, certs, keys)
  - App Service Authentication

Layer 5: Data
  - Azure SQL TDE (Transparent Data Encryption)
  - Storage Service Encryption
  - Customer-managed keys (Key Vault)
  - Azure Purview (data governance)

Layer 6: Identity
  - Azure AD Conditional Access
  - PIM (Privileged Identity Management)
  - MFA enforcement
  - Azure AD Identity Protection

Layer 7: Detection
  - Microsoft Defender for Cloud (CSPM)
  - Microsoft Sentinel (SIEM/SOAR)
  - Azure Monitor + Log Analytics
  - Activity Log (audit)
```

### Azure AD Workload Identity (AKS)
```
Pod → ServiceAccount → Workload Identity annotation
  → Azure AD OIDC → Managed Identity / Service Principal
  → Azure Key Vault, Storage, SQL (no credentials in pod)
```

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-app
  annotations:
    azure.workload.identity/client-id: "00000000-0000-0000-0000-000000000000"
```

### Zero Trust on Azure
```
Verify explicitly:
  - Azure AD Conditional Access
  - MFA for all users
  - Device compliance checks

Use least privilege:
  - Azure RBAC (minimum required roles)
  - PIM (just-in-time access)
  - Managed Identities (no passwords)

Assume breach:
  - Micro-segmentation (NSGs, Azure Firewall)
  - Encrypt everything (TLS, TDE, CMK)
  - Monitor everything (Sentinel, Defender)
  - Incident response plan
```

---

## 6. Azure High Availability Patterns

### Availability Zones Architecture
```
Zone 1              Zone 2              Zone 3
[App Service]       [App Service]       [App Service]
[AKS Node Pool]     [AKS Node Pool]     [AKS Node Pool]
[Azure SQL]         [Azure SQL]         [Azure SQL]
(Primary)           (Secondary)         (Secondary)
[Azure Cache]       [Azure Cache]       [Azure Cache]
```

Zone-redundant services:
- Azure SQL Business Critical: 3 replicas across zones
- Cosmos DB: Automatic zone redundancy
- Azure Cache for Redis Premium: Zone redundancy
- Application Gateway v2: Zone redundancy
- Azure Load Balancer Standard: Zone redundancy

### Multi-Region Active-Active
```
[Azure Traffic Manager (Performance routing)]
  ↓                              ↓
[East US]                    [West Europe]
  ↓                              ↓
[Front Door PoP]             [Front Door PoP]
  ↓                              ↓
[Application Gateway]        [Application Gateway]
  ↓                              ↓
[AKS]                        [AKS]
  ↓                              ↓
[Cosmos DB — East US] ←→ [Cosmos DB — West Europe]
(multi-region writes)
[Azure SQL Geo-replication]
[Blob Storage GRS]
```

### Azure Site Recovery (DR)
```
Primary Region                  Secondary Region
[VMs / App Service]  →  ASR  →  [Replicated VMs]
[Azure SQL]          →  Geo  →  [Secondary SQL]
[Blob Storage]       →  GRS  →  [Secondary Storage]

Failover:
1. Detect failure (Azure Monitor alert)
2. Initiate failover (manual or automated)
3. DNS update (Traffic Manager)
4. Secondary becomes primary
5. RTO: 15-30 minutes (warm standby)
```

---

## 7. Azure Reference Architecture: Microservices on AKS

```
[Azure Front Door + WAF]
  ↓
[Azure API Management (APIM)]
  - Authentication (Azure AD, OAuth)
  - Rate limiting
  - API versioning
  - Developer portal
  ↓
[Application Gateway + WAF]
  ↓
[AKS Cluster]
  ├── [nginx Ingress Controller]
  │       ↓
  ├── [User Service] → [Azure SQL]
  ├── [Product Service] → [Cosmos DB]
  ├── [Order Service] → [Azure SQL]
  │       ↓
  │   [Service Bus Queue]
  │       ↓
  │   [Order Processor Function]
  │       ↓
  │   [Event Grid]
  │       ↓
  │   [Notification Function] → [SendGrid / Twilio]
  ├── [Cart Service] → [Azure Cache for Redis]
  ├── [Search Service] → [Azure Cognitive Search]
  └── [Payment Service] → [Stripe API]

[Azure Container Registry] ← [GitHub Actions CI/CD]
[ArgoCD] → [AKS] (GitOps)
[Azure Monitor + Application Insights]
[Azure Key Vault] ← [External Secrets Operator]
```

---

## 8. AWS vs Azure Service Mapping

| Category | AWS | Azure |
|----------|-----|-------|
| Virtual Network | VPC | VNet |
| Kubernetes | EKS | AKS |
| Serverless | Lambda | Azure Functions |
| Container Registry | ECR | ACR |
| Object Storage | S3 | Blob Storage |
| Block Storage | EBS | Azure Disks |
| File Storage | EFS | Azure Files |
| Relational DB | RDS / Aurora | Azure SQL / Flexible Server |
| NoSQL | DynamoDB | Cosmos DB |
| Cache | ElastiCache | Azure Cache for Redis |
| Data Warehouse | Redshift | Synapse Analytics |
| Message Queue | SQS | Service Bus Queues |
| Pub/Sub | SNS | Service Bus Topics / Event Grid |
| Event Streaming | Kinesis | Event Hubs |
| API Gateway | API Gateway | API Management |
| CDN | CloudFront | Azure CDN / Front Door |
| DNS | Route 53 | Azure DNS |
| Global LB | Route 53 + CloudFront | Traffic Manager + Front Door |
| Regional L7 LB | ALB | Application Gateway |
| Regional L4 LB | NLB | Azure Load Balancer |
| Identity | IAM | Azure AD (Entra ID) |
| Secrets | Secrets Manager | Key Vault |
| Key Management | KMS | Key Vault |
| Monitoring | CloudWatch | Azure Monitor |
| Logging | CloudWatch Logs | Log Analytics |
| Tracing | X-Ray | Application Insights |
| SIEM | Security Hub + GuardDuty | Microsoft Sentinel |
| IaC | CloudFormation / CDK | ARM Templates / Bicep |
| CI/CD | CodePipeline / CodeBuild | Azure DevOps / GitHub Actions |
| GitOps | (use ArgoCD/Flux) | (use ArgoCD/Flux) |
| Service Mesh | App Mesh / Istio | Open Service Mesh / Istio |
| Dedicated Connection | Direct Connect | ExpressRoute |
| VPN | Site-to-Site VPN | VPN Gateway |
| Network Hub | Transit Gateway | Virtual WAN / Hub VNet |
| Compliance | AWS Config | Azure Policy |
| Cost Management | Cost Explorer | Azure Cost Management |
