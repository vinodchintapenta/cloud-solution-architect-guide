# GitOps Production Architecture

---

## 🟢 Plain English Summary — Read This First

### What is this document about?
GitOps is a way of managing infrastructure and deployments where **Git is the single source of truth**. Instead of someone running commands to deploy, a robot (ArgoCD or Flux) watches your Git repository and automatically makes the cluster match what's in Git.

Think of it like this: instead of telling your team "deploy version 2.0 now," you update a file in Git that says "the desired version is 2.0," and a robot automatically makes it happen. If someone manually changes something in the cluster, the robot detects the drift and reverts it.

---

### Abbreviation Decoder

| Abbreviation | Full Name | Plain English |
|---|---|---|
| GitOps | Git Operations | Using Git as the single source of truth for infrastructure and deployments. |
| ArgoCD | (not an acronym) | A popular GitOps tool that watches a Git repo and syncs a Kubernetes cluster to match it. |
| Flux | (not an acronym) | Another GitOps tool, similar to ArgoCD. |
| CI | Continuous Integration | Automatically building and testing code every time someone pushes to Git. |
| CD | Continuous Deployment | Automatically deploying code after it passes tests. |
| CI/CD | Continuous Integration / Continuous Deployment | The full pipeline from code push to production deployment. |
| Helm | (not an acronym) | A package manager for Kubernetes. Like npm for Node.js but for K8s apps. |
| Kustomize | (not an acronym) | A tool for customizing Kubernetes YAML files for different environments without duplicating them. |
| YAML | Yet Another Markup Language | The file format used for Kubernetes configuration. |
| SHA | Secure Hash Algorithm | A unique fingerprint for a Git commit. Used as an image tag to know exactly which version is deployed. |
| OCI | Open Container Initiative | A standard for container images and runtimes. |
| Crossplane | (not an acronym) | A tool that lets you manage cloud resources (RDS, S3, etc.) using Kubernetes manifests. |
| IDP | Internal Developer Platform | A self-service platform that lets developers provision infrastructure without needing to know cloud details. |
| Backstage | (not an acronym) | An open-source developer portal (created by Spotify) for building IDPs. |
| GPG | GNU Privacy Guard | A tool for signing Git commits to prove they came from you. |
| PR | Pull Request | A request to merge code changes into the main branch. Requires review and approval. |
| DLQ | Dead Letter Queue | Where failed messages go after too many retry attempts. |
| TTL | Time To Live | How long something is valid before it expires. |
| SealedSecret | (not an acronym) | An encrypted Kubernetes Secret that is safe to store in Git. |
| ESO | External Secrets Operator | A Kubernetes operator that syncs secrets from AWS Secrets Manager / Azure Key Vault into Kubernetes Secrets. |

---

### GitOps vs Traditional Deployment in Plain English

**Traditional (Push) Deployment:**
1. Developer writes code
2. CI pipeline builds and tests it
3. CI pipeline runs `kubectl apply` to deploy it
4. The CI pipeline needs cluster credentials (security risk!)

**GitOps (Pull) Deployment:**
1. Developer writes code
2. CI pipeline builds and tests it, pushes Docker image
3. CI pipeline updates a config file in Git (changes image tag)
4. ArgoCD (running inside the cluster) notices the Git change
5. ArgoCD pulls the change and applies it to the cluster
6. The CI pipeline NEVER needs cluster credentials!

The key insight: the cluster pulls changes from Git, rather than CI pushing changes to the cluster. This is more secure and gives you a complete audit trail.

---

### Why GitOps Matters for Swiss Re

1. **Audit trail**: Every deployment is a Git commit. You can see who deployed what, when, and why.
2. **Rollback**: `git revert` undoes a deployment. No special rollback commands needed.
3. **Drift detection**: If someone manually changes something in production, ArgoCD detects it and reverts it.
4. **Security**: CI pipelines don't need cluster credentials. Only ArgoCD (inside the cluster) has access.
5. **Compliance**: Every change is reviewed via Pull Request before it reaches production.

---

### Secrets in GitOps — The Problem and Solutions

**The Problem**: You can't store passwords in Git (it's public or at least accessible to many people).

**Solution 1 — Sealed Secrets**: Encrypt the secret with the cluster's public key. The encrypted version is safe to commit to Git. Only the cluster can decrypt it.

**Solution 2 — External Secrets Operator**: Store secrets in AWS Secrets Manager or Azure Key Vault. A Kubernetes operator syncs them into the cluster automatically. Git only has a reference to the secret, not the secret itself.

**Solution 3 — HashiCorp Vault**: A dedicated secrets management system. Pods authenticate to Vault and get secrets injected at runtime.

---

> GitOps is an operational framework that applies DevOps best practices (version control, collaboration, CI/CD) to infrastructure automation.

---

## 1. GitOps Principles

1. **Declarative**: Describe desired state, not imperative steps
2. **Versioned and immutable**: Git is the single source of truth
3. **Pulled automatically**: Software agents pull and apply changes
4. **Continuously reconciled**: Agents detect and correct drift

