# Visual Architecture Guide — 50 Diagrams

> ASCII/text diagrams for whiteboard practice. Study these until you can draw them from memory.

---

## KUBERNETES DIAGRAMS

### 1. Kubernetes Cluster Architecture
```
┌──────────────────────────────────────────────────────────────┐
│                        CONTROL PLANE                         │
│  [API Server] ←→ [etcd]   [Scheduler]   [Controller Mgr]    │
└──────────────────────────────────────────────────────────────┘
         ↕ (kubelet watches API server)
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│     NODE 1      │  │     NODE 2      │  │     NODE 3      │
│ [kubelet]       │  │ [kubelet]       │  │ [kubelet]       │
│ [kube-proxy]    │  │ [kube-proxy]    │  │ [kube-proxy]    │
│ [containerd]    │  │ [containerd]    │  │ [containerd]    │
│ [Pod A] [Pod B] │  │ [Pod C] [Pod D] │  │ [Pod E] [Pod F] │
└─────────────────┘  └─────────────────┘  └─────────────────┘
```

### 2. Kubernetes Request Flow
```
Internet
  ↓
[Cloud LB] (AWS ALB / Azure LB)
  ↓
[Ingress Controller Pod] (nginx / AWS ALB Ingress / AGIC)
  ↓ (matches host + path rules)
[Service: ClusterIP] (kube-proxy iptables)
  ↓ (selects healthy pod, round-robin)
[Pod: App Container]
  ↓
[Downstream: DB / Cache / Queue]
```

### 3. Kubernetes with Istio Request Flow
```
Internet
  ↓
[Cloud LB]
  ↓
[Istio Ingress Gateway Pod] (Envoy)
  ↓ (Gateway + VirtualService rules)
[Service A]
  ↓
[Pod A: Envoy Sidecar → App Container]
  ↓ (mTLS)
[Pod B: Envoy Sidecar → App Container]
  ↓
[Database]
```

### 4. Kubernetes Rolling Deployment
```
Before:  [v1] [v1] [v1] [v1] [v1]

Step 1:  [v1] [v1] [v1] [v1] [v2]  ← +1 new (maxSurge=1)
Step 2:  [v1] [v1] [v1] [v2] [v2]  ← -1 old (maxUnavailable=1)
Step 3:  [v1] [v1] [v2] [v2] [v2]
Step 4:  [v1] [v2] [v2] [v2] [v2]
After:   [v2] [v2] [v2] [v2] [v2]
```

### 5. Kubernetes HPA Flow
```
[Metrics Server] ← scrapes pod CPU/memory
      ↓
[HPA Controller] ← reads metrics every 15s
      ↓
  desired = ceil(current * (currentMetric / targetMetric))
      ↓
[Deployment] ← scale replicas up/down
      ↓
[Pods] ← created/terminated
```

### 6. Kubernetes Storage Flow
```
[Pod] → requests storage via [PVC]
[PVC] → bound to [PV]
[PV] → backed by [StorageClass] → [Cloud Storage: EBS/Azure Disk]

Dynamic Provisioning:
[PVC created] → [StorageClass] → [CSI Driver] → [Cloud API] → [PV created] → [PVC bound]
```

### 7. Kubernetes RBAC
```
[User / ServiceAccount]
  ↓ subject of
[RoleBinding / ClusterRoleBinding]
  ↓ references
[Role / ClusterRole]
  ↓ contains
[Rules: apiGroups + resources + verbs]

Example:
ServiceAccount: my-app
  → RoleBinding: my-app-binding (namespace: prod)
  → Role: pod-reader
  → Rules: pods [get, list, watch]
```

### 8. GitOps with ArgoCD
```
Developer
  ↓ git push
[App Source Repo] → CI Pipeline → [Build + Test + Push Image]
                                → [Update Config Repo: new image tag]
[Config Repo] ← ArgoCD watches (poll every 3min or webhook)
  ↓ detects drift
[ArgoCD] → compares Git state vs Cluster state
  ↓ syncs
[Kubernetes Cluster] ← applies manifests
```

