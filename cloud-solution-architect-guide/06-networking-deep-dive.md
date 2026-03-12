# Networking Deep Dive
## Ingress, Istio, Load Balancing, DNS, Routing — AWS & Azure

---

## 🟢 Plain English Summary — Read This First

### What is this document about?
When a user types "api.example.com" in their browser, a LOT happens before your application receives the request. This document traces that entire journey — from DNS lookup to load balancer to Kubernetes ingress to your pod — and explains every component along the way.

Think of it like tracing a letter from the moment you drop it in a mailbox to when it arrives at the recipient's door. Every stop along the way has a name and a purpose.

---

### Abbreviation Decoder

| Abbreviation | Full Name | Plain English |
|---|---|---|
| DNS | Domain Name System | The internet's phone book. Translates "google.com" into an IP address like "142.250.80.46". |
| TTL | Time To Live | How long a DNS answer is cached. Low TTL = faster failover but more DNS queries. |
| CDN | Content Delivery Network | Servers around the world that cache your content. Users get served from the nearest server. |
| ALB | Application Load Balancer | AWS load balancer that understands HTTP. Can route based on URL path, hostname, headers. |
| NLB | Network Load Balancer | AWS load balancer for TCP/UDP. Extremely fast. Gets a static IP address. |
| CLB | Classic Load Balancer | Old AWS load balancer. Don't use for new architectures. |
| WAF | Web Application Firewall | Filters malicious requests (SQL injection, XSS) before they reach your app. |
| SSL | Secure Sockets Layer | Old name for TLS. You'll still see "SSL certificate" but it's actually TLS. |
| TLS | Transport Layer Security | Encrypts traffic between client and server. The "S" in HTTPS. |
| ACM | AWS Certificate Manager | AWS service that provides free TLS certificates. |
| NSG | Network Security Group | Azure's firewall rules for subnets and network interfaces. |
| VNet | Virtual Network | Azure's private cloud network. |
| VPC | Virtual Private Cloud | AWS's private cloud network. |
| AGIC | Application Gateway Ingress Controller | Azure's Kubernetes Ingress Controller that uses Application Gateway. |
| nginx | (not an acronym) | A popular web server also used as a Kubernetes Ingress Controller. |
| Istio | (not an acronym) | A service mesh — adds security, observability, and traffic management to all service-to-service calls. |
| mTLS | Mutual TLS | Both sides of a connection prove their identity. Not just "is the server real?" but also "is the client real?" |
| Envoy | (not an acronym) | A high-performance proxy used by Istio as a sidecar. Intercepts all traffic in/out of a pod. |
| istiod | (not an acronym) | The Istio control plane. Manages certificates, pushes config to all Envoy sidecars. |
| xDS | (not an acronym) | The protocol istiod uses to push configuration to Envoy proxies. |
| SPIFFE | Secure Production Identity Framework For Everyone | A standard for giving each service a cryptographic identity (like a passport). |
| SVID | SPIFFE Verifiable Identity Document | The actual certificate that proves a service's identity. |
| PoP | Point of Presence | A CDN/Front Door edge location. The nearest server to the user. |
| CNAME | Canonical Name | A DNS record that points one domain name to another. |
| A Record | Address Record | A DNS record that points a domain name to an IP address. |
| BGP | Border Gateway Protocol | The routing protocol that makes the internet work. Routers use it to find paths. |
| QUIC | Quick UDP Internet Connections | The protocol behind HTTP/3. Faster connection setup than TCP. |
| SNI | Server Name Indication | A TLS extension that lets one server host multiple SSL certificates (for different domains). |

---

### The Request Journey in Plain English

When you type "api.example.com" in your browser:

**Step 1 — DNS Lookup:**
Your computer asks "what's the IP address for api.example.com?" It checks its own cache, then asks your ISP's DNS server, which eventually asks the authoritative DNS server (Route 53 or Azure DNS). You get back an IP address.

**Step 2 — CDN/Edge (optional):**
If the company uses CloudFront or Azure Front Door, your request goes to the nearest edge server (maybe in your city). If the content is cached there, you get it immediately. If not, the edge server fetches it from the origin.

