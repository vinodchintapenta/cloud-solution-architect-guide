# Kubernetes Production Architecture Deep Dive

---

## 🟢 Plain English Summary — Read This First

### What is this document about?
Kubernetes (often written as "K8s") is a system that manages running applications packaged in containers. Think of it like an air traffic control system — it decides where each plane (container) lands (runs), keeps them flying (restarts crashed ones), and scales up when more planes are needed.

This document covers how Kubernetes works internally, how to run it in production, and all the patterns you need to know for an architect interview.

---

### Abbreviation Decoder

| Abbreviation | Full Name | Plain English |
|---|---|---|
| K8s | Kubernetes | The container orchestration system. "K8s" = K + 8 letters + s. |
| Pod | (not an acronym) | The smallest unit in Kubernetes. One or more containers that run together and share network/storage. |
| etcd | (not an acronym) | The database where Kubernetes stores ALL its state. If etcd dies, the cluster is blind. Must be backed up. |
| API Server | kube-apiserver | The front door to Kubernetes. Every command goes through it. |
| CNI | Container Network Interface | A plugin that handles how containers get IP addresses and talk to each other. |
| CRI | Container Runtime Interface | A standard interface for container runtimes (containerd, CRI-O). |
| HPA | Horizontal Pod Autoscaler | Automatically adds more pods when CPU/memory is high. |
| VPA | Vertical Pod Autoscaler | Automatically adjusts how much CPU/memory each pod gets. |
| KEDA | Kubernetes Event-Driven Autoscaling | Scales pods based on queue depth, Kafka lag, etc. Can scale to zero. |
| PV | Persistent Volume | Storage that survives pod restarts. Like a hard drive attached to the cluster. |
| PVC | Persistent Volume Claim | A pod's request for storage. "I need 10GB of storage." |
| CSI | Container Storage Interface | A standard for storage plugins. |
| RBAC | Role-Based Access Control | Controls who can do what in the cluster. |
| OPA | Open Policy Agent | A policy engine. "No pod can run as root." "All pods must have resource limits." |
| mTLS | Mutual TLS | Both services prove their identity with certificates. Used by Istio for service-to-service security. |
| SPIFFE | Secure Production Identity Framework For Everyone | A standard for giving each service a cryptographic identity. |
| SVID | SPIFFE Verifiable Identity Document | The actual certificate that proves a service's identity. |
| PDB | Pod Disruption Budget | A rule that says "always keep at least 2 pods running, even during maintenance." |
| DaemonSet | (not an acronym) | A Kubernetes object that runs exactly one pod per node. Used for log collectors, monitoring agents. |
| StatefulSet | (not an acronym) | Like a Deployment but for stateful apps (databases). Pods get stable names (pod-0, pod-1). |
| ConfigMap | (not an acronym) | Stores non-secret configuration (environment variables, config files). |
| Secret | (not an acronym) | Stores sensitive configuration. WARNING: only base64 encoded by default, not encrypted. |
| Ingress | (not an acronym) | Rules for routing external HTTP traffic into the cluster. |
| AGIC | Application Gateway Ingress Controller | Azure's Ingress Controller that uses Azure Application Gateway. |
| ArgoCD | (not an acronym) | A GitOps tool that watches a Git repo and keeps the cluster in sync with it. |
| Helm | (not an acronym) | A package manager for Kubernetes. Like apt/npm but for K8s apps. |
| Kustomize | (not an acronym) | A tool for customizing Kubernetes manifests for different environments (dev/staging/prod). |
| IPVS | IP Virtual Server | A faster alternative to iptables for routing traffic to pods. |
| BGP | Border Gateway Protocol | A routing protocol used by Calico CNI for pod networking. |
| eBPF | Extended Berkeley Packet Filter | A technology used by Cilium CNI for very fast, programmable networking. |

---

### Kubernetes Architecture in Plain English

Think of Kubernetes as a company with two departments:

**Control Plane (Management Department):**
- **API Server**: The receptionist. All requests go through here.
- **etcd**: The filing cabinet. Stores everything about the cluster's state.
- **Scheduler**: The HR department. Decides which worker (node) gets which job (pod).
- **Controller Manager**: The supervisor. Constantly checks "is reality matching what we want?" and fixes it if not.

**Data Plane (Worker Department):**
- **Nodes**: The actual computers where your apps run.
- **kubelet**: The foreman on each node. Receives instructions from the API server and manages pods.
- **kube-proxy**: The traffic director on each node. Routes network traffic to the right pod.
- **Container Runtime**: The engine that actually runs containers (like containerd).

---

### Pod vs Container in Plain English

