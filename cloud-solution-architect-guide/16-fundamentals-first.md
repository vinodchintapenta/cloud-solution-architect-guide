# Fundamentals First — Read This Before Everything Else

---

## 🟢 Plain English Summary — Read This First

### What is this document about?
This is the foundation document — the mental model you need before diving into any cloud technology. It explains how architects think differently from developers, and covers the core concepts that every cloud interview question traces back to.

Think of it like learning the rules of chess before memorizing specific moves. You need to understand WHY things work the way they do, not just memorize facts.

---

### Abbreviation Decoder

| Abbreviation | Full Name | Plain English |
|---|---|---|
| CAP | Consistency, Availability, Partition Tolerance | A rule that says: in a distributed system (many computers working together), you can only guarantee 2 out of 3 things at once. |
| ACID | Atomicity, Consistency, Isolation, Durability | The 4 guarantees that traditional databases (like PostgreSQL) make. All-or-nothing transactions. |
| BASE | Basically Available, Soft state, Eventually consistent | The looser guarantees that NoSQL databases make. Faster, but data might be slightly out of date. |
| NFR | Non-Functional Requirement | Requirements about HOW the system behaves (fast, reliable, secure) rather than WHAT it does. |
| RTO | Recovery Time Objective | How long can the system be down before it's a problem? "We can be down for max 1 hour." |
| RPO | Recovery Point Objective | How much data can we afford to lose? "We can lose at most 5 minutes of data." |
| HA | High Availability | The system keeps running even when individual parts fail. Like a car with a spare tire. |
| DR | Disaster Recovery | The plan for when something catastrophic happens (entire data center goes down). |
| IaaS | Infrastructure as a Service | You rent the raw computers/servers. You manage the operating system and everything above. (e.g., AWS EC2, Azure VMs) |
| PaaS | Platform as a Service | You rent a platform to run your app. The cloud manages the OS and runtime. (e.g., AWS Lambda, Azure App Service) |
| SaaS | Software as a Service | You just use the software. The cloud manages everything. (e.g., Gmail, Office 365) |
| JWT | JSON Web Token | A small digital "ticket" that proves who you are. Like a wristband at a concert — shows you paid to get in. |
| TLS | Transport Layer Security | The encryption that protects data traveling over the internet. The "S" in HTTPS. |
| AES | Advanced Encryption Standard | The algorithm used to encrypt data at rest (stored on disk). AES-256 means very strong encryption. |
| DEK | Data Encryption Key | The key that encrypts your actual data. |
| KEK | Key Encryption Key | The key that encrypts the DEK. This is "envelope encryption." |
| KMS | Key Management Service | AWS's service for managing encryption keys. |
| OAuth | Open Authorization | A standard for letting apps access your data without giving them your password. Like "Login with Google." |
| OIDC | OpenID Connect | Built on top of OAuth — adds identity (who you are) on top of authorization (what you can do). |
| gRPC | Google Remote Procedure Call | A fast way for services to call each other. Like REST but faster and uses binary format. |
| CDN | Content Delivery Network | A network of servers around the world that cache your content close to users. Makes websites load faster. |
| DNS | Domain Name System | The internet's phone book. Translates "google.com" into an IP address. |
| AZ | Availability Zone | A physically separate data center within a cloud region. If one AZ fails, others keep running. |
| SLA | Service Level Agreement | A contract that says "we guarantee X% uptime." If we fail, you get money back. |
| SLO | Service Level Objective | An internal target. "We aim for 99.9% uptime" (even if the SLA only requires 99.5%). |
| SLI | Service Level Indicator | The actual measurement. "Our uptime this month was 99.95%." |
| CI/CD | Continuous Integration / Continuous Deployment | Automatically testing and deploying code changes. Push code → tests run → deploys to production. |
| K8s | Kubernetes | Container orchestration system. Manages running containers at scale. |
| etcd | (no acronym) | A distributed key-value store used by Kubernetes to store all cluster state. |
| LRU | Least Recently Used | A cache eviction strategy — when the cache is full, remove the item that was used least recently. |
| LFU | Least Frequently Used | Remove the item that was accessed the fewest times. |
| TTL | Time To Live | How long something is valid before it expires. Cache TTL = how long before the cache is refreshed. |
| 2PC | Two-Phase Commit | A way to do transactions across multiple databases. Slow and fragile — architects avoid it. |