**Step 3 — Load Balancer:**
The request hits a load balancer (ALB or Application Gateway). It checks the URL path and hostname, applies WAF rules, terminates SSL, and routes to the right backend.

**Step 4 — Kubernetes Ingress:**
Inside Kubernetes, the Ingress Controller (nginx or AGIC) reads the routing rules and sends the request to the right Kubernetes Service.

**Step 5 — Service → Pod:**
kube-proxy routes the request from the Service's virtual IP to an actual pod IP.

**Step 6 — Your Application:**
The request arrives at your container. It processes it and sends back a response.

---

### Load Balancer Types in Plain English

**Layer 4 (Transport Layer) Load Balancer:**
- Sees: IP addresses and port numbers
- Doesn't understand: HTTP, URLs, cookies
- Fast and simple
- AWS: NLB | Azure: Azure Load Balancer

**Layer 7 (Application Layer) Load Balancer:**
- Sees: HTTP headers, URLs, cookies, hostnames
- Can route: /api → service A, /web → service B
- Can do: SSL termination, WAF, sticky sessions
- AWS: ALB | Azure: Application Gateway

**Global Load Balancer:**
- Routes users to the nearest region
- AWS: Route 53 + CloudFront | Azure: Traffic Manager + Front Door

---

### Istio Service Mesh in Plain English

Without Istio: Service A calls Service B directly. If you want encryption, retries, or tracing, every service must implement it themselves.

With Istio: A tiny proxy (Envoy) is injected into every pod as a sidecar. All traffic goes through this proxy. The proxy handles:
- **mTLS**: Automatically encrypts all service-to-service traffic
- **Retries**: Automatically retries failed requests
- **Circuit Breaking**: Stops calling a failing service
- **Tracing**: Records how long each call takes
- **Traffic Splitting**: Send 10% of traffic to the new version

Your application code doesn't change at all — the sidecar handles everything.

---

### SSL/TLS Termination in Plain English

"Termination" means decrypting the HTTPS traffic. You have choices:

- **Terminate at the edge**: CDN decrypts it, sends plain HTTP internally. Simple but traffic is unencrypted inside your network.
- **Re-encrypt**: CDN decrypts, then re-encrypts before sending to the load balancer. More secure.
- **End-to-end (passthrough)**: Never decrypted until it reaches your app. Most secure, but load balancer can't inspect the traffic.

For most companies: terminate at the load balancer, use mTLS (Istio) for internal service-to-service traffic.

---

> This document explains the complete request flow from a user's browser to your application, covering every networking component along the way.

---

## 1. The Big Picture — Request Flow

### Generic Request Flow (Any Cloud)

```
[User Browser]
    ↓ DNS lookup: api.example.com
[DNS Resolver] (ISP or 8.8.8.8)
    ↓ recursive lookup
[Authoritative DNS] (Route 53 / Azure DNS)
    ↓ returns IP of load balancer
[Global Load Balancer / CDN Edge]  ← CloudFront / Azure Front Door
    ↓ routes to nearest region
[Regional Load Balancer]           ← ALB / Application Gateway
    ↓ L7 routing (host, path, headers)
[Ingress Controller]               ← nginx / AWS ALB Ingress / AGIC
    ↓ Kubernetes routing rules
[Service (ClusterIP)]              ← kube-proxy iptables/IPVS
    ↓ pod selection
[Pod / Container]                  ← your application
    ↓ (optional) service mesh
[Envoy Sidecar]                    ← Istio mTLS, policies
    ↓
[Downstream Service / Database]
```

---

## 2. DNS Deep Dive

### How DNS Works (Step by Step)

```
1. Browser checks local cache → not found
2. OS checks /etc/hosts → not found
3. OS queries local DNS resolver (ISP or 8.8.8.8)
4. Resolver checks its cache → not found
5. Resolver queries Root DNS servers (.) → returns .com nameservers
6. Resolver queries .com TLD servers → returns example.com nameservers
7. Resolver queries example.com nameservers (Route 53 / Azure DNS)
8. Returns A record (IPv4) or CNAME
9. Resolver caches result (TTL)
10. Browser connects to returned IP
```

### DNS Record Types