A **container** is like a single worker with a specific job.
A **pod** is like a small team of workers who share an office (same network, same storage). Usually a pod has 1 container, but sometimes 2 (e.g., app + sidecar for logging).

---

### Services in Plain English

Pods come and go (they crash, get replaced, scale up/down). Their IP addresses change. A **Service** is a stable address that always points to the right pods, no matter how many there are or what their IPs are.

- **ClusterIP**: Only accessible inside the cluster. For internal service-to-service calls.
- **NodePort**: Accessible from outside via a port on each node. For development/testing.
- **LoadBalancer**: Creates a cloud load balancer with a public IP. For production external services.

---

### Ingress in Plain English

A Service with type LoadBalancer creates one load balancer per service — expensive. **Ingress** is a smarter router: one load balancer handles all traffic, and routes to different services based on the URL path or hostname.

```
api.example.com/users  →  user-service
api.example.com/orders →  order-service
shop.example.com       →  shop-service
```

---

### Resource Requests vs Limits in Plain English

- **Request**: "I need at least this much CPU/memory to run." The scheduler uses this to decide which node to place the pod on.
- **Limit**: "I'm not allowed to use more than this." If a pod exceeds its memory limit, it gets killed (OOMKilled).

Always set both. Without requests, the scheduler can't make good decisions. Without limits, one bad pod can starve all others.

---

> Complete reference for Kubernetes architecture, from fundamentals to production patterns.

---

## 1. Kubernetes Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                     CONTROL PLANE                           │
│                                                             │
│  ┌──────────────┐  ┌──────────┐  ┌─────────────────────┐  │
│  │  API Server  │  │  etcd    │  │  Controller Manager  │  │
│  │  (kube-      │  │  (state  │  │  (reconciliation     │  │
│  │  apiserver)  │  │  store)  │  │   loops)             │  │
│  └──────────────┘  └──────────┘  └─────────────────────┘  │
│         │                              │                    │
│  ┌──────────────┐                      │                    │
│  │  Scheduler   │                      │                    │
│  │  (pod        │                      │                    │
│  │  placement)  │                      │                    │
│  └──────────────┘                      │                    │
└─────────────────────────────────────────────────────────────┘
         │ (watches API server)
┌─────────────────────────────────────────────────────────────┐
│                      DATA PLANE (Nodes)                     │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  Node 1                                              │  │
│  │  ┌──────────┐  ┌────────────┐  ┌─────────────────┐  │  │
│  │  │ kubelet  │  │ kube-proxy │  │ Container       │  │  │
│  │  │(pod mgmt)│  │(iptables/  │  │ Runtime         │  │  │
│  │  │          │  │ ipvs)      │  │ (containerd)    │  │  │
│  │  └──────────┘  └────────────┘  └─────────────────┘  │  │
│  │  ┌──────────────────────────────────────────────┐    │  │
│  │  │  Pod A          Pod B          Pod C          │    │  │
│  │  │  [Container]    [Container]    [Container]    │    │  │
│  │  └──────────────────────────────────────────────┘    │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. Control Plane Components

### API Server (kube-apiserver)
- The front door to Kubernetes — all communication goes through it
- RESTful API, validates and processes requests
- Stores state in etcd
- Authentication → Authorization (RBAC) → Admission Control → etcd
- Horizontally scalable (run multiple replicas behind a load balancer)

### etcd
- Distributed key-value store — the brain of Kubernetes
- Stores ALL cluster state: pods, services, configmaps, secrets, etc.
- Uses Raft consensus algorithm (needs odd number of nodes: 3, 5, 7)
- Critical: back up etcd regularly
- Performance: etcd needs fast SSDs (low latency I/O)

### Scheduler (kube-scheduler)
- Watches for unscheduled pods
- Selects the best node based on:
  - Resource requests/limits
  - Node affinity/anti-affinity
  - Taints and tolerations
  - Pod affinity/anti-affinity
  - Topology spread constraints
- Two phases: Filtering (eliminate unsuitable nodes) → Scoring (rank remaining)

### Controller Manager (kube-controller-manager)
- Runs control loops (reconciliation loops)
- Key controllers:
  - **ReplicaSet controller**: Ensures desired pod count
  - **Deployment controller**: Manages rolling updates
  - **Node controller**: Monitors node health
  - **Service Account controller**: Creates default service accounts
  - **Endpoint controller**: Populates Endpoints objects

### Cloud Controller Manager
- Integrates with cloud provider APIs
- Manages: Load balancers, node lifecycle, routes, volumes

---

## 3. Data Plane Components

### kubelet
- Agent running on every node
- Watches API server for pods assigned to its node
- Manages pod lifecycle: create, start, stop, delete containers
- Reports node and pod status back to API server
- Runs liveness/readiness/startup probes