---

### The Architect Mindset in Plain English

A developer asks: "How do I write this code?"
An architect asks: "Should we build this at all? What breaks when we have 10 million users? What happens if the database goes down?"

The 5 questions an architect always asks:
1. What problem are we actually solving? (not "what feature do we want")
2. What are the constraints? (how fast, how reliable, how cheap, what regulations)
3. What are our options? (always have at least 2-3 alternatives)
4. What are the trade-offs? (every choice has a downside)
5. What can go wrong? (failure modes — what breaks and how do we recover)

---

### CAP Theorem in Plain English

Imagine 3 friends sharing a Google Doc:
- **Consistency**: Everyone always sees the same version of the document
- **Availability**: Everyone can always open and edit the document
- **Partition Tolerance**: The system keeps working even if the internet connection between friends breaks

The rule: if the internet breaks (partition), you must choose:
- **CP** (Consistency + Partition): "I'd rather show an error than show wrong data" — banking systems
- **AP** (Availability + Partition): "I'd rather show slightly old data than show an error" — social media feeds

---

### ACID vs BASE in Plain English

**ACID** (traditional databases like PostgreSQL, MySQL):
- Like a bank transaction: either the money moves from Account A to Account B completely, or nothing happens. No half-transfers.

**BASE** (NoSQL databases like DynamoDB, Cassandra):
- Like a social media like count: it might show 1,000 likes on your phone and 1,001 on your friend's phone for a few seconds. Eventually they'll match. That's fine for likes — not fine for bank balances.

---

### RTO and RPO in Plain English

Imagine your company's database crashes at 2pm on a Tuesday.

**RPO** (Recovery Point Objective): "How much data can we lose?"
- RPO = 1 hour means: we can restore from a backup taken at 1pm, losing 1 hour of data
- RPO = 0 means: we cannot lose ANY data (requires real-time replication — expensive)

**RTO** (Recovery Time Objective): "How long can we be down?"
- RTO = 4 hours means: we have 4 hours to get back online before it's a serious problem
- RTO = 15 minutes means: we need to be back up in 15 minutes (requires hot standby — expensive)

Lower RTO/RPO = more expensive architecture. Always ask the business: "How much is 1 hour of downtime worth to you?"

---

> This is your starting point. Master these concepts cold. Every interview question traces back to one of these.

---

## 1. The Architect's Mental Model

### You are NOT a developer in this interview. You are an architect.
- Developers ask: "How do I implement this?"
- Architects ask: "Should we build this? What are the trade-offs? What breaks at scale?"

### The 5 Questions Every Architect Asks
1. What problem are we solving? (requirements)
2. What are the constraints? (NFRs: scale, latency, cost, compliance)
3. What are the options? (at least 2-3 alternatives)
4. What are the trade-offs? (pros/cons of each)
5. What can go wrong? (failure modes, edge cases)

---

## 2. Core Distributed Systems Concepts

### CAP Theorem
- **C**onsistency: every read gets the latest write
- **A**vailability: every request gets a response
- **P**artition Tolerance: system works despite network splits
- You MUST choose CP or AP — partitions always happen
- CP: etcd, Zookeeper, HBase — "I'd rather fail than give wrong data"
- AP: DynamoDB, Cassandra, Cosmos DB — "I'd rather give stale data than fail"

### ACID vs BASE
```
ACID (relational DBs):
  Atomicity — all or nothing
  Consistency — valid state always
  Isolation — transactions don't interfere
  Durability — committed = persisted

BASE (NoSQL):
  Basically Available — system available most of the time
  Soft state — state may change without input
  Eventually consistent — will converge given time
```

### Consistency Models (weakest to strongest)
1. Eventual — will converge, reads may be stale
2. Read-your-writes — you see your own writes
3. Session — consistent within a session
4. Bounded staleness — at most X seconds old
5. Strong — every read sees latest write (expensive)

