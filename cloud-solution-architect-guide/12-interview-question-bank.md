# Cloud Architect Interview Question Bank — 150 Questions

---

## AWS Architecture (50 Questions)

**Compute**
1. When would you choose EKS over ECS? Trade-offs?
2. How does EKS VPC CNI networking work? How do pods get IPs?
3. What is IRSA and why is it better than node-level IAM roles?
4. When would you use Lambda over ECS Fargate?
5. How do you handle Lambda cold starts in latency-sensitive apps?
6. What is Karpenter vs Cluster Autoscaler?
7. How do you implement blue-green deployments on EKS?
8. Explain EC2 Auto Scaling vs Kubernetes HPA.
9. When would you use Spot instances? How do you handle interruptions?
10. What is AWS Fargate and what are its limitations?

**Networking**
11. Difference between Security Groups and NACLs?
12. When would you use Transit Gateway vs VPC Peering?
13. How does AWS PrivateLink work? When would you use it?
14. ALB vs NLB — when to use each?
15. How does Route 53 latency routing work?
16. What is Direct Connect vs Site-to-Site VPN?
17. How do you design multi-region active-active on AWS?
18. What is AWS Global Accelerator vs CloudFront?
19. How do you restrict internet access for private subnet instances?
20. Explain VPC Flow Logs and their use in security analysis.

**Storage and Databases**
21. When would you choose DynamoDB over RDS?
22. Explain DynamoDB partition key design. What is a hot partition?
23. How does Aurora differ from RDS? When would you choose Aurora?
24. Explain Aurora Global Database replication lag.
25. ElastiCache Redis vs Memcached — when to use each?
26. How do you implement zero-downtime database migrations?
27. When would you use DynamoDB Streams vs Kinesis?
28. How do you design a data lake on AWS?
29. Explain S3 storage classes and lifecycle policies.
30. How does RDS Proxy help with connection pooling?

**Security**
31. Explain the AWS Shared Responsibility Model.
32. How do you implement least privilege IAM?
33. What are SCPs and how do they differ from IAM policies?
34. How does KMS envelope encryption work?
35. Secrets Manager vs Parameter Store — when to use each?
36. How would you respond to a compromised EC2 instance?
37. What does GuardDuty analyze? What threats does it detect?
38. How do you secure an S3 bucket with sensitive data?
39. What is AWS Config and how do you use it for compliance?
40. How do you implement network segmentation in AWS?

**Architecture Patterns**
41. Explain the SNS + SQS fan-out pattern.
42. How do you implement idempotency in a distributed system?
43. Saga pattern — choreography vs orchestration?
44. How do you design for eventual consistency?
45. What is the strangler fig pattern for monolith migration?
46. How do you implement rate limiting in an API?
47. Explain CQRS. When would you use it?
48. How do you handle distributed transactions across microservices?
49. What is the outbox pattern and when would you use it?
50. How do you design a multi-tenant SaaS on AWS?

---

## Azure Architecture (50 Questions)

**Compute**
51. When would you choose AKS over App Service?
52. AKS networking: kubenet vs Azure CNI vs Azure CNI Overlay?
53. What is Azure Workload Identity and how does it work with AKS?
54. How do you implement auto-scaling in AKS? HPA, VPA, KEDA, Cluster Autoscaler?
55. What is KEDA and how does it enable event-driven scaling?
56. When would you use Azure Functions over AKS?
57. Explain Durable Functions patterns: chaining, fan-out, async HTTP.
58. What is Azure Container Apps vs AKS?
59. How do you manage node pools in AKS? When to use spot node pools?
60. How do you implement blue-green deployments on AKS?

**Networking**
61. Azure NSG vs Azure Firewall — when to use each?
62. What is Hub-Spoke topology? When would you use Azure Virtual WAN?
63. How does Azure Private Link work? Private Link vs Private Endpoint?
64. Application Gateway vs Azure Front Door — when to use each?
65. Traffic Manager vs Azure Front Door?
66. ExpressRoute vs VPN Gateway?
67. How do you design multi-region active-active on Azure?
68. What is Azure Bastion and why is it better than a jump server?
69. How do you implement micro-segmentation in Azure?
70. What is Azure DNS Private Resolver?

**Storage and Databases**
71. When would you choose Cosmos DB over Azure SQL?
72. Explain Cosmos DB consistency levels. When would you use each?
73. How does Cosmos DB multi-region writes work? How are conflicts resolved?
74. Azure SQL Hyperscale vs Business Critical?
75. What are Azure SQL elastic pools?
76. Explain Azure Blob Storage redundancy: LRS, ZRS, GRS, GZRS.
77. When would you use Event Hubs vs Service Bus?
78. How do you design a data platform on Azure?
79. How do you handle zero-downtime migrations on Azure SQL?
80. When would you use Azure Cache for Redis vs Cosmos DB?