### 9. ArgoCD App of Apps
```
[Root App] → points to clusters/production/
  ↓
[clusters/production/]
  ├── [infrastructure-app] → points to infrastructure/
  │     ├── cert-manager
  │     ├── ingress-nginx
  │     └── prometheus
  └── [workloads-app] → points to apps/
        ├── [frontend-app] → apps/frontend/overlays/production
        └── [backend-app] → apps/backend/overlays/production
```

### 10. Istio Architecture
```
┌─────────────────────────────────────┐
│           istiod (Control Plane)    │
│  [Pilot] [Citadel] [Galley]         │
│  pushes config via xDS              │
└─────────────────────────────────────┘
         ↓ config push
┌──────────────────┐  ┌──────────────────┐
│ Pod A            │  │ Pod B            │
│ [Envoy Sidecar]  │←mTLS→[Envoy Sidecar]│
│ [App Container]  │  │ [App Container]  │
└──────────────────┘  └──────────────────┘
```

---

## AWS DIAGRAMS

### 11. AWS 3-Tier Web Architecture
```
Internet
  ↓
[Route 53] → DNS routing
  ↓
[CloudFront + WAF] → CDN + edge security
  ↓
[ALB] → Layer 7 load balancing
  ↓              ↓
[AZ-a]          [AZ-b]
[EC2/ECS/EKS]   [EC2/ECS/EKS]  ← App tier
  ↓              ↓
[ElastiCache Redis] ← Cache tier
  ↓
[RDS Aurora Multi-AZ] ← Data tier
Primary ←sync→ Standby
  ↓
[S3] ← Object storage
```

### 12. AWS VPC Architecture
```
VPC 10.0.0.0/16
├── Public Subnet AZ-a 10.0.1.0/24
│   [NAT GW] [ALB node] [Bastion]
│   ↕ Internet Gateway
├── Public Subnet AZ-b 10.0.2.0/24
│   [NAT GW] [ALB node]
├── App Subnet AZ-a 10.0.3.0/24
│   [EC2/EKS nodes]
├── App Subnet AZ-b 10.0.4.0/24
│   [EC2/EKS nodes]
├── Data Subnet AZ-a 10.0.5.0/24
│   [RDS Primary] [ElastiCache]
└── Data Subnet AZ-b 10.0.6.0/24
    [RDS Standby] [ElastiCache Replica]
```

### 13. AWS Multi-Region Active-Active
```
[Route 53: Latency Routing]
  ↓                    ↓
[us-east-1]         [eu-west-1]
[CloudFront]        [CloudFront]
[ALB]               [ALB]
[EKS]               [EKS]
[Aurora Global]←1s→[Aurora Global]
[DynamoDB GT]←→[DynamoDB GT]
[S3 CRR]←→[S3 CRR]
```

### 14. AWS EKS Networking
```
VPC 10.0.0.0/16
  ↓
[EKS Node: 10.0.1.5]
  ├── Pod A: 10.0.1.10 (VPC IP via VPC CNI)
  ├── Pod B: 10.0.1.11 (VPC IP via VPC CNI)
  └── Pod C: 10.0.1.12 (VPC IP via VPC CNI)

Pods get real VPC IPs → direct routing, no overlay
Security Groups can be applied to pods
```

### 15. AWS Serverless Event-Driven
```
[API Gateway] → [Lambda: API handler] → [DynamoDB]
                                      → [SQS Queue]
                                            ↓
                                      [Lambda: Worker]
                                            ↓
                                      [SNS Topic]
                                      /    |    \
                               [SQS A] [SQS B] [Lambda C]
                               (email) (SMS)   (push)
```

### 16. AWS Data Lake
```
[Sources: RDS, S3, Kinesis, APIs]
  ↓
[Ingestion: DMS, Kinesis Firehose, Glue]
  ↓
[S3 Data Lake]
  ├── /raw (landing zone)
  ├── /processed (cleaned)
  └── /curated (business-ready)
  ↓
[Glue Data Catalog] ← metadata
  ↓
[Athena] [Redshift] [EMR] [SageMaker]
  ↓
[QuickSight] ← dashboards
```

