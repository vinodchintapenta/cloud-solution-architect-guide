# Observability & Security Architecture

---

## 🟢 Plain English Summary — Read This First

### What is this document about?
This document covers two critical topics:

**Observability**: How do you know if your system is healthy? How do you find problems before users do? How do you debug issues in a distributed system with 50 microservices?

**Security**: How do you protect your system from attackers? How do you detect when something bad is happening? How do you respond to incidents?

---

### Abbreviation Decoder

| Abbreviation | Full Name | Plain English |
|---|---|---|
| SLI | Service Level Indicator | A measurement of how your service is performing. "99.95% of requests succeeded this month." |
| SLO | Service Level Objective | Your internal target. "We aim for 99.9% success rate." |
| SLA | Service Level Agreement | A contract with customers. "We guarantee 99.5% uptime or you get a refund." |
| SRE | Site Reliability Engineering | A discipline that applies software engineering to operations. Google invented it. |
| RED | Rate, Errors, Duration | The 3 key metrics for any service. How many requests? How many errors? How long do they take? |
| USE | Utilization, Saturation, Errors | The 3 key metrics for infrastructure. How busy is it? Is it overloaded? Are there errors? |
| APM | Application Performance Monitoring | Tools that track how your application performs (response times, error rates, etc.). |
| OTel | OpenTelemetry | An open standard for collecting metrics, logs, and traces. Works with any backend. |
| EFK | Elasticsearch, Fluentd, Kibana | A popular logging stack. Fluentd collects logs, Elasticsearch stores them, Kibana visualizes them. |
| ELK | Elasticsearch, Logstash, Kibana | Similar to EFK but uses Logstash instead of Fluentd. |
| PromQL | Prometheus Query Language | The query language for Prometheus metrics. |
| KQL | Kusto Query Language | Azure's query language for Log Analytics. |
| SIEM | Security Information and Event Management | Collects security logs from everywhere and analyzes them for threats. AWS: Security Hub. Azure: Microsoft Sentinel. |
| SOAR | Security Orchestration, Automation and Response | Automatically responds to security threats. |
| CSPM | Cloud Security Posture Management | Continuously checks your cloud setup for security weaknesses and gives a score. |
| WAF | Web Application Firewall | Blocks malicious web traffic (SQL injection, XSS, etc.). |
| DDoS | Distributed Denial of Service | An attack where thousands of computers flood your service to overwhelm it. |
| mTLS | Mutual TLS | Both sides of a connection prove their identity with certificates. |
| Zero Trust | (not an acronym) | A security model: never trust anything by default, always verify. Even internal traffic. |
| MDM | Mobile Device Management | Software that manages and secures company mobile devices. |
| DLP | Data Loss Prevention | Tools that prevent sensitive data from leaving the organization. |
| SBOM | Software Bill of Materials | A list of all components in your software. Helps identify vulnerable dependencies. |
| CVE | Common Vulnerabilities and Exposures | A public database of known security vulnerabilities. |
| IAM | Identity and Access Management | Controls who can do what. |
| PIM | Privileged Identity Management | Just-in-time access for privileged roles. |
| MFA | Multi-Factor Authentication | Login requires two proofs of identity. |
| SSO | Single Sign-On | One login for all systems. |
| SAML | Security Assertion Markup Language | A standard for SSO. Used by enterprise identity providers. |
| OAuth | Open Authorization | A standard for delegated access. "Login with Google." |
| OIDC | OpenID Connect | OAuth + identity. Proves who you are AND what you can access. |
| JWT | JSON Web Token | A small digital ticket that proves your identity. |
| PKI | Public Key Infrastructure | The system of certificates and certificate authorities that makes TLS work. |
| CA | Certificate Authority | An organization that issues and signs digital certificates. |
| CRL | Certificate Revocation List | A list of certificates that have been revoked (invalidated). |
| OCSP | Online Certificate Status Protocol | A way to check if a certificate is still valid in real time. |
| HSM | Hardware Security Module | A physical device that stores encryption keys securely. Can't be extracted. |
| OPA | Open Policy Agent | A policy engine. "No pod can run as root." Enforces rules as code. |
| Falco | (not an acronym) | A runtime security tool for Kubernetes. Detects suspicious behavior (e.g., a container trying to read /etc/passwd). |
| Trivy | (not an acronym) | A vulnerability scanner for container images. |
| Snyk | (not an acronym) | A security tool that scans code and containers for vulnerabilities. |
| Cosign | (not an acronym) | A tool for signing container images. Proves the image hasn't been tampered with. |
| SPIFFE | Secure Production Identity Framework For Everyone | A standard for giving services cryptographic identities. |
| SVID | SPIFFE Verifiable Identity Document | The certificate that proves a service's identity. |
| TTL | Time To Live | How long something is valid. |
| P50/P95/P99 | Percentile 50/95/99 | P99 latency = 99% of requests are faster than this. The slowest 1% are slower. |
| MTTR | Mean Time to Recovery | Average time to fix something when it breaks. |
| MTBF | Mean Time Between Failures | Average time between failures. Higher = more reliable. |