| Record | Purpose | Example |
|--------|---------|---------|
| A | IPv4 address | api.example.com → 1.2.3.4 |
| AAAA | IPv6 address | api.example.com → 2001:db8::1 |
| CNAME | Alias to another name | www → api.example.com |
| MX | Mail server | example.com → mail.example.com |
| TXT | Text (SPF, DKIM, verification) | "v=spf1 include:..." |
| NS | Nameserver | example.com → ns1.route53.com |
| SOA | Start of Authority | Zone metadata |
| SRV | Service location | _http._tcp.example.com |
| PTR | Reverse DNS | 1.2.3.4 → api.example.com |

### AWS Route 53

**Routing Policies:**

```
Simple Routing
  → Single resource, no health checks
  → Use for: single server, development

Weighted Routing
  → Route 10% to v2, 90% to v1
  → Use for: A/B testing, gradual migration

Latency Routing
  → Route to lowest latency region
  → Use for: global applications

Failover Routing
  → Primary: us-east-1 (active)
  → Secondary: eu-west-1 (passive, only if primary fails)
  → Use for: active-passive DR

Geolocation Routing
  → EU users → EU region
  → US users → US region
  → Use for: data residency, localization

Geoproximity Routing
  → Route based on geographic distance
  → Bias: expand/shrink routing area
  → Use for: fine-grained geographic control

Multi-Value Routing
  → Return multiple IPs (up to 8)
  → Health checks on each
  → Use for: simple load distribution
```

**Route 53 Health Checks:**
- HTTP/HTTPS/TCP health checks
- Can check string in response body
- Calculated health checks (combine multiple)
- CloudWatch alarm-based health checks

### Azure DNS

**Public DNS Zones:**
- Manage DNS records for internet-facing domains
- Delegate domain to Azure DNS nameservers

**Private DNS Zones:**
- Name resolution within VNets
- Auto-registration of VM hostnames
- Link to multiple VNets

**Azure DNS Private Resolver:**
- Inbound endpoint: on-prem → Azure DNS
- Outbound endpoint: Azure → on-prem DNS
- Conditional forwarding rules

**Azure Traffic Manager (DNS-based load balancing):**
```
Routing methods:
- Priority: Primary/failover
- Weighted: Distribute traffic by weight
- Performance: Route to lowest latency endpoint
- Geographic: Route by user location
- Multivalue: Return multiple healthy endpoints
- Subnet: Route by client IP subnet
```

---

## 3. Load Balancing Deep Dive

### OSI Model Context

```
Layer 7 (Application) — HTTP/HTTPS, host-based, path-based routing
Layer 4 (Transport)   — TCP/UDP, IP:Port routing
Layer 3 (Network)     — IP routing
```

### AWS Load Balancers

#### Application Load Balancer (ALB) — Layer 7

```
Features:
- Host-based routing: api.example.com → service A, web.example.com → service B
- Path-based routing: /api/* → service A, /static/* → S3
- Header-based routing: X-Version: v2 → new deployment
- Query string routing: ?version=2 → new deployment
- Weighted target groups: 90% → v1, 10% → v2
- SSL/TLS termination
- WebSocket support
- HTTP/2 support
- WAF integration
- Lambda as target

Target Groups:
- EC2 instances
- ECS tasks
- Lambda functions
- IP addresses (on-prem, other VPCs)

Health Checks:
- HTTP/HTTPS path check
- Configurable: interval, timeout, healthy/unhealthy threshold
```

#### Network Load Balancer (NLB) — Layer 4

```
Features:
- Ultra-low latency (millions of requests/sec)
- Static IP per AZ (or Elastic IP)
- Preserve client IP
- TCP, UDP, TLS
- No WAF integration
- PrivateLink support

Use NLB when:
- Need static IP
- Non-HTTP protocols (MQTT, custom TCP)
- Extreme performance requirements
- PrivateLink endpoint
```

#### Classic Load Balancer (CLB) — Legacy
- Avoid for new architectures
- Use ALB or NLB instead

#### AWS Load Balancer Decision

```
HTTP/HTTPS with routing rules → ALB
TCP/UDP, static IP, extreme performance → NLB
Kubernetes Ingress on EKS → AWS Load Balancer Controller (creates ALB)
```