### 17. AWS Security Architecture
```
Internet
  ↓
[CloudFront + WAF + Shield]
  ↓
[ALB + WAF]
  ↓
[Security Group: allow 443 from ALB only]
  ↓
[EC2/EKS: IMDSv2, no public IP]
  ↓
[Security Group: allow 5432 from App SG only]
  ↓
[RDS: encrypted, private subnet]

Monitoring:
[CloudTrail] → [S3] → [Athena queries]
[GuardDuty] → [Security Hub] → [EventBridge] → [Lambda: auto-remediate]
```

### 18. AWS Transit Gateway Hub-Spoke
```
                [Transit Gateway]
               /        |        \
    [Network VPC]  [Prod VPC]  [Dev VPC]
    (hub: firewall,   (spoke)    (spoke)
     DNS, bastion)
          |
    [Direct Connect / VPN]
          |
    [On-Premises]
```

### 19. AWS Multi-Account Strategy
```
Root (Management Account)
├── Security OU
│   ├── Log Archive Account
│   └── Security Tooling Account
├── Infrastructure OU
│   ├── Network Account (TGW, DNS)
│   └── Shared Services Account (CI/CD)
├── Workloads OU
│   ├── Production OU
│   │   └── Prod Accounts
│   └── Non-Prod OU
│       └── Dev/Staging Accounts
└── Sandbox OU
    └── Developer Accounts
```

### 20. AWS DR Strategies
```
Backup & Restore (RTO: hours, RPO: hours)
  [Primary] → backup → [S3] → restore → [Secondary]

Pilot Light (RTO: 10-30min, RPO: minutes)
  [Primary: full] → replicate → [Secondary: DB only, minimal compute]
  On failover: scale up secondary compute

Warm Standby (RTO: minutes, RPO: seconds)
  [Primary: full] → replicate → [Secondary: scaled-down running copy]
  On failover: scale up to full capacity

Multi-Site Active-Active (RTO: seconds, RPO: near-zero)
  [Primary: full] ←→ [Secondary: full]
  Both serving traffic simultaneously
```

---

## AZURE DIAGRAMS

### 21. Azure Hub-Spoke Network
```
[Hub VNet 10.0.0.0/16]
  ├── [Azure Firewall]
  ├── [VPN Gateway]
  ├── [ExpressRoute GW]
  └── [Azure Bastion]
  ↕ VNet Peering
[Spoke 1: Prod 10.1.0.0/16]  [Spoke 2: Dev 10.2.0.0/16]
  ├── App Subnet               ├── App Subnet
  └── Data Subnet              └── Data Subnet
```

### 22. Azure AKS Architecture
```
[Azure Front Door + WAF]
  ↓
[Application Gateway + WAF]
  ↓
[AKS Cluster]
  ├── System Node Pool (Standard_D2s_v3) × 3 AZs
  │   [CoreDNS] [kube-proxy] [KEDA] [Cluster Autoscaler]
  └── User Node Pool (Standard_D4s_v3) × 3 AZs
      [App Pods]
  ↓
[Azure AD Workload Identity] → [Key Vault] [Storage] [SQL]
[AGIC] → [Application Gateway]
[Azure Monitor + Container Insights]
```

### 23. Azure Resource Hierarchy
```
Azure AD Tenant
└── Root Management Group
    ├── Platform MG
    │   ├── Identity Sub
    │   ├── Management Sub
    │   └── Connectivity Sub (Hub VNet)
    └── Landing Zones MG
        ├── Corp MG
        │   ├── Production Sub
        │   └── Non-Prod Sub
        └── Online MG
            └── Public Sub
```

### 24. Azure Multi-Region Active-Active
```
[Azure Traffic Manager: Performance routing]
  ↓                         ↓
[East US]               [West Europe]
[Front Door PoP]        [Front Door PoP]
[App Gateway]           [App Gateway]
[AKS]                   [AKS]
[Cosmos DB East] ←→ [Cosmos DB West]
(multi-region writes, session consistency)
[Azure SQL Geo-replication]
[Blob GRS]
```