### GitOps vs Traditional CI/CD

```
Traditional CI/CD (Push):
Developer → Git → CI Pipeline → kubectl apply → Cluster
                                    ↑
                              (has cluster credentials)

GitOps (Pull):
Developer → Git → CI Pipeline → Git (manifests updated)
                                    ↓
                              GitOps Agent (ArgoCD/Flux)
                              watches Git, pulls changes
                                    ↓
                              Cluster (agent has credentials)
```

Benefits of GitOps:
- Audit trail: every change is a Git commit
- Rollback: git revert
- Drift detection: agent detects manual changes
- No cluster credentials in CI pipeline
- Declarative, self-healing

---

## 2. ArgoCD Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    ArgoCD                               │
│                                                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐ │
│  │  API Server  │  │  Repo Server │  │  Application │ │
│  │  (REST/gRPC) │  │  (git clone, │  │  Controller  │ │
│  │              │  │   template)  │  │  (reconcile) │ │
│  └──────────────┘  └──────────────┘  └──────────────┘ │
│         ↓                  ↓                  ↓        │
│  ┌──────────────────────────────────────────────────┐  │
│  │              Redis (cache)                       │  │
│  └──────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
         ↓ watches                    ↓ syncs
[Git Repository]              [Kubernetes Cluster]
```

### ArgoCD Components

**API Server**: REST/gRPC API, web UI, CLI interface
**Repo Server**: Clones Git repos, generates manifests (Helm, Kustomize, plain YAML)
**Application Controller**: Watches cluster state, compares with desired state, syncs

### ArgoCD Application Resource
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/myorg/my-app-config
    targetRevision: HEAD
    path: environments/production
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true      # delete resources removed from Git
      selfHeal: true   # revert manual changes
    syncOptions:
    - CreateNamespace=true
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m
```

---

## 3. GitOps Repository Structure

### Mono-repo Pattern
```
gitops-repo/
├── apps/
│   ├── frontend/
│   │   ├── base/
│   │   │   ├── deployment.yaml
│   │   │   ├── service.yaml
│   │   │   └── kustomization.yaml
│   │   └── overlays/
│   │       ├── dev/
│   │       │   └── kustomization.yaml  (patches for dev)
│   │       ├── staging/
│   │       │   └── kustomization.yaml
│   │       └── production/
│   │           └── kustomization.yaml
│   └── backend/
│       └── (same structure)
├── infrastructure/
│   ├── cert-manager/
│   ├── ingress-nginx/
│   ├── prometheus/
│   └── argocd/
└── clusters/
    ├── dev/
    │   └── apps.yaml  (ArgoCD ApplicationSet)
    ├── staging/
    │   └── apps.yaml
    └── production/
        └── apps.yaml
```

### Multi-repo Pattern
```
app-source-repo/          ← developers own this
  src/
  Dockerfile
  .github/workflows/
    ci.yaml               ← builds image, updates config repo

app-config-repo/          ← platform team owns this
  environments/
    dev/
    staging/
    production/
```

### App of Apps Pattern
```
root-app (ArgoCD Application)
  → points to: clusters/production/
  → contains: ArgoCD Application manifests

clusters/production/
  ├── infrastructure-app.yaml  → points to infrastructure/
  ├── monitoring-app.yaml      → points to monitoring/
  └── workloads-app.yaml       → points to workloads/

workloads/
  ├── frontend-app.yaml        → points to apps/frontend/overlays/production
  └── backend-app.yaml         → points to apps/backend/overlays/production
```

---

## 4. Complete GitOps Pipeline

### CI Pipeline (GitHub Actions)
```yaml
name: CI
on:
  push:
    branches: [main]

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Build Docker image
      run: docker build -t myapp:${{ github.sha }} .
    
    - name: Run tests
      run: docker run myapp:${{ github.sha }} npm test
    
    - name: Security scan
      uses: aquasecurity/trivy-action@master
      with:
        image-ref: myapp:${{ github.sha }}
    
    - name: Push to registry
      run: |
        docker tag myapp:${{ github.sha }} myregistry/myapp:${{ github.sha }}
        docker push myregistry/myapp:${{ github.sha }}
    
    - name: Update config repo
      run: |
        git clone https://github.com/myorg/app-config
        cd app-config
        # Update image tag in production overlay
        sed -i "s|image: myregistry/myapp:.*|image: myregistry/myapp:${{ github.sha }}|" \
          environments/production/deployment.yaml
        git commit -am "Update production image to ${{ github.sha }}"
        git push
```

### CD Flow (ArgoCD)
```
1. GitHub Actions pushes new image tag to config repo
2. ArgoCD detects change in config repo (polls every 3 min or webhook)
3. ArgoCD compares desired state (Git) with actual state (cluster)
4. ArgoCD syncs: applies new deployment with updated image
5. Kubernetes performs rolling update
6. ArgoCD reports sync status (Synced/OutOfSync/Degraded)
```

