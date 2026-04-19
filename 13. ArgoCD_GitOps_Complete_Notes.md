# 🚀 ArgoCD + GitOps Complete Notes — Interview Prep

> **Stack Covered:** ArgoCD · GitHub Actions · Prometheus · Grafana · Amazon EKS  
> **Purpose:** Full GitOps pipeline setup, architecture deep-dive, and interview preparation

---

## Table of Contents

1. [What is GitOps?](#1-what-is-gitops)
2. [Why ArgoCD?](#2-why-argocd)
3. [ArgoCD Architecture](#3-argocd-architecture)
4. [Core ArgoCD Concepts](#4-core-argocd-concepts)
5. [ArgoCD Installation on EKS](#5-argocd-installation-on-eks)
6. [Full GitOps Pipeline with GitHub Actions](#6-full-gitops-pipeline-with-github-actions)
7. [Prometheus & Grafana Monitoring](#7-prometheus--grafana-monitoring)
8. [Amazon EKS Deep Dive](#8-amazon-eks-deep-dive)
9. [End-to-End GitOps Architecture](#9-end-to-end-gitops-architecture)
10. [Interview Q&A](#10-interview-qa)

---

## 1. What is GitOps?

### Definition

GitOps is a set of practices where **Git is the single source of truth** for both application code and infrastructure configuration. Every change to the system is made through a Git commit, and automated processes ensure the live system always matches what is declared in Git.

### GitOps Principles (OpenGitOps)

| Principle | Description |
|-----------|-------------|
| **Declarative** | The entire system is described declaratively (YAML, Helm, Kustomize) |
| **Versioned & Immutable** | All desired state is stored in Git with full history |
| **Pulled Automatically** | Software agents (like ArgoCD) continuously pull and apply state |
| **Continuously Reconciled** | Agents detect drift and self-heal toward desired state |

### Push vs Pull Model

```
❌ Traditional CI/CD (Push Model):
  CI Pipeline → kubectl apply → Cluster
  Problem: CI pipeline needs cluster credentials, hard to audit, no self-healing

✅ GitOps (Pull Model):
  Developer → Git → ArgoCD (inside cluster) → Cluster
  Benefit: Cluster pulls its own config, no external credentials needed, self-healing
```

---

## 2. Why ArgoCD?

### Problems Without ArgoCD

- Deployments are done manually with `kubectl apply` — error-prone
- No central visibility into what version is deployed where
- No automated rollback if deployment fails
- No drift detection (someone does `kubectl edit` manually and nobody knows)
- CI pipeline needs cluster admin credentials — security risk

### How ArgoCD Solves These

| Problem | ArgoCD Solution |
|---------|-----------------|
| Manual deployments | Auto-sync from Git |
| No visibility | Beautiful UI + CLI + API |
| No rollback | One-click rollback to any Git commit |
| Drift detection | Continuous reconciliation loop |
| Credential exposure | ArgoCD runs inside the cluster |
| Multi-cluster management | Single ArgoCD instance manages N clusters |
| Multi-env support | App-of-Apps pattern, ApplicationSets |

### ArgoCD vs Flux (Common Interview Question)

| Feature | ArgoCD | Flux v2 |
|---------|--------|---------|
| UI | ✅ Rich web UI | ❌ No built-in UI |
| Multi-tenancy | ✅ Projects + RBAC | ✅ |
| Helm support | ✅ Native | ✅ |
| Kustomize | ✅ Native | ✅ |
| Multi-cluster | ✅ | ✅ |
| Notifications | ✅ (plugin) | ✅ |
| Community | Large, CNCF graduated | Large, CNCF graduated |

---

## 3. ArgoCD Architecture

### High-Level Architecture Diagram

```
┌────────────────────────────────────────────────────────────────┐
│                        Git Repository                          │
│   ┌──────────────┐   ┌──────────────┐   ┌──────────────┐      │
│   │  app-code/   │   │  k8s-manifests│  │  helm-charts/│      │
│   └──────────────┘   └──────────────┘   └──────────────┘      │
└────────────────┬───────────────────────────────────────────────┘
                 │  (ArgoCD polls / webhook trigger)
                 ▼
┌────────────────────────────────────────────────────────────────┐
│                    ArgoCD (runs in EKS)                        │
│                                                                │
│  ┌──────────────┐  ┌─────────────┐  ┌────────────────────┐   │
│  │  API Server  │  │  Repo Server│  │  Application       │   │
│  │  (gRPC/REST) │  │  (clones &  │  │  Controller        │   │
│  │              │  │   renders   │  │  (reconcile loop)  │   │
│  └──────┬───────┘  └─────────────┘  └────────────────────┘   │
│         │                                                      │
│  ┌──────┴───────┐  ┌─────────────┐  ┌────────────────────┐   │
│  │  Web UI      │  │  Redis      │  │  Dex (OIDC)        │   │
│  │  (Dashboard) │  │  (cache)    │  │  (SSO/Auth)        │   │
│  └──────────────┘  └─────────────┘  └────────────────────┘   │
└────────────────────────────────────────────────────────────────┘
                 │  (applies manifests)
                 ▼
┌────────────────────────────────────────────────────────────────┐
│                  Target Kubernetes Cluster (EKS)               │
│   ┌──────────────┐   ┌──────────────┐   ┌──────────────┐      │
│   │  Namespace:  │   │  Namespace:  │   │  Namespace:  │      │
│   │    dev       │   │   staging    │   │   prod       │      │
│   └──────────────┘   └──────────────┘   └──────────────┘      │
└────────────────────────────────────────────────────────────────┘
```

### ArgoCD Components Explained

#### 1. API Server
- Exposes gRPC and REST APIs
- Used by Web UI, CLI (`argocd` CLI), and external systems
- Handles authentication, RBAC, app management operations
- Listens on port `443` (HTTPS)

#### 2. Repository Server
- Clones Git repositories
- Renders Kubernetes manifests from Helm charts, Kustomize, raw YAML, Jsonnet
- Caches rendered manifests in Redis
- Stateless — can be scaled horizontally

#### 3. Application Controller
- The **heart of ArgoCD** — runs the reconciliation loop
- Compares **desired state** (Git) vs **live state** (Kubernetes API)
- Applies changes when drift is detected (if auto-sync is on)
- Runs as a Kubernetes StatefulSet (one shard per N applications)

#### 4. Redis
- Cache layer for rendered manifests and cluster state
- Reduces load on the Kubernetes API server and Git

#### 5. Dex
- Optional OpenID Connect (OIDC) provider for SSO
- Integrates with GitHub, Google, LDAP, SAML, etc.

#### 6. ApplicationSet Controller
- Generates multiple ArgoCD Applications from a template
- Used for multi-cluster, multi-environment deployments

---

## 4. Core ArgoCD Concepts

### 4.1 Application

An **Application** is the core custom resource of ArgoCD. It defines:
- **Source**: where to get the config (Git repo, path, revision)
- **Destination**: which cluster and namespace to deploy to
- **Sync Policy**: manual or automatic

```yaml
# application.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/your-org/your-repo.git
    targetRevision: HEAD       # Branch, tag, or commit SHA
    path: k8s/overlays/prod    # Path inside the repo
  destination:
    server: https://kubernetes.default.svc   # Same cluster
    namespace: production
  syncPolicy:
    automated:
      prune: true              # Delete resources removed from Git
      selfHeal: true           # Auto-fix drift
    syncOptions:
      - CreateNamespace=true   # Create namespace if it doesn't exist
```

### 4.2 Sync Status vs Health Status

These are two separate concepts:

| Concept | Values | Meaning |
|---------|--------|---------|
| **Sync Status** | `Synced` / `OutOfSync` | Does live state match Git? |
| **Health Status** | `Healthy` / `Progressing` / `Degraded` / `Missing` | Is the app running correctly? |

You can be `Synced` but `Degraded` (deployed but crashlooping), or `OutOfSync` but `Healthy` (someone patched live state manually).

### 4.3 Sync Phases & Waves

ArgoCD supports **ordering** of resource deployment using sync waves:

```yaml
metadata:
  annotations:
    argocd.argoproj.io/sync-wave: "0"   # Deploy first (e.g., CRDs, Namespaces)
---
metadata:
  annotations:
    argocd.argoproj.io/sync-wave: "1"   # Deploy second (e.g., Deployments)
---
metadata:
  annotations:
    argocd.argoproj.io/sync-wave: "2"   # Deploy last (e.g., Ingress)
```

Default wave is `0`. Resources in lower waves must be healthy before higher waves start.

### 4.4 Projects (AppProject)

Projects are a way to **group applications** and enforce policies:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: team-payments
  namespace: argocd
spec:
  description: Payments team applications
  sourceRepos:
    - https://github.com/org/payments-repo.git
  destinations:
    - namespace: payments-*      # Only deploy to payments namespaces
      server: https://kubernetes.default.svc
  clusterResourceWhitelist:
    - group: ''
      kind: Namespace
  namespaceResourceBlacklist:
    - group: ''
      kind: ResourceQuota        # Block creating ResourceQuotas
  roles:
    - name: developer
      policies:
        - p, proj:team-payments:developer, applications, sync, team-payments/*, allow
        - p, proj:team-payments:developer, applications, get, team-payments/*, allow
```

### 4.5 ApplicationSet

Generates multiple Applications from one template — ideal for multi-cluster/multi-env:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: guestbook
  namespace: argocd
spec:
  generators:
    - list:
        elements:
          - cluster: dev
            url: https://dev.eks.example.com
            env: dev
          - cluster: staging
            url: https://staging.eks.example.com
            env: staging
          - cluster: prod
            url: https://prod.eks.example.com
            env: prod
  template:
    metadata:
      name: 'guestbook-{{cluster}}'
    spec:
      project: default
      source:
        repoURL: https://github.com/org/guestbook.git
        targetRevision: HEAD
        path: 'overlays/{{env}}'
      destination:
        server: '{{url}}'
        namespace: guestbook
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
```

### 4.6 App-of-Apps Pattern

A parent Application that manages other Applications — used for bootstrapping:

```
argocd-bootstrap (App)
    ├── monitoring-app (Application CR)
    │       └── deploys: prometheus, grafana
    ├── ingress-app (Application CR)
    │       └── deploys: nginx-ingress
    └── my-service-app (Application CR)
            └── deploys: your microservice
```

```yaml
# parent app points to a folder of Application manifests
spec:
  source:
    path: apps/            # This folder contains Application YAMLs
    repoURL: https://github.com/org/gitops-repo.git
```

### 4.7 Resource Hooks

Hooks let you run Jobs before/after sync:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: db-migration
  annotations:
    argocd.argoproj.io/hook: PreSync           # Runs before sync
    argocd.argoproj.io/hook-delete-policy: HookSucceeded
spec:
  template:
    spec:
      containers:
        - name: migrate
          image: my-app:latest
          command: ["./migrate.sh"]
```

Hook types: `PreSync`, `Sync`, `PostSync`, `SyncFail`

---

## 5. ArgoCD Installation on EKS

### Prerequisites

```bash
# Install tools
brew install kubectl helm argocd awscli

# Configure AWS
aws configure

# Create EKS cluster (or use existing)
eksctl create cluster \
  --name my-cluster \
  --region ap-south-1 \
  --nodegroup-name standard-workers \
  --node-type t3.medium \
  --nodes 2 \
  --managed

# Update kubeconfig
aws eks update-kubeconfig --region ap-south-1 --name my-cluster
```

### Install ArgoCD

```bash
# Create namespace
kubectl create namespace argocd

# Install ArgoCD (stable)
kubectl apply -n argocd -f \
  https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Wait for pods to be ready
kubectl wait --for=condition=Ready pods --all -n argocd --timeout=300s

# Check pods
kubectl get pods -n argocd
# NAME                                                READY   STATUS
# argocd-application-controller-0                    1/1     Running
# argocd-applicationset-controller-xxx               1/1     Running
# argocd-dex-server-xxx                              1/1     Running
# argocd-notifications-controller-xxx                1/1     Running
# argocd-redis-xxx                                   1/1     Running
# argocd-repo-server-xxx                             1/1     Running
# argocd-server-xxx                                  1/1     Running
```

### Access ArgoCD UI

```bash
# Option 1: Port-forward (local dev)
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Option 2: LoadBalancer (EKS)
kubectl patch svc argocd-server -n argocd \
  -p '{"spec": {"type": "LoadBalancer"}}'

# Get external URL
kubectl get svc argocd-server -n argocd

# Get initial admin password
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d && echo
```

### Login via CLI

```bash
argocd login localhost:8080 \
  --username admin \
  --password <password> \
  --insecure

# Change password
argocd account update-password
```

### Connect Your Repo (Private Repos)

```bash
# SSH key
argocd repo add git@github.com:org/repo.git \
  --ssh-private-key-path ~/.ssh/id_rsa

# HTTPS with token
argocd repo add https://github.com/org/repo.git \
  --username git \
  --password <GITHUB_TOKEN>
```

---

## 6. Full GitOps Pipeline with GitHub Actions

### Repository Structure

```
├── app-repo/                        # Application code repo
│   ├── src/
│   ├── Dockerfile
│   └── .github/
│       └── workflows/
│           └── ci-cd.yaml           # CI: build & push image, update gitops repo
│
└── gitops-repo/                     # GitOps manifests repo (separate!)
    ├── apps/                        # App-of-Apps parent Applications
    ├── base/                        # Kustomize base
    │   └── my-service/
    │       ├── deployment.yaml
    │       ├── service.yaml
    │       └── kustomization.yaml
    └── overlays/                    # Environment-specific patches
        ├── dev/
        │   ├── kustomization.yaml
        │   └── patch-replicas.yaml
        ├── staging/
        │   └── kustomization.yaml
        └── prod/
            ├── kustomization.yaml
            └── patch-hpa.yaml
```

### Step 1: Dockerfile

```dockerfile
# Dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY . .
EXPOSE 3000
USER node
CMD ["node", "server.js"]
```

### Step 2: Kubernetes Base Manifests (Kustomize)

```yaml
# base/my-service/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-service
  labels:
    app: my-service
spec:
  replicas: 2
  selector:
    matchLabels:
      app: my-service
  template:
    metadata:
      labels:
        app: my-service
    spec:
      containers:
        - name: my-service
          image: ghcr.io/your-org/my-service:latest    # Will be patched by CI
          ports:
            - containerPort: 3000
          resources:
            requests:
              memory: "64Mi"
              cpu: "100m"
            limits:
              memory: "128Mi"
              cpu: "500m"
          readinessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 5
            periodSeconds: 10
          livenessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 15
            periodSeconds: 20
```

```yaml
# base/my-service/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - deployment.yaml
  - service.yaml
```

```yaml
# overlays/prod/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namespace: production

resources:
  - ../../base/my-service

patches:
  - path: patch-replicas.yaml

images:
  - name: ghcr.io/your-org/my-service
    newTag: "abc123"    # Updated by CI pipeline
```

### Step 3: GitHub Actions CI/CD Workflow

```yaml
# .github/workflows/ci-cd.yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  # ─── JOB 1: Test ───────────────────────────────────────────────────────────
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test

      - name: Run linting
        run: npm run lint

  # ─── JOB 2: Build & Push Docker Image ─────────────────────────────────────
  build-and-push:
    runs-on: ubuntu-latest
    needs: test
    if: github.ref == 'refs/heads/main'
    permissions:
      contents: read
      packages: write
    outputs:
      image-tag: ${{ steps.meta.outputs.version }}
      image-digest: ${{ steps.build.outputs.digest }}

    steps:
      - uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to GitHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract Docker metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=sha,prefix=,suffix=,format=short    # e.g., abc1234
            type=ref,event=branch
            type=semver,pattern={{version}}

      - name: Build and push Docker image
        id: build
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

      - name: Run Trivy vulnerability scan
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ steps.meta.outputs.version }}
          format: 'sarif'
          output: 'trivy-results.sarif'

  # ─── JOB 3: Update GitOps Repo ─────────────────────────────────────────────
  update-gitops:
    runs-on: ubuntu-latest
    needs: build-and-push
    if: github.ref == 'refs/heads/main'

    steps:
      - name: Checkout GitOps repo
        uses: actions/checkout@v4
        with:
          repository: your-org/gitops-repo       # Separate GitOps repo
          token: ${{ secrets.GITOPS_PAT }}        # PAT with repo write access
          path: gitops-repo

      - name: Install Kustomize
        run: |
          curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh" | bash
          sudo mv kustomize /usr/local/bin/

      - name: Update image tag in staging
        working-directory: gitops-repo
        run: |
          cd overlays/staging
          kustomize edit set image \
            ghcr.io/your-org/my-service=ghcr.io/your-org/my-service:${{ needs.build-and-push.outputs.image-tag }}

      - name: Commit and push to GitOps repo
        working-directory: gitops-repo
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          git add .
          git diff --staged --quiet || git commit -m "chore: update my-service image to ${{ needs.build-and-push.outputs.image-tag }}

          Source: ${{ github.repository }}@${{ github.sha }}
          Workflow: ${{ github.run_id }}"
          git push

  # ─── JOB 4: Promote to Production (Manual Approval) ───────────────────────
  promote-to-prod:
    runs-on: ubuntu-latest
    needs: update-gitops
    environment: production     # Requires manual approval in GitHub Environments

    steps:
      - name: Checkout GitOps repo
        uses: actions/checkout@v4
        with:
          repository: your-org/gitops-repo
          token: ${{ secrets.GITOPS_PAT }}

      - name: Install Kustomize
        run: |
          curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh" | bash
          sudo mv kustomize /usr/local/bin/

      - name: Promote image to production
        run: |
          cd overlays/prod
          kustomize edit set image \
            ghcr.io/your-org/my-service=ghcr.io/your-org/my-service:${{ needs.build-and-push.outputs.image-tag }}

      - name: Commit and push production update
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          git add .
          git commit -m "chore(prod): promote my-service to ${{ needs.build-and-push.outputs.image-tag }}"
          git push
```

### Step 4: ArgoCD Applications for Each Environment

```yaml
# gitops-repo/apps/my-service-staging.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-service-staging
  namespace: argocd
  finalizers:
    - resources-finalizer.argocd.argoproj.io    # Cascade delete on app removal
spec:
  project: default
  source:
    repoURL: https://github.com/your-org/gitops-repo.git
    targetRevision: HEAD
    path: overlays/staging
  destination:
    server: https://kubernetes.default.svc
    namespace: staging
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
      - PrunePropagationPolicy=foreground
      - PruneLast=true
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m
```

```yaml
# gitops-repo/apps/my-service-prod.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-service-prod
  namespace: argocd
spec:
  project: production
  source:
    repoURL: https://github.com/your-org/gitops-repo.git
    targetRevision: HEAD
    path: overlays/prod
  destination:
    server: https://prod-cluster.eks.example.com
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true    # Be careful with selfHeal in prod
    syncOptions:
      - CreateNamespace=true
```

### Complete Flow Diagram

```
Developer pushes code
        │
        ▼
GitHub Actions: CI
 ├── Run tests
 ├── Build Docker image
 ├── Push to GHCR (sha tag)
 └── Scan with Trivy
        │
        ▼
GitHub Actions: Update GitOps Repo
 └── Update image tag in overlays/staging/kustomization.yaml
        │
        ▼
ArgoCD detects change in GitOps repo (poll / webhook)
        │
        ▼
ArgoCD syncs staging namespace
 └── Applies updated Deployment with new image
        │
        ▼
Manual Approval via GitHub Environments
        │
        ▼
GitHub Actions: Promote to Production
 └── Update image tag in overlays/prod/kustomization.yaml
        │
        ▼
ArgoCD syncs production namespace
```

---

## 7. Prometheus & Grafana Monitoring

### Why Prometheus + Grafana with ArgoCD?

- Prometheus scrapes metrics from your app, ArgoCD, and Kubernetes nodes
- Grafana visualizes them in dashboards
- You get full observability: app metrics + deployment health + cluster health

### Installation via Helm

```bash
# Add repos
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

# Install kube-prometheus-stack (includes Prometheus + Grafana + Alertmanager)
kubectl create namespace monitoring

helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --set grafana.adminPassword=admin123 \
  --set prometheus.prometheusSpec.serviceMonitorSelectorNilUsesHelmValues=false \
  --set prometheus.prometheusSpec.podMonitorSelectorNilUsesHelmValues=false
```

### ArgoCD Metrics Exposed

ArgoCD exposes Prometheus metrics natively:

| Service | Port | Metrics |
|---------|------|---------|
| `argocd-metrics` | 8082 | App sync status, health, reconciliation |
| `argocd-server-metrics` | 8083 | API server request latency, errors |
| `argocd-repo-server` | 8084 | Repo clone/render duration |

### ServiceMonitor for ArgoCD

```yaml
# argocd-servicemonitor.yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: argocd-metrics
  namespace: monitoring
  labels:
    release: kube-prometheus-stack    # Must match Prometheus selector
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: argocd-metrics
  namespaceSelector:
    matchNames:
      - argocd
  endpoints:
    - port: metrics
      interval: 30s
      path: /metrics
```

### Your App Metrics (Node.js Example)

```javascript
// metrics.js — expose Prometheus metrics from your app
const client = require('prom-client');
const register = new client.Registry();

// Collect default metrics (CPU, memory, event loop)
client.collectDefaultMetrics({ register });

// Custom counter
const httpRequests = new client.Counter({
  name: 'http_requests_total',
  help: 'Total HTTP requests',
  labelNames: ['method', 'route', 'status_code'],
  registers: [register]
});

// Histogram for latency
const httpDuration = new client.Histogram({
  name: 'http_request_duration_seconds',
  help: 'HTTP request duration in seconds',
  labelNames: ['method', 'route', 'status_code'],
  buckets: [0.001, 0.005, 0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1, 2.5, 5],
  registers: [register]
});

// Expose /metrics endpoint
app.get('/metrics', async (req, res) => {
  res.set('Content-Type', register.contentType);
  res.end(await register.metrics());
});
```

```yaml
# ServiceMonitor for your app
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: my-service-monitor
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app: my-service
  namespaceSelector:
    matchNames:
      - production
  endpoints:
    - port: http
      path: /metrics
      interval: 15s
```

### Key Prometheus Queries (PromQL)

```promql
# ArgoCD — Apps out of sync
argocd_app_info{sync_status="OutOfSync"}

# ArgoCD — Apps unhealthy
argocd_app_info{health_status!="Healthy"}

# ArgoCD — Number of syncs per app (rate)
rate(argocd_app_reconcile_count[5m])

# App — Request rate
rate(http_requests_total[5m])

# App — Error rate (5xx)
rate(http_requests_total{status_code=~"5.."}[5m])

# App — P99 latency
histogram_quantile(0.99, rate(http_request_duration_seconds_bucket[5m]))

# Pod CPU usage
rate(container_cpu_usage_seconds_total{namespace="production"}[5m])

# Pod Memory usage
container_memory_working_set_bytes{namespace="production"}
```

### PrometheusRule (Alerting)

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: argocd-alerts
  namespace: monitoring
spec:
  groups:
    - name: argocd
      rules:
        - alert: ArgoCDAppOutOfSync
          expr: argocd_app_info{sync_status="OutOfSync"} == 1
          for: 5m
          labels:
            severity: warning
          annotations:
            summary: "ArgoCD app {{ $labels.name }} is OutOfSync"
            description: "App {{ $labels.name }} in project {{ $labels.project }} has been out of sync for 5 minutes."

        - alert: ArgoCDAppDegraded
          expr: argocd_app_info{health_status="Degraded"} == 1
          for: 2m
          labels:
            severity: critical
          annotations:
            summary: "ArgoCD app {{ $labels.name }} is Degraded"
```

### Access Grafana

```bash
# Port forward
kubectl port-forward svc/kube-prometheus-stack-grafana -n monitoring 3000:80

# Default credentials: admin / admin123
# Import dashboards:
# - ArgoCD: Dashboard ID 14584
# - Kubernetes cluster: Dashboard ID 7249
# - Node Exporter: Dashboard ID 1860
```

---

## 8. Amazon EKS Deep Dive

### What is EKS?

Amazon Elastic Kubernetes Service (EKS) is a managed Kubernetes control plane. AWS manages the API server, etcd, and controller manager. You only manage worker nodes.

### EKS Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    AWS Account                          │
│                                                         │
│  ┌──────────────────────────────────────────────────┐  │
│  │          EKS Control Plane (Managed by AWS)      │  │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────────┐  │  │
│  │  │ API      │  │  etcd    │  │ Controller   │  │  │
│  │  │ Server   │  │ (cluster │  │ Manager +    │  │  │
│  │  │          │  │  state)  │  │ Scheduler    │  │  │
│  │  └──────────┘  └──────────┘  └──────────────┘  │  │
│  └──────────────────────────────────────────────────┘  │
│                         │                               │
│         ┌───────────────┼───────────────┐              │
│         ▼               ▼               ▼              │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐        │
│  │  Worker    │  │  Worker    │  │  Worker    │        │
│  │  Node AZ-a │  │  Node AZ-b │  │  Node AZ-c │        │
│  │  (EC2)     │  │  (EC2)     │  │  (EC2)     │        │
│  └────────────┘  └────────────┘  └────────────┘        │
│                                                         │
│  ┌────────────────────────────────────────────────┐    │
│  │            VPC                                 │    │
│  │  ┌──────────────┐  ┌──────────────┐           │    │
│  │  │ Public Subnet│  │Private Subnet│           │    │
│  │  │ (Load Balancer│  │ (Worker Nodes│           │    │
│  │  └──────────────┘  └──────────────┘           │    │
│  └────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────┘
```

### Create EKS Cluster with eksctl

```yaml
# cluster-config.yaml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: production-cluster
  region: ap-south-1
  version: "1.29"

vpc:
  cidr: "10.0.0.0/16"
  nat:
    gateway: HighlyAvailable     # NAT gateway per AZ

managedNodeGroups:
  - name: general
    instanceType: t3.medium
    minSize: 2
    maxSize: 10
    desiredCapacity: 3
    volumeSize: 50
    privateNetworking: true        # Nodes in private subnets
    amiFamily: AmazonLinux2
    labels:
      role: general
    iam:
      withAddonPolicies:
        ebs: true                  # EBS CSI driver
        albIngress: true           # AWS Load Balancer Controller
        cloudWatch: true           # CloudWatch logging
        autoScaler: true           # Cluster Autoscaler

addons:
  - name: vpc-cni
  - name: coredns
  - name: kube-proxy
  - name: aws-ebs-csi-driver

cloudWatch:
  clusterLogging:
    enableTypes: ["api", "audit", "authenticator"]
```

```bash
eksctl create cluster -f cluster-config.yaml
```

### IRSA (IAM Roles for Service Accounts)

Avoid storing AWS credentials in pods. Use IRSA instead:

```bash
# Enable OIDC provider
eksctl utils associate-iam-oidc-provider \
  --cluster production-cluster \
  --approve

# Create IAM role for a service account
eksctl create iamserviceaccount \
  --name my-service-sa \
  --namespace production \
  --cluster production-cluster \
  --attach-policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess \
  --approve
```

```yaml
# Use in deployment
spec:
  serviceAccountName: my-service-sa   # Automatically gets AWS credentials via IRSA
```

### Cluster Autoscaler

```yaml
# Automatically scales EC2 nodes based on pod scheduling
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cluster-autoscaler
  namespace: kube-system
spec:
  template:
    spec:
      containers:
        - name: cluster-autoscaler
          image: registry.k8s.io/autoscaling/cluster-autoscaler:v1.29.0
          command:
            - ./cluster-autoscaler
            - --v=4
            - --stderrthreshold=info
            - --cloud-provider=aws
            - --skip-nodes-with-local-storage=false
            - --expander=least-waste
            - --node-group-auto-discovery=asg:tag=k8s.io/cluster-autoscaler/enabled,k8s.io/cluster-autoscaler/production-cluster
```

### HPA (Horizontal Pod Autoscaler)

```yaml
# Scale pods based on CPU/memory
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-service-hpa
  namespace: production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-service
  minReplicas: 2
  maxReplicas: 20
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80
```

---

## 9. End-to-End GitOps Architecture

### Complete Picture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         Developer Workflow                              │
│                                                                         │
│   git push  →  Pull Request  →  Code Review  →  Merge to main          │
└─────────────────────────────┬───────────────────────────────────────────┘
                              │  triggers
                              ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                      GitHub Actions (CI/CD)                             │
│                                                                         │
│  1. Test  →  2. Build Image  →  3. Push to GHCR  →  4. Trivy Scan     │
│  5. Update GitOps Repo (image tag)  →  6. Manual approval (prod)       │
└─────────────────────────────┬───────────────────────────────────────────┘
                              │  commits to
                              ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                         GitOps Repository                               │
│                                                                         │
│   overlays/dev/      overlays/staging/      overlays/prod/              │
│   (auto-deploy)      (auto-deploy)          (manual approval)           │
└─────────────────────────────┬───────────────────────────────────────────┘
                              │  ArgoCD polls every 3 min
                              ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                      ArgoCD (EKS cluster)                               │
│                                                                         │
│   Detects drift  →  Renders manifests  →  Applies to cluster           │
│   Self-heals if someone manually edits live state                       │
└─────────────────────────────┬───────────────────────────────────────────┘
                              │  deploys to
                              ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                      Amazon EKS Cluster                                 │
│                                                                         │
│   ┌──────────┐   ┌──────────┐   ┌─────────────────────────────────┐   │
│   │  Pods    │   │  Services│   │  Prometheus scrapes metrics      │   │
│   │  (App)   │   │  (LB)    │   │  Grafana shows dashboards       │   │
│   └──────────┘   └──────────┘   │  Alertmanager sends alerts      │   │
│                                  └─────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────┘
```

### Security Best Practices

| Area | Practice |
|------|----------|
| **Secrets** | Use AWS Secrets Manager + External Secrets Operator (never store secrets in Git) |
| **RBAC** | Use ArgoCD Projects to limit which teams can deploy where |
| **Image** | Always use image digest (`sha256:...`) in production, not tags |
| **Network** | Use Network Policies to restrict pod-to-pod traffic |
| **Scanning** | Trivy in CI, plus admission controller (Kyverno/OPA Gatekeeper) |
| **IRSA** | Use IRSA instead of EC2 instance profiles or hardcoded keys |

### External Secrets Operator (No Secrets in Git)

```yaml
# Sync from AWS Secrets Manager to Kubernetes Secret
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: my-service-secrets
  namespace: production
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: aws-secrets-manager
    kind: ClusterSecretStore
  target:
    name: my-service-secret    # Creates this Kubernetes Secret
  data:
    - secretKey: DB_PASSWORD
      remoteRef:
        key: production/my-service/database
        property: password
    - secretKey: API_KEY
      remoteRef:
        key: production/my-service/api
        property: key
```

---

## 10. Interview Q&A

### ArgoCD Core Concepts

**Q: What is the difference between Sync Status and Health Status in ArgoCD?**

> **Sync Status** compares desired state (Git) with live state (cluster). Values: `Synced`, `OutOfSync`. **Health Status** checks if resources are actually running correctly. Values: `Healthy`, `Progressing`, `Degraded`, `Missing`, `Suspended`. A deployment can be `Synced` but `Degraded` if pods are crashlooping after a sync.

**Q: What happens when selfHeal is enabled in ArgoCD?**

> If someone manually changes a resource in the cluster (e.g., `kubectl edit deployment`), ArgoCD detects the drift and automatically reverts it to match the Git state. This is the "self-healing" capability that enforces Git as the single source of truth.

**Q: What is the App-of-Apps pattern?**

> A parent ArgoCD Application that points to a directory of other Application manifests. When ArgoCD syncs the parent, it creates all child Applications, which then sync their own resources. This is used to bootstrap entire clusters or manage many applications declaratively.

**Q: What is the difference between ArgoCD and traditional CI/CD?**

> Traditional CI/CD uses a **push model** — the pipeline runs `kubectl apply` which requires cluster credentials in the CI system. ArgoCD uses a **pull model** — it runs inside the cluster and continuously pulls from Git. This is more secure (no external credential storage), provides self-healing, and gives better auditability through Git history.

**Q: How does ArgoCD handle multi-cluster deployments?**

> ArgoCD can manage multiple clusters from a single control plane. Each cluster is registered using `argocd cluster add`, and Applications can specify `destination.server` to point to any registered cluster. ApplicationSets make this powerful by generating Applications for all clusters from a template.

**Q: What is an ApplicationSet and when would you use it?**

> An ApplicationSet generates multiple ArgoCD Applications from a single template using generators (List, Cluster, Git, Matrix). You'd use it when deploying the same application across multiple environments or clusters to avoid copy-pasting Application manifests.

### GitOps & Pipeline

**Q: Why should the application code repo and GitOps repo be separate?**

> Separation of concerns: the app repo focuses on development workflows, while the GitOps repo is the operational source of truth. Developers can deploy without understanding cluster internals. The CI pipeline commits to the GitOps repo rather than running kubectl directly, maintaining the pull-based model.

**Q: How do you promote from staging to production in GitOps?**

> The CI pipeline updates the image tag in the staging overlay and commits. After staging validation, a separate job (gated by manual approval using GitHub Environments) updates the production overlay. ArgoCD detects the change and deploys. The entire promotion history is in Git.

**Q: How do you handle secrets in GitOps without storing them in Git?**

> Use **External Secrets Operator** with AWS Secrets Manager, HashiCorp Vault, or GCP Secret Manager. The ESO CRD defines which external secret to fetch, and ESO creates the Kubernetes Secret automatically. Only the reference (not the value) lives in Git.

### Kubernetes & EKS

**Q: What is IRSA and why is it better than instance profiles?**

> IRSA (IAM Roles for Service Accounts) allows individual pods to assume specific IAM roles. Unlike instance profiles (which give all pods on a node the same permissions), IRSA follows the principle of least privilege — each service account gets only the permissions it needs.

**Q: Explain the difference between HPA, VPA, and Cluster Autoscaler.**

> **HPA** (Horizontal Pod Autoscaler) adds/removes pod replicas based on metrics. **VPA** (Vertical Pod Autoscaler) adjusts CPU/memory requests of existing pods. **Cluster Autoscaler** adds/removes EC2 nodes when pods can't be scheduled or nodes are underutilized. In practice: HPA handles load, Cluster Autoscaler handles capacity.

**Q: What is the reconciliation loop in ArgoCD?**

> The Application Controller runs a continuous loop that compares the desired state (rendered from Git manifests) with the live state (queried from the Kubernetes API). If they differ (OutOfSync), and auto-sync is enabled, it applies the desired state. This loop runs every 3 minutes by default and is also triggered by Git webhook events.

### Monitoring

**Q: What ArgoCD metrics would you alert on in production?**

> Key alerts: `argocd_app_info{sync_status="OutOfSync"}` (app drifted from Git), `argocd_app_info{health_status="Degraded"}` (app unhealthy after sync), `argocd_app_reconcile_duration_seconds` (reconciliation taking too long — indicates performance issue), and `argocd_cluster_api_request_duration_seconds` (cluster connectivity issues).

**Q: What is the difference between a ServiceMonitor and a PodMonitor in Prometheus?**

> A **ServiceMonitor** scrapes metrics from a Kubernetes Service's endpoints — it selects pods via a Service. A **PodMonitor** scrapes pods directly without needing a Service. ServiceMonitor is more common as it respects service discovery and load balancing.

---

## Quick Reference Cheat Sheet

```bash
# ArgoCD CLI common commands
argocd app list                              # List all apps
argocd app get my-app                       # Get app details
argocd app sync my-app                      # Manually sync
argocd app sync my-app --dry-run            # Preview what would change
argocd app rollback my-app 3               # Rollback to history revision 3
argocd app history my-app                  # View sync history
argocd app diff my-app                     # Diff Git vs live state
argocd app delete my-app --cascade         # Delete app + all resources

# Useful kubectl commands for GitOps debugging
kubectl get applications -n argocd          # List ArgoCD apps
kubectl describe application my-app -n argocd
kubectl get events -n production --sort-by='.lastTimestamp'
kubectl rollout status deployment/my-service -n production
kubectl rollout undo deployment/my-service -n production   # Emergency rollback
```

---

*Notes maintained at: `github.com/your-username/devops-notes`*  
*Last updated: 2026 | Stack: ArgoCD · GitHub Actions · Prometheus · Grafana · EKS*