---

### The Three Pillars of Observability in Plain English

**Metrics** — Numbers over time:
- "How many requests per second?"
- "What's the CPU usage?"
- "What percentage of requests are failing?"
- Tool: Prometheus (collects) + Grafana (visualizes)

**Logs** — Text records of events:
- "User 123 logged in at 2pm"
- "Payment failed: connection timeout"
- "Pod crashed with OOMKilled"
- Tool: EFK stack (Elasticsearch + Fluentd + Kibana) or Loki + Grafana

**Traces** — Following a request through multiple services:
- "This request took 500ms total: 10ms in API Gateway, 200ms in Order Service, 290ms waiting for the database"
- Tool: Jaeger, Zipkin, or AWS X-Ray

Without all three, you're flying blind. Metrics tell you SOMETHING is wrong. Logs tell you WHAT happened. Traces tell you WHERE in the system the problem is.

---

### SLI/SLO/SLA/Error Budget in Plain English

Imagine a pizza delivery service:

- **SLI**: "We delivered 98% of pizzas within 30 minutes this month" (the measurement)
- **SLO**: "We aim to deliver 99% of pizzas within 30 minutes" (our internal target)
- **SLA**: "We guarantee 95% on-time delivery or you get a free pizza" (the contract)
- **Error Budget**: 100% - 99% = 1% of deliveries can be late. That's our "budget" for failures.

When the error budget is running low, you stop adding new features and focus on reliability.

---

### Zero Trust in Plain English

Traditional security: "If you're inside the company network, you're trusted."
Problem: If an attacker gets inside the network, they can access everything.

Zero Trust: "Nobody is trusted by default, even inside the network. Every request must prove its identity."

The three principles:
1. **Never trust, always verify**: Every request must authenticate, even from internal services.
2. **Least privilege**: Give the minimum access needed. Don't give admin access when read-only is enough.
3. **Assume breach**: Design as if an attacker is already inside. Encrypt everything, log everything, segment everything.

---

### Incident Response in Plain English

When something bad happens:
1. **Detect**: An alert fires (GuardDuty found suspicious activity)
2. **Triage**: How bad is it? P1 (critical) or P3 (minor)?
3. **Contain**: Stop the bleeding. Revoke compromised credentials, isolate affected servers.
4. **Investigate**: What happened? Check CloudTrail, VPC Flow Logs, application logs.
5. **Eradicate**: Fix the root cause. Patch the vulnerability, rotate all credentials.
6. **Recover**: Restore service. Verify everything is clean.
7. **Post-mortem**: Learn from it. What can we do to prevent this next time?

---

# PART 1: OBSERVABILITY

## 1. The Three Pillars of Observability

