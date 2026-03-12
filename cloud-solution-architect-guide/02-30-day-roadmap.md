# 30-Day Cloud Architect Roadmap

---

## 🟢 Plain English Summary — Read This First

### What is this document about?
This is a structured 30-day plan to go from "knows cloud basics" to "confident in any Cloud Architect interview." Each week builds on the previous. By Day 30, you should be able to walk into any Cloud Architect interview with confidence.

### Key Abbreviations in This Document

| Abbreviation | Full Name | Plain English |
|---|---|---|
| DDD | Domain-Driven Design | A way of designing software around business domains. Each domain (Orders, Payments) becomes a service. |
| OLTP | Online Transaction Processing | Databases for many small, fast transactions. (PostgreSQL, MySQL) |
| OLAP | Online Analytical Processing | Databases for complex queries over large datasets. (Redshift, Synapse) |
| CDC | Change Data Capture | Capturing every database change and streaming it elsewhere. |
| ETL | Extract, Transform, Load | Moving and transforming data from one place to another. |
| ELT | Extract, Load, Transform | Load raw data first, then transform in the destination. |
| RTO | Recovery Time Objective | Max acceptable downtime. "We can be down for 1 hour max." |
| RPO | Recovery Point Objective | Max acceptable data loss. "We can lose 5 minutes of data max." |
| HA | High Availability | System keeps running even when parts fail. |
| DR | Disaster Recovery | Plan for catastrophic failures (entire region goes down). |
| FinOps | Financial Operations | Managing and optimizing cloud costs. |
| SASE | Secure Access Service Edge | A security model that combines network security and WAN into a cloud service. |
| SBOM | Software Bill of Materials | A list of all components in your software. Helps find vulnerable dependencies. |
| CSPM | Cloud Security Posture Management | Continuously checks your cloud for security weaknesses. |
| DevSecOps | Development, Security, Operations | Integrating security into the development process from the start. |
| IDP | Internal Developer Platform | A self-service platform for developers to provision infrastructure. |
| ADR | Architecture Decision Record | A document recording WHY an architecture decision was made. |
| RFC | Request for Comments | A proposal document shared for feedback before implementation. |
| TCO | Total Cost of Ownership | Full cost including hidden costs. |
| ATAM | Architecture Trade-off Analysis Method | A formal method for evaluating architecture trade-offs. |
| C4 | Context, Container, Component, Code | A model for drawing architecture diagrams at 4 levels. |
| STAR | Situation, Task, Action, Result | A framework for answering behavioral interview questions. |
| CKA | Certified Kubernetes Administrator | A Kubernetes certification. |
| CKAD | Certified Kubernetes Application Developer | Another Kubernetes certification. |
| CKS | Certified Kubernetes Security Specialist | Kubernetes security certification. |
| SAP | Solutions Architect Professional | AWS's advanced architect certification. |

### The 4 Weeks at a Glance

**Week 1 — Foundations**: Cloud fundamentals, networking, containers, AWS, Azure, GitOps, security. (See `01-1-week-roadmap.md` for details.)

**Week 2 — Deep Dives**: Kubernetes production patterns, service mesh (Istio), multi-account strategy, data architecture, microservices patterns.

**Week 3 — Architecture Patterns**: High availability, disaster recovery, performance, security architecture, cost optimization, system design practice.

**Week 4 — Interview Mastery**: Interview frameworks, mock interviews, behavioral questions, whiteboard practice.

### The 30-Day Checklist Summary

By Day 30, you should be able to:
- Design any 3-tier system on AWS and Azure
- Explain microservices trade-offs (when to use, when not to)
- Design for HA and DR with specific RTO/RPO numbers
- Explain Kubernetes architecture end-to-end
- Trace a request through Ingress → Service Mesh → Service → Pod
- Use the RESHADED framework for any system design question
- Explain trade-offs for every architecture decision

---

> This is a structured, progressive plan. Each week builds on the previous. By Day 30 you should be able to walk into any Cloud Architect interview with confidence.

---