### 25. Azure Microservices on AKS
```
[Azure Front Door]
  ↓
[Azure APIM] ← auth, rate limit, versioning
  ↓
[Application Gateway]
  ↓
[AKS: nginx Ingress]
  ├── [User Service] → [Azure SQL]
  ├── [Product Service] → [Cosmos DB]
  ├── [Order Service] → [Azure SQL]
  │       ↓ [Service Bus Queue]
  │       ↓ [Azure Functions: Order Processor]
  │       ↓ [Event Grid] → [Notification Function]
  ├── [Cart Service] → [Azure Cache for Redis]
  └── [Search Service] → [Azure Cognitive Search]
[ACR] ← [GitHub Actions CI]
[ArgoCD] → [AKS]
```

---

## MICROSERVICES & PATTERNS

### 26. Microservices Architecture
```
[Client]
  ↓
[API Gateway] ← auth, routing, rate limiting
  ↓
[Service Mesh: Istio]
  ├── [User Service] → [User DB]
  ├── [Order Service] → [Order DB]
  │       ↓ async
  │   [Message Bus: Kafka/SQS]
  │       ↓
  ├── [Inventory Service] → [Inventory DB]
  ├── [Payment Service] → [Payment DB]
  └── [Notification Service] → [Email/SMS/Push]
```

### 27. CQRS Pattern
```
[Client]
  ↓ commands          ↓ queries
[Command API]      [Query API]
  ↓                    ↓
[Write DB]          [Read DB]
(normalized,        (denormalized,
 ACID)               optimized for reads)
  ↓
[Event Stream]
  ↓
[Projections] → update Read DB
```

### 28. Saga Pattern — Choreography
```
[Order Service] → OrderCreated event
  ↓
[Payment Service] listens → PaymentProcessed event
  ↓
[Inventory Service] listens → InventoryReserved event
  ↓
[Shipping Service] listens → ShipmentCreated event

On failure: each service publishes compensating event
PaymentFailed → OrderService cancels order
```

### 29. Saga Pattern — Orchestration
```
[Saga Orchestrator]
  → 1. Call Payment Service
  ← PaymentProcessed
  → 2. Call Inventory Service
  ← InventoryReserved
  → 3. Call Shipping Service
  ← ShipmentCreated
  → Complete

On failure:
  → Compensate: Cancel Inventory
  → Compensate: Refund Payment
  → Mark Order Failed
```

### 30. Event-Driven Architecture
```
[Producer Services]
  ↓
[Event Bus: Kafka / EventBridge / Event Grid]
  ↓ (topic routing)
[Consumer A: Analytics]
[Consumer B: Notifications]
[Consumer C: Audit Log]
[Consumer D: Search Index]

Dead Letter Queue ← failed events after max retries
```

### 31. Strangler Fig Pattern
```
Phase 1: All traffic → Monolith
  [Client] → [Monolith]

Phase 2: Proxy routes some traffic
  [Client] → [API Gateway/Proxy]
                ├── /users → [New User Service]
                └── /* → [Monolith]

Phase 3: More services extracted
  [Client] → [API Gateway]
                ├── /users → [User Service]
                ├── /orders → [Order Service]
                └── /* → [Monolith (shrinking)]

Phase 4: Monolith retired
  [Client] → [API Gateway] → [Microservices]
```

### 32. Circuit Breaker States
```
CLOSED (normal)
  requests → service
  count failures
  if failures > threshold → OPEN

OPEN (tripped)
  requests → fast fail (no call to service)
  after timeout → HALF-OPEN

HALF-OPEN (testing)
  1 test request → service
  if success → CLOSED
  if failure → OPEN
```

### 33. API Gateway Pattern
```
[Clients: Web, Mobile, 3rd Party]
  ↓
[API Gateway]
  ├── Authentication (JWT, OAuth)
  ├── Authorization (RBAC)
  ├── Rate Limiting (token bucket)
  ├── Request Routing
  ├── Request/Response Transform
  ├── Caching
  ├── Logging & Tracing
  └── SSL Termination
  ↓
[Backend Services]
```

