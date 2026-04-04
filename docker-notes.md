# 🐳 Docker — Complete Notes for Interviews & Job Prep

> Comprehensive notes covering all core Docker concepts with explanations, commands, and interview tips.

---

## Table of Contents

1. [Why Docker? The Problem Before Docker](#1-why-docker-the-problem-before-docker)
2. [What is Containerization?](#2-what-is-containerization)
3. [Virtualization vs Containerization](#3-virtualization-vs-containerization)
4. [Docker Architecture Overview](#4-docker-architecture-overview)
   - [Docker Client](#41-docker-client)
   - [Docker Engine](#42-docker-engine)
   - [Docker Daemon](#43-docker-daemon)
   - [Docker Registry](#44-docker-registry)
5. [Dockerfile → Docker Image → Docker Container](#5-dockerfile--docker-image--docker-container)
6. [How to Write a Dockerfile](#6-how-to-write-a-dockerfile)
7. [Port Mapping](#7-port-mapping)
8. [Multi-Stage Docker Build](#8-multi-stage-docker-build)
9. [Distroless Images](#9-distroless-images)
10. [Data Persistence in Docker](#10-data-persistence-in-docker)
11. [Volume Mapping](#11-volume-mapping)
12. [Networking in Docker](#12-networking-in-docker)
    - [Bridge (Default)](#121-bridge-network-default)
    - [User-Defined Bridge](#122-user-defined-bridge)
    - [Host Network](#123-host-network)
    - [None Network](#124-none-network)
    - [MacVLAN](#125-macvlan)
    - [IPVlan](#126-ipvlan)
    - [Overlay Network](#127-overlay-network)
13. [Docker Compose](#13-docker-compose)
14. [ENTRYPOINT vs CMD](#14-entrypoint-vs-cmd)
15. [Important Commands Cheat Sheet](#15-important-commands-cheat-sheet)
16. [Interview Tips](#16-interview-tips)

---

## 1. Why Docker? The Problem Before Docker

### 🔴 The "Works on My Machine" Problem

Before Docker, developers constantly faced this nightmare scenario:

- A developer builds an app on their **local machine** (say, Ubuntu with Node.js 16, Python 3.9).
- They push the code to a **staging server** running CentOS with Node.js 14, Python 3.7.
- The app **breaks** in staging even though it worked perfectly locally.
- Then it breaks **differently** in production.

This was known as the **"It works on my machine"** problem.

### 🔴 Other Problems Developers Faced

| Problem | Explanation |
|---|---|
| **Environment mismatch** | Different OS, library versions, runtime versions between dev, staging, prod |
| **Dependency conflicts** | App A needs Python 2.7, App B needs Python 3.9 — impossible on same server |
| **Slow onboarding** | New devs spent days setting up local environments |
| **Heavy VMs** | Using Virtual Machines to solve this was too slow, resource-heavy |
| **Inconsistent deployments** | Each deployment had to be manually configured |
| **Scalability issues** | Spinning up new VMs took minutes, not seconds |

### ✅ How Docker Solved This

Docker packages the application **along with its entire environment** (OS libraries, runtime, config, dependencies) into a single unit called a **container**. This container runs **identically** everywhere — on any laptop, server, or cloud platform.

> **Key Idea:** Docker ensures the environment travels with the application.

---

## 2. What is Containerization?

**Containerization** is the process of packaging an application and all its dependencies (libraries, config files, runtime) into a single lightweight, portable unit called a **container**.

### How It Works

- Containers share the **host OS kernel** — they do NOT include a full OS.
- Each container is **isolated** from others using Linux kernel features:
  - **Namespaces** → isolate processes, network, file system (each container sees its own world)
  - **cgroups (Control Groups)** → limit and allocate CPU, memory, disk I/O per container
- This makes containers **extremely lightweight** and **fast to start** (milliseconds vs minutes for VMs).

### Container vs Process

A container is essentially a **process** (or group of processes) running on the host, but with:
- Its own isolated filesystem
- Its own isolated network stack
- Its own process tree
- Resource limits applied via cgroups

### Benefits of Containerization

- **Portability** — runs the same on dev, staging, and prod
- **Isolation** — one container can't affect another
- **Efficiency** — shares host kernel, uses far less RAM than VMs
- **Speed** — containers start in milliseconds
- **Scalability** — easy to spin up multiple identical containers
- **Microservices-friendly** — each service can live in its own container

---

## 3. Virtualization vs Containerization

This is one of the most common interview questions.

### Virtualization

- Uses a **Hypervisor** (e.g., VMware, VirtualBox, KVM) to create Virtual Machines (VMs).
- Each VM includes a **full OS** (kernel + userspace).
- VMs are **heavy**, typically GBs in size.
- Startup time: **minutes**.
- Strong **hardware-level isolation**.

### Containerization

- Uses the **host OS kernel** directly, no separate kernel per container.
- Containers only contain the app + its libraries.
- Containers are **lightweight**, typically MBs in size.
- Startup time: **milliseconds**.
- Isolation via **namespaces and cgroups** (OS-level, not hardware-level).

### Comparison Table

| Feature | Virtualization (VMs) | Containerization (Docker) |
|---|---|---|
| OS per instance | Full OS (kernel + userspace) | Shares host kernel |
| Size | GBs | MBs |
| Startup time | Minutes | Milliseconds |
| Resource usage | High | Low |
| Isolation level | Hardware-level (stronger) | OS-level (slightly weaker) |
| Portability | Lower | High |
| Use case | Full OS isolation, legacy apps | Microservices, cloud-native apps |

### Architecture Diagram

```
┌─────────────────────────────────────┐    ┌─────────────────────────────────────┐
│          VIRTUALIZATION             │    │         CONTAINERIZATION            │
│                                     │    │                                     │
│  ┌────────┐ ┌────────┐ ┌────────┐  │    │  ┌────────┐ ┌────────┐ ┌────────┐  │
│  │ App A  │ │ App B  │ │ App C  │  │    │  │ App A  │ │ App B  │ │ App C  │  │
│  ├────────┤ ├────────┤ ├────────┤  │    │  ├────────┤ ├────────┤ ├────────┤  │
│  │Guest OS│ │Guest OS│ │Guest OS│  │    │  │  Libs  │ │  Libs  │ │  Libs  │  │
│  ├────────┴─┴────────┴─┴────────┤  │    │  ├────────┴─┴────────┴─┴────────┤  │
│  │         Hypervisor           │  │    │  │       Container Runtime       │  │
│  ├──────────────────────────────┤  │    │  │      (Docker Engine)          │  │
│  │           Host OS            │  │    │  ├──────────────────────────────┤   │
│  ├──────────────────────────────┤  │    │  │        Host OS Kernel         │  │
│  │           Hardware           │  │    │  ├──────────────────────────────┤   │
│  └──────────────────────────────┘  │    │  │           Hardware            │  │
└─────────────────────────────────────┘    └─────────────────────────────────────┘
```

> **Interview Tip:** VMs are better for strong security isolation. Containers are better for speed, efficiency, and microservices.

---

## 4. Docker Architecture Overview

Docker follows a **client-server architecture**.

```
┌──────────────┐        REST API       ┌───────────────────────────────────┐
│ Docker Client│ ◄──────────────────► │           Docker Host              │
│  (docker CLI)│                       │                                    │
└──────────────┘                       │  ┌─────────────┐                  │
                                       │  │Docker Daemon│                  │
                                       │  │  (dockerd)  │                  │
                                       │  └──────┬──────┘                  │
                                       │         │                          │
                                       │  ┌──────▼──────┐  ┌───────────┐  │
                                       │  │  Containers  │  │  Images   │  │
                                       │  └─────────────┘  └───────────┘  │
                                       └───────────────────────────────────┘
                                                      │
                                                      ▼
                                          ┌───────────────────┐
                                          │   Docker Registry  │
                                          │   (Docker Hub)     │
                                          └───────────────────┘
```

---

### 4.1 Docker Client

- The **CLI tool** you interact with — the `docker` command.
- Translates your commands into **REST API calls** to the Docker Daemon.
- Can connect to a **local or remote** Docker daemon.
- Examples: `docker build`, `docker run`, `docker pull`

### 4.2 Docker Engine

Docker Engine is the **core** of Docker. It is a client-server application composed of:

1. **Docker Daemon** (`dockerd`) — background service that manages containers
2. **REST API** — interface the client uses to talk to the daemon
3. **Docker CLI** — the client that uses the REST API

Docker Engine also includes **containerd** (manages container lifecycle) and **runc** (low-level container runner).

```
Docker Engine Stack:
  docker CLI  →  REST API  →  dockerd  →  containerd  →  runc  →  container
```

### 4.3 Docker Daemon

- Also called **`dockerd`**.
- A **background process** that does the actual work.
- Listens for Docker API requests.
- Manages: **images, containers, volumes, networks**.
- Communicates with other daemons (for Docker Swarm).

### 4.4 Docker Registry

- A **storage and distribution system** for Docker images.
- **Docker Hub** is the default public registry (`hub.docker.com`).
- You can also set up a **private registry** (e.g., AWS ECR, GCR, Harbor, self-hosted).

**Common registry commands:**

```bash
docker pull nginx                        # Pull from Docker Hub
docker push myusername/myapp:v1          # Push to Docker Hub
docker login                             # Authenticate with registry
docker tag myapp myusername/myapp:v1     # Tag before pushing
```

**Private registry example:**

```bash
docker pull 123456.dkr.ecr.ap-south-1.amazonaws.com/myapp:latest   # AWS ECR
```

---

## 5. Dockerfile → Docker Image → Docker Container

These three are the **core lifecycle** of Docker. Understanding this flow is essential.

```
Dockerfile  ──(docker build)──►  Docker Image  ──(docker run)──►  Docker Container
   (recipe)                         (template)                        (running instance)
```

### Dockerfile

- A **text file** containing instructions to build an image.
- Think of it as a **recipe** or **blueprint**.
- Every instruction creates a **layer** in the image.

### Docker Image

- A **read-only template** built from a Dockerfile.
- Made up of **layers** (each instruction = one layer).
- Layers are **cached** — if a layer hasn't changed, Docker reuses the cache (faster builds).
- Images are **stored** in a registry (Docker Hub, ECR, etc.).

```bash
docker build -t myapp:v1 .       # Build image from Dockerfile in current directory
docker images                    # List all images
docker image inspect myapp:v1    # Inspect image details and layers
docker rmi myapp:v1              # Remove an image
```

### Docker Container

- A **running instance** of an image.
- Has its own **isolated filesystem, network, and processes**.
- Is **ephemeral by default** — data is lost when the container stops/deletes.
- You can run **multiple containers** from the same image simultaneously.

```bash
docker run myapp:v1               # Run a container
docker run -d myapp:v1            # Run in detached (background) mode
docker ps                         # List running containers
docker ps -a                      # List all containers (including stopped)
docker stop <container_id>        # Stop a container
docker rm <container_id>          # Remove a container
docker logs <container_id>        # View container logs
docker exec -it <container_id> bash  # Enter a running container
```

### Image Layering (Important Concept)

```
┌──────────────────────┐
│   Your App Layer      │  ← COPY . .
├──────────────────────┤
│   Dependency Layer    │  ← RUN npm install
├──────────────────────┤
│   Config Layer        │  ← WORKDIR /app
├──────────────────────┤
│   Base Image Layer    │  ← FROM node:18
└──────────────────────┘
```

Each layer is cached. If `package.json` doesn't change, `npm install` layer is reused → **faster builds**.

---

## 6. How to Write a Dockerfile

### Dockerfile Instructions Reference

| Instruction | Purpose |
|---|---|
| `FROM` | Base image to build on |
| `WORKDIR` | Set working directory inside container |
| `COPY` | Copy files from host to container |
| `ADD` | Like COPY, but also handles URLs and tar extraction |
| `RUN` | Execute command during build (creates a layer) |
| `ENV` | Set environment variables |
| `ARG` | Build-time variables (not available at runtime) |
| `EXPOSE` | Document which port the app uses (informational) |
| `CMD` | Default command to run when container starts |
| `ENTRYPOINT` | Configure container to run as an executable |
| `VOLUME` | Create a mount point for persistent data |
| `USER` | Set the user to run subsequent commands |
| `LABEL` | Add metadata to the image |
| `HEALTHCHECK` | Define a health check command |

### Example: Node.js Application Dockerfile

```dockerfile
# Stage 1: Use official Node.js base image
FROM node:18-alpine

# Set working directory inside the container
WORKDIR /app

# Copy package files first (for better layer caching)
COPY package*.json ./

# Install dependencies
RUN npm install --production

# Copy rest of the application code
COPY . .

# Set environment variable
ENV NODE_ENV=production

# Expose port 3000
EXPOSE 3000

# Command to run the app
CMD ["node", "server.js"]
```

### Example: Python Application Dockerfile

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 5000

CMD ["python", "app.py"]
```

### Best Practices for Dockerfiles

- **Use specific tags** — `FROM node:18-alpine`, not `FROM node:latest`
- **Copy dependency files first** — helps Docker cache the install layer
- **Use `.dockerignore`** — exclude `node_modules`, `.git`, etc.
- **Run as non-root user** — for security
- **Minimize layers** — combine multiple `RUN` commands with `&&`
- **Use slim/alpine images** — smaller attack surface and faster pulls

### .dockerignore Example

```
node_modules
.git
.env
*.log
dist
__pycache__
.DS_Store
```

---

## 7. Port Mapping

By default, containers are **isolated** from the host network. Port mapping (also called **port publishing**) exposes a container's port to the host machine.

### Syntax

```bash
docker run -p <host_port>:<container_port> <image>
```

### Examples

```bash
# Map host port 8080 to container port 3000
docker run -p 8080:3000 myapp

# Map host port 80 to container port 80
docker run -p 80:80 nginx

# Map all interfaces (default) or specific IP
docker run -p 127.0.0.1:8080:3000 myapp   # Only accessible on localhost

# Multiple port mappings
docker run -p 8080:80 -p 443:443 nginx
```

After running the above, you can access the app at `http://localhost:8080`.

### How It Works

```
Browser → localhost:8080 → Docker Host (port 8080) → Container (port 3000) → App
```

### EXPOSE vs -p flag

- `EXPOSE` in Dockerfile is just **documentation** — it doesn't actually publish the port.
- `-p` flag in `docker run` actually **publishes** the port.

---

## 8. Multi-Stage Docker Build

Multi-stage builds allow you to use **multiple FROM statements** in a single Dockerfile. This is used to create **smaller, production-ready images** by separating build-time dependencies from runtime dependencies.

### The Problem Without Multi-Stage

If you build a Go or Java app inside the container, you need build tools (compiler, SDK) in the image. These tools are only needed at **build time**, not at runtime. Without multi-stage builds, your final image is bloated with unnecessary tools.

### How Multi-Stage Builds Work

```dockerfile
# ─── Stage 1: Build Stage ───────────────────────────────
FROM golang:1.21 AS builder

WORKDIR /app
COPY . .
RUN go build -o myapp .

# ─── Stage 2: Final Stage ───────────────────────────────
FROM alpine:3.18

WORKDIR /app

# Copy only the compiled binary from the builder stage
COPY --from=builder /app/myapp .

EXPOSE 8080
CMD ["./myapp"]
```

### Benefits

- Final image contains **only the binary** — no Go compiler, no build tools.
- Image size drops from **~800MB to ~10MB** (in Go apps).
- Smaller surface area = **more secure**.

### Multi-Stage for Node.js

```dockerfile
# Stage 1: Build
FROM node:18 AS build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Stage 2: Production
FROM node:18-alpine AS production
WORKDIR /app
COPY --from=build /app/dist ./dist
COPY --from=build /app/node_modules ./node_modules
EXPOSE 3000
CMD ["node", "dist/server.js"]
```

---

## 9. Distroless Images

**Distroless images** are container images that contain **only your application and its runtime dependencies** — no shell (`bash`/`sh`), no package manager (`apt`, `yum`), no OS utilities.

They were pioneered by **Google** and are available at `gcr.io/distroless/`.

### Why Distroless?

| Feature | Regular Image | Distroless Image |
|---|---|---|
| Shell (bash/sh) | ✅ Present | ❌ Absent |
| Package manager | ✅ Present | ❌ Absent |
| Attack surface | Larger | Minimal |
| CVEs | More | Far fewer |
| Image size | Larger | Smaller |
| Debug-ability | Easy | Hard |

### Example: Distroless with Go

```dockerfile
# Stage 1: Build
FROM golang:1.21 AS builder
WORKDIR /app
COPY . .
RUN CGO_ENABLED=0 go build -o myapp .

# Stage 2: Distroless Final Image
FROM gcr.io/distroless/static-debian12

COPY --from=builder /app/myapp /myapp

EXPOSE 8080
ENTRYPOINT ["/myapp"]
```

### Example: Distroless with Java

```dockerfile
FROM eclipse-temurin:21 AS builder
WORKDIR /app
COPY . .
RUN ./mvnw package -DskipTests

FROM gcr.io/distroless/java21-debian12
COPY --from=builder /app/target/myapp.jar /app.jar
ENTRYPOINT ["java", "-jar", "/app.jar"]
```

### Distroless vs Alpine

| | Distroless | Alpine |
|---|---|---|
| Shell | No | Yes (ash) |
| Size | ~2MB (static) | ~5MB |
| Security | Better | Good |
| Debugging | Harder | Easier |
| Package manager | No | Yes (apk) |

> **Interview Tip:** Distroless is preferred in production for security. Alpine is a good middle ground.

---

## 10. Data Persistence in Docker

By default, data written inside a container is **ephemeral** — when the container is deleted, all data is lost. This is because containers use a **writable layer** on top of the read-only image layers, and this layer is removed with the container.

### The Problem

```bash
docker run -it ubuntu bash
# Inside container:
echo "hello" > /data/file.txt
exit

docker rm <container_id>
# file.txt is GONE forever!
```

### Solutions for Data Persistence

There are **3 ways** to persist data in Docker:

| Method | Description | Use Case |
|---|---|---|
| **Volumes** | Docker-managed storage on host | Production databases, persistent data |
| **Bind Mounts** | Mount a specific host directory into container | Development (live code sync) |
| **tmpfs Mounts** | In-memory storage (Linux only) | Sensitive temp data, performance |

---

## 11. Volume Mapping

### Docker Volumes (Recommended)

Volumes are managed by Docker and stored in `/var/lib/docker/volumes/` on the host.

```bash
# Create a named volume
docker volume create mydata

# Run container with a volume
docker run -v mydata:/app/data myapp

# List volumes
docker volume ls

# Inspect a volume
docker volume inspect mydata

# Remove a volume
docker volume rm mydata

# Remove all unused volumes
docker volume prune
```

### Bind Mounts

Bind mounts link a **specific host path** to a container path.

```bash
# Mount current directory into container (useful for dev)
docker run -v $(pwd):/app myapp

# Mount a specific directory
docker run -v /home/user/data:/app/data myapp

# Read-only bind mount
docker run -v $(pwd):/app:ro myapp
```

### Differences: Volumes vs Bind Mounts

| Feature | Volumes | Bind Mounts |
|---|---|---|
| Managed by | Docker | You (host filesystem) |
| Location | `/var/lib/docker/volumes/` | Any path on host |
| Portability | High | Low (path must exist on host) |
| Performance | Better | Slightly lower |
| Use case | Production data | Development |
| Backup | Easy (docker volume) | Manual |

### tmpfs Mounts

```bash
docker run --tmpfs /tmp myapp    # Store /tmp in memory
```

Data in tmpfs is stored in **RAM only** — very fast, but lost when container stops.

---

## 12. Networking in Docker

Docker networking allows containers to communicate with each other and with the outside world. Docker provides several **network drivers**.

```bash
docker network ls             # List networks
docker network inspect <name> # Inspect a network
docker network create <name>  # Create a network
docker network rm <name>      # Remove a network
```

---

### 12.1 Bridge Network (Default)

- **Default network** for containers.
- Docker creates a virtual bridge called `docker0` on the host.
- All containers on the same bridge can communicate via **IP address**.
- Containers on the **default bridge cannot communicate by container name** (no DNS resolution).

```bash
docker run --network bridge nginx    # Explicitly use bridge (it's the default)
docker inspect bridge                # Inspect default bridge network
```

**Architecture:**
```
Host
 └── docker0 (bridge: 172.17.0.0/16)
      ├── container1 (172.17.0.2)
      └── container2 (172.17.0.3)
```

---

### 12.2 User-Defined Bridge

- A **custom bridge network** you create yourself.
- Containers get **automatic DNS resolution by container name** (most important feature!).
- Better isolation and control.
- **Recommended for production** over the default bridge.

```bash
# Create custom bridge network
docker network create mynetwork

# Run containers on custom network
docker run -d --name webapp --network mynetwork nginx
docker run -d --name db --network mynetwork postgres

# Now webapp can reach db by name:
# ping db       ← works!
# ping 172.18.0.3  ← also works
```

> **Interview Tip:** Always use user-defined bridge over default bridge. The DNS-based name resolution is the key advantage.

---

### 12.3 Host Network

- The container **shares the host's network stack** — no network isolation.
- The container's ports are **directly accessible** on the host without port mapping.
- **Only available on Linux** (not Mac/Windows Docker Desktop).
- Best for **performance-critical** applications.

```bash
docker run --network host nginx
# Nginx now runs on host's port 80 directly — no -p flag needed
```

**Use cases:** High-performance networking, when port mapping overhead matters.

---

### 12.4 None Network

- The container has **no network interface** (except loopback `lo`).
- Completely **isolated** from all networks.
- Used for maximum security isolation or batch processing jobs.

```bash
docker run --network none myapp
# Container cannot reach internet or other containers
```

---

### 12.5 MacVLAN

- Assigns a **real MAC address** to the container, making it appear as a **physical device** on the network.
- The container gets an IP from the **physical network's subnet** (like DHCP).
- Bypasses the host's network stack entirely.
- Used when containers need to be **directly accessible on the LAN**.

```bash
docker network create -d macvlan \
  --subnet=192.168.1.0/24 \
  --gateway=192.168.1.1 \
  -o parent=eth0 \
  mymacvlan

docker run --network mymacvlan --ip 192.168.1.100 nginx
```

**Use cases:** Legacy apps that expect to be on the physical network, network monitoring tools.

---

### 12.6 IPVlan

Similar to MacVLAN but operates at **Layer 3 (IP level)** instead of Layer 2 (MAC level).

- Does **not assign unique MAC addresses** — all containers share the host's MAC.
- Better suited for environments where switches limit the number of MAC addresses.
- Two modes: **L2 mode** (like macvlan) and **L3 mode** (routing).

```bash
docker network create -d ipvlan \
  --subnet=192.168.1.0/24 \
  --gateway=192.168.1.1 \
  -o parent=eth0 \
  myipvlan
```

**MacVLAN vs IPVlan:**

| Feature | MacVLAN | IPVlan |
|---|---|---|
| MAC addresses | Unique per container | Shared with host |
| Layer | L2 | L2 or L3 |
| Promiscuous mode needed | Yes | No |
| Switch compatibility | Issues with MAC limits | Better |

---

### 12.7 Overlay Network

- Enables communication between containers **across multiple Docker hosts** (different servers).
- Required for **Docker Swarm** and distributed applications.
- Uses **VXLAN tunneling** to create a virtual network spanning multiple hosts.

```bash
# Requires Docker Swarm to be initialized
docker swarm init

docker network create --driver overlay myoverlay

docker service create --network myoverlay --name web nginx
```

**Use cases:** Microservices distributed across multiple servers, Docker Swarm clusters.

```
Host 1                    Host 2
 └── Container A  ←────────→  Container B
      (overlay network via VXLAN tunnel)
```

### Networking Summary Table

| Network | Isolation | Container-to-Container | DNS by Name | Use Case |
|---|---|---|---|---|
| Bridge (default) | Yes | Same host only | ❌ No | Basic local testing |
| User-defined Bridge | Yes | Same host only | ✅ Yes | Local multi-container apps |
| Host | No | Via host | ✅ Yes | High performance, Linux only |
| None | Complete | ❌ No | ❌ No | Maximum isolation |
| MacVLAN | No (on LAN) | Yes | No (by IP) | LAN-visible containers |
| IPVlan | No (on LAN) | Yes | No (by IP) | Restricted MAC environments |
| Overlay | Yes | Multi-host | ✅ Yes | Docker Swarm, multi-host |

---

## 13. Docker Compose

**Docker Compose** is a tool for defining and running **multi-container Docker applications** using a YAML file (`docker-compose.yml`).

Instead of running multiple `docker run` commands with many flags, you define everything in one file and start everything with a single command.

### Why Docker Compose?

Imagine a 3-tier app:
- Frontend (React)
- Backend (Node.js API)
- Database (PostgreSQL)
- Cache (Redis)

Without Compose: 4 long `docker run` commands with networks, volumes, ports, env vars...

With Compose: One file, one command.

### docker-compose.yml Structure

```yaml
version: '3.8'

services:
  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    depends_on:
      - backend
    networks:
      - appnet

  backend:
    build: ./backend
    ports:
      - "5000:5000"
    environment:
      - DATABASE_URL=postgres://user:pass@db:5432/mydb
      - REDIS_URL=redis://cache:6379
    depends_on:
      - db
      - cache
    networks:
      - appnet

  db:
    image: postgres:15
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: mydb
    volumes:
      - pgdata:/var/lib/postgresql/data
    networks:
      - appnet

  cache:
    image: redis:7-alpine
    networks:
      - appnet

volumes:
  pgdata:

networks:
  appnet:
    driver: bridge
```

### Key Docker Compose Commands

```bash
docker compose up              # Start all services
docker compose up -d           # Start in detached mode
docker compose down            # Stop and remove containers
docker compose down -v         # Stop and remove containers + volumes
docker compose ps              # List running services
docker compose logs            # View logs
docker compose logs -f backend # Follow logs for a specific service
docker compose build           # Build/rebuild images
docker compose exec backend bash  # Execute command in running service
docker compose restart backend    # Restart a specific service
```

### Important Compose Concepts

**`depends_on`:** Controls startup **order** but does NOT wait for the service to be **ready** (e.g., PostgreSQL accepting connections). Use health checks for that.

**Health Checks:**

```yaml
db:
  image: postgres:15
  healthcheck:
    test: ["CMD-SHELL", "pg_isready -U user"]
    interval: 10s
    timeout: 5s
    retries: 5
```

**Environment Variables from .env file:**

```bash
# .env file
DB_PASSWORD=secret
```

```yaml
environment:
  - DB_PASSWORD=${DB_PASSWORD}
```

> **Interview Tip:** Docker Compose is for **single-host** multi-container apps. For **multi-host** orchestration, use **Kubernetes** or **Docker Swarm**.

---

## 14. ENTRYPOINT vs CMD

This is a very common interview question and also easy to confuse.

### CMD

- Defines the **default command** to run when a container starts.
- Can be **completely overridden** by passing a command at `docker run`.
- Can be used to pass **default arguments** to `ENTRYPOINT`.

```dockerfile
CMD ["node", "server.js"]
```

```bash
docker run myapp                    # Runs: node server.js
docker run myapp python app.py      # Overrides CMD → Runs: python app.py
```

### ENTRYPOINT

- Defines the **executable** that always runs.
- Arguments from `docker run` are **appended** to `ENTRYPOINT`, not replaced.
- The container **always runs this command** — it can't be overridden without `--entrypoint` flag.

```dockerfile
ENTRYPOINT ["node", "server.js"]
```

```bash
docker run myapp                    # Runs: node server.js
docker run myapp --port 4000        # Runs: node server.js --port 4000 (appended!)
docker run --entrypoint bash myapp  # Override entrypoint: runs bash
```

### ENTRYPOINT + CMD Together

The most powerful and flexible pattern — `ENTRYPOINT` sets the executable, `CMD` sets default arguments that can be overridden.

```dockerfile
ENTRYPOINT ["python", "app.py"]
CMD ["--port", "5000"]
```

```bash
docker run myapp                        # Runs: python app.py --port 5000 (default)
docker run myapp --port 8000            # Runs: python app.py --port 8000 (CMD overridden)
```

### Shell vs Exec Form

```dockerfile
# Shell form (runs via /bin/sh -c) — signals NOT forwarded properly
CMD node server.js
ENTRYPOINT node server.js

# Exec form (preferred) — signals forwarded correctly
CMD ["node", "server.js"]
ENTRYPOINT ["node", "server.js"]
```

> **Always use exec form** (JSON array syntax) for `ENTRYPOINT` and `CMD`.

### Comparison Summary

| Feature | CMD | ENTRYPOINT |
|---|---|---|
| Purpose | Default arguments / default command | Main executable |
| Override with `docker run args` | Yes (completely replaced) | No (args appended) |
| Override with `--entrypoint` | N/A | Yes |
| Used together | Provides default args to ENTRYPOINT | Sets the executable |
| Best for | Variable commands | Fixed executables |

---

## 15. Important Commands Cheat Sheet

### Image Commands

```bash
docker build -t myapp:v1 .           # Build image
docker images                        # List images
docker rmi myapp:v1                  # Remove image
docker pull nginx                    # Pull from registry
docker push myusername/myapp:v1      # Push to registry
docker tag myapp myusername/myapp:v1 # Tag an image
docker image prune                   # Remove dangling images
docker save myapp > myapp.tar        # Export image to tar
docker load < myapp.tar              # Import image from tar
docker history myapp:v1              # Show image layers
```

### Container Commands

```bash
docker run -d -p 8080:3000 --name mycontainer myapp:v1   # Run container
docker ps                            # List running containers
docker ps -a                         # List all containers
docker stop mycontainer              # Stop container
docker start mycontainer             # Start stopped container
docker restart mycontainer           # Restart container
docker rm mycontainer                # Remove container
docker rm -f mycontainer             # Force remove running container
docker logs mycontainer              # View logs
docker logs -f mycontainer           # Follow logs
docker exec -it mycontainer bash     # Shell into container
docker inspect mycontainer           # Detailed container info
docker stats                         # Live resource usage stats
docker top mycontainer               # Running processes in container
docker cp mycontainer:/app/file.txt . # Copy file from container
docker container prune               # Remove all stopped containers
```

### Volume Commands

```bash
docker volume create myvolume        # Create volume
docker volume ls                     # List volumes
docker volume inspect myvolume       # Inspect volume
docker volume rm myvolume            # Remove volume
docker volume prune                  # Remove unused volumes
```

### Network Commands

```bash
docker network ls                    # List networks
docker network create mynetwork      # Create network
docker network inspect mynetwork     # Inspect network
docker network rm mynetwork          # Remove network
docker network connect mynetwork mycontainer   # Connect container to network
docker network disconnect mynetwork mycontainer # Disconnect
```

### System Commands

```bash
docker system df                     # Show disk usage
docker system prune                  # Remove all unused resources
docker system prune -a               # Remove everything including images
docker info                          # Docker system information
docker version                       # Docker version
```

---

## 16. Interview Tips

### Common Interview Questions

**Q: What is the difference between `docker stop` and `docker kill`?**
> `docker stop` sends `SIGTERM` first (graceful shutdown), then `SIGKILL` after a timeout. `docker kill` sends `SIGKILL` immediately (forceful).

**Q: What is a dangling image?**
> An image with no tag (shows as `<none>:<none>`) — usually old intermediate build images. Remove with `docker image prune`.

**Q: How do you reduce Docker image size?**
> Use alpine/slim base images, multi-stage builds, distroless images, minimize layers, use `.dockerignore`, and clean up package cache in the same `RUN` command.

**Q: Can two containers communicate without `-p` flag?**
> Yes — if they are on the same Docker network, they can communicate internally without exposing ports to the host.

**Q: What is the difference between `ADD` and `COPY`?**
> Both copy files into the image. `ADD` also handles URLs and automatically extracts `.tar` files. Best practice: prefer `COPY` unless you specifically need `ADD`'s extra features.

**Q: What is a Docker registry vs Docker repository?**
> A **registry** is the server that stores images (e.g., Docker Hub, ECR). A **repository** is a collection of images with the same name but different tags (e.g., `nginx:1.24`, `nginx:latest`, `nginx:alpine`).

**Q: How does Docker achieve isolation?**
> Via Linux **namespaces** (process, network, mount, UTS, IPC isolation) and **cgroups** (resource limits).

**Q: What happens when you run `docker run`?**
> 1. Docker CLI sends request to Docker Daemon. 2. Daemon checks for image locally. 3. If not found, pulls from registry. 4. Creates a new writable container layer on top of the image. 5. Sets up networking. 6. Starts the process defined by CMD/ENTRYPOINT.

**Q: What is Docker Compose vs Docker Swarm vs Kubernetes?**
> - **Docker Compose**: Single-host, multi-container management
> - **Docker Swarm**: Multi-host container orchestration (native Docker, simpler)
> - **Kubernetes**: Multi-host container orchestration (industry standard, more complex and feature-rich)

### Key Things to Remember for Interviews

- Containers share the **host kernel** (unlike VMs)
- Images are **immutable**; containers add a writable layer
- `ENTRYPOINT` = what runs; `CMD` = default args (can be overridden)
- Always use **user-defined bridge** over default bridge (DNS resolution)
- **Volumes** outlive containers; data persists after container removal
- Multi-stage builds dramatically reduce final image size
- Distroless = no shell = more secure production images
- Docker Compose is for **single host**; Kubernetes for **multi-host orchestration**

---

*Notes by: [Your Name] | Last Updated: 2025*

*References:*
- *[Docker Official Documentation](https://docs.docker.com)*
- *[Docker Hub](https://hub.docker.com)*
- *[Google Distroless](https://github.com/GoogleContainerTools/distroless)*