## Week 1: Foundations & Architect Mindset (Days 1–7)

See `01-1-week-roadmap.md` for detailed daily breakdown.

**Week 1 Outcomes:**
- [ ] Can explain CAP theorem with real examples
- [ ] Can draw 3-tier architecture on AWS and Azure
- [ ] Understand VPC/VNet networking fundamentals
- [ ] Can trace a request end-to-end through a cloud system
- [ ] Know when to use containers vs serverless vs VMs

---

## Week 2: Deep Dives (Days 8–14)

### Day 8 — Kubernetes Production Patterns
- Multi-tenancy in Kubernetes (namespaces, RBAC, network policies)
- Resource management: requests, limits, LimitRange, ResourceQuota
- Pod Disruption Budgets, Horizontal Pod Autoscaler, Vertical Pod Autoscaler
- Node affinity, taints, tolerations
- Kubernetes storage: PV, PVC, StorageClass, CSI drivers

### Day 9 — Service Mesh & Advanced Networking
- What is a service mesh and why it exists
- Istio architecture: Envoy sidecar, Pilot, Citadel, Galley
- Traffic management: VirtualService, DestinationRule, Gateway
- mTLS between services
- Canary deployments with Istio
- Linkerd as a lighter alternative

### Day 10 — AWS Advanced Patterns
- Multi-account strategy: AWS Organizations, Control Tower, Landing Zone
- AWS Service Control Policies (SCPs)
- VPC design patterns: single VPC, multi-VPC, Transit Gateway hub
- AWS Direct Connect vs VPN
- EKS networking: VPC CNI, pod networking, security groups for pods
- AWS WAF, Shield, GuardDuty, Security Hub

### Day 11 — Azure Advanced Patterns
- Azure Management Groups, Subscriptions, Resource Groups hierarchy
- Azure Policy, Blueprints, Defender for Cloud
- AKS networking: kubenet vs Azure CNI
- Azure Private Link, Private Endpoints
- Azure ExpressRoute vs VPN Gateway
- Azure AD, Managed Identities, RBAC

### Day 12 — Data Architecture
- OLTP vs OLAP — when to use which
- Data lake vs Data warehouse vs Lakehouse
- Lambda architecture (batch + speed layer)
- Kappa architecture (stream only)
- CDC (Change Data Capture) patterns
- AWS: Glue, Athena, Redshift, Lake Formation
- Azure: Data Factory, Synapse, Databricks

### Day 13 — Microservices Architecture
- Microservices vs Monolith vs Modular Monolith
- Domain-Driven Design (DDD) basics: bounded contexts, aggregates
- Service communication: REST, gRPC, GraphQL, AsyncAPI
- Saga pattern (choreography vs orchestration)
- CQRS + Event Sourcing
- API Gateway patterns
- Circuit breaker, retry, bulkhead patterns

### Day 14 — Week 2 Review & Practice
- Draw a complete microservices architecture on Kubernetes
- Design a data pipeline (batch + streaming)
- Practice explaining Istio traffic flow to a non-technical stakeholder

---

## Week 3: Architecture Patterns & System Design (Days 15–21)

### Day 15 — High Availability & Disaster Recovery
- RTO vs RPO — know these cold
- Active-Active vs Active-Passive
- Pilot Light vs Warm Standby vs Multi-Site
- Database replication strategies
- Cross-region replication: S3, DynamoDB Global Tables, Aurora Global
- Azure: Geo-redundant storage, Cosmos DB multi-region writes
- Chaos Engineering principles

### Day 16 — Performance & Scalability
- Horizontal vs Vertical scaling
- Caching strategies: CDN, application cache, database cache
- Cache invalidation patterns
- Read replicas, connection pooling
- Async processing with queues
- Database sharding strategies
- Content delivery: CloudFront vs Azure CDN vs Cloudflare

