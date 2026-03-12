# 1-Week Cloud Architect Intensive Kickstart

---

## 🟢 Plain English Summary — Read This First

### What is this document about?
This is a day-by-day plan for your first week of Cloud Architect preparation. Each day has a morning, afternoon, and evening section. By the end of the week, you should be able to draw and explain architecture diagrams for common cloud systems.

### Key Abbreviations in This Document

| Abbreviation | Full Name | Plain English |
|---|---|---|
| IaaS | Infrastructure as a Service | You rent raw servers. You manage the OS. (EC2, Azure VMs) |
| PaaS | Platform as a Service | You rent a platform. Cloud manages the OS. (Lambda, App Service) |
| SaaS | Software as a Service | You just use the software. Cloud manages everything. (Gmail, Office 365) |
| CaaS | Container as a Service | You run containers. Cloud manages the container platform. (ECS, AKS) |
| CAP | Consistency, Availability, Partition Tolerance | You can only guarantee 2 of 3 in a distributed system. |
| PACELC | Partition, Availability, Consistency, Else, Latency, Consistency | Extension of CAP — even without partitions, there's a latency vs consistency trade-off. |
| VPC | Virtual Private Cloud | AWS's private network in the cloud. |
| VNet | Virtual Network | Azure's private network in the cloud. |
| CIDR | Classless Inter-Domain Routing | A way to specify IP address ranges. "10.0.0.0/16" = 65,536 IPs. |
| NAT | Network Address Translation | Lets private servers access the internet without being directly exposed. |
| NACL | Network Access Control List | Subnet-level firewall in AWS. Stateless. |
| NSG | Network Security Group | Azure's firewall for subnets and network interfaces. |
| ALB | Application Load Balancer | AWS HTTP/HTTPS load balancer. Routes by URL path/hostname. |
| NLB | Network Load Balancer | AWS TCP/UDP load balancer. Very fast. |
| DNS | Domain Name System | Translates domain names to IP addresses. |
| ECS | Elastic Container Service | AWS's container service. |
| EKS | Elastic Kubernetes Service | AWS's managed Kubernetes. |
| AKS | Azure Kubernetes Service | Azure's managed Kubernetes. |
| RESHADED | Requirements, Estimation, Storage, High-level, API, Detailed, Edge cases, Diagrams | Framework for system design interviews. |
| IAM | Identity and Access Management | Controls who can do what. |
| WAF | Web Application Firewall | Blocks malicious web traffic. |
| DDoS | Distributed Denial of Service | Attack that floods your service with fake requests. |
| KMS | Key Management Service | AWS's encryption key management. |
| SSO | Single Sign-On | One login for all systems. |
| SAML | Security Assertion Markup Language | A standard for enterprise SSO. |
| OAuth | Open Authorization | Standard for "Login with Google" type flows. |
| OIDC | OpenID Connect | OAuth + identity. |
| JWT | JSON Web Token | A digital ticket that proves your identity. |
| GitOps | Git Operations | Using Git as the single source of truth for deployments. |
| SLI | Service Level Indicator | A measurement of service performance. |
| SLO | Service Level Objective | Your internal performance target. |
| SLA | Service Level Agreement | A contract with customers about uptime. |
| ELK | Elasticsearch, Logstash, Kibana | A popular logging stack. |
| EFK | Elasticsearch, Fluentd, Kibana | Similar to ELK but uses Fluentd. |

### The Week at a Glance

- **Day 1**: Cloud fundamentals + architect thinking (CAP theorem, trade-offs)
- **Day 2**: Networking (VPC/VNet, DNS, load balancers)
- **Day 3**: Containers and Kubernetes
- **Day 4**: AWS deep dive (compute, storage, databases, messaging)
- **Day 5**: Azure deep dive (same topics, Azure services)
- **Day 6**: GitOps, observability, security
- **Day 7**: System design practice (put it all together)

### The Most Important Mindset Shift

Stop thinking like a developer ("how do I implement this?") and start thinking like an architect ("should we build this? what breaks at scale? what are the trade-offs?").

---

> Goal: Build strong foundational thinking as an architect — not a developer. By end of week you should be able to draw, explain, and defend architecture decisions.

---

## Architect Mindset First

Before any technology, internalize this:
- Architects solve **business problems** with technology, not the other way around
- Every decision has **trade-offs** — your job is to articulate them
- You think in **systems**, not functions
- You own **non-functional requirements**: scalability, reliability, security, cost, observability
- You draw **before** you code

---

## Day 1 — Cloud Fundamentals & Architect Thinking

### Morning: Core Concepts
- Regions, Availability Zones, Edge Locations
- Shared Responsibility Model (AWS & Azure)
- IaaS vs PaaS vs SaaS vs CaaS (Container as a Service)
- Well-Architected Framework (AWS) / Azure Well-Architected Framework
  - Operational Excellence
  - Security
  - Reliability
  - Performance Efficiency
  - Cost Optimization
  - Sustainability (AWS adds this)

### Afternoon: Architect Thinking Models
- CAP Theorem — Consistency, Availability, Partition Tolerance
- PACELC — Extension of CAP (latency vs consistency)
- 12-Factor App principles
- Event-driven vs Request-driven architectures
- Synchronous vs Asynchronous communication