---

## NETWORKING DIAGRAMS

### 34. DNS Resolution Flow
```
Browser: api.example.com
  ↓ check local cache → miss
OS: query local resolver (8.8.8.8)
  ↓ check cache → miss
Resolver: query Root DNS (.)
  ← returns .com nameservers
Resolver: query .com TLD
  ← returns example.com nameservers
Resolver: query example.com NS (Route 53)
  ← returns A record: 1.2.3.4
Browser: connect to 1.2.3.4
```

### 35. Load Balancer Layers
```
Layer 7 (ALB / Application Gateway):
  - HTTP/HTTPS
  - Host-based routing: api.example.com → Service A
  - Path-based routing: /api/* → Service A, /static/* → S3
  - Header routing, cookie affinity
  - SSL termination, WAF

Layer 4 (NLB / Azure LB):
  - TCP/UDP
  - IP:Port routing
  - Static IP
  - Ultra-low latency
  - No HTTP awareness
```

### 36. SSL/TLS Termination Options
```
Option 1: Terminate at Edge
Client ──TLS──→ CDN ──HTTP──→ LB ──HTTP──→ App
(simple, CDN can cache)

Option 2: Re-encrypt
Client ──TLS──→ CDN ──TLS──→ LB ──TLS──→ App
(defense in depth)

Option 3: Passthrough
Client ──TLS──→ NLB (passthrough) ──TLS──→ App
(true E2E, no L7 routing)

Option 4: mTLS
Client ──mTLS──→ App (both present certs)
(service-to-service, zero trust)
```

### 37. Ingress Controller Flow
```
[Ingress Resource] (YAML rules: host, path → service)
  ↓ read by
[Ingress Controller] (nginx/traefik/ALB)
  ↓ configures
[Load Balancer / Proxy]
  ↓ routes traffic
[Kubernetes Services]
  ↓
[Pods]
```

---

## OBSERVABILITY DIAGRAMS

### 38. Observability Stack
```
[Applications]
  ↓ metrics (/metrics endpoint)    ↓ logs (stdout)    ↓ traces (OTel SDK)
[Prometheus]                    [Fluent Bit]         [OTel Collector]
  ↓                               ↓                      ↓
[Alertmanager]                 [Loki/ES]              [Jaeger/Tempo]
  ↓                               ↓                      ↓
[PagerDuty/Slack]              [Grafana]              [Grafana]
```

### 39. Distributed Trace
```
Request: GET /api/orders/123
Trace ID: abc-123

Span 1: API Gateway (2ms)
  └── Span 2: Order Service (45ms)
        ├── Span 3: DB Query (20ms)
        ├── Span 4: Cache Lookup (2ms)
        └── Span 5: Payment Service (15ms)
              └── Span 6: Payment DB (8ms)

Total: 47ms end-to-end
```

### 40. SLO / Error Budget
```
SLO: 99.9% availability
Error Budget: 0.1% = 43.8 min/month

Month start: [████████████████████] 100% budget
Week 1 incident (5min): [███████████████████░] 88% remaining
Week 2 incident (10min): [█████████████████░░░] 77% remaining
Week 3 incident (20min): [█████████████░░░░░░░] 54% remaining
Week 4 incident (15min): [██████████░░░░░░░░░░] 20% remaining

Policy: < 20% → freeze non-critical deployments
```

---

## SECURITY DIAGRAMS

### 41. Zero Trust Architecture
```
Traditional (Castle and Moat):
  [Internet] → [Firewall] → [Trusted Internal Network]
  Once inside: implicit trust

Zero Trust:
  [Internet] → [Identity Verification] → [Device Check] → [Resource]
  Every request verified regardless of location
  Micro-segmentation: no lateral movement
```