```
┌─────────────────────────────────────────────────────────┐
│                   OBSERVABILITY                         │
│                                                         │
│  ┌──────────┐    ┌──────────┐    ┌──────────────────┐  │
│  │ METRICS  │    │  LOGS    │    │     TRACES       │  │
│  │          │    │          │    │                  │  │
│  │ What is  │    │ What     │    │ How did a        │  │
│  │ happening│    │ happened │    │ request flow     │  │
│  │ (numbers)│    │ (events) │    │ through services │  │
│  └──────────┘    └──────────┘    └──────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

### Metrics
- Numerical measurements over time
- Types: Counter (always increases), Gauge (up/down), Histogram (distribution), Summary
- Examples: request rate, error rate, latency, CPU usage, memory

### Logs
- Timestamped records of events
- Structured (JSON) vs unstructured (plain text)
- Use structured logging always
- Examples: application errors, audit events, access logs

### Traces
- Track a request as it flows through multiple services
- Span: single operation within a trace
- Trace: collection of spans with parent-child relationships
- Correlation ID: links spans across services

---

## 2. SLI, SLO, SLA, Error Budget

### Definitions
- **SLI (Service Level Indicator)**: Metric that measures service behavior
  - Example: "99th percentile latency of HTTP requests"
  - Example: "Percentage of successful requests"

- **SLO (Service Level Objective)**: Target value for an SLI
  - Example: "99th percentile latency < 200ms"
  - Example: "99.9% of requests succeed"

- **SLA (Service Level Agreement)**: Contract with consequences
  - Example: "If availability < 99.9%, customer gets credit"

- **Error Budget**: 100% - SLO = allowed failures
  - 99.9% SLO → 0.1% error budget → 43.8 min/month downtime allowed

### Error Budget Policy
```
Error budget remaining > 50%: Deploy freely, experiment
Error budget remaining 25-50%: Normal deployments, caution
Error budget remaining 0-25%: Freeze non-critical deployments
Error budget exhausted: Only reliability work, no features
```

---

## 3. Prometheus + Grafana Stack

### Architecture
```
[Applications] → expose /metrics endpoint
      ↓
[Prometheus] → scrapes metrics (pull model)
      ↓
[Alertmanager] → routes alerts (email, Slack, PagerDuty)
      ↓
[Grafana] → visualizes metrics
```

### Prometheus Deployment on Kubernetes
```yaml
# kube-prometheus-stack (Helm chart) includes:
# - Prometheus Operator
# - Prometheus
# - Alertmanager
# - Grafana
# - Node Exporter (node metrics)
# - kube-state-metrics (K8s object metrics)
```

### Key Metrics to Monitor

**Application (RED Method)**:
- **R**ate: requests per second
- **E**rrors: error rate (%)
- **D**uration: latency (P50, P95, P99)

**Infrastructure (USE Method)**:
- **U**tilization: CPU, memory, disk usage (%)
- **S**aturation: queue depth, wait time
- **E**rrors: error count

**Kubernetes**:
- Pod restart count
- OOMKilled events
- Pending pods
- Node pressure (CPU, memory, disk)
- PVC usage

### PromQL Examples
```promql
# Request rate (per second, 5-min window)
rate(http_requests_total[5m])

# Error rate
rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m])

# P99 latency
histogram_quantile(0.99, rate(http_request_duration_seconds_bucket[5m]))

# CPU usage by pod
sum(rate(container_cpu_usage_seconds_total[5m])) by (pod)

# Memory usage
container_memory_working_set_bytes{container!=""}
```

---

## 4. Logging Architecture

### EFK Stack (Kubernetes)
```
[Pod logs] → stdout/stderr
      ↓
[Fluentd/Fluent Bit] (DaemonSet, collects from /var/log/containers)
      ↓
[Elasticsearch] (stores and indexes)
      ↓
[Kibana] (search and visualize)
```

### Loki Stack (Grafana)
```
[Pod logs] → stdout/stderr
      ↓
[Promtail] (DaemonSet, like Fluentd but for Loki)
      ↓
[Loki] (stores logs, indexed by labels only — cheaper than ES)
      ↓