### Evening: Draw
- Draw a 3-tier web application on AWS
- Draw the same on Azure
- Label every component and explain why each exists

---

## Day 2 — Networking Deep Dive

### Morning: Core Networking
- VPC (AWS) / VNet (Azure) — subnets, CIDR, routing tables
- Public vs Private subnets
- NAT Gateway, Internet Gateway
- Security Groups vs NACLs (AWS) / NSG (Azure)
- Peering, Transit Gateway (AWS) / VNet Peering + Hub-Spoke (Azure)

### Afternoon: DNS & Load Balancing
- Route 53 (AWS) vs Azure DNS
- ALB vs NLB vs CLB (AWS)
- Azure Load Balancer vs Application Gateway vs Front Door
- Health checks, sticky sessions, weighted routing
- Global load balancing patterns

### Evening: Draw
- Draw a multi-region active-active setup on AWS
- Draw Hub-Spoke network topology on Azure
- Trace a request from browser → DNS → LB → App → DB

---

## Day 3 — Containers & Kubernetes

### Morning: Container Fundamentals
- Docker: image, container, registry, layer caching
- Container networking: bridge, host, overlay
- Container security: non-root, read-only FS, image scanning

### Afternoon: Kubernetes Architecture
- Control Plane: API Server, etcd, Scheduler, Controller Manager
- Data Plane: kubelet, kube-proxy, container runtime
- Pod, ReplicaSet, Deployment, StatefulSet, DaemonSet
- Services: ClusterIP, NodePort, LoadBalancer, ExternalName
- Ingress, Ingress Controller, IngressClass

### Evening: Draw
- Draw Kubernetes cluster architecture (control + data plane)
- Trace a request: Internet → Ingress → Service → Pod
- Draw a rolling deployment

---

## Day 4 — AWS Deep Dive

### Morning: Compute & Storage
- EC2 instance types, Auto Scaling Groups, Launch Templates
- ECS vs EKS vs Lambda (when to use each)
- S3: storage classes, lifecycle, versioning, replication
- EBS vs EFS vs S3 — decision matrix

### Afternoon: Databases & Messaging
- RDS, Aurora, DynamoDB, ElastiCache, Redshift
- SQS vs SNS vs EventBridge vs Kinesis
- API Gateway + Lambda pattern
- CloudFront CDN

### Evening: Draw
- Design a serverless event-driven order processing system on AWS
- Design a data lake architecture on AWS

---

## Day 5 — Azure Deep Dive

### Morning: Compute & Storage
- Azure VMs, VMSS, App Service, Azure Functions
- AKS (Azure Kubernetes Service)
- Azure Blob Storage, Azure Files, Azure Disks
- Storage tiers: Hot, Cool, Archive

### Afternoon: Databases & Integration
- Azure SQL, Cosmos DB, Azure Cache for Redis, Synapse Analytics
- Service Bus vs Event Hub vs Event Grid
- API Management (APIM)
- Azure CDN, Azure Front Door

### Evening: Draw
- Design a microservices architecture on AKS with APIM
- Design a real-time analytics pipeline on Azure

---

## Day 6 — GitOps, Observability & Security

### Morning: GitOps
- GitOps principles: Git as single source of truth
- ArgoCD architecture and workflow
- Flux CD
- Helm charts, Kustomize
- CI/CD pipeline: GitHub Actions → ArgoCD → Kubernetes

### Afternoon: Observability
- The 3 pillars: Metrics, Logs, Traces
- Prometheus + Grafana (metrics)
- ELK / EFK stack (logs)
- Jaeger / Zipkin (distributed tracing)
- AWS: CloudWatch, X-Ray, CloudTrail
- Azure: Azure Monitor, Application Insights, Log Analytics

### Evening: Security
- Zero Trust Architecture
- IAM: least privilege, roles, policies
- Secrets management: AWS Secrets Manager, Azure Key Vault, HashiCorp Vault
- Network security: WAF, DDoS protection
- Container security: OPA/Gatekeeper, Falco

---

## Day 7 — System Design Practice

### Morning: Framework
- RESHADED framework for system design:
  - **R**equirements (functional + non-functional)
  - **E**stimation (scale, storage, bandwidth)
  - **S**torage (data model, DB choice)
  - **H**igh-level design (components)
  - **A**PI design
  - **D**etailed design (deep dive)
  - **E**dge cases

### Afternoon: Practice Problems
1. Design a URL shortener (warm-up)
2. Design a notification system
3. Design a ride-sharing backend (Uber-like)

### Evening: Review & Consolidate
- Review all diagrams you drew this week
- Write 3 bullet points of trade-offs for each design
- Identify gaps — add to your 30-day plan

---

## Key Architect Phrases to Master

- "The trade-off here is..."
- "Given the non-functional requirements of X scale and Y latency..."
- "I would choose X over Y because..."
- "The failure mode here is... and we mitigate it by..."
- "This is a synchronous call — if we need to decouple, we can introduce a queue..."
- "For multi-region, we need to think about data residency and replication lag..."