### Latency Numbers (memorize these)
```
L1 cache: 1 ns
L2 cache: 4 ns
RAM: 100 ns
SSD read: 100 μs
HDD read: 10 ms
Same datacenter network: 0.5 ms
Cross-region network: 100-300 ms
```

---

## 3. Scalability Fundamentals

### Horizontal vs Vertical
- Vertical: bigger machine — simple, has limits, downtime
- Horizontal: more machines — complex, unlimited, stateless required

### Stateless vs Stateful
- Stateless: any instance can handle any request — easy to scale
- Stateful: request must go to specific instance — hard to scale
- Rule: push state to external store (Redis, DB), keep services stateless

### Caching Fundamentals
```
Cache-aside (lazy loading):
  Read: check cache → miss → read DB → write cache → return
  Write: write DB → invalidate cache

Write-through:
  Write: write cache + DB simultaneously

Write-behind:
  Write: write cache → async write to DB (risk: data loss)

TTL: always set TTL — stale cache is a bug
Cache eviction: LRU (most common), LFU, FIFO
```

### Database Scaling
```
Read scaling: read replicas + caching
Write scaling: sharding (partition data across nodes)
  - Range sharding: by date, ID range (hot spots possible)
  - Hash sharding: consistent hashing (even distribution)
  - Directory sharding: lookup table (flexible, single point of failure)
```

---

## 4. Networking Fundamentals

### OSI Model (what architects care about)
- Layer 7 (Application): HTTP, HTTPS, gRPC, WebSocket
- Layer 4 (Transport): TCP, UDP — port-based routing
- Layer 3 (Network): IP — address-based routing

### TCP vs UDP
- TCP: reliable, ordered, connection-oriented — HTTP, databases
- UDP: fast, unreliable, connectionless — video streaming, gaming, DNS

### HTTP Fundamentals
- REST: stateless, resource-based, HTTP verbs (GET/POST/PUT/DELETE/PATCH)
- HTTP/2: multiplexing (multiple requests on one connection), header compression
- HTTP/3: QUIC protocol (UDP-based), faster connection setup
- WebSocket: bidirectional, persistent connection — real-time apps

### DNS Resolution
```
Browser → OS cache → Local resolver → Root DNS → TLD DNS → Authoritative DNS
TTL controls caching — low TTL = faster failover, more DNS queries
```

### Load Balancing Algorithms
- Round Robin: equal distribution
- Least Connections: route to least busy server
- IP Hash: same client → same server (session affinity)
- Weighted: route more to powerful servers

---

## 5. Security Fundamentals

### Authentication vs Authorization
- Authentication: who are you? (identity)
- Authorization: what can you do? (permissions)

### OAuth 2.0 / OIDC Flow
```
User → App → Authorization Server (Azure AD / Cognito)
  → User logs in → Authorization Code
  → App exchanges code for Access Token + ID Token
  → App calls API with Bearer token
  → API validates token (JWT signature)
```

### JWT Structure
```
Header.Payload.Signature
Header: {"alg": "RS256", "typ": "JWT"}
Payload: {"sub": "user123", "exp": 1234567890, "roles": ["admin"]}
Signature: HMAC or RSA signature
```

### Encryption
- In-transit: TLS 1.2+ (HTTPS) — protects data moving over network
- At-rest: AES-256 — protects data stored on disk
- Envelope encryption: data encrypted with DEK, DEK encrypted with KEK (KMS)

### Zero Trust Principles
1. Never trust, always verify
2. Least privilege access
3. Assume breach — design as if attacker is inside

---

## 6. Microservices Fundamentals

### When to use Microservices
- Multiple teams working independently
- Different scaling requirements per service
- Different technology requirements
- NOT for: small teams, simple apps, early-stage products

### Service Communication
```
Synchronous: REST, gRPC
  + Simple, immediate response
  - Tight coupling, cascading failures

Asynchronous: Message queues, event streaming
  + Loose coupling, resilient
  - Eventual consistency, harder to debug
```

### Key Patterns
- **Circuit Breaker**: stop calling failing service, fast-fail
- **Retry with backoff**: retry transient failures with exponential delay
- **Bulkhead**: isolate resources per downstream service
- **Timeout**: always set timeouts — never wait forever
- **Idempotency**: same request = same result (safe to retry)
- **Saga**: distributed transactions without 2PC