[Grafana] (query with LogQL)
```

Loki vs Elasticsearch:
- Loki: Cheaper, label-based indexing, integrates with Grafana
- Elasticsearch: Full-text search, more powerful queries, more expensive

### Structured Logging Best Practices
```json
{
  "timestamp": "2024-01-15T10:30:00Z",
  "level": "ERROR",
  "service": "order-service",
  "trace_id": "abc123",
  "span_id": "def456",
  "user_id": "user-789",
  "order_id": "order-101",
  "message": "Payment processing failed",
  "error": "connection timeout",
  "duration_ms": 5000
}
```

---

## 5. Distributed Tracing

### OpenTelemetry (OTel) — The Standard
```
[Application] → OTel SDK → [OTel Collector] → [Backend]
                                                  ├── Jaeger
                                                  ├── Zipkin
                                                  ├── Tempo (Grafana)
                                                  ├── AWS X-Ray
                                                  └── Azure App Insights
```

### Trace Flow
```
Request: GET /api/orders/123

Trace ID: abc-123
  Span 1: API Gateway (10ms)
    Span 2: Order Service (45ms)
      Span 3: DB Query (20ms)
      Span 4: Cache Lookup (2ms)
      Span 5: Payment Service call (15ms)
        Span 6: Payment DB (8ms)
```

### Sampling Strategies
- **Head-based**: Decision at trace start (simple, may miss errors)
- **Tail-based**: Decision after trace completes (captures errors, more complex)
- **Rate-based**: Sample X% of traces
- **Always sample errors**: Never drop error traces

---

## 6. AWS Observability

### CloudWatch
```
Metrics: EC2, RDS, Lambda, custom metrics (PutMetricData)
Logs: CloudWatch Logs (log groups, log streams)
Alarms: Threshold-based, anomaly detection
Dashboards: Visualize metrics
Insights: Query logs with CloudWatch Logs Insights
Container Insights: EKS/ECS metrics and logs
```

### AWS X-Ray (Distributed Tracing)
```
[Application] → X-Ray SDK → [X-Ray Daemon] → [X-Ray Service]
                                                    ↓
                                            Service Map
                                            Trace Analysis
                                            Error Analysis
```

### CloudTrail (Audit)
- Records all API calls (who, what, when, from where)
- Management events: control plane (create/delete resources)
- Data events: S3 object access, Lambda invocations
- Insights: Detect unusual API activity

### AWS Observability Stack
```
Metrics: CloudWatch + Prometheus (for Kubernetes)
Logs: CloudWatch Logs + Fluent Bit (for Kubernetes)
Traces: X-Ray + OpenTelemetry
Dashboards: CloudWatch Dashboards + Grafana
Alerting: CloudWatch Alarms + SNS
```

---

## 7. Azure Observability

### Azure Monitor
```
[Resources] → Diagnostic Settings → [Log Analytics Workspace]
                                           ↓
                                    [Azure Monitor]
                                    ├── Metrics (time-series)
                                    ├── Logs (KQL queries)
                                    ├── Alerts
                                    └── Dashboards
```

### Application Insights
- APM (Application Performance Monitoring)
- Auto-instrumentation for .NET, Java, Node.js, Python
- Distributed tracing (Application Map)
- Live Metrics Stream
- Availability tests (synthetic monitoring)
- Smart Detection (anomaly detection)

### Container Insights (AKS)
```
[AKS] → Azure Monitor Agent (DaemonSet) → [Log Analytics]
                                                ↓
                                        Container Insights
                                        - Node metrics
                                        - Pod metrics
                                        - Container logs
                                        - Live data
```

### Azure Observability Stack
```
Metrics: Azure Monitor Metrics + Prometheus (AKS)
Logs: Log Analytics Workspace + Container Insights
Traces: Application Insights + OpenTelemetry
Dashboards: Azure Dashboards + Grafana (Azure Managed Grafana)
Alerting: Azure Monitor Alerts + Action Groups
```

---

# PART 2: SECURITY ARCHITECTURE

## 8. Zero Trust Architecture

### Principles
1. **Never trust, always verify** — No implicit trust based on network location
2. **Least privilege access** — Minimum required permissions
3. **Assume breach** — Design as if attacker is already inside

### Zero Trust Components
```
[Identity] → Verify who is accessing
  - MFA, Conditional Access, PIM