### 42. Defense in Depth
```
Layer 1: [DDoS Protection + WAF]
Layer 2: [Firewall + NSG/SG]
Layer 3: [Network Segmentation: private subnets]
Layer 4: [Identity: MFA + least privilege]
Layer 5: [Application: input validation, auth]
Layer 6: [Data: encryption at rest + in transit]
Layer 7: [Monitoring: SIEM + anomaly detection]
```

### 43. Secrets Management Flow
```
[Application Pod]
  ↓ (no hardcoded secrets)
[External Secrets Operator]
  ↓ (uses Workload Identity / IRSA)
[AWS Secrets Manager / Azure Key Vault]
  ↓ (fetches secret)
[Kubernetes Secret] (created in cluster)
  ↓ (mounted as env var or volume)
[Application Pod] ← reads secret
```

---

## DATA ARCHITECTURE DIAGRAMS

### 44. Lambda Architecture
```
[Data Sources]
  ↓              ↓
[Batch Layer]  [Speed Layer]
(Hadoop/Spark) (Kafka/Flink)
  ↓              ↓
[Batch Views]  [Real-time Views]
  ↓              ↓
[Serving Layer: merge batch + real-time]
  ↓
[Query]
```

### 45. Data Mesh
```
[Domain A: Orders]     [Domain B: Users]     [Domain C: Products]
  ├── Data Product        ├── Data Product       ├── Data Product
  ├── Data Contract       ├── Data Contract      ├── Data Contract
  └── Data Owner          └── Data Owner         └── Data Owner
         ↓                       ↓                      ↓
[Data Platform: storage, catalog, governance, security]
         ↓
[Data Consumers: Analytics, ML, Reporting]
```

### 46. CDC (Change Data Capture)
```
[PostgreSQL / MySQL]
  ↓ (binlog / WAL)
[Debezium / AWS DMS]
  ↓
[Kafka / Kinesis]
  ↓
[Consumers]
  ├── [Data Warehouse: Redshift/Synapse]
  ├── [Search Index: Elasticsearch]
  ├── [Cache Invalidation: Redis]
  └── [Event-driven services]
```

---

## SYSTEM DESIGN DIAGRAMS

### 47. URL Shortener
```
Write path:
[Client] → [API] → [ID Generator (Snowflake)] → [DynamoDB: shortCode→longURL]
                                               → [Redis Cache]

Read path:
[Client] → [CDN] → cache hit → [301 Redirect]
                → cache miss → [API] → [Redis] → [DynamoDB] → [301 Redirect]
```

### 48. Twitter Timeline (Fan-out on Write)
```
[User posts tweet]
  ↓
[Tweet Service] → [Cassandra: tweets]
  ↓
[Kafka: new-tweet event]
  ↓
[Fan-out Service]
  ↓ (for each follower)
[Redis: follower's timeline list] ← LPUSH tweetId
  (max 800 tweets per list)

[User reads timeline]
  ↓
[Timeline Service] → [Redis: LRANGE 0 19] → top 20 tweets
  ↓
[Hydrate tweet data from Cassandra]
  ↓
[Return to client]
```

### 49. Ride Sharing (Uber)
```
[Driver App] → [Location Service] → [Redis GEOADD: driverLoc]
                                  → [Kafka: location stream]

[Rider requests ride]
  ↓
[Ride Service] → [Matching Service]
                    → [Redis GEORADIUS: find drivers within 2km]
                    → [Score: distance + rating + car type]
                    → [Assign nearest driver]
  ↓
[WebSocket: notify driver + rider]
  ↓
[Trip Service] → [PostgreSQL: trip record]
  ↓
[Payment Service] → [Stripe]
```

### 50. Payment System
```
[Client] → [API GW] → [Payment Service]
                           ↓
                    [Idempotency Check: Redis]
                    key exists? → return cached response
                    key new? → continue
                           ↓
                    [Write to DB + Outbox table] (same transaction)
                           ↓
                    [Outbox Worker] → [Payment Processor: Stripe]
                           ↓
                    [Mark outbox processed]
                           ↓
                    [Ledger Service] → [Append-only ledger DB]
                           ↓
                    [Notification: email/SMS receipt]
```
