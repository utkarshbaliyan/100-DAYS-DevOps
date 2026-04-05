# ☸️ Kubernetes Complete Study Notes

> **Purpose:** Comprehensive Kubernetes notes for interview preparation and job applications.  
> **Level:** Beginner → Intermediate → Advanced concepts  
> **Covers:** Architecture, Workloads, Networking, Storage, Scaling, Helm & more

---

## 📋 Table of Contents

1. [CNCF & Why Kubernetes?](#1-cncf--why-kubernetes)
2. [Kubernetes Architecture](#2-kubernetes-architecture)
3. [Control Plane Components](#3-control-plane-components)
4. [Worker Node Components](#4-worker-node-components)
5. [kubectl & CNI Network](#5-kubectl--cni-network)
6. [Creating a Kubernetes Cluster](#6-creating-a-kubernetes-cluster)
7. [Manifest Files & Core Objects](#7-manifest-files--core-objects)
8. [Pods](#8-pods)
9. [Namespaces](#9-namespaces)
10. [Deployments & Replica Sets](#10-deployments--replica-sets)
11. [Services](#11-services)
12. [Labels & Selectors](#12-labels--selectors)
13. [Flow of Kubernetes (End-to-End)](#13-flow-of-kubernetes-end-to-end)
14. [Ingress & Gateway API](#14-ingress--gateway-api)
15. [ConfigMaps & Secrets](#15-configmaps--secrets)
16. [Persistent Volumes & Claims](#16-persistent-volumes--claims)
17. [Deployment Strategies](#17-deployment-strategies)
18. [Health Checks & Probes](#18-health-checks--probes)
19. [Container Orchestration](#19-container-orchestration)
20. [Resource Requests & Limits](#20-resource-requests--limits)
21. [Auto Scaling](#21-auto-scaling)
22. [Metrics Server](#22-metrics-server)
23. [Helm](#23-helm)
24. [Kubernetes Dashboard](#24-kubernetes-dashboard)
25. [Quick Reference & Cheat Sheet](#25-quick-reference--cheat-sheet)

---

## 1. CNCF & Why Kubernetes?

### What is CNCF?

**CNCF** stands for **Cloud Native Computing Foundation**. It is a vendor-neutral open-source foundation (part of the Linux Foundation) that hosts and promotes cloud-native projects.

- Founded in **2015**
- Kubernetes was the **first project** donated to CNCF (by Google)
- Other CNCF projects include: Prometheus, Envoy, Helm, Argo, Istio (sandbox), Fluentd, Containerd, etc.
- CNCF maintains the **Cloud Native Landscape** — a map of all cloud-native tools

> **Cloud Native** means building and running applications that are scalable, resilient, manageable, and observable in modern cloud environments.

---

### Why Kubernetes?

Before Kubernetes, teams used **Docker** to run containers. But as applications grew, managing containers manually became painful:

| Problem (without K8s) | Solution (with K8s) |
|---|---|
| Manually restart crashed containers | Self-healing — auto-restarts failed containers |
| Hard to scale containers up/down | Auto-scaling based on load |
| No built-in load balancing | Built-in service discovery & load balancing |
| Manual deployment across servers | Automated rollouts & rollbacks |
| No resource management | CPU/memory request & limit enforcement |
| Secrets management is difficult | Native Secrets & ConfigMaps |
| Complex networking between containers | CNI-based networking |

**In short:** Kubernetes automates the deployment, scaling, and management of containerized applications.

---

## 2. Kubernetes Architecture

Kubernetes follows a **Master-Worker** architecture.

```
┌─────────────────────────────────────────────────────────────────┐
│                        KUBERNETES CLUSTER                        │
│                                                                   │
│  ┌─────────────────────────┐    ┌──────────────────────────────┐ │
│  │     CONTROL PLANE        │    │        WORKER NODES          │ │
│  │  (Master Node)           │    │                              │ │
│  │                          │    │  ┌──────────┐ ┌──────────┐  │ │
│  │  ┌──────────────────┐    │    │  │  Node 1  │ │  Node 2  │  │ │
│  │  │    API Server     │◄───┼────┼──│ kubelet  │ │ kubelet  │  │ │
│  │  └──────────────────┘    │    │  │ kube-    │ │ kube-    │  │ │
│  │  ┌──────────────────┐    │    │  │ proxy    │ │ proxy    │  │ │
│  │  │  Controller Mgr   │    │    │  │ Pods     │ │ Pods     │  │ │
│  │  └──────────────────┘    │    │  └──────────┘ └──────────┘  │ │
│  │  ┌──────────────────┐    │    └──────────────────────────────┘ │
│  │  │    Scheduler      │    │                                     │
│  │  └──────────────────┘    │                                     │
│  │  ┌──────────────────┐    │                                     │
│  │  │      ETCD         │    │                                     │
│  │  └──────────────────┘    │                                     │
│  └─────────────────────────┘                                     │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3. Control Plane Components

The **Control Plane** is the brain of Kubernetes. It manages the cluster state and makes global decisions.

---

### 🔷 API Server (`kube-apiserver`)

- **The central hub** of Kubernetes — all communication goes through it
- Exposes the Kubernetes REST API
- Every component (kubectl, kubelet, controller manager, scheduler) talks to the API Server
- Validates and processes API requests
- Stores the result in **etcd**

```
kubectl apply -f pod.yaml
       ↓
  API Server  →  validates →  stores in etcd  →  notifies scheduler
```

---

### 🔷 ETCD

- A **distributed key-value store** used as Kubernetes' backing database
- Stores ALL cluster state: nodes, pods, configs, secrets, service accounts, etc.
- **Only the API Server** directly reads/writes to etcd
- Highly available — uses Raft consensus algorithm
- Critical component — losing etcd = losing your cluster state

> **Interview tip:** etcd is the single source of truth in a Kubernetes cluster.

---

### 🔷 Scheduler (`kube-scheduler`)

- Watches for **newly created Pods** that have no assigned node
- Decides **which node** to place a Pod on based on:
  - Resource availability (CPU, memory)
  - Node affinity / anti-affinity rules
  - Taints and tolerations
  - Pod topology spread constraints
  - Resource requests vs node capacity
- Does NOT actually run the Pod — it just assigns it to a node

---

### 🔷 Controller Manager (`kube-controller-manager`)

- Runs multiple **controllers** in a single binary
- A controller is a control loop that watches the current state and tries to move it to the desired state

| Controller | Responsibility |
|---|---|
| **Node Controller** | Monitors node health, handles node failures |
| **Replication Controller** | Ensures correct number of pod replicas |
| **Endpoints Controller** | Populates the Endpoints object (links Services to Pods) |
| **Service Account Controller** | Creates default service accounts for namespaces |
| **Job Controller** | Manages one-off Jobs |
| **Deployment Controller** | Manages Deployment rollouts |

> Think of controllers as "reconciliation loops" — they constantly compare desired state vs actual state.

---

## 4. Worker Node Components

Worker Nodes are where your actual application containers run.

---

### 🔶 kubelet

- An **agent** running on every worker node
- Communicates with the API Server
- Ensures containers described in PodSpecs are running and healthy
- Does NOT manage containers not created by Kubernetes
- Reports node and pod status back to the API Server

```
API Server → sends PodSpec → kubelet → tells container runtime → container starts
```

---

### 🔶 Pods

- The **smallest deployable unit** in Kubernetes
- A pod wraps one or more containers that share:
  - Network namespace (same IP address)
  - Storage volumes
  - IPC namespace
- Containers in a pod communicate via `localhost`
- Pods are ephemeral — they can die and be replaced

---

### 🔶 kube-proxy (Service Proxy)

- Runs on every node
- Maintains **network rules** (iptables or IPVS) to allow communication to Services
- Handles routing of traffic to the correct pod backend
- Implements the Kubernetes **Service** concept at the network level

---

### 🔶 Container Runtime

- The software that actually **runs the containers**
- Kubernetes uses the **CRI (Container Runtime Interface)** to talk to runtimes
- Examples: **containerd** (most common), **CRI-O**, Docker (deprecated in K8s 1.24+)

---

## 5. kubectl & CNI Network

### kubectl

- The **command-line tool** for interacting with Kubernetes
- Communicates with the Kubernetes API Server
- Uses a config file: `~/.kube/config` (called **kubeconfig**)

```bash
# Common kubectl commands
kubectl get pods                          # List pods in default namespace
kubectl get pods -n <namespace>           # List pods in a specific namespace
kubectl get pods -A                       # List pods in ALL namespaces
kubectl describe pod <pod-name>           # Detailed info about a pod
kubectl logs <pod-name>                   # View pod logs
kubectl logs <pod-name> -c <container>    # Logs from a specific container
kubectl exec -it <pod-name> -- /bin/sh    # Shell into a pod
kubectl apply -f <file.yaml>              # Apply a manifest file
kubectl delete -f <file.yaml>             # Delete resources from manifest
kubectl get nodes                         # List all nodes
kubectl get all                           # Get all resources
kubectl get events                        # View cluster events (great for debugging)
kubectl top pods                          # Resource usage (needs metrics-server)
kubectl top nodes
kubectl rollout status deployment/<name>  # Check rollout status
kubectl rollout undo deployment/<name>    # Rollback a deployment
kubectl scale deployment/<name> --replicas=3
kubectl port-forward pod/<name> 8080:80   # Forward local port to pod
```

---

### CNI Network (Container Network Interface)

- **CNI** is a standard interface/specification for container networking
- Kubernetes uses CNI plugins to provide networking between pods across nodes
- Without a CNI plugin, pods cannot communicate across nodes

**Popular CNI Plugins:**

| Plugin | Key Features |
|---|---|
| **Calico** | Network policy enforcement, BGP routing, very popular |
| **Flannel** | Simple overlay network, easy to set up |
| **Weave Net** | Encrypted networking, auto-discovery |
| **Cilium** | eBPF-based, high performance, strong observability |
| **AWS VPC CNI** | Used in EKS, assigns VPC IPs to pods |

**Key networking rules in Kubernetes:**
1. Every Pod gets its own unique IP address
2. All pods can communicate with each other without NAT
3. Nodes can communicate with all pods without NAT

---

## 6. Creating a Kubernetes Cluster

### Local Development Tools

#### `minikube`
- Runs a single-node Kubernetes cluster locally (inside a VM or Docker)
- Best for **learning and local development**

```bash
minikube start
minikube start --driver=docker
minikube status
minikube stop
minikube delete
minikube dashboard         # Opens browser dashboard
minikube addons enable ingress
```

---

#### `kind` (Kubernetes IN Docker)
- Runs Kubernetes clusters inside **Docker containers**
- Great for **CI pipelines** and local multi-node testing
- Each "node" is a Docker container

```bash
kind create cluster
kind create cluster --name my-cluster
kind get clusters
kind delete cluster --name my-cluster

# Port forwarding needed for NodePort services with kind
kubectl port-forward svc/<service-name> 8080:80
```

---

#### `kubeadm`
- **Official tool** to bootstrap a production-grade Kubernetes cluster on bare metal or VMs
- Used for on-premise clusters

```bash
# On master node
kubeadm init --pod-network-cidr=10.244.0.0/16

# Then apply CNI plugin (e.g., Flannel)
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

# On worker nodes
kubeadm join <master-ip>:6443 --token <token> --discovery-token-ca-cert-hash <hash>
```

---

#### `vind` (Virtual Nodes in Docker)
- Similar to kind but focuses on virtual node simulation
- Less commonly used; useful for simulating large clusters

---

### Managed Cloud Kubernetes Services

These are **production-grade**, fully managed Kubernetes services — you don't manage the control plane.

| Service | Cloud Provider | Notes |
|---|---|---|
| **EKS** (Elastic Kubernetes Service) | AWS | Most popular in enterprises |
| **AKS** (Azure Kubernetes Service) | Microsoft Azure | Deep Azure integration |
| **GKE** (Google Kubernetes Engine) | GCP | Most mature, Google invented K8s |

**Advantages of managed services:**
- Control plane is managed for you (upgrades, HA, backups)
- Integration with cloud services (IAM, load balancers, storage)
- Auto-scaling of node pools
- Pay only for worker nodes

---

## 7. Manifest Files & Core Objects

### What is a Manifest File?

A **manifest file** is a YAML (or JSON) file that describes the desired state of a Kubernetes resource.

```yaml
apiVersion: apps/v1       # API group and version
kind: Deployment          # Type of resource
metadata:                 # Resource metadata
  name: my-app
  namespace: default
  labels:
    app: my-app
spec:                     # Desired state specification
  replicas: 3
  ...
```

**Key fields in every manifest:**
- `apiVersion` — which API version to use (e.g., `v1`, `apps/v1`, `networking.k8s.io/v1`)
- `kind` — type of resource (`Pod`, `Deployment`, `Service`, etc.)
- `metadata` — name, namespace, labels, annotations
- `spec` — the desired configuration of the resource

---

## 8. Pods

### What is a Pod?

A **Pod** is the smallest deployable unit in Kubernetes. It represents one or more containers running together on a node.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
    app: my-app
spec:
  containers:
    - name: my-container
      image: nginx:1.21
      ports:
        - containerPort: 80
      resources:
        requests:
          memory: "64Mi"
          cpu: "250m"
        limits:
          memory: "128Mi"
          cpu: "500m"
```

**Key Pod characteristics:**
- Each pod gets a unique IP address within the cluster
- Containers in a pod share the same network namespace (communicate via localhost)
- Pods are **ephemeral** — when deleted, they are gone permanently
- You rarely create pods directly — you use Deployments instead

**Multi-container Pod patterns:**

| Pattern | Description |
|---|---|
| **Sidecar** | Helper container alongside main (e.g., log shipper) |
| **Ambassador** | Proxy for external services |
| **Adapter** | Transforms output of main container |
| **Init Container** | Runs before main containers start (e.g., DB migration) |

---

## 9. Namespaces

### What is a Namespace?

A **Namespace** is a logical partition inside a Kubernetes cluster. It allows you to divide cluster resources between multiple teams or environments.

```bash
kubectl get namespaces        # List all namespaces
kubectl create namespace dev  # Create a namespace
kubectl get pods -n dev       # Get pods in 'dev' namespace
```

**Default namespaces in Kubernetes:**

| Namespace | Purpose |
|---|---|
| `default` | Default namespace for user resources |
| `kube-system` | Kubernetes system components (DNS, proxy, etc.) |
| `kube-public` | Publicly readable data (cluster info) |
| `kube-node-lease` | Node heartbeat leases (for node health) |

**Example: Creating a Namespace**

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
```

**Resource Quotas per Namespace:**

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-quota
  namespace: dev
spec:
  hard:
    requests.cpu: "4"
    requests.memory: 4Gi
    limits.cpu: "8"
    limits.memory: 8Gi
    pods: "20"
```

> **Interview tip:** Namespaces do NOT provide network isolation by default. You need **NetworkPolicies** for that.

---

## 10. Deployments & Replica Sets

### ReplicaSet

A **ReplicaSet** ensures a specified number of pod replicas are running at any time.

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: my-replicaset
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: nginx
          image: nginx:1.21
```

> You rarely use ReplicaSets directly — Deployments manage them for you.

---

### Deployment

A **Deployment** provides declarative updates for Pods and ReplicaSets. It's the standard way to run stateless applications in Kubernetes.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
  namespace: default
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-container
          image: nginx:1.21
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
            limits:
              cpu: "500m"
              memory: "256Mi"
```

**Deployment capabilities:**
- Rolling updates (zero-downtime deployments)
- Rollback to previous versions
- Scaling up and down
- Pause and resume rollouts

```bash
kubectl get deployments
kubectl describe deployment my-deployment
kubectl scale deployment my-deployment --replicas=5
kubectl rollout status deployment/my-deployment
kubectl rollout history deployment/my-deployment
kubectl rollout undo deployment/my-deployment          # Rollback one version
kubectl rollout undo deployment/my-deployment --to-revision=2  # Rollback to specific
```

---

## 11. Services

### What is a Service?

A **Service** is an abstraction that defines a logical set of Pods and a policy to access them. Since Pods are ephemeral (they die and get new IPs), Services provide a **stable IP address and DNS name**.

Services use **label selectors** to find the target pods.

---

### Types of Services

#### 1. ClusterIP (Default)
- Exposes the service on an **internal cluster IP**
- Only accessible from within the cluster
- Used for internal microservice communication

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: ClusterIP
  selector:
    app: my-app
  ports:
    - port: 80          # Port the service listens on
      targetPort: 8080  # Port the container listens on
```

---

#### 2. NodePort
- Exposes the service on **each Node's IP** at a static port (30000–32767)
- Accessible from outside the cluster using `<NodeIP>:<NodePort>`
- In **kind**, you need port-forwarding since nodes are Docker containers

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nodeport-service
spec:
  type: NodePort
  selector:
    app: my-app
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30080   # Optional; auto-assigned if not set
```

```bash
# In kind — port forward since NodePort isn't directly accessible
kubectl port-forward svc/my-nodeport-service 8080:80
```

---

#### 3. LoadBalancer
- Exposes the service using a **cloud provider's load balancer** (AWS ELB, GCP LB, Azure LB)
- Gets an **external IP** automatically in cloud environments
- In local clusters, it stays in `<pending>` state (use MetalLB for on-premise)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-lb-service
spec:
  type: LoadBalancer
  selector:
    app: my-app
  ports:
    - port: 80
      targetPort: 8080
```

---

#### 4. ExternalName
- Maps the service to an **external DNS name** (not to pods)
- Returns a CNAME record — no proxying
- Used to connect to external services from within the cluster

```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-db
spec:
  type: ExternalName
  externalName: mydb.example.com
```

---

#### 5. Headless Service
- Created by setting `clusterIP: None`
- **No load balancing** — DNS returns individual pod IPs directly
- Used with **StatefulSets** (databases, Kafka, etc.) for direct pod addressing

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-headless-service
spec:
  clusterIP: None    # This makes it headless
  selector:
    app: my-db
  ports:
    - port: 5432
```

**Service comparison table:**

| Type | Accessible From | Use Case |
|---|---|---|
| ClusterIP | Inside cluster only | Internal microservices |
| NodePort | External via Node IP | Dev/testing, basic external access |
| LoadBalancer | External via cloud LB | Production external access |
| ExternalName | Inside cluster | Access external services |
| Headless | Inside cluster (direct pod) | StatefulSets, databases |

---

## 12. Labels & Selectors

### Labels

**Labels** are key-value pairs attached to Kubernetes objects. They are used to organize and select subsets of objects.

```yaml
metadata:
  labels:
    app: my-app
    environment: production
    version: "1.2.0"
    tier: frontend
```

---

### Selectors

**Selectors** are used to filter/select Kubernetes objects based on their labels.

**Equality-based:**
```yaml
selector:
  matchLabels:
    app: my-app
    environment: production
```

**Set-based:**
```yaml
selector:
  matchExpressions:
    - key: environment
      operator: In
      values: [production, staging]
    - key: tier
      operator: NotIn
      values: [backend]
```

**Using selectors with kubectl:**
```bash
kubectl get pods -l app=my-app
kubectl get pods -l environment=production,app=my-app
kubectl get pods -l 'environment in (production,staging)'
```

> **Why labels matter:** Services use selectors to route traffic to pods. Deployments use selectors to manage their pods. Without correct labels, nothing connects.

---

## 13. Flow of Kubernetes (End-to-End)

Here is what happens when you run `kubectl apply -f deployment.yaml`:

```
1. kubectl  →  sends HTTP request to API Server

2. API Server  →  authenticates & authorizes the request
                →  validates the manifest (schema check)
                →  writes desired state to etcd

3. Controller Manager  →  Deployment controller detects new Deployment
                       →  creates/updates a ReplicaSet
                       →  ReplicaSet controller creates Pod objects

4. Scheduler  →  sees unscheduled Pods
              →  scores nodes based on resources, taints, affinity
              →  assigns Pods to best-fit nodes
              →  updates Pod spec in etcd via API Server

5. kubelet (on assigned node)  →  watches API Server for pods assigned to its node
                               →  pulls container image
                               →  starts containers via container runtime (containerd)
                               →  reports pod status back to API Server

6. kube-proxy  →  updates iptables/IPVS rules on nodes
               →  routes Service traffic to correct pod IPs

7. CNI Plugin  →  assigns IP to the new pod
               →  sets up network routes so pods can communicate
```

---

## 14. Ingress & Gateway API

### What is Ingress?

An **Ingress** is a Kubernetes resource that manages **external HTTP/HTTPS** access to services inside the cluster. It acts as an **L7 (Layer 7) load balancer** and router.

**Without Ingress:** Each service needs its own LoadBalancer → expensive and complex.  
**With Ingress:** One load balancer routes traffic to multiple services based on rules.

```
Internet → LoadBalancer → Ingress Controller → Services → Pods
```

**Popular Ingress Controllers:**
- **NGINX Ingress Controller** (most popular)
- **Traefik**
- **HAProxy**
- **AWS ALB Ingress Controller** (for EKS)
- **Kong**

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
    - host: myapp.example.com
      http:
        paths:
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: api-service
                port:
                  number: 80
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend-service
                port:
                  number: 80
  tls:                            # TLS/HTTPS configuration
    - hosts:
        - myapp.example.com
      secretName: tls-secret
```

---

### What is Gateway API?

**Gateway API** is the **next generation** of Kubernetes ingress — it's more expressive, extensible, and role-oriented than Ingress.

**Key differences from Ingress:**

| Feature | Ingress | Gateway API |
|---|---|---|
| Role separation | None | Yes (infra team vs app team) |
| Protocol support | HTTP/HTTPS only | HTTP, HTTPS, TCP, UDP, gRPC |
| Extensibility | Annotations (hacky) | Native via policies |
| Stability | Stable | GA since K8s 1.28+ |

**Gateway API resources:**
- `GatewayClass` — defines the type of gateway (like IngressClass)
- `Gateway` — defines how traffic enters the cluster (managed by infra team)
- `HTTPRoute` — defines routing rules (managed by app team)
- `TCPRoute`, `GRPCRoute` — for non-HTTP traffic

```yaml
# HTTPRoute example
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: my-route
spec:
  parentRefs:
    - name: my-gateway
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /api
      backendRefs:
        - name: api-service
          port: 80
```

> Gateway API is the future. Many new projects now support it directly.

---

## 15. ConfigMaps & Secrets

### ConfigMap

A **ConfigMap** stores **non-sensitive** configuration data as key-value pairs. It decouples configuration from container images.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  DATABASE_HOST: "postgres.default.svc.cluster.local"
  DATABASE_PORT: "5432"
  LOG_LEVEL: "info"
  config.yaml: |
    server:
      port: 8080
      timeout: 30s
```

**Using ConfigMap in a Pod:**
```yaml
spec:
  containers:
    - name: my-app
      image: my-app:1.0
      # Option 1: As environment variables
      envFrom:
        - configMapRef:
            name: app-config
      # Option 2: Specific key as env var
      env:
        - name: DB_HOST
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: DATABASE_HOST
      # Option 3: Mount as a volume (file)
      volumeMounts:
        - name: config-volume
          mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        name: app-config
```

---

### Secrets

A **Secret** stores **sensitive data** such as passwords, tokens, and keys. Data is base64-encoded (NOT encrypted by default — use encryption at rest for production).

**Types of Secrets:**

| Type | Use Case |
|---|---|
| `Opaque` | Default; arbitrary key-value data |
| `kubernetes.io/dockerconfigjson` | Docker registry credentials |
| `kubernetes.io/tls` | TLS certificates |
| `kubernetes.io/service-account-token` | Service account tokens |
| `kubernetes.io/basic-auth` | Basic authentication |
| `kubernetes.io/ssh-auth` | SSH credentials |

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  username: YWRtaW4=       # base64 encoded "admin"
  password: cGFzc3dvcmQ=   # base64 encoded "password"
```

```bash
# Create a secret imperatively
kubectl create secret generic db-secret \
  --from-literal=username=admin \
  --from-literal=password=password

# View secret (decoded)
kubectl get secret db-secret -o jsonpath='{.data.password}' | base64 -d
```

**Using Secrets in a Pod:**
```yaml
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: db-secret
        key: password
```

> **Production best practice:** Use external secret managers like **HashiCorp Vault**, **AWS Secrets Manager**, or **External Secrets Operator** — don't store secrets directly in etcd.

---

## 16. Persistent Volumes & Claims

By default, container storage is **ephemeral** — data is lost when a pod dies. Persistent Volumes solve this.

### Persistent Volume (PV)

A **PersistentVolume** is a piece of storage in the cluster provisioned by an admin or dynamically provisioned using a **StorageClass**.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce   # Can be mounted by one node at a time
  persistentVolumeReclaimPolicy: Retain
  storageClassName: standard
  hostPath:                       # For local/dev; use cloud storage in prod
    path: /data/my-pv
```

**Access Modes:**

| Mode | Description |
|---|---|
| `ReadWriteOnce (RWO)` | Mounted read-write by a single node |
| `ReadOnlyMany (ROX)` | Mounted read-only by many nodes |
| `ReadWriteMany (RWX)` | Mounted read-write by many nodes |

**Reclaim Policies:**

| Policy | Behavior after PVC deletion |
|---|---|
| `Retain` | PV is kept; data preserved; manual cleanup |
| `Delete` | PV and underlying storage are deleted |
| `Recycle` | *(Deprecated)* Data scrubbed, PV reused |

---

### Persistent Volume Claim (PVC)

A **PVC** is a request for storage by a user. It's like a pod requesting CPU/memory, but for storage.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: standard
```

**Using PVC in a Pod:**
```yaml
spec:
  containers:
    - name: my-app
      image: my-app:1.0
      volumeMounts:
        - mountPath: /data
          name: storage
  volumes:
    - name: storage
      persistentVolumeClaim:
        claimName: my-pvc
```

---

### StorageClass

A **StorageClass** enables **dynamic provisioning** of PVs automatically when a PVC is created.

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-storage
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
reclaimPolicy: Delete
allowVolumeExpansion: true
```

---

## 17. Deployment Strategies

### RollingUpdate (Default)

**RollingUpdate** replaces pods gradually — old pods are taken down one by one as new ones come up. This ensures **zero downtime**.

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1        # Max pods above desired count during update
      maxUnavailable: 0  # Max pods that can be unavailable during update
```

**Process:**
```
v1 v1 v1  →  v1 v1 v2  →  v1 v2 v2  →  v2 v2 v2
```

---

### Recreate

**Recreate** terminates ALL existing pods before creating new ones. Results in **downtime** but is simpler.

```yaml
spec:
  strategy:
    type: Recreate
```

**Process:**
```
v1 v1 v1  →  [DOWN]  →  v2 v2 v2
```

Use when: your app can't run two versions simultaneously (e.g., database schema migrations).

---

### Other Strategies (beyond built-in K8s)

| Strategy | Description | Tool |
|---|---|---|
| **Blue/Green** | Run v1 and v2 simultaneously, switch traffic | Argo Rollouts, manual |
| **Canary** | Route small % of traffic to new version | Argo Rollouts, Istio |
| **A/B Testing** | Route traffic based on headers/cookies | Istio, Nginx |

```bash
# Check rollout status
kubectl rollout status deployment/my-app

# Rollback
kubectl rollout undo deployment/my-app

# Pause and resume
kubectl rollout pause deployment/my-app
kubectl rollout resume deployment/my-app
```

---

## 18. Health Checks & Probes

Kubernetes uses **probes** to check container health. If a container fails a probe, Kubernetes takes action (restart or stop routing traffic).

### 1. Liveness Probe

- Checks if the container is **alive**
- If it fails → Kubernetes **restarts** the container
- Use when: app can get into a broken state that requires a restart

```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 15   # Wait before first probe
  periodSeconds: 20          # Probe every 20 seconds
  failureThreshold: 3        # Restart after 3 failures
  timeoutSeconds: 5
```

---

### 2. Readiness Probe

- Checks if the container is **ready to serve traffic**
- If it fails → pod is **removed from Service endpoints** (traffic stops)
- Container is NOT restarted
- Use when: app needs time to warm up (load configs, connect to DB, etc.)

```yaml
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 10
  failureThreshold: 3
```

---

### 3. Startup Probe

- Checks if the container has **finished starting up**
- Disables liveness and readiness probes until it succeeds
- If it fails after `failureThreshold` attempts → container is killed and restarted
- Use when: app has a slow start (e.g., JVM apps, large ML models)

```yaml
startupProbe:
  httpGet:
    path: /healthz
    port: 8080
  failureThreshold: 30    # Give up to 30 * 10 = 300 seconds to start
  periodSeconds: 10
```

---

### Probe Types

| Probe Type | Description |
|---|---|
| `httpGet` | HTTP GET request to the container |
| `tcpSocket` | TCP connection check |
| `exec` | Run a command inside the container |
| `grpc` | gRPC health check |

```yaml
# exec example
livenessProbe:
  exec:
    command:
      - cat
      - /tmp/healthy
```

**Probe comparison:**

| | Liveness | Readiness | Startup |
|---|---|---|---|
| Failure action | Restart container | Remove from Service | Kill & restart |
| Use case | Detect deadlocks | Control traffic | Slow start |
| During startup | Can cause premature restart | - | Protects liveness |

---

## 19. Container Orchestration

### What is Container Orchestration?

**Container Orchestration** is the automated management of containerized application lifecycles across multiple hosts. It handles:

- **Scheduling** — placing containers on the right hosts
- **Scaling** — adding/removing containers based on load
- **Networking** — connecting containers
- **Load Balancing** — distributing traffic
- **Storage Management** — attaching volumes
- **Health Monitoring** — restarting failed containers
- **Rolling Updates** — zero-downtime deployments
- **Self-Healing** — replacing failed nodes/pods

**Kubernetes** is the industry-standard container orchestrator. Alternatives: Docker Swarm, Apache Mesos, Nomad.

---

## 20. Resource Requests & Limits

### What are Requests and Limits?

**Requests** = the minimum amount of CPU/memory Kubernetes **guarantees** to a container (used by scheduler).  
**Limits** = the maximum amount a container can use.

```yaml
resources:
  requests:
    memory: "128Mi"    # 128 mebibytes
    cpu: "250m"        # 250 millicores = 0.25 CPU core
  limits:
    memory: "256Mi"
    cpu: "500m"
```

**CPU units:**
- `1` = 1 CPU core
- `500m` = 0.5 CPU core
- `250m` = 0.25 CPU core (250 millicores)

**Memory units:** Ki, Mi, Gi (kibibytes, mebibytes, gibibytes)

---

### What happens when limits are exceeded?

| Resource | Over Limit Behavior |
|---|---|
| **CPU** | Container is **throttled** (slowed down), NOT killed |
| **Memory** | Container is **OOMKilled** (Out Of Memory, killed and restarted) |

---

### QoS Classes (Quality of Service)

Kubernetes assigns a QoS class to pods based on their resource config:

| Class | Condition | Eviction Priority |
|---|---|---|
| **Guaranteed** | requests == limits for all containers | Last to be evicted |
| **Burstable** | requests < limits | Middle priority |
| **BestEffort** | No requests or limits set | First to be evicted |

```bash
kubectl describe pod <name> | grep QoS
```

---

### LimitRange

Set default requests/limits for a namespace:

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limits
  namespace: dev
spec:
  limits:
    - type: Container
      default:
        cpu: "500m"
        memory: "256Mi"
      defaultRequest:
        cpu: "100m"
        memory: "128Mi"
```

---

## 21. Auto Scaling

Kubernetes supports three types of auto scaling:

### 1. HPA — Horizontal Pod Autoscaler

Scales the **number of pod replicas** based on CPU, memory, or custom metrics.

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-deployment
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70   # Scale when avg CPU > 70%
```

```bash
kubectl get hpa
kubectl autoscale deployment my-deployment --cpu-percent=70 --min=2 --max=10
```

> **HPA requires Metrics Server** to be installed.

---

### 2. VPA — Vertical Pod Autoscaler

Adjusts the **CPU and memory requests/limits** of containers automatically based on historical usage.

- Not built-in — requires installing the VPA controller
- Can recommend values or automatically update pods
- Pods need to be restarted to apply new values

---

### 3. Cluster Autoscaler

Scales the **number of nodes** in the cluster when:
- Pods can't be scheduled due to insufficient resources → adds nodes
- Nodes are underutilized for a period → removes nodes

Works with cloud providers (EKS, AKS, GKE) and their node groups/pools.

---

## 22. Metrics Server

The **Metrics Server** is a cluster-wide aggregator of resource usage data. It collects CPU and memory metrics from kubelets and exposes them via the Kubernetes API.

**Required for:**
- `kubectl top pods` and `kubectl top nodes`
- HPA (Horizontal Pod Autoscaler)

```bash
# Install metrics server (for local clusters)
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# For minikube
minikube addons enable metrics-server

# Usage
kubectl top nodes
kubectl top pods
kubectl top pods -n kube-system
```

> Metrics Server provides short-term metrics only. For long-term storage and dashboards, use **Prometheus + Grafana**.

---

## 23. Helm

### What is Helm?

**Helm** is the **package manager for Kubernetes**. Just like `apt` for Ubuntu or `pip` for Python, Helm manages Kubernetes applications called **Charts**.

- A **Chart** is a collection of pre-configured Kubernetes manifests
- **Values** allow customization of charts without modifying the templates
- **Releases** are instances of a chart deployed in a cluster

### Why Helm?

Without Helm: manually manage 10+ YAML files, hardcode values, difficult to version  
With Helm: install complex apps in one command, version-controlled, configurable

---

### Helm Architecture

```
Chart (templates + values.yaml)
       ↓
Helm CLI  →  renders manifests  →  kubectl apply  →  Kubernetes
```

---

### Helm Commands

```bash
# Install Helm
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Add a chart repository
helm repo add stable https://charts.helm.sh/stable
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

# Search for charts
helm search repo nginx
helm search hub wordpress

# Install a chart
helm install my-nginx bitnami/nginx
helm install my-release bitnami/mysql --set auth.rootPassword=secret

# Install with custom values file
helm install my-app ./my-chart -f custom-values.yaml

# List installed releases
helm list
helm list -A  # all namespaces

# Upgrade a release
helm upgrade my-nginx bitnami/nginx --set replicaCount=3

# Rollback
helm rollback my-nginx 1  # rollback to revision 1

# Uninstall
helm uninstall my-nginx

# Template rendering (dry run)
helm template my-app ./my-chart

# Show chart values
helm show values bitnami/nginx
```

---

### Chart Structure

```
my-chart/
├── Chart.yaml          # Chart metadata (name, version, description)
├── values.yaml         # Default configuration values
├── templates/          # Kubernetes manifest templates
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── _helpers.tpl    # Template helpers/partials
│   └── NOTES.txt       # Post-install notes
└── charts/             # Dependency charts
```

**values.yaml example:**
```yaml
replicaCount: 2
image:
  repository: nginx
  tag: "1.21"
  pullPolicy: IfNotPresent
service:
  type: ClusterIP
  port: 80
```

**Template using values:**
```yaml
spec:
  replicas: {{ .Values.replicaCount }}
  template:
    spec:
      containers:
        - image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
```

---

## 24. Kubernetes Dashboard

The **Kubernetes Dashboard** is a web-based UI for managing and monitoring your cluster.

```bash
# Install dashboard
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml

# Create admin service account
kubectl create serviceaccount admin-user -n kubernetes-dashboard
kubectl create clusterrolebinding admin-user \
  --clusterrole=cluster-admin \
  --serviceaccount=kubernetes-dashboard:admin-user

# Get login token
kubectl -n kubernetes-dashboard create token admin-user

# Access dashboard
kubectl proxy
# Visit: http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/

# In minikube
minikube dashboard
```

**Dashboard features:**
- View and manage workloads (Deployments, Pods, StatefulSets)
- View logs and exec into containers
- Manage Configs and Secrets
- View cluster events and resource usage
- Apply/edit YAML manifests directly

> For production monitoring, prefer **Prometheus + Grafana** over the dashboard.

---

## 25. Quick Reference & Cheat Sheet

### Common kubectl Commands

```bash
# Cluster info
kubectl cluster-info
kubectl get nodes -o wide
kubectl get componentstatuses

# Pods
kubectl run nginx --image=nginx --port=80    # Create a pod imperatively
kubectl get pods -o wide                     # Show pod IPs and nodes
kubectl get pods --watch                     # Watch pods in real time
kubectl delete pod <pod-name> --force        # Force delete pod
kubectl cp <pod>:/path/to/file ./local-file  # Copy file from pod

# Debugging
kubectl describe pod <pod-name>
kubectl logs <pod-name> --previous           # Logs from crashed container
kubectl logs -f <pod-name>                   # Follow/stream logs
kubectl exec -it <pod-name> -- bash
kubectl get events --sort-by='.lastTimestamp'

# Context and namespace
kubectl config get-contexts
kubectl config use-context <context-name>
kubectl config set-context --current --namespace=dev  # Set default namespace

# Resource management
kubectl get all -n <namespace>
kubectl delete all --all -n <namespace>     # ⚠️ Deletes everything in namespace
kubectl api-resources                        # List all available resource types

# Dry run (test without applying)
kubectl apply -f manifest.yaml --dry-run=client
kubectl apply -f manifest.yaml --dry-run=server

# Generate YAML from imperative commands
kubectl create deployment my-app --image=nginx --dry-run=client -o yaml > deployment.yaml
kubectl expose deployment my-app --port=80 --dry-run=client -o yaml > service.yaml
```

---

### Important Kubernetes Objects Summary

| Object | Purpose |
|---|---|
| Pod | Smallest deployable unit; wraps containers |
| ReplicaSet | Ensures N pod replicas are running |
| Deployment | Manages ReplicaSets; handles rolling updates |
| StatefulSet | Like Deployment but for stateful apps (stable IDs) |
| DaemonSet | Runs one pod per node (log agents, monitoring) |
| Job | Run-to-completion workload |
| CronJob | Scheduled job (like Linux cron) |
| Service | Stable network endpoint for pods |
| Ingress | HTTP/HTTPS external routing |
| ConfigMap | Non-sensitive configuration data |
| Secret | Sensitive data (passwords, tokens) |
| PersistentVolume | Storage resource in the cluster |
| PersistentVolumeClaim | Storage request by a pod |
| StorageClass | Dynamic storage provisioning |
| HPA | Scale pods based on metrics |
| Namespace | Logical cluster partition |
| ServiceAccount | Identity for pods to access K8s API |
| NetworkPolicy | Firewall rules for pod communication |
| RBAC (Role/ClusterRole) | Access control |

---

### Kubernetes Ports Reference

| Component | Default Port |
|---|---|
| API Server | 6443 |
| etcd | 2379-2380 |
| Scheduler | 10259 |
| Controller Manager | 10257 |
| kubelet | 10250 |
| NodePort range | 30000-32767 |

---

### Topics to Also Know for Interviews

- **RBAC** — Role-Based Access Control; `Role`, `ClusterRole`, `RoleBinding`, `ClusterRoleBinding`
- **NetworkPolicy** — restrict pod-to-pod communication
- **ServiceAccount** — identity for pods
- **Taints & Tolerations** — control which pods run on which nodes
- **Node Affinity / Pod Affinity** — prefer or require certain node/pod placement
- **StatefulSet** — ordered, stable pod identity for databases
- **DaemonSet** — run exactly one pod per node
- **CronJob** — scheduled recurring tasks
- **Resource Quotas** — limit total resources per namespace
- **etcd backup & restore** — critical for DR
- **Pod Disruption Budgets (PDB)** — maintain minimum available pods during maintenance
- **Admission Controllers** — intercept API requests (OPA Gatekeeper, Kyverno)
- **Operators** — extend Kubernetes to manage complex stateful apps
- **Prometheus + Grafana** — standard monitoring stack
- **Argo CD** — GitOps continuous delivery for Kubernetes
- **Cert-manager** — automate TLS certificate management

---

*Notes compiled for Kubernetes job preparation. Last updated: 2025.*