[Device] → Verify device health
  - MDM, device compliance, certificate

[Network] → Micro-segmentation
  - No implicit trust on internal network
  - Encrypt all traffic (mTLS)

[Application] → Verify app access
  - OAuth 2.0, OIDC, API keys

[Data] → Classify and protect
  - Encryption, DLP, access controls
```

---

## 9. IAM Security Patterns

### AWS IAM Best Practices
```
1. Root account: MFA + no access keys
2. Break-glass account: emergency access, heavily monitored
3. Service accounts: IAM roles, not users
4. Cross-account: assume role (not long-term credentials)
5. Permission boundaries: limit what delegated admins can grant
6. SCPs: organization-wide guardrails
7. Access Analyzer: find unintended external access
8. Credential rotation: automate with Secrets Manager
```

### Azure Identity Best Practices
```
1. Managed Identities: no credentials for Azure resources
2. Workload Identity: for Kubernetes pods
3. PIM: just-in-time privileged access
4. Conditional Access: MFA + device compliance
5. Service Principals: for non-Azure workloads
6. App Registrations: for OAuth 2.0 flows
7. Azure AD Identity Protection: risk-based policies
```

---

## 10. Network Security Architecture

### AWS Network Security Layers
```
Internet
  ↓
[CloudFront + WAF]          ← L7 filtering, DDoS
  ↓
[ALB + WAF]                 ← L7 filtering, SSL termination
  ↓
[Security Groups]           ← Stateful, instance-level
  ↓
[NACLs]                     ← Stateless, subnet-level
  ↓
[VPC Flow Logs]             ← Traffic visibility
  ↓
[Application]
  ↓
[Security Groups]           ← Database tier
  ↓
[Database]
```

### Azure Network Security Layers
```
Internet
  ↓
[Azure Front Door + WAF]    ← Global L7, DDoS
  ↓
[Application Gateway + WAF] ← Regional L7
  ↓
[Azure Firewall]            ← Stateful, centralized
  ↓
[NSG (subnet)]              ← Subnet-level rules
  ↓
[NSG (NIC)]                 ← Instance-level rules
  ↓
[Application]
  ↓
[Private Endpoint]          ← Database access
  ↓
[Database]
```

---

## 11. Container & Kubernetes Security

### Image Security
```
Build time:
  - Use minimal base images (distroless, alpine)
  - Multi-stage builds (no build tools in final image)
  - No secrets in Dockerfile or image layers
  - Image scanning: Trivy, Snyk, Clair, ECR scanning, ACR scanning

Registry:
  - Private registry (ECR, ACR, GCR)
  - Image signing (Cosign, Notary)
  - Admission controller: only allow signed images

Runtime:
  - Non-root user
  - Read-only root filesystem
  - Drop capabilities
  - No privileged containers
```

### Pod Security Context
```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  runAsGroup: 3000
  fsGroup: 2000
  readOnlyRootFilesystem: true
  allowPrivilegeEscalation: false
  capabilities:
    drop:
    - ALL
    add:
    - NET_BIND_SERVICE  # only if needed
  seccompProfile:
    type: RuntimeDefault
```

### OPA Gatekeeper (Policy as Code)
```yaml
# Constraint Template: define policy
apiVersion: templates.gatekeeper.sh/v1beta1
kind: ConstraintTemplate
metadata:
  name: k8srequiredlabels
spec:
  crd:
    spec:
      names:
        kind: K8sRequiredLabels
  targets:
  - target: admission.k8s.gatekeeper.sh
    rego: |
      package k8srequiredlabels
      violation[{"msg": msg}] {
        provided := {label | input.review.object.metadata.labels[label]}
        required := {label | label := input.parameters.labels[_]}
        missing := required - provided
        count(missing) > 0
        msg := sprintf("Missing required labels: %v", [missing])
      }