### Promotion Flow (Dev → Staging → Production)
```
Feature branch → PR → Review → Merge to main
                                    ↓
                              CI: build + test + push image
                                    ↓
                              Auto-update dev environment
                                    ↓
                              Manual PR: promote to staging
                                    ↓
                              Staging tests pass
                                    ↓
                              Manual PR: promote to production
                                    ↓
                              Production deployment (ArgoCD)
```

---

## 5. Helm in GitOps

### Helm Chart Structure
```
my-app/
├── Chart.yaml          ← chart metadata
├── values.yaml         ← default values
├── values-dev.yaml     ← dev overrides
├── values-prod.yaml    ← prod overrides
└── templates/
    ├── deployment.yaml
    ├── service.yaml
    ├── ingress.yaml
    ├── hpa.yaml
    └── _helpers.tpl    ← template helpers
```

### ArgoCD with Helm
```yaml
spec:
  source:
    repoURL: https://github.com/myorg/helm-charts
    chart: my-app
    targetRevision: 1.2.3
    helm:
      valueFiles:
      - values-production.yaml
      parameters:
      - name: image.tag
        value: "abc123"
```

---

## 6. Kustomize in GitOps

### Kustomize Overlay Pattern
```yaml
# base/kustomization.yaml
resources:
- deployment.yaml
- service.yaml

# overlays/production/kustomization.yaml
bases:
- ../../base
patches:
- patch: |-
    - op: replace
      path: /spec/replicas
      value: 5
  target:
    kind: Deployment
    name: my-app
images:
- name: myapp
  newTag: "1.2.3"
configMapGenerator:
- name: app-config
  literals:
  - LOG_LEVEL=info
  - ENVIRONMENT=production
```

---

## 7. Secrets in GitOps

### Problem: Secrets cannot be stored in Git (plaintext)

### Solution 1: Sealed Secrets
```
Developer → kubeseal (encrypt with cluster public key) → SealedSecret (safe to commit)
                                                              ↓
                                                    Sealed Secrets Controller
                                                    (decrypts with private key)
                                                              ↓
                                                    Kubernetes Secret
```

### Solution 2: External Secrets Operator
```
ExternalSecret resource (safe to commit)
  ↓
External Secrets Operator
  ↓
AWS Secrets Manager / Azure Key Vault / HashiCorp Vault
  ↓
Kubernetes Secret (created in cluster, not in Git)
```

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: db-credentials
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: aws-secrets-manager
    kind: ClusterSecretStore
  target:
    name: db-credentials
  data:
  - secretKey: password
    remoteRef:
      key: production/db/password
```

### Solution 3: HashiCorp Vault + Vault Agent
```
Pod → Vault Agent Sidecar → Vault → Secret
         (init container)    (authenticate via K8s SA)
         (writes secret to shared volume)
```

---

## 8. Multi-Cluster GitOps

### ArgoCD Hub-Spoke
```
Management Cluster
  [ArgoCD] → manages → [Dev Cluster]
           → manages → [Staging Cluster]
           → manages → [Production Cluster]
```

### ArgoCD ApplicationSet
```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: my-app-all-clusters
spec:
  generators:
  - list:
      elements:
      - cluster: dev
        url: https://dev-cluster.example.com
      - cluster: staging
        url: https://staging-cluster.example.com
      - cluster: production
        url: https://prod-cluster.example.com
  template:
    metadata:
      name: 'my-app-{{cluster}}'
    spec:
      source:
        repoURL: https://github.com/myorg/app-config
        path: 'environments/{{cluster}}'
      destination:
        server: '{{url}}'
        namespace: my-app
```

---

## 9. GitOps for Infrastructure (Crossplane)

```
Git (infrastructure manifests)
  ↓
ArgoCD
  ↓
Crossplane (runs in Kubernetes)
  ↓
Cloud Provider APIs (AWS, Azure, GCP)
  ↓
Cloud Resources (RDS, S3, AKS, etc.)
```

Crossplane Composite Resource:
```yaml
apiVersion: database.example.com/v1alpha1
kind: PostgreSQLInstance
metadata:
  name: my-db
spec:
  parameters:
    storageGB: 20
    region: us-east-1
  compositionRef:
    name: postgresql-aws
```

---

## 10. GitOps Best Practices

### Repository Strategy
- Separate app code repo from config repo
- Config repo: only manifests, no application code
- Branch protection on main/production branches
- Required PR reviews for production changes

### Security
- Signed commits (GPG)
- Branch protection rules
- Least privilege for ArgoCD service account
- Audit log for all sync operations
- Secrets never in Git (use External Secrets or Sealed Secrets)

### Observability
- ArgoCD metrics → Prometheus → Grafana
- Alert on: sync failures, degraded apps, OutOfSync state
- Notification controller: Slack/Teams alerts on sync events

### Rollback Strategy
```
Option 1: Git revert
  git revert <commit>
  git push → ArgoCD detects → syncs old state

Option 2: ArgoCD UI/CLI
  argocd app rollback my-app <revision>

Option 3: Helm rollback
  helm rollback my-app 1
```