### Azure Load Balancers

#### Azure Load Balancer — Layer 4

```
SKUs: Basic (free, limited) vs Standard (production)

Features:
- TCP/UDP load balancing
- Inbound NAT rules
- Outbound rules
- HA ports (all ports at once)
- Zone-redundant (Standard)

Use for:
- Non-HTTP workloads
- Internal load balancing
- VM scale sets
```

#### Application Gateway — Layer 7

```
Features:
- URL-based routing
- Multi-site hosting
- SSL/TLS termination and re-encryption
- Cookie-based session affinity
- WebSocket and HTTP/2
- WAF (Web Application Firewall)
- Autoscaling (v2)
- Zone redundancy (v2)
- Private frontend IP

Tiers:
- Standard v2: L7 load balancing
- WAF v2: Includes WAF

Use for:
- Regional HTTP/S load balancing
- WAF requirement
- SSL offloading
- AKS Ingress (AGIC - Application Gateway Ingress Controller)
```

#### Azure Front Door — Global Layer 7

```
Features:
- Global HTTP/S load balancing
- Anycast routing (nearest PoP)
- SSL/TLS termination at edge
- WAF at edge
- CDN capabilities
- URL rewrite, redirect
- Health probes
- Session affinity
- Private Link to origins

Use for:
- Global web applications
- Multi-region active-active
- CDN + WAF + LB in one service
```

#### Traffic Manager — DNS-based Global

```
Features:
- DNS-based routing (not proxy)
- Works with any internet-facing endpoint
- Routing methods: Priority, Weighted, Performance, Geographic, Multivalue, Subnet
- Nested profiles

Use for:
- Non-HTTP global routing
- Combining with other load balancers
- Failover between regions
```

#### Azure Load Balancer Decision

```
Internal TCP/UDP → Azure Load Balancer (internal)
Regional HTTP/S + WAF → Application Gateway
Global HTTP/S + CDN + WAF → Azure Front Door
Global DNS-based routing → Traffic Manager
AKS Ingress → AGIC (Application Gateway) or nginx Ingress
```

---

## 4. Ingress Deep Dive (Kubernetes)

### What is Ingress?

Ingress is a Kubernetes API object that manages external access to services in a cluster, typically HTTP/HTTPS. It provides:
- Host-based routing
- Path-based routing
- SSL/TLS termination
- Name-based virtual hosting

### Ingress Controller

The Ingress resource is just a configuration — you need an Ingress Controller to implement it.

```
[Ingress Resource] ← defines rules (what)
[Ingress Controller] ← implements rules (how)
```

Popular Ingress Controllers:

| Controller | Best For |
|------------|----------|
| nginx-ingress | General purpose, most popular |
| AWS Load Balancer Controller | EKS, creates ALB per Ingress |
| AGIC (Azure) | AKS, uses Application Gateway |
| Traefik | Dynamic config, Let's Encrypt |
| HAProxy | High performance |
| Istio Gateway | Service mesh environments |
| Kong | API Gateway features |

### nginx Ingress Controller

```
Internet → [AWS NLB / Azure LB] → [nginx pods] → [Services] → [Pods]
```

nginx Ingress annotations (powerful customization):
```yaml
annotations:
  nginx.ingress.kubernetes.io/rewrite-target: /
  nginx.ingress.kubernetes.io/ssl-redirect: "true"
  nginx.ingress.kubernetes.io/proxy-body-size: "10m"
  nginx.ingress.kubernetes.io/rate-limit: "100"
  nginx.ingress.kubernetes.io/auth-url: "https://auth.example.com/verify"
  nginx.ingress.kubernetes.io/canary: "true"
  nginx.ingress.kubernetes.io/canary-weight: "10"
```

### AWS Load Balancer Controller (EKS)

```
Ingress resource → ALB Controller → Creates AWS ALB
                                  → Creates Target Groups
                                  → Registers pods as targets
```

Each Ingress creates a new ALB (or share with IngressGroup annotation).