### kube-proxy
- Network proxy on every node
- Maintains network rules (iptables or IPVS)
- Implements Service abstraction (ClusterIP → Pod IPs)
- IPVS mode: Better performance for large clusters (hash table vs iptables chains)

### Container Runtime
- Implements CRI (Container Runtime Interface)
- containerd: Default in most distributions
- CRI-O: Lightweight, OCI-compliant
- Docker: Deprecated as runtime (still used for building images)

---

## 4. Kubernetes Networking

### Pod Networking
- Every pod gets a unique IP address
- Pods can communicate with any other pod without NAT
- CNI (Container Network Interface) plugins implement this:
  - **Flannel**: Simple overlay, good for small clusters
  - **Calico**: Network policies, BGP routing, high performance
  - **Cilium**: eBPF-based, L7 policies, observability
  - **Weave**: Simple, encrypted by default
  - **AWS VPC CNI**: Pods get VPC IPs (native routing)
  - **Azure CNI**: Pods get VNet IPs

### Service Types

```
ClusterIP (default)
  - Internal cluster IP
  - Only accessible within cluster
  - Use for: internal service communication

NodePort
  - Exposes service on each node's IP at a static port (30000-32767)
  - Accessible from outside cluster via NodeIP:NodePort
  - Use for: development, testing

LoadBalancer
  - Creates external load balancer (cloud provider)
  - Gets external IP
  - Use for: production external services

ExternalName
  - Maps service to external DNS name
  - Use for: accessing external services by internal name
```

### Ingress

```
Internet
  ↓
[Ingress Controller]           ← nginx, traefik, AWS ALB, Azure AGIC
  ↓ (routes based on rules)
[Service A]  [Service B]  [Service C]
  ↓              ↓              ↓
[Pods A]     [Pods B]      [Pods C]
```

Ingress resource defines routing rules:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: api.example.com
    http:
      paths:
      - path: /users
        pathType: Prefix
        backend:
          service:
            name: user-service
            port:
              number: 80
      - path: /orders
        pathType: Prefix
        backend:
          service:
            name: order-service
            port:
              number: 80
  tls:
  - hosts:
    - api.example.com
    secretName: tls-secret
```

### Network Policies
- Default: all pods can communicate with all pods
- NetworkPolicy restricts traffic (requires CNI support: Calico, Cilium)

```yaml
# Deny all ingress, allow only from specific namespace
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-from-frontend
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: frontend
    - podSelector:
        matchLabels:
          app: frontend
```

---

## 5. Kubernetes Workloads

### Pod
- Smallest deployable unit
- One or more containers sharing network namespace and volumes
- Ephemeral: don't rely on pod IP or hostname

### ReplicaSet
- Ensures N replicas of a pod are running
- Rarely used directly — use Deployment instead

### Deployment
- Manages ReplicaSets for rolling updates and rollbacks
- Strategies: RollingUpdate (default) or Recreate
- RollingUpdate: maxSurge (extra pods during update), maxUnavailable

### StatefulSet
- For stateful applications (databases, Kafka, Zookeeper)
- Stable network identity: pod-0, pod-1, pod-2
- Stable storage: PVC per pod, not deleted on pod deletion
- Ordered deployment and scaling

### DaemonSet
- Runs one pod per node (or per selected nodes)
- Use for: log collectors (Fluentd), monitoring agents (Prometheus node exporter), CNI plugins

### Job / CronJob
- Job: Run to completion (batch processing)
- CronJob: Scheduled jobs (like cron)

---

## 6. Kubernetes Storage

```
[Pod] → [PVC (claim)] → [PV (volume)] → [Storage Backend]
                              ↑
                    [StorageClass] (dynamic provisioning)
```

### PersistentVolume (PV)
- Cluster-level storage resource
- Provisioned by admin or dynamically via StorageClass
- Access modes: ReadWriteOnce, ReadOnlyMany, ReadWriteMany, ReadWriteOncePod

### PersistentVolumeClaim (PVC)
- Pod's request for storage
- Bound to a PV that satisfies the request

### StorageClass
- Defines storage "class" (type, performance, reclaim policy)
- Enables dynamic provisioning
- AWS: gp2, gp3, io1 EBS volumes
- Azure: managed-premium, managed-standard, azurefile

### CSI (Container Storage Interface)
- Standard interface for storage plugins
- Allows third-party storage vendors to write drivers

---

## 7. Kubernetes Security

### RBAC (Role-Based Access Control)

```
[ServiceAccount / User / Group]
  ↓ bound by
[RoleBinding / ClusterRoleBinding]
  ↓ references
[Role / ClusterRole]
  ↓ defines