---

## 7. Cloud Fundamentals

### Shared Responsibility Model
```
AWS/Azure manages: physical hardware, network, hypervisor, managed services
You manage: OS (IaaS), application, data, identity, network config, encryption

IaaS (EC2/VM): you manage OS and above
PaaS (App Service/Lambda): you manage app and data
SaaS (Office 365): you manage data and access
```

### High Availability vs Disaster Recovery
```
HA: keep system running during component failures
  - Multiple AZs, load balancers, auto-scaling, health checks
  - RTO: seconds to minutes

DR: recover from regional/catastrophic failure
  - Multi-region, backups, replication
  - RTO: minutes to hours (depends on strategy)
```

### RTO and RPO
- **RTO** (Recovery Time Objective): how long can we be down?
- **RPO** (Recovery Point Objective): how much data can we lose?
- Lower RTO/RPO = more expensive architecture

### Cloud Cost Model
- Pay for what you use (vs capex for on-prem)
- Compute: per second/hour
- Storage: per GB/month
- Network: egress costs money, ingress usually free
- Managed services: premium over self-managed but less ops overhead

---

## 8. Kubernetes Fundamentals

### Why Kubernetes?
- Container orchestration: scheduling, scaling, self-healing
- Declarative: describe desired state, K8s makes it happen
- Portable: same manifests work on any cloud

### Core Objects
```
Pod: smallest unit, 1+ containers, shared network/storage
Deployment: manages pods, rolling updates, rollbacks
Service: stable endpoint for pods (ClusterIP/NodePort/LoadBalancer)
ConfigMap: non-secret configuration
Secret: sensitive configuration (base64, not encrypted by default)
Ingress: HTTP routing rules
Namespace: logical isolation within cluster
```

### Key Commands (know these)
```bash
kubectl get pods -n namespace
kubectl describe pod podname
kubectl logs podname -c containername
kubectl exec -it podname -- /bin/sh
kubectl apply -f manifest.yaml
kubectl rollout status deployment/myapp
kubectl rollout undo deployment/myapp
```

---

## 9. Terraform Fundamentals

### Core Concepts
```
Provider: plugin to interact with cloud API (azurerm, aws, alicloud)
Resource: infrastructure object (azurerm_kubernetes_cluster)
Data source: read existing infrastructure
Variable: input parameters
Output: exported values
Module: reusable configuration
State: tracks what Terraform manages (store in remote backend)
```

### Workflow
```
terraform init    → download providers
terraform plan    → show what will change
terraform apply   → make changes
terraform destroy → remove all resources
```

### State Management
- Always use remote state: Azure Blob Storage, AWS S3 + DynamoDB lock
- Never commit state files to Git
- State locking prevents concurrent modifications

---

## 10. GitOps Fundamentals

### Core Principle
Git is the single source of truth for infrastructure and application state.

### Flow
```
Developer → git push → CI pipeline (build, test, push image)
                     → update config repo (new image tag)
GitOps agent (ArgoCD) → watches config repo
                      → detects drift
                      → syncs cluster to desired state
```

### Benefits
- Audit trail: every change is a git commit
- Rollback: git revert
- No cluster credentials in CI pipeline
- Self-healing: agent corrects manual changes

---

## Swiss Re Interview Focus Areas

Based on your feedback, they want to verify:

### 1. Cloud Governance (your gap — see file 17)
- Azure Policy, Management Groups, Subscriptions
- RBAC, PIM, Conditional Access
- Cost governance, tagging strategy
- Compliance frameworks

### 2. Architectural Depth
- Draw and explain any system design
- Trade-off articulation
- Failure mode analysis
- Multi-region, HA, DR

### 3. Java for Architects (see file 18)
- Not deep coding — architectural Java concepts
- Design patterns in Java
- Concurrency, JVM, Spring Boot
- Microservices with Java

### 4. Microservices Design Patterns (see file 19)
- Saga, CQRS, Event Sourcing, Circuit Breaker
- API Gateway, Service Mesh
- Database per service

### 5. Whiteboarding (see file 20)
- How to approach the board
- What to draw first
- How to handle ambiguity