```yaml
annotations:
  kubernetes.io/ingress.class: alb
  alb.ingress.kubernetes.io/scheme: internet-facing
  alb.ingress.kubernetes.io/target-type: ip
  alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:...
  alb.ingress.kubernetes.io/group.name: my-group  # share ALB
```

### AGIC (Application Gateway Ingress Controller) — AKS

```
Ingress resource → AGIC → Configures Azure Application Gateway
                        → Creates backend pools
                        → Creates routing rules
```

---

## 5. Istio Service Mesh Deep Dive

### Why Service Mesh?

Without service mesh, each service must implement:
- mTLS between services
- Retries, timeouts, circuit breaking
- Distributed tracing
- Traffic splitting for canary
- Authorization policies

Service mesh moves this to the infrastructure layer (sidecar proxy).

### Istio Architecture

```
┌─────────────────────────────────────────────────────┐
│                  CONTROL PLANE                      │
│                                                     │
│  ┌─────────────────────────────────────────────┐   │
│  │              istiod                          │   │
│  │  ┌──────────┐ ┌──────────┐ ┌─────────────┐ │   │
│  │  │  Pilot   │ │ Citadel  │ │   Galley    │ │   │
│  │  │(traffic  │ │(certs,   │ │(config      │ │   │
│  │  │ mgmt)    │ │ mTLS)    │ │ validation) │ │   │
│  │  └──────────┘ └──────────┘ └─────────────┘ │   │
│  └─────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────┘
         ↓ pushes config via xDS protocol
┌─────────────────────────────────────────────────────┐
│                   DATA PLANE                        │
│                                                     │
│  Pod A                    Pod B                     │
│  ┌──────────────────┐    ┌──────────────────┐      │
│  │ [Envoy Sidecar]  │    │ [Envoy Sidecar]  │      │
│  │ [App Container]  │    │ [App Container]  │      │
│  └──────────────────┘    └──────────────────┘      │
│         ↕ mTLS                  ↕ mTLS              │
└─────────────────────────────────────────────────────┘
```

### Istio Traffic Management Resources

#### VirtualService
Defines routing rules for traffic to a service:
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  - match:
    - headers:
        end-user:
          exact: jason
    route:
    - destination:
        host: reviews
        subset: v2
  - route:
    - destination:
        host: reviews
        subset: v1
      weight: 90
    - destination:
        host: reviews
        subset: v2
      weight: 10
```

#### DestinationRule
Defines policies for traffic to a service (after routing):
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: reviews
spec:
  host: reviews
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100
    outlierDetection:
      consecutive5xxErrors: 5
      interval: 30s
      baseEjectionTime: 30s
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
    trafficPolicy:
      loadBalancer:
        simple: ROUND_ROBIN
```

#### Gateway
Manages inbound/outbound traffic at the mesh boundary:
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: my-gateway
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 443
      name: https
      protocol: HTTPS
    tls:
      mode: SIMPLE
      credentialName: my-cert
    hosts:
    - api.example.com
```

### Istio Traffic Patterns

#### Canary Deployment
```
100% → v1
  ↓ (gradually shift)
90% → v1, 10% → v2
  ↓
50% → v1, 50% → v2
  ↓
0% → v1, 100% → v2
```

#### Circuit Breaker
```
Normal: requests → service
Tripped: service returning errors → circuit opens → fast fail
Half-open: test request → if success, close circuit
```

#### Retry Policy
```yaml
retries:
  attempts: 3
  perTryTimeout: 2s
  retryOn: gateway-error,connect-failure,retriable-4xx
```

### Istio mTLS

```
Service A (Envoy) ←──── mTLS ────→ Service B (Envoy)
     ↑                                    ↑
  cert from istiod                   cert from istiod
  (SPIFFE/SVID)                      (SPIFFE/SVID)
```

- Certificates issued by istiod (acts as CA)
- SPIFFE (Secure Production Identity Framework For Everyone)
- SVID: SPIFFE Verifiable Identity Document
- Automatic rotation

### Istio Authorization Policy
```yaml
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: allow-frontend
spec:
  selector:
    matchLabels:
      app: backend
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/default/sa/frontend"]
    to:
    - operation:
        methods: ["GET"]
        paths: ["/api/*"]