**Security**
81. Explain the Azure Shared Responsibility Model.
82. How does Azure RBAC work? Built-in vs custom roles?
83. What is Azure PIM and when would you use it?
84. How does Azure Key Vault work? Secrets vs Keys vs Certificates?
85. What is Microsoft Defender for Cloud?
86. How would you respond to a compromised Azure VM?
87. What is Microsoft Sentinel vs Defender for Cloud?
88. How do you implement Zero Trust on Azure?
89. How do you enforce compliance across subscriptions with Azure Policy?
90. How do you secure an AKS cluster? List 5 security controls.

**Architecture Patterns**
91. How do you implement the Saga pattern on Azure?
92. What does Azure API Management provide?
93. How do you implement event-driven architecture on Azure?
94. What is Service Bus dead-letter queue?
95. How do you implement CQRS on Azure?
96. Azure Logic Apps vs Azure Functions — when to use each?
97. How do you design a multi-tenant SaaS on Azure?
98. What is Azure Arc and when would you use it?
99. How do you implement GitOps on AKS?
100. How do you design a DR strategy on Azure?

---

## Kubernetes and Cloud Native (25 Questions)

101. Explain Kubernetes control plane components and their roles.
102. How does Kubernetes scheduling work? What factors influence pod placement?
103. Deployment vs StatefulSet vs DaemonSet — when to use each?
104. How does Kubernetes networking work? Explain the CNI model.
105. What is an Ingress Controller? How does it differ from LoadBalancer service?
106. Explain Kubernetes RBAC. How do you implement least privilege?
107. How does Kubernetes handle secrets? What are the security concerns?
108. HPA vs VPA vs KEDA — when would you use each?
109. What is a Pod Disruption Budget and why is it important?
110. How do you implement multi-tenancy in Kubernetes?
111. Explain Istio architecture. What problem does it solve?
112. How does Istio mTLS work? What is SPIFFE/SVID?
113. Explain Istio VirtualService, DestinationRule, Gateway.
114. How do you implement canary deployments with Istio?
115. Istio vs Linkerd — trade-offs?
116. How do you monitor a Kubernetes cluster? What metrics matter?
117. Explain Kubernetes storage: PV, PVC, StorageClass, CSI.
118. How do you handle stateful applications in Kubernetes?
119. Kustomize vs Helm — when to use each?
120. How do you implement GitOps with ArgoCD? Explain App of Apps.
121. What is Crossplane and how does it enable infrastructure as code?
122. How do you secure container images? What is image signing?
123. Explain OPA Gatekeeper. How do you write a policy?
124. How do you handle node failures in a Kubernetes cluster?
125. Cluster Autoscaler vs Karpenter?

---

## System Design and Architecture Thinking (25 Questions)

126. Explain CAP theorem with real-world examples.
127. Horizontal vs vertical scaling — when to use each?
128. How do you design for high availability? HA vs DR?
129. Explain RTO and RPO. How do they influence architecture?
130. How do you implement rate limiting at scale?
131. Explain consistent hashing. When would you use it?
132. How do you design a distributed cache?
133. Synchronous vs asynchronous communication — when to use each?
134. How do you handle distributed transactions?
135. Explain the outbox pattern. How does it ensure at-least-once delivery?
136. How do you design for idempotency?
137. Message queue vs event stream — what is the difference?
138. How do you implement a circuit breaker?
139. Explain the bulkhead pattern.
140. How do you design a multi-region active-active system?
141. What is data sharding? What are the sharding strategies?
142. How do you handle schema migrations in microservices?
143. Explain the strangler fig pattern for monolith migration.
144. How do you measure and improve system observability?
145. What is chaos engineering? How would you implement it?
146. How do you design a system for 99.99% availability?
147. Push vs pull models for metrics collection?
148. How do you implement feature flags in a distributed system?
149. What is a service mesh? When would you use one?
150. How do you approach a cloud migration? What is the 6 R's framework?

---

## Answer Framework

For every question, structure your answer:
1. Define the concept (1 sentence)
2. How it works (2-3 sentences)
3. When to use it (use cases)
4. Trade-offs (pros and cons)
5. Real example or well-known system reference

## The 6 R's of Cloud Migration
- Rehost: lift and shift
- Replatform: minor optimizations (managed DB instead of self-managed)
- Repurchase: move to SaaS
- Refactor: redesign for cloud-native
- Retire: decommission unused apps
- Retain: keep on-premises (compliance, latency)