### Day 17 — Security Architecture
- Zero Trust: never trust, always verify
- Identity: SSO, SAML, OAuth 2.0, OIDC, JWT
- Network security layers: WAF → LB → Firewall → NSG/SG → App
- Encryption: in-transit (TLS), at-rest (KMS/Key Vault)
- Secrets rotation
- SIEM: AWS Security Hub + GuardDuty / Azure Sentinel
- Compliance: SOC2, ISO27001, GDPR, HIPAA considerations

### Day 18 — Cost Optimization Architecture
- FinOps principles
- Reserved vs On-Demand vs Spot (AWS) / Reserved vs Pay-as-you-go vs Spot (Azure)
- Right-sizing strategies
- Auto-scaling for cost
- S3 Intelligent Tiering, Azure Blob lifecycle
- Cost allocation tags
- Multi-cloud cost management

### Day 19 — System Design: Large Scale Systems
- Design Twitter/X
- Design YouTube
- Design a distributed cache (Redis-like)
- Key concepts: consistent hashing, replication, partitioning

### Day 20 — System Design: Real-Time Systems
- Design a real-time chat system (WhatsApp)
- Design a live streaming platform
- WebSockets vs Server-Sent Events vs Long Polling
- Message ordering guarantees

### Day 21 — Week 3 Review
- Draw a complete DR architecture (active-passive, cross-region)
- Explain Zero Trust to a CISO
- Practice cost optimization conversation

---

## Week 4: Interview Mastery & Consolidation (Days 22–30)

### Day 22 — Interview Framework Mastery
- RESHADED framework (see `11-system-design-frameworks.md`)
- How to structure a 45-minute design interview
- How to handle ambiguous requirements
- How to drive the conversation as an architect
- Common mistakes architects make in interviews

### Day 23 — Advanced Interview Problems
- Design Uber/Lyft
- Design Netflix
- Design Google Drive
- Design a payment system

### Day 24 — Cloud-Specific Interview Questions
- AWS: 50 deep questions (see `12-interview-question-bank.md`)
- Azure: 50 deep questions
- Kubernetes: 25 questions
- GitOps: 10 questions

### Day 25 — Behavioral & Leadership Questions
- "Tell me about a time you made an architecture decision that failed"
- "How do you handle disagreement with a senior engineer on architecture?"
- "How do you balance technical debt vs new features?"
- "Describe your approach to migrating a monolith to microservices"
- STAR method for architect-level answers

### Day 26 — Whiteboard Practice
- Practice drawing on paper/whiteboard without looking at notes
- Time yourself: 5 min requirements, 10 min high-level, 15 min deep dive
- Record yourself explaining an architecture

### Day 27 — Mock Interview 1
- Full 45-minute mock: Design a ride-sharing system
- Self-evaluate: Did you ask clarifying questions? Did you cover NFRs?

### Day 28 — Mock Interview 2
- Full 45-minute mock: Design a global e-commerce platform
- Focus on: multi-region, data consistency, payment processing

### Day 29 — Gaps & Weak Areas
- Review any topics you're still unsure about
- Re-read relevant sections of the CSA Bible
- Focus on your weakest area

### Day 30 — Final Review & Confidence
- Review all diagrams
- Read through your architect phrases
- You are ready. Trust the process.

---

## 30-Day Checklist

### Architecture Skills
- [ ] Can design any 3-tier system on AWS and Azure
- [ ] Can explain microservices trade-offs
- [ ] Can design for HA and DR with specific RTO/RPO
- [ ] Can design a data pipeline (batch + streaming)
- [ ] Can explain Kubernetes architecture end-to-end
- [ ] Can trace a request through Ingress → Istio → Service → Pod

### Cloud Skills
- [ ] AWS: VPC, EKS, RDS, S3, SQS, Lambda, CloudFront
- [ ] Azure: VNet, AKS, SQL, Blob, Service Bus, Functions, Front Door
- [ ] Multi-account/subscription strategy
- [ ] Cost optimization strategies

### Interview Skills
- [ ] RESHADED framework memorized
- [ ] Can handle ambiguous requirements
- [ ] Can explain trade-offs for every decision
- [ ] Completed 5+ full mock designs
- [ ] Behavioral questions prepared with STAR
