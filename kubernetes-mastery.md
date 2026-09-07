# 🚀 The Complete Kubernetes Mastery Guide
### From Zero to Hero — Practical Learning on Arch Linux with Minikube

> **How to use this guide:** Don't just read it — open a terminal and type every command as you go. Kubernetes is a "hands" skill, not a "reading" skill. Each section builds on the last, so go in order the first time through.

---

## 📖 Table of Contents

1. [Why Kubernetes Exists](#1-why-kubernetes-exists)
2. [Core Concepts You Must Know First](#2-core-concepts-you-must-know-first)
3. [Setting Up Your Arch Linux Environment](#3-setting-up-your-arch-linux-environment)
4. [Installing Minikube, kubectl & Friends](#4-installing-minikube-kubectl--friends)
5. [Starting Your First Cluster](#5-starting-your-first-cluster)
6. [kubectl Basics — Talking to Your Cluster](#6-kubectl-basics--talking-to-your-cluster)
7. [Pods — The Smallest Unit](#7-pods--the-smallest-unit)
8. [Deployments — Running Apps for Real](#8-deployments--running-apps-for-real)
9. [Services — Networking & Exposure](#9-services--networking--exposure)
10. [ConfigMaps & Secrets — Configuration](#10-configmaps--secrets--configuration)
11. [Volumes & Persistent Storage](#11-volumes--persistent-storage)
12. [Namespaces — Organizing Your Cluster](#12-namespaces--organizing-your-cluster)
13. [Scaling & Rolling Updates](#13-scaling--rolling-updates)
14. [Health Checks — Probes](#14-health-checks--probes)
15. [Jobs & CronJobs](#15-jobs--cronjobs)
16. [StatefulSets & DaemonSets](#16-statefulsets--daemonsets)
17. [Ingress — Real-World URL Routing](#17-ingress--real-world-url-routing)
18. [Helm — The Package Manager for Kubernetes](#18-helm--the-package-manager-for-kubernetes)
19. [Monitoring, Logs & Debugging](#19-monitoring-logs--debugging)
20. [RBAC & Security Basics](#20-rbac--security-basics)
21. [Custom Resources & Operators (Advanced)](#21-custom-resources--operators-advanced)
22. [Troubleshooting Cheat Sheet](#22-troubleshooting-cheat-sheet)
23. [Hands-On Practice Projects](#23-hands-on-practice-projects)
24. [Full kubectl Command Cheat Sheet](#24-full-kubectl-command-cheat-sheet)
25. [Where to Go From Here](#25-where-to-go-from-here)

---

## 1. Why Kubernetes Exists

### The problem before Kubernetes

Imagine you built an app and packaged it into a **Docker container**. That's great for one machine. But real production systems need:

| Problem | What happens without Kubernetes |
|---|---|
| A container crashes | Someone has to notice and restart it manually |
| Traffic spikes | Someone has to manually spin up more containers |
| You have 50 microservices | Managing them by hand across many servers is chaos |
| A server dies | All containers on it are gone until someone intervenes |
| You want zero-downtime updates | You'd need custom scripts to swap containers safely |
| Services need to find each other | You'd hardcode IPs that constantly change |

### What Kubernetes actually is

**Kubernetes (K8s)** is a **container orchestration system**. Think of it as an operating system for your *entire data center/cluster*, instead of for one machine.

You describe the **desired state** ("I want 3 copies of my app running, always") in a YAML file, and Kubernetes constantly works to make reality match that description. This is called the **reconciliation loop** or **control loop** — the single most important idea in all of Kubernetes.

```
   You declare:                Kubernetes constantly checks:
  "3 replicas of my app"  →   "Are there 3 running? If not, fix it."
```

### Why learn it now

- It's the industry standard for deploying software at any serious scale.
- Every major cloud (AWS EKS, GCP GKE, Azure AKS) is built around it.
- It teaches you networking, storage, and distributed systems concepts that transfer everywhere.

### Why Minikube for learning

**Minikube** creates a *real*, fully functional single-node Kubernetes cluster inside a VM or container on your own Arch Linux machine. You get the real `kubectl` experience without needing a cloud account or multiple servers.

---

## 2. Core Concepts You Must Know First

Read this section twice. Everything else in this guide is just details on top of these ideas.

| Concept | Plain-English Definition |
|---|---|
| **Cluster** | A set of machines (nodes) that Kubernetes manages as one unit |
| **Node** | A single machine (VM or physical) in the cluster that runs your workloads |
| **Pod** | The smallest deployable unit — one or more containers that share network/storage |
| **Deployment** | A blueprint that tells Kubernetes "keep N copies of this Pod running" |
| **Service** | A stable network address that routes traffic to a set of Pods |
| **Namespace** | A virtual "folder" to separate resources (e.g. dev vs prod) |
| **ConfigMap** | Non-secret configuration data injected into Pods |
| **Secret** | Sensitive configuration data (passwords, tokens) injected into Pods |
| **Volume** | Storage that a Pod can read/write, which can outlive the container |
| **Ingress** | A rule set that routes external HTTP(S) traffic to Services by hostname/path |
| **kubectl** | The command-line tool you use to talk to a Kubernetes cluster |
| **etcd** | The database Kubernetes uses internally to store all cluster state |
| **Control Plane** | The "brain" of Kubernetes — decides what should run where |
| **kubelet** | The agent on every node that actually starts/stops containers |

### The architecture picture

```
                         ┌─────────────────────────────┐
                         │        CONTROL PLANE         │
                         │  (the brain of the cluster)  │
                         │                               │
                         │  ┌───────────┐  ┌──────────┐ │
                         │  │ API Server│  │  etcd DB │ │
                         │  └───────────┘  └──────────┘ │
                         │  ┌───────────┐  ┌──────────┐ │
                         │  │ Scheduler │  │Controller│ │
                         │  └───────────┘  │ Manager  │ │
                         │                 └──────────┘ │
                         └───────────────┬───────────────┘
                                         │ (kubectl talks here)
                    ┌────────────────────┼────────────────────┐
                    │                    │                    │
             ┌──────▼──────┐      ┌──────▼──────┐      ┌──────▼──────┐
             │   NODE 1    │      │   NODE 2    │      │   NODE 3    │
             │  ┌───────┐  │      │  ┌───────┐  │      │  ┌───────┐  │
             │  │kubelet│  │      │  │kubelet│  │      │  │kubelet│  │
             │  └───┬───┘  │      │  └───┬───┘  │      │  └───┬───┘  │
             │  ┌───▼───┐  │      │  ┌───▼───┐  │      │  ┌───▼───┐  │
             │  │ Pods  │  │      │  │ Pods  │  │      │  │ Pods  │  │
             │  └───────┘  │      │  └───────┘  │      │  └───────┘  │
             └─────────────┘      └─────────────┘      └─────────────┘
```

With Minikube, **all of this runs on one node** (your machine), which is perfect for learning — the concepts are identical to a 1000-node production cluster.

---

## 3. Setting Up Your Arch Linux Environment

### 3.1 Update your system first

```bash
sudo pacman -Syu
```

### 3.2 Install a container/VM driver

Minikube needs something to run the cluster *inside*. On Arch Linux, the most reliable and lightweight option is **Docker**.

```bash
sudo pacman -S docker
sudo systemctl enable --now docker
sudo usermod -aG docker $USER
```

> ⚠️ **Important:** After running `usermod`, log out and log back in (or run `newgrp docker`) so your user picks up the `docker` group without needing `sudo` for every command.

Verify Docker works:

```bash
docker run hello-world
```

You should see a "Hello from Docker!" message.

### 3.3 (Optional) Install an AUR helper if you don't have one

Some tools are easier to grab from the AUR. If you already use `yay` or `paru`, skip this.

```bash
sudo pacman -S --needed git base-devel
git clone https://aur.archlinux.org/yay.git
cd yay && makepkg -si
```

---

## 4. Installing Minikube, kubectl & Friends

### 4.1 Install `kubectl` (the CLI you'll use constantly)

```bash
sudo pacman -S kubectl
```

Verify:
```bash
kubectl version --client
```

### 4.2 Install Minikube

Arch's official repos don't always carry the latest Minikube, so the AUR is the most current option:

```bash
yay -S minikube
```

*(Alternative without an AUR helper — download the binary directly:)*

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
rm minikube-linux-amd64
```

Verify:
```bash
minikube version
```

### 4.3 (Recommended) Install `kubectx` and `kubens`

These make switching between contexts/namespaces much faster later:

```bash
yay -S kubectx
```

### 4.4 (Recommended for Helm section later) Install Helm

```bash
sudo pacman -S helm
```

---

## 5. Starting Your First Cluster

### 5.1 Start Minikube using the Docker driver

```bash
minikube start --driver=docker
```

This downloads a small VM image (first time only) and boots a single-node Kubernetes cluster inside a Docker container. Takes a few minutes the first time.

### 5.2 Confirm it's running

```bash
minikube status
```

Expected output looks like:
```
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

### 5.3 Check kubectl is pointed at Minikube

```bash
kubectl cluster-info
kubectl get nodes
```

You should see one node, `minikube`, in `Ready` status.

### 5.4 Open the built-in dashboard (great for visual learners)

```bash
minikube dashboard
```

This opens a web UI in your browser showing everything running in your cluster. Keep this open in a tab while you learn — it's a great sanity check.

### 5.5 Useful Minikube lifecycle commands

```bash
minikube stop        # pause the cluster (keeps state)
minikube start        # resume it
minikube delete        # completely destroy it (fresh start)
minikube pause        # pause without stopping the VM
minikube ssh           # SSH into the Minikube VM itself
```

---

## 6. kubectl Basics — Talking to Your Cluster

`kubectl` follows a consistent pattern:

```
kubectl <verb> <resource> <name> [flags]
```

Examples:
```bash
kubectl get pods
kubectl describe pod my-pod
kubectl delete deployment my-app
kubectl logs my-pod
```

### The most-used verbs

| Verb | What it does |
|---|---|
| `get` | List resources |
| `describe` | Show detailed info + recent events (your #1 debugging tool) |
| `create` | Create a resource from a file or flags |
| `apply` | Create OR update a resource declaratively (the professional way) |
| `delete` | Remove a resource |
| `logs` | View container logs |
| `exec` | Run a command inside a container |
| `edit` | Open a resource in your editor to modify it live |

### `apply` vs `create` — know this early

- `kubectl create -f file.yaml` → fails if the resource already exists.
- `kubectl apply -f file.yaml` → creates it if missing, updates it if it exists.

**Always prefer `apply`.** It matches how real teams work (GitOps-style, declarative).

---

## 7. Pods — The Smallest Unit

A Pod wraps one or more containers that share a network namespace and can share storage. Most of the time, **you won't create Pods directly** — you'll use a Deployment. But you must understand them first.

### 7.1 Create your first Pod (imperative way, for learning only)

```bash
kubectl run nginx-pod --image=nginx:alpine --port=80
```

### 7.2 Inspect it

```bash
kubectl get pods
kubectl describe pod nginx-pod
kubectl logs nginx-pod
```

### 7.3 Get a shell inside the running container

```bash
kubectl exec -it nginx-pod -- sh
```

Type `exit` to leave.

### 7.4 Delete it

```bash
kubectl delete pod nginx-pod
```

### 7.5 The declarative way (the way you should actually work)

Create a file `pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
    - name: nginx
      image: nginx:alpine
      ports:
        - containerPort: 80
```

Apply it:
```bash
kubectl apply -f pod.yaml
```

> 💡 **Key habit to build now:** Every resource you make from here on should live in a `.yaml` file, tracked like code. This is how Kubernetes is used in the real world.

---

## 8. Deployments — Running Apps for Real

A **Deployment** manages a set of identical Pods (via a **ReplicaSet** underneath) and gives you self-healing, scaling, and rolling updates for free.

### 8.1 Create a Deployment YAML — `deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:alpine
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: "100m"
              memory: "64Mi"
            limits:
              cpu: "250m"
              memory: "128Mi"
```

### 8.2 Apply and observe

```bash
kubectl apply -f deployment.yaml
kubectl get deployments
kubectl get pods -o wide
kubectl get replicasets
```

You'll see 3 Pods, all owned by one ReplicaSet, owned by the Deployment.

### 8.3 Prove self-healing

Delete one Pod manually and watch Kubernetes replace it instantly:

```bash
kubectl delete pod <one-of-the-pod-names>
kubectl get pods -w
```

The `-w` (watch) flag streams live changes — you'll see a new Pod appear within seconds. **This is the core magic of Kubernetes.**

---

## 9. Services — Networking & Exposure

Pods are ephemeral — their IPs change constantly. A **Service** gives you a stable name and IP that automatically load-balances across matching Pods.

### 9.1 Service types

| Type | Use case |
|---|---|
| `ClusterIP` (default) | Internal-only access, other Pods in the cluster can reach it |
| `NodePort` | Exposes a port on the node itself — good for local testing |
| `LoadBalancer` | Cloud provider provisions an external load balancer (not needed on Minikube directly, but Minikube can simulate it) |

### 9.2 Expose the Deployment — `service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
```

Apply:
```bash
kubectl apply -f service.yaml
kubectl get services
```

### 9.3 Access it from your browser

Minikube makes this trivial:

```bash
minikube service nginx-service
```

This opens your default browser pointed at the correct IP:port automatically. You should see the Nginx welcome page.

### 9.4 The `selector` connection — this is critical to understand

A Service finds Pods purely by matching **labels**. Notice:

```yaml
# In the Deployment's pod template:
labels:
  app: nginx

# In the Service:
selector:
  app: nginx
```

If these labels don't match, the Service will have **zero endpoints** and route nowhere. This is the #1 beginner mistake — always check labels first when a Service isn't working.

```bash
kubectl get endpoints nginx-service
```

---

## 10. ConfigMaps & Secrets — Configuration

Never hardcode configuration or passwords into your container images. Kubernetes gives you two objects for this.

### 10.1 ConfigMap example — `configmap.yaml`

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_MODE: "production"
  MAX_CONNECTIONS: "100"
```

### 10.2 Secret example — `secret.yaml`

Secrets store values Base64-encoded (NOT encrypted by default — treat with care):

```bash
echo -n 'supersecretpassword' | base64
```

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  DB_PASSWORD: c3VwZXJzZWNyZXRwYXNzd29yZA==
```

### 10.3 Use both inside a Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: config-demo
spec:
  containers:
    - name: demo
      image: busybox
      command: ["sleep", "3600"]
      envFrom:
        - configMapRef:
            name: app-config
        - secretRef:
            name: app-secret
```

Apply everything and verify the environment variables landed inside the container:

```bash
kubectl apply -f configmap.yaml -f secret.yaml -f pod-config-demo.yaml
kubectl exec -it config-demo -- env | grep -E "APP_MODE|DB_PASSWORD"
```

---

## 11. Volumes & Persistent Storage

Containers are stateless by default — data disappears when a Pod dies. **Volumes** solve this.

### 11.1 Key storage objects

| Object | Purpose |
|---|---|
| `Volume` | Storage attached directly to a Pod's lifecycle |
| `PersistentVolume (PV)` | A piece of real storage in the cluster (disk) |
| `PersistentVolumeClaim (PVC)` | A Pod's "request" for storage, bound to a PV |
| `StorageClass` | A template for dynamically creating PVs on demand |

### 11.2 Minikube already ships a default StorageClass

```bash
kubectl get storageclass
```

### 11.3 Create a PVC — `pvc.yaml`

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
      storage: 1Gi
```

### 11.4 Mount it in a Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: storage-demo
spec:
  containers:
    - name: writer
      image: busybox
      command: ["sh", "-c", "echo Hello Persistent World > /data/hello.txt && sleep 3600"]
      volumeMounts:
        - mountPath: /data
          name: my-storage
  volumes:
    - name: my-storage
      persistentVolumeClaim:
        claimName: my-pvc
```

Apply, then verify the file survives a Pod restart:

```bash
kubectl apply -f pvc.yaml -f storage-demo.yaml
kubectl exec storage-demo -- cat /data/hello.txt
kubectl delete pod storage-demo
kubectl apply -f storage-demo.yaml
kubectl exec storage-demo -- cat /data/hello.txt   # still there!
```

---

## 12. Namespaces — Organizing Your Cluster

Namespaces let you separate resources logically (e.g., `dev`, `staging`, `prod`) within one cluster.

```bash
kubectl get namespaces
kubectl create namespace dev
kubectl apply -f deployment.yaml -n dev
kubectl get pods -n dev
```

### Switch your default namespace (using kubens, installed earlier)

```bash
kubens dev      # everything now defaults to the dev namespace
kubens default  # switch back
```

### View resources across all namespaces

```bash
kubectl get pods --all-namespaces
```

---

## 13. Scaling & Rolling Updates

### 13.1 Manual scaling

```bash
kubectl scale deployment nginx-deployment --replicas=5
kubectl get pods
```

### 13.2 Rolling updates — zero downtime deploys

Change the image version and re-apply:

```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.25-alpine
kubectl rollout status deployment/nginx-deployment
```

Kubernetes replaces Pods **gradually**, never taking all replicas down at once.

### 13.3 Rollback if something breaks

```bash
kubectl rollout history deployment/nginx-deployment
kubectl rollout undo deployment/nginx-deployment
```

### 13.4 Autoscaling (based on CPU)

Requires the metrics-server addon:

```bash
minikube addons enable metrics-server
kubectl autoscale deployment nginx-deployment --cpu-percent=50 --min=2 --max=10
kubectl get hpa
```

---

## 14. Health Checks — Probes

Kubernetes can automatically detect broken containers and act on it, but only if you tell it how to check.

| Probe | Purpose |
|---|---|
| `livenessProbe` | "Is this container still alive?" — restarts it if it fails |
| `readinessProbe` | "Is this container ready for traffic?" — removes it from Service routing if it fails |
| `startupProbe` | "Has this slow-starting container finished booting?" |

```yaml
livenessProbe:
  httpGet:
    path: /
    port: 80
  initialDelaySeconds: 5
  periodSeconds: 10
readinessProbe:
  httpGet:
    path: /
    port: 80
  initialDelaySeconds: 5
  periodSeconds: 5
```

Add this under `containers:` in your Deployment YAML and re-apply. Watch `kubectl describe pod` to see probe results in the Events section.

---

## 15. Jobs & CronJobs

Not everything runs forever — sometimes you need a task that runs once and finishes, or on a schedule.

### 15.1 A one-off Job — `job.yaml`

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: hello-job
spec:
  template:
    spec:
      containers:
        - name: hello
          image: busybox
          command: ["echo", "Hello from a Kubernetes Job"]
      restartPolicy: Never
```

```bash
kubectl apply -f job.yaml
kubectl get jobs
kubectl logs job/hello-job
```

### 15.2 A scheduled CronJob — `cronjob.yaml`

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello-cron
spec:
  schedule: "*/2 * * * *"    # every 2 minutes
  jobTemplate:
    spec:
      template:
        spec:
          containers:
            - name: hello
              image: busybox
              command: ["date"]
          restartPolicy: OnFailure
```

```bash
kubectl apply -f cronjob.yaml
kubectl get cronjobs
kubectl get jobs --watch
```

---

## 16. StatefulSets & DaemonSets

### 16.1 StatefulSet — for apps that need stable identity (databases, queues)

Unlike Deployments, each Pod gets a **stable, predictable name** (`app-0`, `app-1`...) and its own persistent storage that survives rescheduling.

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "web"
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: nginx
          image: nginx:alpine
```

Use this for things like PostgreSQL, MongoDB, Kafka — anything where "which instance is which" matters.

### 16.2 DaemonSet — one Pod per node, always

Useful for log collectors, monitoring agents, or networking tools that must run everywhere.

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-logger
spec:
  selector:
    matchLabels:
      name: node-logger
  template:
    metadata:
      labels:
        name: node-logger
    spec:
      containers:
        - name: logger
          image: busybox
          command: ["sh", "-c", "while true; do echo I am on this node; sleep 30; done"]
```

On a single-node Minikube cluster, this looks identical to a normal Pod — but on a real multi-node cluster, one copy runs on *every* node automatically.

---

## 17. Ingress — Real-World URL Routing

`NodePort` is fine for testing, but real applications route traffic by **hostname and path** using an **Ingress**. This is how `example.com/api` and `example.com/app` can point to different Services on port 80/443.

### 17.1 Enable the Ingress addon in Minikube

```bash
minikube addons enable ingress
```

### 17.2 Create an Ingress — `ingress.yaml`

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
    - host: nginx.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: nginx-service
                port:
                  number: 80
```

### 17.3 Point your local DNS at Minikube

```bash
echo "$(minikube ip) nginx.local" | sudo tee -a /etc/hosts
```

Apply and test:
```bash
kubectl apply -f ingress.yaml
curl http://nginx.local
```

---

## 18. Helm — The Package Manager for Kubernetes

Writing raw YAML for complex apps (with dozens of resources) gets tedious. **Helm** packages Kubernetes resources into reusable, configurable "charts" — think of it like `pacman`, but for Kubernetes apps.

### 18.1 Add a chart repository

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

### 18.2 Install something real — e.g. WordPress

```bash
helm install my-wordpress bitnami/wordpress
kubectl get pods
```

### 18.3 Useful Helm commands

```bash
helm list                     # see installed releases
helm status my-wordpress      # check a release
helm upgrade my-wordpress bitnami/wordpress --set replicaCount=2
helm uninstall my-wordpress   # remove it completely
```

### 18.4 Create your own chart (advanced habit worth building)

```bash
helm create mychart
```

This scaffolds a full chart structure (`templates/`, `values.yaml`, `Chart.yaml`) you can customize — this is how professional teams package internal apps.

---

## 19. Monitoring, Logs & Debugging

### 19.1 The debugging trio — memorize this order

```bash
kubectl get pods                 # 1. Is it even running?
kubectl describe pod <name>      # 2. What do the Events say?
kubectl logs <name>              # 3. What is the app itself saying?
```

### 19.2 Common flags that save time

```bash
kubectl logs <pod> --previous          # logs from a crashed container's last run
kubectl logs <pod> -f                  # follow logs live (like tail -f)
kubectl logs <pod> -c <container-name> # multi-container pod: pick one
kubectl get pods -o wide               # see node + IP assignment
kubectl top pods                       # CPU/memory usage (needs metrics-server)
kubectl top nodes
```

### 19.3 Get a full YAML dump of any live resource

```bash
kubectl get deployment nginx-deployment -o yaml
```

This is incredibly useful for learning — you see every default field Kubernetes filled in that you didn't write yourself.

### 19.4 Events across the whole cluster

```bash
kubectl get events --sort-by=.metadata.creationTimestamp
```

### 19.5 Dashboard reminder

```bash
minikube dashboard
```

Keep this open while learning — visually watching Pods appear/disappear cements the mental model fast.

---

## 20. RBAC & Security Basics

**Role-Based Access Control (RBAC)** governs *who* can do *what* in your cluster.

### 20.1 Key objects

| Object | Scope | Purpose |
|---|---|---|
| `Role` | Single namespace | Defines permissions |
| `ClusterRole` | Whole cluster | Defines permissions cluster-wide |
| `RoleBinding` | Single namespace | Grants a Role to a user/group/service account |
| `ClusterRoleBinding` | Whole cluster | Grants a ClusterRole cluster-wide |

### 20.2 Example — read-only access to Pods in one namespace

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: dev
  name: pod-reader
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "watch", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: dev
subjects:
  - kind: User
    name: some-user
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

### 20.3 Service Accounts — identity for Pods

Every Pod runs as a **ServiceAccount** (default one if unspecified). Applications running inside Pods use this identity to talk to the Kubernetes API itself.

```bash
kubectl create serviceaccount my-app-sa
kubectl get serviceaccounts
```

### 20.4 Security basics worth practicing

- Never store real passwords in plain ConfigMaps — always use Secrets.
- Set resource `limits` on every container to avoid one Pod starving the node.
- Use `readinessProbe` so broken Pods never receive live traffic.
- Avoid running containers as root when possible (`securityContext.runAsNonRoot: true`).

---

## 21. Custom Resources & Operators (Advanced)

This is where Kubernetes becomes a true platform, not just a container runner.

### 21.1 Custom Resource Definitions (CRDs)

A CRD lets you teach Kubernetes about a *brand new kind of object* beyond the built-ins (Pod, Service, etc.). For example, a database vendor might define a `PostgresCluster` resource.

```bash
kubectl get crds
```

### 21.2 Operators

An **Operator** is a controller (a program) that watches a Custom Resource and takes real action to reconcile it — e.g., a `PostgresOperator` watches for `PostgresCluster` objects and actually provisions, backs up, and heals real Postgres clusters automatically.

**Mental model:** Deployments/Services are built-in "operators" for generic apps. A custom Operator is the same idea, purpose-built for one specific piece of software (Postgres, Kafka, Elasticsearch, etc.).

### 21.3 Try one hands-on: install an Operator via Helm

```bash
helm repo add cnpg https://cloudnative-pg.github.io/charts
helm install cnpg cnpg/cloudnative-pg
kubectl get pods -n default
```

This installs a real, production-grade Postgres Operator. Explore its CRDs:

```bash
kubectl get crd | grep postgresql
```

---

## 22. Troubleshooting Cheat Sheet

| Symptom | Likely cause | Command to check |
|---|---|---|
| Pod stuck in `Pending` | Not enough resources, or unbound PVC | `kubectl describe pod <name>` |
| Pod stuck in `CrashLoopBackOff` | App crashes on startup | `kubectl logs <name> --previous` |
| Pod stuck in `ImagePullBackOff` | Wrong image name/tag, or no internet | `kubectl describe pod <name>` |
| Service has no traffic | Label selector mismatch | `kubectl get endpoints <svc>` |
| `connection refused` from curl | Wrong port, or app not listening yet | `kubectl port-forward` and test locally |
| Changes don't seem to apply | Forgot to re-apply, or wrong namespace | `kubectl get all -n <ns>` |
| Minikube won't start | Docker not running, or resources too low | `minikube start --alsologtostderr -v=7` |

### Port-forward for quick local debugging (bypasses Services entirely)

```bash
kubectl port-forward pod/nginx-pod 8080:80
# then in browser: localhost:8080
```

---

## 23. Hands-On Practice Projects

Do these **in order**. Each one forces you to combine multiple concepts above.

### 🥉 Project 1 — Beginner: Static website
Deploy an Nginx Deployment (3 replicas) serving a custom `index.html` via a ConfigMap mounted as a volume. Expose it with a NodePort Service.

### 🥈 Project 2 — Intermediate: Two-tier app
Deploy a simple backend (e.g. a small Python/Node API in your own image) + a Service for it, plus a frontend Deployment that calls the backend by its **Service DNS name** (`http://backend-service.default.svc.cluster.local`). This teaches internal service discovery.

### 🥇 Project 3 — Advanced: Stateful app with Ingress
Deploy a StatefulSet running Postgres (or use the CloudNativePG operator from Section 21), a Secret for its password, a PVC for data, and an Ingress routing `myapp.local` to a frontend that reads/writes to that database.

### 🏆 Project 4 — Master: Full CI-style workflow
1. Write all manifests as Helm chart templates.
2. Add a `livenessProbe`, `readinessProbe`, resource `limits`, and an HPA.
3. Simulate a rolling update and a rollback.
4. Add RBAC so a "read-only" ServiceAccount can only `get`/`list` Pods, and prove it with `kubectl auth can-i`:
```bash
kubectl auth can-i delete pods --as=system:serviceaccount:default:my-app-sa
```

---

## 24. Full kubectl Command Cheat Sheet

```bash
# Cluster info
kubectl cluster-info
kubectl get nodes
kubectl version

# Viewing resources
kubectl get pods|deployments|services|all -n <namespace>
kubectl get pods --all-namespaces
kubectl get pods -o wide
kubectl get pods -o yaml
kubectl describe <resource> <name>

# Creating / updating
kubectl apply -f file.yaml
kubectl apply -f ./manifests/          # apply a whole folder
kubectl delete -f file.yaml
kubectl edit deployment <name>

# Scaling & updates
kubectl scale deployment <name> --replicas=5
kubectl set image deployment/<name> <container>=<image>:<tag>
kubectl rollout status deployment/<name>
kubectl rollout undo deployment/<name>
kubectl rollout history deployment/<name>

# Debugging
kubectl logs <pod> [-f] [--previous] [-c container]
kubectl exec -it <pod> -- sh
kubectl port-forward pod/<pod> 8080:80
kubectl top pods
kubectl top nodes
kubectl get events --sort-by=.metadata.creationTimestamp

# Namespaces
kubectl create namespace <name>
kubectl config set-context --current --namespace=<name>

# Cleanup
kubectl delete pod <name>
kubectl delete deployment <name>
kubectl delete -f file.yaml
kubectl delete namespace <name>   # deletes everything inside it too!

# Minikube-specific
minikube start --driver=docker
minikube stop
minikube delete
minikube status
minikube dashboard
minikube service <svc-name>
minikube addons list
minikube addons enable <addon>
minikube ip
minikube ssh
```

---

## 25. Where to Go From Here

Once you're comfortable with everything above, these are the natural next steps:

- **Kustomize** — a native alternative to Helm for managing environment-specific config overlays.
- **GitOps tools** — ArgoCD or FluxCD, which auto-sync your cluster to a Git repository.
- **Service meshes** — Istio or Linkerd, for advanced traffic control, mTLS, and observability.
- **Multi-node practice** — try `minikube start --nodes=3` to simulate a real multi-node cluster locally.
- **Certified Kubernetes Administrator (CKA)** — the industry-recognized certification; everything in this guide covers a large portion of its curriculum.
- **Read real Helm charts** on [ArtifactHub](https://artifacthub.io) to see how production-grade apps are packaged.

---

### 🎯 Final Tip

The fastest way to actually learn Kubernetes is to **break things on purpose** in this Minikube cluster and then fix them using Section 22's troubleshooting flow. You cannot break anything that matters — that's the entire point of learning on Minikube. Delete Pods, corrupt a Service selector, scale to zero, and practice bringing it all back.

```bash
minikube delete   # your undo button for literally everything
minikube start --driver=docker
```

Good luck — you now have everything you need to go from typing your first `kubectl get pods` to running a self-healing, auto-scaling, secured mini production system on your own Arch Linux machine.