```

---

## 6. Complete Request Flow — AWS (EKS + Istio)

```
Step 1: DNS Resolution
  User → api.example.com
  Route 53 → returns ALB DNS name (CNAME)
  ALB DNS → returns ALB IP

Step 2: Global CDN (optional)
  CloudFront edge → checks cache
  Cache miss → forwards to ALB

Step 3: AWS ALB
  - SSL/TLS termination (ACM certificate)
  - WAF rules checked
  - Host/path routing rules
  - Routes to Istio Ingress Gateway target group

Step 4: Istio Ingress Gateway
  - Envoy proxy (runs as pod)
  - Reads Istio Gateway + VirtualService rules
  - Routes to correct service

Step 5: Service Mesh (Envoy Sidecar)
  - Outbound: Envoy in calling pod intercepts
  - mTLS handshake with destination Envoy
  - Traffic policies applied (retry, timeout, circuit breaker)
  - Inbound: Envoy in target pod intercepts
  - Authorization policy checked
  - Forwards to application container

Step 6: Application
  - Processes request
  - May call other services (same flow)
  - Returns response

Step 7: Response
  - Flows back through same path
  - Metrics/traces collected at each Envoy
```

---

## 7. Complete Request Flow — Azure (AKS + nginx Ingress)

```
Step 1: DNS Resolution
  User → api.example.com
  Azure DNS → returns Azure Front Door hostname (CNAME)
  Front Door DNS → returns Front Door IP (anycast)

Step 2: Azure Front Door
  - Routes to nearest PoP
  - WAF rules checked
  - SSL/TLS termination
  - Routes to Application Gateway (origin)

Step 3: Application Gateway
  - Regional L7 load balancing
  - Additional WAF rules
  - Routes to nginx Ingress Controller pods

Step 4: nginx Ingress Controller
  - Reads Ingress resources
  - Host/path matching
  - Routes to Kubernetes Service

Step 5: Kubernetes Service
  - kube-proxy iptables rules
  - Selects healthy pod
  - Routes to pod IP

Step 6: Pod
  - Application processes request
  - Returns response

Step 7: Response
  - Flows back through same path
```

---

## 8. SSL/TLS Termination Strategies

### Terminate at Edge (Most Common)
```
Client ──TLS──→ CDN/Front Door ──HTTP──→ LB ──HTTP──→ App
```
- Pros: Simple, CDN can cache, less compute on app
- Cons: Traffic unencrypted inside network

### Re-encrypt (Defense in Depth)
```
Client ──TLS──→ CDN ──TLS──→ LB ──TLS──→ App
```
- Pros: Encrypted end-to-end
- Cons: More compute, certificate management

### Passthrough (E2E Encryption)
```
Client ──TLS──→ NLB (passthrough) ──TLS──→ App
```
- Pros: True end-to-end, no decryption at LB
- Cons: LB can't do L7 routing, no WAF

### mTLS (Mutual TLS)
```
Client ──mTLS──→ App (both sides present certificates)
```
- Use for: Service-to-service, API clients with certificates

---

## 9. Key Networking Concepts Summary

| Concept | AWS | Azure |
|---------|-----|-------|
| Virtual Network | VPC | VNet |
| Subnet | Subnet | Subnet |
| Firewall (instance) | Security Group | NSG |
| Firewall (subnet) | NACL | NSG (subnet level) |
| Managed Firewall | AWS Firewall Manager | Azure Firewall |
| DNS | Route 53 | Azure DNS |
| Global LB | Route 53 + CloudFront | Azure Front Door + Traffic Manager |
| Regional L7 LB | ALB | Application Gateway |
| Regional L4 LB | NLB | Azure Load Balancer |
| CDN | CloudFront | Azure CDN / Front Door |
| Private connectivity | PrivateLink | Private Link |
| Dedicated connection | Direct Connect | ExpressRoute |
| VPN | Site-to-Site VPN | VPN Gateway |
| Network hub | Transit Gateway | Virtual WAN / Hub VNet |
| Kubernetes Ingress | AWS LB Controller (ALB) | AGIC (App Gateway) or nginx |
| Service Mesh | Istio / App Mesh | Istio / Open Service Mesh |