[Rules: verbs on resources]
```

```yaml
# Role: can read pods in namespace
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: production
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
---
# Bind role to service account
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: production
subjects:
- kind: ServiceAccount
  name: my-app
  namespace: production
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

### Pod Security
- **Pod Security Standards**: Privileged, Baseline, Restricted
- **Pod Security Admission**: Enforce standards at namespace level
- **OPA Gatekeeper**: Policy-as-code (more flexible)
- **Falco**: Runtime security, detect anomalous behavior

### Secrets Management
- Kubernetes Secrets: base64 encoded (NOT encrypted by default)
- Enable encryption at rest for etcd
- External Secrets Operator: Sync from AWS Secrets Manager / Azure Key Vault
- Sealed Secrets: Encrypt secrets for GitOps

---

## 8. Kubernetes Autoscaling

### Horizontal Pod Autoscaler (HPA)
- Scales pod replicas based on CPU/memory or custom metrics
- Requires metrics-server
- Custom metrics: Prometheus Adapter, KEDA

### Vertical Pod Autoscaler (VPA)
- Adjusts CPU/memory requests/limits
- Modes: Off (recommend only), Initial (set on creation), Auto (update running pods)
- Cannot be used with HPA on same metric

### Cluster Autoscaler
- Adds/removes nodes based on pending pods and underutilized nodes
- Works with cloud provider node groups
- AWS: Cluster Autoscaler or Karpenter (faster, more flexible)
- Azure: AKS cluster autoscaler

### KEDA (Kubernetes Event-Driven Autoscaling)
- Scale based on external events: queue depth, Kafka lag, HTTP requests
- Scales to zero (unlike HPA)
- 50+ scalers: SQS, Service Bus, Prometheus, Redis, etc.

---

## 9. Production Kubernetes Patterns

### Multi-Tenancy
```
Cluster per team (hard isolation)
  vs
Namespace per team (soft isolation)
  vs
Virtual clusters (vcluster - medium isolation)
```

Namespace isolation:
- RBAC: teams can only access their namespace
- NetworkPolicy: restrict cross-namespace traffic
- ResourceQuota: limit CPU/memory per namespace
- LimitRange: default requests/limits for pods

### High Availability
- Control plane: 3+ nodes, etcd 3+ nodes
- Worker nodes: spread across AZs
- Pod anti-affinity: spread pods across nodes/AZs
- Pod Disruption Budgets: ensure minimum availability during disruptions

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: my-app-pdb
spec:
  minAvailable: 2  # or maxUnavailable: 1
  selector:
    matchLabels:
      app: my-app
```

### Resource Management
```yaml
resources:
  requests:          # Scheduler uses this for placement
    cpu: "100m"      # 100 millicores = 0.1 CPU
    memory: "128Mi"
  limits:            # Container cannot exceed this
    cpu: "500m"
    memory: "512Mi"
```

Best practices:
- Always set requests (required for scheduling)
- Set limits to prevent noisy neighbors
- CPU limits cause throttling; memory limits cause OOM kills
- Use VPA recommendations to right-size

### GitOps with ArgoCD

```
Developer → Git Push → GitHub/GitLab
                           ↓
                      ArgoCD watches repo
                           ↓
                      Detects drift
                           ↓
                      Syncs to cluster
                           ↓
                      Kubernetes applies manifests
```

ArgoCD App of Apps pattern:
```
root-app (ArgoCD Application)
  → app-of-apps (ArgoCD Application)
      → service-a (ArgoCD Application)
      → service-b (ArgoCD Application)
      → infrastructure (ArgoCD Application)
```

---

## 10. Request Flow Through Kubernetes

```
1. User/Client sends HTTP request to api.example.com

2. DNS Resolution
   api.example.com → [Route 53 / Azure DNS] → Load Balancer IP

3. Cloud Load Balancer
   AWS ALB / Azure Load Balancer → Ingress Controller pods

4. Ingress Controller (nginx/traefik)
   - Reads Ingress rules
   - Matches host + path
   - Routes to correct Service

5. Service (ClusterIP)
   - kube-proxy maintains iptables/IPVS rules
   - Selects a healthy pod (round-robin by default)
   - Translates ClusterIP:Port → PodIP:Port

6. Pod
   - Request arrives at container port
   - Application processes request

7. Response flows back through same path
```

With Istio service mesh:
```
1-4. Same as above

5. Envoy sidecar (in calling pod)
   - Intercepts outbound traffic
   - Applies traffic policies (retries, timeouts, circuit breaking)
   - mTLS encryption

6. Envoy sidecar (in target pod)
   - Intercepts inbound traffic
   - Verifies mTLS certificate
   - Applies authorization policies
   - Forwards to application container

7. Response flows back through Envoy sidecars
```
