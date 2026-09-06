# Docker & Kubernetes — Complete CLI Guide

> From `docker build` to a full `kubectl`-driven Kubernetes cluster — entirely from the command line.
> This guide first covers Docker end-to-end, then explains Kubernetes, **including exactly how Kubernetes uses Docker under the hood**, all the way through to running a real cluster.

---

## Table of Contents

**Part 1 — Docker**
1. [What Is Docker? (Containers vs. VMs)](#1-what-is-docker-containers-vs-vms)
2. [Installing & Verifying Docker](#2-installing--verifying-docker)
3. [Docker Images — Pulling & Inspecting](#3-docker-images--pulling--inspecting)
4. [Writing a Dockerfile & Building Images](#4-writing-a-dockerfile--building-images)
5. [Running Containers](#5-running-containers)
6. [Managing Running Containers](#6-managing-running-containers)
7. [Volumes & Networking](#7-volumes--networking)
8. [Docker Compose (Multi-Container Apps)](#8-docker-compose-multi-container-apps)
9. [Docker Hub & Registries](#9-docker-hub--registries)
10. [Cleaning Up & System Commands](#10-cleaning-up--system-commands)
11. [Docker CLI Cheat Sheet](#11-docker-cli-cheat-sheet)

**Part 2 — Kubernetes**
12. [What Is Kubernetes? How It Uses Docker](#12-what-is-kubernetes-how-it-uses-docker)
13. [Kubernetes Architecture](#13-kubernetes-architecture)
14. [Installing kubectl & Starting a Local Cluster](#14-installing-kubectl--starting-a-local-cluster)
15. [Core kubectl Commands](#15-core-kubectl-commands)
16. [Pods, Deployments & ReplicaSets](#16-pods-deployments--replicasets)
17. [Services & Networking](#17-services--networking)
18. [ConfigMaps & Secrets](#18-configmaps--secrets)
19. [Scaling, Rolling Updates & Rollbacks](#19-scaling-rolling-updates--rollbacks)
20. [Namespaces & Contexts](#20-namespaces--contexts)
21. [Kubernetes CLI Cheat Sheet](#21-kubernetes-cli-cheat-sheet)
22. [Best Practices](#22-best-practices)

---
---

# PART 1 — DOCKER

## 1. What Is Docker? (Containers vs. VMs)

Docker is a platform for building, running, and shipping applications inside **containers** — lightweight, isolated environments that package an application together with everything it needs to run: code, runtime, system libraries, and settings.

### Containers vs. Virtual Machines

A **Virtual Machine (VM)** virtualizes an entire computer, including its own operating system kernel — making it heavy (gigabytes) and slow to start (minutes).

A **container** shares the host machine's OS kernel and only packages the application and its dependencies — making it lightweight (megabytes) and fast to start (seconds).

| Aspect | Virtual Machine | Docker Container |
|---|---|---|
| Size | Gigabytes | Megabytes |
| Startup time | Minutes | Seconds |
| OS | Full guest OS per VM | Shares host OS kernel |
| Isolation | Full hardware-level | Process-level |

### Key Docker Concepts

- **Image** — a read-only template/blueprint for a container (like a class in code)
- **Container** — a running (or stopped) instance of an image (like an object/instance)
- **Dockerfile** — a text file with instructions to build an image
- **Registry** — a place to store and share images, e.g. Docker Hub

> 💡 **In short:** An image is the recipe. A container is the dish made from that recipe, running right now.

### How a Typical Docker Workflow Flows

1. **Write** a Dockerfile describing how to build your app's image
2. **Build** the image with `docker build`
3. **Run** one or more containers from that image
4. **Manage** running containers — logs, exec, stop/start
5. **Push** the image to a registry (like Docker Hub) to share it
6. **Pull & run** that same image anywhere — your laptop, a server, or later, a Kubernetes cluster

---

## 2. Installing & Verifying Docker

### Install Docker

| Operating System | Command / Method |
|---|---|
| Windows / macOS | Install **Docker Desktop** from docker.com |
| Ubuntu / Debian | `curl -fsSL https://get.docker.com \| sh` |
| Fedora / RHEL | `sudo dnf install docker-ce docker-ce-cli` |

### Verify the Installation

```bash
docker --version              # check Docker CLI version
docker info                   # detailed system + engine info
docker run hello-world        # pulls and runs a test container
```

### Run Docker Without sudo (Linux)

```bash
sudo usermod -aG docker $USER
# log out and back in for this to take effect
```

> 📝 **Note:** On Windows/macOS, Docker Desktop runs a lightweight Linux VM in the background so containers work identically across platforms.

### Post-Install Checklist

1. **Daemon running?** → `docker info` returns system details without errors
2. **Can pull images?** → `docker pull hello-world` downloads successfully
3. **Can run containers?** → `docker run hello-world` prints a welcome message
4. **Non-root access works?** (Linux) → `docker ps` runs without needing `sudo`

### Common Installation Issues

| Symptom | Likely Fix |
|---|---|
| "Cannot connect to the Docker daemon" | Start Docker Desktop, or `sudo systemctl start docker` on Linux |
| "Permission denied" on docker.sock | Add your user to the `docker` group and re-login |
| `docker` command not found | Re-open your terminal, or re-check your PATH |
| Slow performance on macOS/Windows | Increase CPU/memory allocated in Docker Desktop settings |

---

## 3. Docker Images — Pulling & Inspecting

### Pull an Image From Docker Hub

```bash
docker pull ubuntu               # latest tag by default
docker pull node:20-alpine       # specific version + lightweight variant
docker pull nginx:latest
```

### List & Inspect Images

```bash
docker images                    # list all local images
docker image ls                  # same as above, newer syntax
docker inspect <image-id>        # full metadata as JSON
docker history <image-id>        # see each build layer
```

### Remove Images

```bash
docker rmi <image-id>            # remove one image
docker image prune               # remove unused (dangling) images
```

### Search Docker Hub From the CLI

```bash
docker search nginx                        # search Docker Hub by keyword
docker search nginx --limit 5              # limit number of results
docker search nginx --filter stars=100     # only well-starred images
```

### Understanding Image Tags

| Reference | Meaning |
|---|---|
| `ubuntu` | Shorthand for `ubuntu:latest` |
| `ubuntu:22.04` | A specific, pinned version |
| `ubuntu@sha256:abcd...` | An exact, immutable image digest |

> 💡 **Tip:** Avoid relying on `:latest` in production — it can silently point to a different image tomorrow. Pin an explicit version tag instead.

---

## 4. Writing a Dockerfile & Building Images

A **Dockerfile** is a set of instructions Docker follows to build an image, layer by layer.

### Example Dockerfile (Node.js App)

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

### What Each Instruction Does

| Instruction | Purpose |
|---|---|
| `FROM` | Base image to build on top of |
| `WORKDIR` | Sets the working directory inside the container |
| `COPY` | Copies files from your machine into the image |
| `RUN` | Executes a command while building the image |
| `EXPOSE` | Documents which port the app listens on |
| `CMD` | Default command to run when the container starts |

### Build an Image From a Dockerfile

```bash
docker build -t my-app:1.0 .          # -t tags the image; . is the build context
docker build -t my-app:latest -f Dockerfile.prod .   # custom Dockerfile name
```

> 💡 **Tip:** Add a `.dockerignore` file (same idea as `.gitignore`) to exclude `node_modules/`, `.git/`, and secrets from the build context.

---

## 5. Running Containers

### Basic Run

```bash
docker run my-app:1.0                 # run in foreground
docker run -d my-app:1.0              # run detached (in background)
docker run --name my-container my-app:1.0   # give it a custom name
```

### Port Mapping

Containers are isolated by default — you must explicitly map a container's port to a port on your host machine.

```bash
docker run -d -p 8080:3000 my-app:1.0
# host port 8080  ->  container port 3000
```

### Environment Variables

```bash
docker run -d -e NODE_ENV=production -e PORT=3000 my-app:1.0
docker run -d --env-file .env my-app:1.0
```

### Mounting Volumes

```bash
docker run -d -v $(pwd):/app my-app:1.0                # bind mount (live sync)
docker run -d -v my-data-volume:/app/data my-app:1.0   # named volume
```

### Interactive Shell Inside a Container

```bash
docker run -it ubuntu bash          # start container + open a shell
docker exec -it my-container bash   # open a shell in an ALREADY running container
```

---

## 6. Managing Running Containers

### List Containers

```bash
docker ps                     # running containers only
docker ps -a                  # ALL containers, including stopped
```

### Start, Stop, Restart

```bash
docker stop my-container
docker start my-container
docker restart my-container
docker pause my-container
docker unpause my-container
```

### Logs & Debugging

```bash
docker logs my-container              # view logs
docker logs -f my-container           # follow logs live (like tail -f)
docker stats                          # live CPU/memory usage of containers
docker top my-container               # running processes inside a container
```

### Remove Containers

```bash
docker rm my-container                 # remove a stopped container
docker rm -f my-container               # force-remove a running container
docker container prune                  # remove all stopped containers
```

> 💡 **Good habit:** Run `docker ps -a` before `docker container prune` so you know exactly what you're about to delete.

---

## 7. Volumes & Networking

### Why Volumes Matter

Containers are ephemeral — when a container is removed, its internal filesystem changes are lost. **Volumes** let data persist independently of any single container's lifecycle.

```bash
docker volume create my-data-volume
docker volume ls
docker volume inspect my-data-volume
docker volume rm my-data-volume
```

### Docker Networks

By default, containers on the same custom network can reach each other **by container name**.

```bash
docker network create my-network
docker network ls
docker run -d --network my-network --name db postgres
docker run -d --network my-network --name api my-app:1.0
# 'api' container can now reach the database at host "db"
```

### Common Network Drivers

| Driver | Use Case |
|---|---|
| `bridge` | Default — an isolated private network on a single host |
| `host` | Container shares the host's network stack directly (no isolation) |
| `none` | Completely disables networking for the container |
| `overlay` | Connects containers across multiple Docker hosts (Swarm mode) |

### Volume Mount Types at a Glance

| Type | Example Flag | Best For |
|---|---|---|
| Named volume | `-v my-data:/app/data` | Persistent data managed by Docker |
| Bind mount | `-v $(pwd):/app` | Live-syncing local source during development |
| tmpfs mount | `--tmpfs /app/cache` | Temporary, in-memory data that shouldn't persist |

### Putting It Together — a Database Container

```bash
docker network create app-network
docker volume create pg-data

docker run -d \
  --name postgres-db \
  --network app-network \
  -v pg-data:/var/lib/postgresql/data \
  -e POSTGRES_PASSWORD=secret \
  postgres:16
```

The database's data now survives container restarts and removals (thanks to the volume), and any other container on `app-network` can reach it simply by using the hostname `postgres-db`.

---

## 8. Docker Compose (Multi-Container Apps)

Docker Compose lets you define and run multi-container applications using a single YAML file, instead of typing long `docker run` commands for each service.

### Example docker-compose.yml

```yaml
version: "3.9"
services:
  api:
    build: .
    ports:
      - "8080:3000"
    environment:
      - NODE_ENV=production
    depends_on:
      - db
  db:
    image: postgres:16
    environment:
      - POSTGRES_PASSWORD=secret
    volumes:
      - db-data:/var/lib/postgresql/data
volumes:
  db-data:
```

### Compose CLI Commands

```bash
docker compose up                  # start all services (foreground)
docker compose up -d               # start all services (detached)
docker compose down                # stop and remove containers/network
docker compose ps                  # list running services
docker compose logs -f             # follow logs for all services
docker compose build               # rebuild images
docker compose restart api         # restart one service
```

> 📝 **Note:** Modern Docker uses `docker compose` (space) built into the CLI. The older standalone `docker-compose` (hyphen) tool still works the same way.

---

## 9. Docker Hub & Registries

### Log In, Tag, Push & Pull

```bash
docker login

docker tag my-app:1.0 yourusername/my-app:1.0

docker push yourusername/my-app:1.0
docker pull yourusername/my-app:1.0
```

### Using a Private Registry

```bash
docker login myregistry.example.com
docker tag my-app:1.0 myregistry.example.com/my-app:1.0
docker push myregistry.example.com/my-app:1.0
```

### Image Naming Convention

```
[registry-host]/[namespace-or-username]/[repository]:[tag]

# Examples:
nginx:latest                              # official image, Docker Hub, implied registry
yourusername/my-app:1.0                   # your Docker Hub account
myregistry.example.com/team/my-app:1.0    # private/self-hosted registry
```

### Automating Login in CI/CD

Never hardcode credentials in scripts — pass them as environment variables from your CI provider's secret store.

```bash
echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
```

> 📝 **Note:** Docker Hub's free tier applies pull-rate limits for anonymous/unauthenticated pulls — run `docker login` even for public images if you're hitting rate limits.

---

## 10. Cleaning Up & System Commands

Docker keeps stopped containers, unused images, build cache, and orphaned volumes around by default, since they can be reused later. Over time this consumes disk space.

### Check Disk Usage

```bash
docker system df               # summary: images, containers, volumes, cache
docker system df -v            # verbose — per-item breakdown
```

### Targeted Cleanup (Safer)

```bash
docker container prune          # remove all stopped containers
docker image prune              # remove dangling (untagged) images only
docker volume prune             # remove unused volumes
docker network prune            # remove unused networks
docker builder prune            # clear the build cache
```

### Broad Cleanup (More Aggressive)

```bash
docker system prune                 # remove unused containers/images/networks
docker system prune -a              # also remove ALL unused images, not just dangling
docker system prune --volumes       # also remove unused volumes
docker system prune -a --volumes    # everything unused, all at once
```

> ⚠️ **Danger:** `docker system prune -a --volumes` deletes everything not actively in use — including images and volumes you may still need. Always review `docker system df` first.

---

## 11. Docker CLI Cheat Sheet

**Images**

| Command | Purpose |
|---|---|
| `docker pull <image>` | Download an image |
| `docker build -t <name> .` | Build an image from a Dockerfile |
| `docker images` | List local images |
| `docker rmi <image>` | Remove an image |

**Containers**

| Command | Purpose |
|---|---|
| `docker run -d -p 8080:80 <image>` | Run a container, detached, port-mapped |
| `docker ps -a` | List all containers |
| `docker stop / start <name>` | Stop / start a container |
| `docker exec -it <name> bash` | Open a shell in a running container |
| `docker logs -f <name>` | Follow container logs |
| `docker rm -f <name>` | Force-remove a container |

**Compose & Registry**

| Command | Purpose |
|---|---|
| `docker compose up -d` | Start multi-container app |
| `docker compose down` | Stop and remove it |
| `docker tag <img> user/img` | Tag for a registry |
| `docker push user/img` | Upload to a registry |

---
---

# PART 2 — KUBERNETES

## 12. What Is Kubernetes? How It Uses Docker

**Kubernetes** (often shortened to "K8s") is a container **orchestration** platform. Where Docker runs one container on one machine, Kubernetes automatically manages many containers, across many machines, at scale.

### Where Docker Fits Into Kubernetes

Kubernetes doesn't replace Docker — it **builds on top of** the same container concept from Part 1. Docker (or a compatible runtime) is the engine that actually creates and runs each container on a machine. Kubernetes is the layer above it that decides:

- **Which machine** each container should run on
- **How many copies** of a container should be running at once
- **What happens** if a container crashes (Kubernetes restarts it automatically)
- **How containers find and talk to each other** across machines
- **How to roll out updates** without downtime, and roll them back if something breaks

> 🔑 **Key relationship:** You still build your application image with `docker build`, exactly as in Part 1. Kubernetes then takes that *same image* and decides where — and how many times — to run it as containers, across a whole cluster of machines instead of just one.

### A Note on Container Runtimes

Modern Kubernetes talks to container runtimes through a standard interface called the **CRI (Container Runtime Interface)**. Most clusters today use **containerd** (which Docker itself is built on) under the hood, rather than the full Docker Engine directly — but the images you build with `docker build` are standard OCI-format images, so they run on Kubernetes exactly the same either way. **Your Docker workflow from Part 1 does not change.**

### Docker vs. Kubernetes — Quick Comparison

| | Docker | Kubernetes |
|---|---|---|
| Scope | One container, one machine | Many containers, many machines |
| Self-healing | No — you restart manually | Yes — automatic |
| Scaling | Manual, per-container | Declarative, automatic |
| Update strategy | You swap containers by hand | Rolling updates built-in |
| Networking across hosts | Not built-in | Built-in via Services/DNS |

---

## 13. Kubernetes Architecture

A Kubernetes **cluster** is a set of machines (nodes) working together, split into two roles.

### Control Plane (the brain)

- **API Server** — the front door; every `kubectl` command talks to this
- **Scheduler** — decides which node a new container should run on
- **Controller Manager** — keeps actual state matching desired state (e.g., restarts crashed containers)
- **etcd** — the cluster's database, storing all configuration and state

### Worker Nodes (where your app actually runs)

- **Kubelet** — the agent on each node that talks to the control plane and starts/stops containers
- **Container runtime** (e.g., containerd) — actually runs the containers
- **Kube-proxy** — handles networking rules so traffic reaches the right container

### Core Objects You Will Use

| Object | What It Is |
|---|---|
| Pod | The smallest deployable unit — one or more containers that share storage/network |
| Deployment | Manages a set of identical Pods and handles updates/rollbacks |
| Service | A stable network address that routes traffic to a set of Pods |
| Namespace | A virtual cluster used to divide resources logically |
| ConfigMap / Secret | Externalized configuration and sensitive data for Pods |

### What Happens When You Run `kubectl apply`

1. **You run a command** — e.g. `kubectl apply -f deployment.yaml` from your terminal
2. **kubectl talks to the API Server** — sending your desired state as a request over HTTPS
3. **The API Server validates and stores it** — writing the desired state into etcd
4. **The Scheduler assigns Pods to nodes** — picking machines with enough free CPU/memory
5. **Each node's Kubelet creates the containers** — calling the container runtime (e.g., containerd) to actually start them. *This is the exact point where Kubernetes is directly using the same container technology as Docker.*
6. **The Controller Manager keeps watching** — continuously comparing actual state to desired state, fixing any drift automatically

> 🔑 **Key idea:** You never tell Kubernetes *which* node to use — you declare the desired end state, and the control plane continuously works to make reality match it. This is called the **reconciliation loop**.

---

## 14. Installing kubectl & Starting a Local Cluster

### Install kubectl (the Kubernetes CLI)

| Operating System | Command |
|---|---|
| macOS | `brew install kubectl` |
| Ubuntu / Debian | `sudo apt-get install -y kubectl` |
| Windows | `choco install kubernetes-cli` |

```bash
kubectl version --client      # verify installation
```

### Start a Local Cluster

You don't need a cloud account to learn Kubernetes. **Minikube** or **kind** (Kubernetes IN Docker) spin up a real cluster on your own machine — using Docker itself to simulate the nodes.

```bash
# Option A: Minikube
brew install minikube          # or the equivalent for your OS
minikube start
minikube status

# Option B: kind (runs cluster nodes as Docker containers)
brew install kind
kind create cluster --name dev-cluster
```

> 📝 **Note:** `kind` literally runs each Kubernetes node as a Docker container — a direct, visible example of Kubernetes building on top of Docker.

---

## 15. Core kubectl Commands

### Cluster Info

```bash
kubectl cluster-info
kubectl get nodes
kubectl get nodes -o wide          # more detail
```

### The Universal Pattern: get, describe, apply, delete

```bash
kubectl get <resource>                # list resources (pods, services, etc.)
kubectl describe <resource> <name>    # detailed info + recent events
kubectl apply -f <file.yaml>          # create or update from a YAML file
kubectl delete <resource> <name>      # remove a resource
```

### Examples

```bash
kubectl get pods
kubectl get pods -o wide
kubectl get all                       # pods, services, deployments, etc.
kubectl describe pod my-pod
kubectl logs my-pod                   # view container logs inside a pod
kubectl logs -f my-pod                # follow logs live
kubectl exec -it my-pod -- bash       # shell into a running pod
```

> 💡 **Tip:** `kubectl exec -it <pod> -- bash` is the Kubernetes equivalent of `docker exec -it <container> bash` from Part 1 — same idea, one layer up.

---

## 16. Pods, Deployments & ReplicaSets

You will almost never create a Pod directly in real use — instead, you create a **Deployment**, and Kubernetes creates and manages the Pods for you.

### Example Deployment YAML

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
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
        - name: my-app
          image: yourusername/my-app:1.0
          ports:
            - containerPort: 3000
```

Notice `image: yourusername/my-app:1.0` — this is the **exact same image** you built with `docker build` and pushed with `docker push` in Part 1. Kubernetes pulls it from the registry, the same way `docker pull` would.

### Apply & Manage the Deployment

```bash
kubectl apply -f deployment.yaml
kubectl get deployments
kubectl get pods                        # 3 pods will appear, one per replica
kubectl get replicasets
kubectl delete -f deployment.yaml       # tears down deployment + its pods
```

> 💡 **Why replicas matter:** If a Pod crashes or its node goes down, the Deployment's ReplicaSet automatically creates a replacement — this is Kubernetes' self-healing behavior, something plain Docker does not do on its own.

---

## 17. Services & Networking

Pods are ephemeral — they get new internal IP addresses whenever they're recreated. A **Service** gives a stable name and address that always routes to the current, healthy Pods.

### Example Service YAML

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app
  ports:
    - port: 80
      targetPort: 3000
  type: ClusterIP
```

### Service Types

| Type | Use Case |
|---|---|
| `ClusterIP` | Default — reachable only from inside the cluster |
| `NodePort` | Exposes the service on a static port on every node |
| `LoadBalancer` | Provisions an external cloud load balancer (cloud providers only) |

### Apply & Access the Service

```bash
kubectl apply -f service.yaml
kubectl get services
kubectl port-forward service/my-app-service 8080:80   # access locally, like docker -p
```

### Service Discovery via DNS

Kubernetes runs an internal DNS server (CoreDNS) automatically. Any Pod can reach a Service by name — similar to how Docker Compose lets containers reach each other by service name.

```bash
# From inside any pod in the same namespace:
curl http://my-app-service

# From a pod in a DIFFERENT namespace, use the fully qualified name:
curl http://my-app-service.default.svc.cluster.local
```

### Exposing Services Externally with Ingress

A `LoadBalancer` Service provisions one cloud load balancer per service, which gets expensive at scale. An **Ingress** routes external HTTP(S) traffic to multiple Services through a single entry point, based on hostname or URL path.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-ingress
spec:
  rules:
    - host: myapp.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-app-service
                port:
                  number: 80
```

```bash
kubectl apply -f ingress.yaml
kubectl get ingress
```

> 📝 **Note:** Ingress requires an Ingress Controller (e.g., NGINX Ingress, Traefik) installed in the cluster — the Ingress resource itself is just a set of routing rules.

---

## 18. ConfigMaps & Secrets

These externalize configuration and sensitive values from your container image — the Kubernetes equivalent of Docker's `-e` flags and `.env` files, but managed centrally by the cluster.

### ConfigMap (Non-Sensitive Config)

```bash
kubectl create configmap app-config --from-literal=NODE_ENV=production
kubectl get configmaps
kubectl describe configmap app-config
```

### Secret (Sensitive Data)

```bash
kubectl create secret generic db-secret \
  --from-literal=DB_PASSWORD=supersecret
kubectl get secrets
```

### Using Them in a Pod

```yaml
spec:
  containers:
    - name: my-app
      image: yourusername/my-app:1.0
      envFrom:
        - configMapRef:
            name: app-config
        - secretRef:
            name: db-secret
```

> ⚠️ **Note:** Kubernetes Secrets are base64-encoded, **not encrypted**, by default. For production-grade secret handling, pair Kubernetes with a dedicated secrets manager.

---

## 19. Scaling, Rolling Updates & Rollbacks

### Manual Scaling

```bash
kubectl scale deployment my-app --replicas=5
kubectl get pods                     # now shows 5 pods
```

### Autoscaling

```bash
kubectl autoscale deployment my-app --min=2 --max=10 --cpu-percent=70
```

### Rolling Update (Deploy a New Version)

This is the Kubernetes equivalent of building a new Docker image and swapping containers — except Kubernetes replaces Pods gradually, with **zero downtime**.

```bash
kubectl set image deployment/my-app my-app=yourusername/my-app:2.0
kubectl rollout status deployment/my-app     # watch the rollout progress
```

### Rollback (Version Control for Deployments)

Just like `git revert` undoes a bad commit, Kubernetes can undo a bad rollout.

```bash
kubectl rollout history deployment/my-app          # see past revisions
kubectl rollout undo deployment/my-app             # roll back to the previous version
kubectl rollout undo deployment/my-app --to-revision=2   # roll back to a specific one
```

> 💡 **Tip:** This rolling-update / rollback system is one of Kubernetes' biggest advantages over running plain Docker containers by hand — deployments become safe and reversible by default.

---

## 20. Namespaces & Contexts

**Namespaces** let you split one cluster into isolated virtual sections — e.g., separating `dev`, `staging`, and `production` resources.

```bash
kubectl get namespaces
kubectl create namespace dev
kubectl apply -f deployment.yaml -n dev
kubectl get pods -n dev
kubectl config set-context --current --namespace=dev   # switch default namespace
```

### Contexts (Switching Between Clusters)

A **context** bundles together a cluster, a user, and a namespace — useful when you work with multiple clusters (local, staging, production).

```bash
kubectl config get-contexts
kubectl config current-context
kubectl config use-context minikube
```

### Viewing Resources Across All Namespaces

```bash
kubectl get pods --all-namespaces
kubectl get pods -A                 # shorthand for --all-namespaces
```

### Deleting a Namespace

Deleting a namespace deletes **every resource inside it**. There is no separate confirmation step, so double-check before running this against anything but a throwaway environment.

```bash
kubectl delete namespace dev
kubectl get namespaces        # confirm it's gone
```

> ⚠️ **Danger:** Never run `kubectl delete namespace` against `default`, `kube-system`, or a production namespace — it is not reversible from the CLI.

---

## 21. Kubernetes CLI Cheat Sheet

**Cluster & Nodes**

| Command | Purpose |
|---|---|
| `kubectl cluster-info` | Show cluster endpoint info |
| `kubectl get nodes` | List cluster nodes |
| `kubectl config use-context <name>` | Switch cluster context |

**Pods & Deployments**

| Command | Purpose |
|---|---|
| `kubectl apply -f file.yaml` | Create/update from YAML |
| `kubectl get pods / deployments` | List resources |
| `kubectl describe pod <name>` | Detailed info + events |
| `kubectl logs -f <pod>` | Follow pod logs |
| `kubectl exec -it <pod> -- bash` | Shell into a pod |
| `kubectl delete -f file.yaml` | Remove resources |

**Scaling & Rollouts**

| Command | Purpose |
|---|---|
| `kubectl scale deployment <n> --replicas=5` | Manually scale |
| `kubectl set image deployment/<n> ...` | Deploy new image version |
| `kubectl rollout status deployment/<n>` | Watch rollout progress |
| `kubectl rollout undo deployment/<n>` | Roll back to previous version |

**Services & Namespaces**

| Command | Purpose |
|---|---|
| `kubectl get services` | List services |
| `kubectl port-forward svc/<n> 8080:80` | Access a service locally |
| `kubectl get namespaces` | List namespaces |
| `kubectl apply -f file.yaml -n <ns>` | Apply within a namespace |

---

## 22. Best Practices

### Docker
- Keep images small — use Alpine-based or slim base images where possible
- Use a `.dockerignore` file to keep secrets and unnecessary files out of the build context
- Never bake secrets into an image — pass them at runtime via environment variables or secret managers
- Tag images with meaningful versions (e.g. `1.2.0`), not just `latest`, so rollbacks are possible
- Use multi-stage builds to keep production images free of build tools

### Kubernetes
- Always deploy through Deployments (or similar controllers), never bare Pods, so self-healing works
- Set resource requests/limits (CPU, memory) on every container so the scheduler places Pods correctly
- Use readiness and liveness probes so Kubernetes only routes traffic to healthy Pods
- Store YAML manifests in version control (Git) — treat cluster config the same way as code
- Use namespaces to separate environments, and RBAC to control who can do what in each
- Always check `kubectl rollout status` after a deploy, and know `kubectl rollout undo` is there if needed

---

<p align="center"><em>Docker & Kubernetes Command Reference — Compiled Guide</em></p>