---
# Constraint: apply policy
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredLabels
metadata:
  name: require-team-label
spec:
  match:
    kinds:
    - apiGroups: [""]
      kinds: ["Namespace"]
  parameters:
    labels: ["team", "environment"]
```

---

## 12. Secrets Management

### HashiCorp Vault Architecture
```
[Applications] → Vault Agent (sidecar) → [Vault Server]
                                               ↓
                                    [Storage Backend]
                                    (Consul, etcd, S3)
                                               ↓
                                    [Secrets Engines]
                                    - KV (static secrets)
                                    - Database (dynamic creds)
                                    - PKI (certificates)
                                    - AWS/Azure (cloud creds)
```

Dynamic Secrets (best practice):
```
App requests DB credentials from Vault
  ↓
Vault creates temporary DB user (TTL: 1 hour)
  ↓
App uses credentials
  ↓
TTL expires → credentials automatically revoked
```

### AWS Secrets Manager
```
[Application] → SDK → [Secrets Manager]
                            ↓
                    [KMS] (encryption)
                            ↓
                    [Lambda] (rotation function)
                            ↓
                    [RDS / other service] (rotated credential)
```

### Azure Key Vault
```
[Application] → Managed Identity → [Key Vault]
                                        ↓
                                   Secrets (passwords, API keys)
                                   Keys (encryption keys)
                                   Certificates (TLS certs)
```

---

## 13. Security Monitoring & Incident Response

### AWS Security Monitoring
```
[CloudTrail] → API audit logs
[VPC Flow Logs] → network traffic
[GuardDuty] → threat detection (ML-based)
[Security Hub] → aggregates findings
[Config] → compliance rules
[Macie] → PII in S3
      ↓
[EventBridge] → route findings
      ↓
[Lambda] → automated remediation
[SNS] → alert security team
[SIEM] → Splunk / Datadog / Sumo Logic
```

### Azure Security Monitoring
```
[Activity Log] → control plane audit
[Diagnostic Logs] → resource logs
[Defender for Cloud] → CSPM + workload protection
[Microsoft Sentinel] → SIEM/SOAR
      ↓
[Logic Apps] → automated response
[Action Groups] → alert security team
```

### Incident Response Playbook
```
1. DETECT: Alert fires (GuardDuty / Sentinel)
2. TRIAGE: Assess severity (P1/P2/P3)
3. CONTAIN: Isolate affected resources
   - Revoke compromised credentials
   - Isolate EC2 instance (security group)
   - Block IP at WAF
4. INVESTIGATE: Root cause analysis
   - CloudTrail / Activity Log
   - VPC Flow Logs
   - Application logs
5. ERADICATE: Remove threat
   - Patch vulnerability
   - Rotate all credentials
   - Rebuild compromised instances
6. RECOVER: Restore service
   - Restore from backup
   - Verify integrity
7. POST-MORTEM: Learn and improve
   - Timeline of events
   - What worked, what didn't
   - Preventive measures
```

---

## 14. Compliance Architecture

### Common Compliance Frameworks

| Framework | Industry | Key Requirements |
|-----------|----------|-----------------|
| SOC 2 | General | Security, availability, confidentiality |
| ISO 27001 | General | Information security management |
| PCI DSS | Payment | Cardholder data protection |
| HIPAA | Healthcare | PHI protection |
| GDPR | EU data | Data privacy, right to erasure |
| FedRAMP | US Government | Cloud security for federal agencies |

### AWS Compliance Tools
- **AWS Config**: Continuous compliance monitoring
- **AWS Audit Manager**: Automated evidence collection
- **AWS Artifact**: Compliance reports and agreements
- **AWS Security Hub**: CIS benchmarks, PCI DSS, FSBP

### Azure Compliance Tools
- **Azure Policy**: Enforce compliance rules
- **Microsoft Defender for Cloud**: Regulatory compliance dashboard
- **Azure Compliance Manager**: Assessment and tracking
- **Microsoft Purview**: Data governance and compliance
