# 📚 Complete Kubernetes Learning Guide: Beginner to Advanced

I'll create a comprehensive guide covering everything from basics to advanced concepts, plus the differences between Kubernetes deployment tools.

---

## 🎯 Table of Contents

1. **Kubernetes Fundamentals**
2. **Core Concepts & Objects**
3. **Deployment Tools (kind vs Minikube vs kops vs kubeadm)**
4. **Networking**
5. **Storage**
6. **Advanced Topics**
7. **Best Practices**

---

# 📖 PART 1: Kubernetes Fundamentals

## 🏗️ What is Kubernetes?

```
Kubernetes = Container Orchestration Platform

┌─────────────────────────────────────────────────────────┐
│                  Kubernetes                              │
│  Automatically manages containerized applications        │
│                                                          │
│  Features:                                               │
│  ├─ Container deployment                                │
│  ├─ Scaling                                             │
│  ├─ Networking                                          │
│  ├─ Storage management                                  │
│  ├─ Self-healing                                        │
│  ├─ Load balancing                                      │
│  └─ Rolling updates                                     │
└─────────────────────────────────────────────────────────┘
```

### Why Kubernetes?

```
Problems it solves:

1. Container Management
   ├─ Run 1000s of containers
   ├─ Automatically schedule them
   └─ Keep them running

2. High Availability
   ├─ Self-healing (restart failed pods)
   ├─ Multi-replica deployments
   └─ Health checks

3. Resource Optimization
   ├─ Bin packing (efficient resource use)
   ├─ CPU/Memory limits
   └─ Auto-scaling based on demand

4. Rolling Updates
   ├─ Zero-downtime deployments
   ├─ Automatic rollback
   └─ Canary deployments

5. Networking
   ├─ Service discovery
   ├─ Load balancing
   └─ Network policies
```

---

## 🏛️ Kubernetes Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                  KUBERNETES CLUSTER                          │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │              CONTROL PLANE (Master)                     │ │
│  │  (Manages the cluster)                                  │ │
│  │                                                          │ │
│  │  ┌────────────────┐  ┌────────────────┐                │ │
│  │  │  API Server    │  │  etcd (Database)              │ │
│  │  │ (REST API)     │  │ (Cluster state)               │ │
│  │  └────────────────┘  └────────────────┘                │ │
│  │                                                          │ │
│  │  ┌────────────────┐  ┌────────────────┐                │ │
│  │  │  Scheduler     │  │  Controller    │                │ │
│  │  │ (Pod placement)│  │  Manager       │                │ │
│  │  └────────────────┘  │ (Reconciliation)               │ │
│  │                       └────────────────┘                │ │
│  └─────────────────────────────────────────────────────────┘ │
│                           │                                  │
│        ┌──────────────────┼──────────────────┐              │
│        │                  │                  │              │
│  ┌─────▼──────┐  ┌─────────▼──────┐  ┌─────────▼──────┐    │
│  │   WORKER 1 │  │   WORKER 2     │  │   WORKER N     │    │
│  │  (Node)    │  │   (Node)       │  │   (Node)       │    │
│  │            │  │                │  │                │    │
│  │ ┌────────┐ │  │ ┌────────┐     │  │ ┌────────┐     │    │
│  │ │ kubelet│ │  │ │ kubelet│     │  │ │ kubelet│     │    │
│  │ └────────┘ │  │ └────────┘     │  │ └────────┘     │    │
│  │            │  │                │  │                │    │
│  │ ┌────────┐ │  │ ┌────────┐     │  │ ┌────────┐     │    │
│  │ │ Pod 1  │ │  │ │ Pod 3  │     │  │ │ Pod 5  │     │    │
│  │ └────────┘ │  │ └────────┘     │  │ └────────┘     │    │
│  │            │  │                │  │                │    │
│  │ ┌────────┐ │  │ ┌────────┐     │  │ ┌────────┐     │    │
│  │ │ Pod 2  │ │  │ │ Pod 4  │     │  │ │ Pod 6  │     │    │
│  │ └────────┘ │  │ └────────┘     │  │ └────────┘     │    │
│  └────────────┘  └────────────────┘  └────────────────┘    │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

### Control Plane Components

| Component | Purpose |
|-----------|---------|
| **API Server** | Central hub, exposes REST API |
| **etcd** | Distributed database storing cluster state |
| **Scheduler** | Assigns pods to nodes |
| **Controller Manager** | Runs controller processes |
| **Cloud Controller Manager** | Integrates with cloud providers |

### Node (Worker) Components

| Component | Purpose |
|-----------|---------|
| **kubelet** | Agent running on each node, manages containers |
| **Container Runtime** | Runs containers (Docker, containerd, etc.) |
| **kube-proxy** | Network proxy, manages networking |

---

# 🧩 PART 2: Core Kubernetes Objects & Concepts

## 1️⃣ Pod (Basic Unit)

**Pod** = Smallest deployable unit in Kubernetes (like a container, but can have multiple containers)

````yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
    app: web
spec:
  containers:
  - name: app
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
````

**Key Points:**
- Pods are ephemeral (temporary)
- Usually managed by higher-level objects
- Can have multiple containers (sidecar pattern)

---

## 2️⃣ ReplicaSet (Multiple Pods)

**ReplicaSet** = Ensures desired number of pods are running

````yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: web-rs
spec:
  replicas: 3           # Keep 3 pods running
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
        image: nginx:1.21
        ports:
        - containerPort: 80
````

**How it works:**
```
ReplicaSet with 3 replicas:
├─ Pod 1 ─┐
├─ Pod 2  ├─ All running the same image
├─ Pod 3 ─┘

If Pod 1 dies → ReplicaSet creates new Pod 4 automatically
If Pod 1 restarts → ReplicaSet deletes the new Pod 4
```

---

## 3️⃣ Deployment (Recommended!)

**Deployment** = ReplicaSet + Rolling Updates + Rollback

````yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
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
        image: nginx:1.21
        ports:
        - containerPort: 80
        livenessProbe:          # Keep pod alive
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:         # Ready to receive traffic
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
````

**Why Deployment > ReplicaSet:**
- Rolling updates (gradual rollout)
- Automatic rollback
- Pause/resume deployments
- Revision history

### Rolling Update Example

```
Version 1: 3 pods running
Pod-v1-1, Pod-v1-2, Pod-v1-3

Update to Version 2:
Step 1: Start Pod-v2-1, keep Pod-v1-1, Pod-v1-2, Pod-v1-3
Step 2: Stop Pod-v1-1, start Pod-v2-2
Step 3: Stop Pod-v1-2, start Pod-v2-3
Step 4: Stop Pod-v1-3

Result: 3 pods running (all v2)
├─ Pod-v2-1
├─ Pod-v2-2
└─ Pod-v2-3

Zero downtime! ✅
```

---

## 4️⃣ Service (Networking)

**Service** = Stable endpoint for pods (load balancer)

### Types of Services

#### A. ClusterIP (Internal Only)

````yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  type: ClusterIP              # Default
  selector:
    app: web
  ports:
  - port: 80                   # Service port
    targetPort: 8080           # Container port
    protocol: TCP
````

```
Access: http://web-service:80 (only from inside cluster)
Use case: Internal communication between pods
```

#### B. NodePort (External Access)

````yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  type: NodePort
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 30007            # Access via any node:30007
````

```
Access: http://<node-ip>:30007 (from outside cluster)
Use case: Development, testing
Port range: 30000-32767
```

#### C. LoadBalancer (Cloud Load Balancer)

````yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 8080
````

```
Access: http://<external-ip>:80 (from anywhere)
External IP: Assigned by cloud provider (AWS, GCP, Azure)
Use case: Production services
```

---

## 5️⃣ Ingress (HTTP/HTTPS Routing)

**Ingress** = HTTP/HTTPS router to multiple services

````yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: myapp.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8080
      - path: /web
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
  - host: admin.myapp.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: admin-service
            port:
              number: 3000
````

```
Routing Logic:
myapp.com/api       → api-service:8080
myapp.com/web       → web-service:80
admin.myapp.com     → admin-service:3000
```

---

## 6️⃣ ConfigMap (Configuration)

**ConfigMap** = Store non-sensitive configuration data

````yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  DATABASE_HOST: postgres.default.svc.cluster.local
  DATABASE_PORT: "5432"
  LOG_LEVEL: info
  APP_ENV: production
````

**Usage in Pod:**

````yaml
spec:
  containers:
  - name: app
    image: myapp:1.0
    env:
    - name: DATABASE_HOST
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: DATABASE_HOST
````

---

## 7️⃣ Secret (Sensitive Data)

**Secret** = Store sensitive data (passwords, tokens, etc.)

````yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  username: dXNlcm5hbWU=       # base64 encoded
  password: cGFzc3dvcmQxMjM=   # base64 encoded
  api-key: YWJjZGVmZ2hpams=
````

**Create from CLI:**

````bash
# Create secret
kubectl create secret generic app-secret \
  --from-literal=username=admin \
  --from-literal=password=secret123

# Create from file
kubectl create secret generic app-secret \
  --from-file=./config.yaml
````

**Usage in Pod:**

````yaml
spec:
  containers:
  - name: app
    env:
    - name: DATABASE_PASSWORD
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: password
````

---

## 8️⃣ Namespace (Isolation)

**Namespace** = Virtual cluster within a cluster

```
Kubernetes Cluster
├─ Namespace: default
│  ├─ Pod: web-1
│  ├─ Pod: web-2
│  └─ Service: web-service
│
├─ Namespace: production
│  ├─ Pod: prod-web-1
│  ├─ Pod: prod-web-2
│  └─ Service: prod-web-service
│
└─ Namespace: monitoring
   ├─ Pod: prometheus-1
   └─ Pod: grafana-1
```

**Create Namespace:**

````bash
# Create namespace
kubectl create namespace monitoring

# Create resource in specific namespace
kubectl apply -f deployment.yaml --namespace=monitoring

# Set default namespace
kubectl config set-context --current --namespace=monitoring

# View all namespaces
kubectl get namespaces
````

---

## 9️⃣ PersistentVolume (Storage)

**PersistentVolume** = Storage resource in cluster

````yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-storage
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: fast-ssd
  nfs:
    server: 192.168.1.100
    path: "/shared"
````

---

## 🔟 PersistentVolumeClaim (Request Storage)

**PersistentVolumeClaim** = Request for storage

````yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-storage
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: fast-ssd
````

**Usage in Pod:**

````yaml
spec:
  containers:
  - name: app
    volumeMounts:
    - name: storage
      mountPath: /data
  volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: pvc-storage
````

---

## 1️⃣1️⃣ StatefulSet (Stateful Apps)

**StatefulSet** = For applications with state (databases, caches)

````yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  replicas: 3
  serviceName: postgres-headless
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:13
        volumeMounts:
        - name: data
          mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 10Gi
````

**Key Differences from Deployment:**
- Ordered pod names (postgres-0, postgres-1, postgres-2)
- Sticky network identity
- Ordered scaling
- Data persistence

---

## 1️⃣2️⃣ DaemonSet (One Pod Per Node)

**DaemonSet** = Ensures pod runs on every node

````yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter
spec:
  selector:
    matchLabels:
      app: node-exporter
  template:
    metadata:
      labels:
        app: node-exporter
    spec:
      containers:
      - name: node-exporter
        image: prom/node-exporter:latest
        ports:
        - containerPort: 9100
````

**Use Cases:**
- Monitoring agents (Prometheus, Datadog)
- Logging agents (Fluentd, Logstash)
- Network plugins
- Security scanning

---

## 1️⃣3️⃣ Job (Run-Once Tasks)

**Job** = Run container to completion

````yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: data-backup
spec:
  backoffLimit: 3            # Retry 3 times
  parallelism: 2             # Run 2 pods in parallel
  completions: 5             # Need 5 completions
  template:
    spec:
      containers:
      - name: backup
        image: backup-tool:1.0
        command: ["./backup.sh"]
      restartPolicy: OnFailure
````

---

## 1️⃣4️⃣ CronJob (Scheduled Tasks)

**CronJob** = Run job on schedule (like cron)

````yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: daily-backup
spec:
  schedule: "0 2 * * *"        # 2 AM every day
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: backup-tool:1.0
            command: ["./backup.sh"]
          restartPolicy: OnFailure
````

---

# 🚀 PART 3: Kubernetes Deployment Tools

Now let's compare **kind vs Minikube vs kops vs kubeadm**

## Comparison Table

```
┌──────────────┬─────────────────┬──────────────┬──────────────┬──────────────┐
│ Feature      │ kind            │ Minikube     │ kops         │ kubeadm      │
├──────────────┼─────────────────┼──────────────┼──────────────┼──────────────┤
│ Purpose      │ Testing,CI/CD   │ Development  │ Production   │ Any          │
│ Setup Time   │ < 1 minute      │ 2-5 min      │ 10-20 min    │ 15-30 min    │
│ Resources    │ Very low        │ Medium       │ High         │ High         │
│ Nodes        │ 1+ containers   │ 1 VM         │ Multiple AWS │ Multiple     │
│ Cloud        │ Local Docker    │ Local VM     │ AWS only     │ Any cloud    │
│ Persistence  │ ❌ No           │ ✅ Yes       │ ✅ Yes       │ ✅ Yes       │
│ Networking   │ Limited         │ Good         │ Excellent    │ Excellent    │
│ Learning     │ Easy            │ Easy         │ Complex      │ Medium       │
│ Production   │ ❌ No           │ ❌ No        │ ✅ Yes       │ ✅ Yes       │
└──────────────┴─────────────────┴──────────────┴──────────────┴──────────────┘
```

---

## 🎯 kind (Kubernetes in Docker)

**kind** = Run Kubernetes clusters inside Docker containers

### What is kind?

```
kind (Kubernetes in Docker)
├─ Runs K8s clusters in Docker containers
├─ No VM needed
├─ Perfect for testing & CI/CD
├─ Multi-node support
└─ Very lightweight
```

### Installation

````bash
# Install kind
brew install kind

# Verify
kind version
````

### Create Cluster

````bash
# Single node cluster
kind create cluster

# Multi-node cluster
cat > kind-config.yaml << 'EOF'
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
EOF

kind create cluster --config kind-config.yaml

# List clusters
kind get clusters

# Delete cluster
kind delete cluster
````

### Load Local Images

````bash
# Build Docker image
docker build -t myapp:latest .

# Load into kind cluster
kind load docker-image myapp:latest

# Use in deployment
kubectl set image deployment/web myapp=myapp:latest
````

### Pros & Cons

```
✅ Pros:
├─ Fast startup (< 1 minute)
├─ Low resource usage
├─ No VM needed
├─ Perfect for CI/CD
└─ Multi-node support

❌ Cons:
├─ No storage persistence
├─ Limited networking features
├─ Not for production
└─ Docker dependency
```

### When to Use kind

```
✅ Use kind when:
├─ Testing K8s manifests
├─ Local development
├─ CI/CD pipelines
├─ Educational purposes
└─ Running multiple quick clusters

❌ Don't use kind when:
├─ Need persistent storage
├─ Testing stateful apps
├─ Need advanced networking
└─ Building production setup
```

---

## 🎯 Minikube (Single-Node VM)

**Minikube** = Single-node Kubernetes cluster in a VM

### What is Minikube?

```
Minikube
├─ Runs K8s in a single VM
├─ Includes Docker built-in
├─ Good for local development
├─ Data persistence
└─ Multiple drivers (VirtualBox, Docker, etc)
```

### Installation

````bash
# Install Minikube
brew install minikube

# Install kubectl
brew install kubectl

# Start Minikube
minikube start

# Stop Minikube
minikube stop

# Delete Minikube
minikube delete

# Get Minikube IP
minikube ip
````

### Key Features

````bash
# SSH into Minikube
minikube ssh

# View Minikube dashboard
minikube dashboard

# Access service via browser
minikube service <service-name>

# Tunnel (if supported)
minikube tunnel

# Logs
minikube logs

# Status
minikube status
````

### Configure Minikube

````bash
# Set memory
minikube config set memory 4096

# Set CPU cores
minikube config set cpus 4

# Change driver
minikube start --driver=docker

# Available drivers: virtualbox, docker, hyperkit, vmware, etc
````

### Pros & Cons

```
✅ Pros:
├─ Easy to install
├─ Good for local development
├─ Data persistence
├─ Multiple drivers
└─ Good networking simulation

❌ Cons:
├─ Single node only
├─ Higher resource usage than kind
├─ Slower startup (2-5 min)
├─ Not for production
└─ VM required
```

### When to Use Minikube

```
✅ Use Minikube when:
├─ Learning Kubernetes
├─ Local development
├─ Testing stateful apps
├─ Need persistent storage
└─ Need realistic single-node setup

❌ Don't use Minikube when:
├─ Testing multi-node scenarios
├─ Need fast CI/CD
├─ Very limited resources
├─ Production deployment
└─ Need advanced networking
```

---

## 🎯 kops (Production AWS)

**kops** = Kubernetes Operations (Production-grade for AWS)

### What is kops?

```
kops (Kubernetes Operations)
├─ Production-grade Kubernetes
├─ AWS-native deployment
├─ Full cluster management
├─ HA (High Availability)
├─ Auto-scaling
└─ Complex but powerful
```

### Installation

````bash
# Install kops
brew install kops

# Install AWS CLI
brew install awscli

# Configure AWS credentials
aws configure

# Verify installation
kops version
````

### Create Cluster

````bash
# Set environment variables
export KOPS_STATE_STORE=s3://my-kops-state-store
export NAME=mycluster.k8s.local

# Create S3 bucket for state
aws s3 mb s3://my-kops-state-store

# Create cluster configuration
kops create cluster \
  --name=$NAME \
  --state=$KOPS_STATE_STORE \
  --zones=us-east-1a,us-east-1b \
  --node-count=3 \
  --node-size=t3.medium \
  --master-size=t3.medium \
  --master-zones=us-east-1a

# Edit cluster (optional)
kops edit cluster --name=$NAME

# Create actual cluster
kops update cluster --name=$NAME --yes

# Validate cluster
kops validate cluster

# Get kubeconfig
kops export kubeconfig --admin
````

### Cluster Management

````bash
# Scale cluster
kops edit ig nodes

# Rolling update
kops rolling-update cluster --name=$NAME --force

# Upgrade Kubernetes version
kops upgrade cluster --name=$NAME
kops update cluster --name=$NAME --yes
kops rolling-update cluster --name=$NAME --force

# Delete cluster
kops delete cluster --name=$NAME --yes
````

### Pros & Cons

```
✅ Pros:
├─ Production-ready
├─ HA and multi-AZ support
├─ Auto-scaling
├─ Full AWS integration
├─ Mature and stable
└─ Great cluster management

❌ Cons:
├─ Complex setup
├─ AWS-only
├─ High cost
├─ Steep learning curve
├─ State management complexity
└─ Slow to create (10-20 min)
```

### When to Use kops

```
✅ Use kops when:
├─ Production workloads on AWS
├─ Need high availability
├─ Want managed K8s cluster
├─ Running on AWS infrastructure
└─ Need auto-scaling

❌ Don't use kops when:
├─ Multi-cloud deployment
├─ GCP or Azure infrastructure
├─ Learning Kubernetes
├─ Local development
├─ Quick prototyping
└─ Don't want AWS vendor lock-in
```

---

## 🎯 kubeadm (Manual Control)

**kubeadm** = Tool to bootstrap Kubernetes cluster

### What is kubeadm?

```
kubeadm
├─ Bootstrap Kubernetes manually
├─ Works on any cloud/machine
├─ More control than kops
├─ Multi-cloud support
├─ Community standard
└─ Requires more setup
```

### Installation Steps

````bash
# 1. Install on all nodes (control-plane & workers)

# Update package list
sudo apt-get update

# Install Docker
sudo apt-get install -y docker.io

# Install kubeadm, kubectl, kubelet
curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://dl.k8s.io/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y kubeadm kubectl kubelet

# 2. On Control Plane node only
sudo kubeadm init \
  --pod-network-cidr=10.244.0.0/16 \
  --apiserver-advertise-address=<control-plane-ip>

# 3. Set up kubeconfig
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# 4. Install CNI plugin (networking)
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

# 5. On Worker nodes
kubeadm join <control-plane-ip>:6443 \
  --token <token> \
  --discovery-token-ca-cert-hash <hash>

# 6. Verify
kubectl get nodes
````

### Pros & Cons

```
✅ Pros:
├─ Multi-cloud (AWS, GCP, Azure, On-prem)
├─ Full control
├─ Community standard
├─ Production ready
├─ Cost effective
└─ Works anywhere

❌ Cons:
├─ Manual setup
├─ Complex configuration
├─ More troubleshooting needed
├─ Steep learning curve
├─ No automatic updates
└─ Time-consuming
```

### When to Use kubeadm

```
✅ Use kubeadm when:
├─ Multi-cloud deployment
├─ On-premises infrastructure
├─ Need maximum control
├─ Production workloads
├─ GCP or Azure (not AWS)
└─ Want to learn deep K8s

❌ Don't use kubeadm when:
├─ Quick local testing
├─ Learning Kubernetes basics
├─ Very limited time
├─ AWS infrastructure (use kops)
└─ Need managed service
```

---

## 📊 Decision Matrix

```
Choose based on your scenario:

SCENARIO 1: Learning Kubernetes
→ Use: Minikube
  Why: Easy, good simulation, local development

SCENARIO 2: Testing in CI/CD
→ Use: kind
  Why: Fast, lightweight, perfect for pipelines

SCENARIO 3: Local Development (Stateful)
→ Use: Minikube
  Why: Persistence, good networking, realistic

SCENARIO 4: Production on AWS
→ Use: kops or EKS
  Why: Production-ready, HA, fully managed

SCENARIO 5: Production Multi-cloud
→ Use: kubeadm
  Why: Works anywhere, full control

SCENARIO 6: Enterprise Cloud
→ Use: Managed Services
  Why: EKS (AWS), GKE (GCP), AKS (Azure)
```

---

# 🌐 PART 4: Kubernetes Networking

## Network Architecture

```
┌────────────────────────────────────────────────────┐
│           Kubernetes Network Model                  │
├────────────────────────────────────────────────────┤
│                                                    │
│  1. Pod-to-Pod Communication                       │
│     └─ Pods can communicate with any pod           │
│        on any node without NAT                     │
│                                                    │
│  2. Pod-to-Service Communication                   │
│     └─ Service provides stable endpoint            │
│        for pod group                               │
│                                                    │
│  3. External-to-Service Communication              │
│     └─ Ingress/LoadBalancer/NodePort               │
│        expose services externally                  │
│                                                    │
│  4. Network Policies                               │
│     └─ Control traffic between pods                │
│                                                    │
└────────────────────────────────────────────────────┘
```

## Service Discovery

**DNS in Kubernetes:**

```
Service DNS Names:

1. Full DNS Name:
   <service-name>.<namespace>.svc.cluster.local

2. Short DNS Name (same namespace):
   <service-name>

3. Headless Service (StatefulSet):
   <pod-name>.<service-name>.<namespace>.svc.cluster.local

Example:
Service: web-service in default namespace
DNS: web-service.default.svc.cluster.local
Or just: web-service (from default namespace)
```

## Network Policies

**NetworkPolicy** = Control pod-to-pod communication

````yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-web
spec:
  podSelector:
    matchLabels:
      app: web
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 8080
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: database
    ports:
    - protocol: TCP
      port: 5432
  - to:
    - namespaceSelector: {}
    ports:
    - protocol: TCP
      port: 53
    - protocol: UDP
      port: 53
````

---

# 💾 PART 5: Storage

## Storage Types

```
┌─────────────────────────────────────────────────────┐
│           Kubernetes Storage Options                │
├─────────────────────────────────────────────────────┤
│                                                     │
│ 1. EmptyDir                                         │
│    ├─ Created when pod starts                       │
│    ├─ Deleted when pod stops                        │
│    └─ Use: Temporary data, cache                    │
│                                                     │
│ 2. HostPath                                         │
│    ├─ Mount host directory into pod                 │
│    ├─ Data persists after pod dies                  │
│    └─ Use: Development, node access                 │
│                                                     │
│ 3. PersistentVolume (PV)                            │
│    ├─ Cluster-wide storage resource                 │
│    ├─ Created by admin                              │
│    └─ Use: Databases, caches                        │
│                                                     │
│ 4. PersistentVolumeClaim (PVC)                      │
│    ├─ Request for storage                           │
│    ├─ Binds to PV                                   │
│    └─ Use: User-level storage requests              │
│                                                     │
│ 5. StorageClass                                     │
│    ├─ Dynamic PV provisioning                       │
│    ├─ Cloud provider integration                    │
│    └─ Use: AWS EBS, GCP PD, Azure Disk              │
│                                                     │
└─────────────────────────────────────────────────────┘
```

### EmptyDir Example

````yaml
apiVersion: v1
kind: Pod
metadata:
  name: temp-pod
spec:
  containers:
  - name: app
    image: myapp:1.0
    volumeMounts:
    - name: temp
      mountPath: /tmp/data
  volumes:
  - name: temp
    emptyDir: {}
````

### HostPath Example

````yaml
apiVersion: v1
kind: Pod
metadata:
  name: host-pod
spec:
  containers:
  - name: app
    image: myapp:1.0
    volumeMounts:
    - name: host-data
      mountPath: /data
  volumes:
  - name: host-data
    hostPath:
      path: /var/lib/data
      type: Directory
````

### PersistentVolume + PVC Example

````yaml
# 1. Create PersistentVolume
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-data
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /data/pv
---
# 2. Create PersistentVolumeClaim
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-data
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
---
# 3. Use in Pod
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: myapp:1.0
    volumeMounts:
    - name: storage
      mountPath: /data
  volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: pvc-data
````

### StorageClass (Dynamic Provisioning)

````yaml
# 1. Create StorageClass
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: ebs.csi.aws.com
parameters:
  type: gp3
  iops: "3000"
  throughput: "125"
allowVolumeExpansion: true
---
# 2. PVC uses StorageClass
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dynamic-pvc
spec:
  storageClassName: fast-ssd
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
---
# 3. Pod uses PVC
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
spec:
  replicas: 1
  template:
    spec:
      containers:
      - name: app
        image: myapp:1.0
        volumeMounts:
        - name: data
          mountPath: /data
      volumes:
      - name: data
        persistentVolumeClaim:
          claimName: dynamic-pvc
````

---

# 🚀 PART 6: Advanced Kubernetes Concepts

## 1. Resource Management

### Resource Requests & Limits

````yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
spec:
  template:
    spec:
      containers:
      - name: app
        image: myapp:1.0
        resources:
          # Minimum guaranteed resources
          requests:
            memory: "256Mi"
            cpu: "250m"              # 0.25 CPU cores
          # Maximum allowed resources
          limits:
            memory: "512Mi"
            cpu: "500m"              # 0.5 CPU cores
        # Pod Quality of Service (QoS)
        # Guaranteed: requests == limits
        # Burstable: requests < limits
        # BestEffort: no requests/limits
````

**How CPU & Memory work:**

```
CPU Units:
├─ 1000m = 1 core
├─ 500m = 0.5 core
├─ 100m = 0.1 core
└─ Example: "250m" = 0.25 core

Memory Units:
├─ Ki = Kibibyte
├─ Mi = Mebibyte (1024 * 1024 bytes)
├─ Gi = Gibibyte (1024 * 1024 * 1024 bytes)
└─ Example: "256Mi" ≈ 256 MB

Requests: What pod needs to be scheduled
Limits: Maximum it can use
```

## 2. Horizontal Pod Autoscaler (HPA)

**HPA** = Automatically scale pods based on metrics

````yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70    # Scale up if > 70% CPU
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80    # Scale up if > 80% memory
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 50                 # Scale down by 50% max
        periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
      - type: Percent
        value: 100               # Scale up by 100% max
        periodSeconds: 30
````

**How HPA works:**

```
Step 1: Monitor metrics
        ↓
Step 2: Calculate desired replicas
        ├─ If CPU > 70% → increase
        ├─ If CPU < 70% → decrease
        └─ If memory > 80% → increase
        ↓
Step 3: Scale deployment
        ├─ Add/remove pods
        └─ Gradual (respects behavior policies)
```

## 3. Vertical Pod Autoscaler (VPA)

**VPA** = Automatically adjust resource requests/limits

````yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: app-vpa
spec:
  targetRef:
    apiVersion: "apps/v1"
    kind: Deployment
    name: myapp
  updatePolicy:
    updateMode: "Auto"            # Modes: Off, Initial, Recreate, Auto
  resourcePolicy:
    containerPolicies:
    - containerName: app
      minAllowed:
        cpu: 100m
        memory: 128Mi
      maxAllowed:
        cpu: 2
        memory: 1Gi
````

## 4. Pod Disruption Budget (PDB)

**PDB** = Ensure minimum availability during disruptions

````yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: app-pdb
spec:
  minAvailable: 2              # Keep min 2 pods running
  selector:
    matchLabels:
      app: myapp
---
# Alternative: maxUnavailable
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: app-pdb
spec:
  maxUnavailable: 1            # Allow max 1 pod down
  selector:
    matchLabels:
      app: myapp
````

## 5. Role-Based Access Control (RBAC)

**RBAC** = Control who can do what in cluster

````yaml
# 1. Create Role (permissions)
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
rules:
- apiGroups: [""]
  resources: ["pods", "services"]
  verbs: ["get", "list", "create", "update", "delete"]
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "create", "update"]
---
# 2. Create ServiceAccount (identity)
apiVersion: v1
kind: ServiceAccount
metadata:
  name: dev-user
---
# 3. Bind Role to ServiceAccount
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dev-user-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: developer
subjects:
- kind: ServiceAccount
  name: dev-user
  namespace: default
````

## 6. Network Policies (Advanced)

````yaml
# Deny all ingress
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
spec:
  podSelector: {}
  policyTypes:
  - Ingress
---
# Allow specific ingress
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    - namespaceSelector:
        matchLabels:
          name: frontend-ns
    ports:
    - protocol: TCP
      port: 8080
````

## 7. Custom Resources (CRDs)

**CRD** = Define custom Kubernetes objects

````yaml
# 1. Define the CRD
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: websites.example.com
spec:
  group: example.com
  names:
    kind: Website
    plural: websites
  scope: Namespaced
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              domain:
                type: string
              replicas:
                type: integer
---
# 2. Use the CRD
apiVersion: example.com/v1
kind: Website
metadata:
  name: mywebsite
spec:
  domain: mywebsite.com
  replicas: 3
````

## 8. Operators (Advanced Automation)

**Operator** = Custom controller managing CRDs

```
Operator Pattern:
└─ Combines CRD + Controller
   ├─ CRD: Defines custom resource
   ├─ Controller: Watches and manages resources
   └─ Reconciliation loop: Makes desired state = actual state

Example: Database Operator
├─ CRD: Database resource
├─ Controller: Creates/updates DB
└─ Watches for changes

Popular Operators:
├─ Prometheus Operator (monitoring)
├─ etcd Operator (database)
├─ PostgreSQL Operator (database)
├─ MySQL Operator (database)
└─ Kafka Operator (messaging)
```

## 9. Helm (Package Manager)

**Helm** = Package manager for Kubernetes

````bash
# Install Helm
brew install helm

# Add repository
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

# Search chart
helm search repo nginx

# Install chart
helm install my-release bitnami/nginx

# List releases
helm list

# Upgrade release
helm upgrade my-release bitnami/nginx --values values.yaml

# Rollback
helm rollback my-release 1

# Uninstall
helm uninstall my-release

# Create your own chart
helm create mychart

# Package chart
helm package mychart/

# Install from local
helm install my-release ./mychart/
````

**Helm Chart Structure:**

```
mychart/
├─ Chart.yaml              (Chart metadata)
├─ values.yaml             (Default values)
├─ values-prod.yaml        (Production values)
├─ templates/
│  ├─ deployment.yaml      (Deployment template)
│  ├─ service.yaml         (Service template)
│  ├─ configmap.yaml       (ConfigMap template)
│  ├─ secret.yaml          (Secret template)
│  └─ _helpers.tpl         (Helper functions)
├─ charts/                 (Dependencies)
└─ README.md
```

## 10. Kustomize (Configuration Management)

**Kustomize** = Template-free customization

````bash
# Create kustomization
kustomize create --autodetect

# Build
kustomize build ./overlay/prod | kubectl apply -f -

# Or kubectl built-in
kubectl apply -k ./overlay/prod
````

**Kustomization Structure:**

```
base/
├─ kustomization.yaml
├─ deployment.yaml
└─ service.yaml

overlay/
├─ dev/
│  ├─ kustomization.yaml (patch for dev)
│  └─ deployment-patch.yaml
└─ prod/
   ├─ kustomization.yaml (patch for prod)
   └─ deployment-patch.yaml
```

---

# 🎯 PART 7: Best Practices

## 1. Resource Management

```yaml
✅ GOOD:
spec:
  containers:
  - name: app
    resources:
      requests:
        memory: "256Mi"
        cpu: "100m"
      limits:
        memory: "512Mi"
        cpu: "500m"

❌ BAD:
spec:
  containers:
  - name: app
    # No resource limits
    # Pod could consume all cluster resources
```

## 2. Health Checks

```yaml
✅ GOOD:
spec:
  containers:
  - name: app
    livenessProbe:
      httpGet:
        path: /health
        port: 8080
      initialDelaySeconds: 30
      periodSeconds: 10
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 5

❌ BAD:
spec:
  containers:
  - name: app
    # No health checks
    # Failed pods stay running
```

## 3. Rolling Updates

```yaml
✅ GOOD:
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1           # One extra pod during update
      maxUnavailable: 0     # Zero pods unavailable
  minReadySeconds: 30       # Wait 30s before considering ready

❌ BAD:
spec:
  strategy:
    type: Recreate         # Kill all, start all (downtime!)
```

## 4. Security

```yaml
✅ GOOD:
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    fsReadOnlyRootFilesystem: true
  containers:
  - name: app
    securityContext:
      allowPrivilegeEscalation: false
      capabilities:
        drop:
        - ALL

❌ BAD:
spec:
  containers:
  - name: app
    securityContext:
      runAsUser: 0          # Running as root!
      privileged: true      # Privileged container!
```

## 5. Image Best Practices

```yaml
✅ GOOD:
containers:
- name: app
  image: myregistry/myapp:v1.2.3    # Specific tag
  imagePullPolicy: IfNotPresent     # Don't pull every time

❌ BAD:
containers:
- name: app
  image: myapp                      # No registry, no tag
  image: myapp:latest               # Latest tag (mutable)
```

## 6. Namespaces & Organization

```yaml
✅ GOOD:
└─ Kubernetes Cluster
   ├─ Namespace: production
   │  ├─ Deployment: app
   │  ├─ Service: app
   │  └─ PVC: app-data
   ├─ Namespace: staging
   │  ├─ Deployment: app
   │  └─ Service: app
   └─ Namespace: monitoring
      ├─ Prometheus
      └─ Grafana

❌ BAD:
└─ Kubernetes Cluster
   └─ Namespace: default
      ├─ Everything mixed together
      └─ Hard to manage
```

## 7. Logging & Monitoring

```yaml
✅ GOOD:
containers:
- name: app
  env:
  - name: LOG_LEVEL
    value: INFO
  resources:
    requests:
      memory: "256Mi"
      cpu: "100m"

# Separate monitoring setup:
- Prometheus for metrics
- Grafana for visualization
- ELK/Loki for logs
- Jaeger for tracing

❌ BAD:
# No monitoring setup
# Application logs nowhere
# No visibility into cluster
```

## 8. GitOps Workflow

```yaml
✅ GOOD:
├─ Git Repository
│  ├─ k8s/base/
│  │  ├─ deployment.yaml
│  │  ├─ service.yaml
│  │  └─ kustomization.yaml
│  ├─ k8s/overlay/prod/
│  │  ├─ kustomization.yaml
│  │  └─ kustomization-patch.yaml
│  └─ .github/workflows/
│     └─ deploy.yml (ArgoCD/Flux)
│
└─ CD Pipeline
   ├─ Commit to Git
   ├─ Pipeline checks manifests
   ├─ Apply to cluster
   └─ ArgoCD syncs desired state

❌ BAD:
# Manual kubectl apply
# No version control
# No audit trail
# No rollback capability
```

---

# 📚 Complete Kubernetes Cheat Sheet

## kubectl Commands

```bash
# Cluster Info
kubectl cluster-info
kubectl get nodes
kubectl describe node <node-name>

# Pods
kubectl get pods
kubectl get pods -o wide
kubectl get pods --all-namespaces
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl logs <pod-name> -f              # Follow logs
kubectl logs <pod-name> -c <container>  # Specific container
kubectl exec -it <pod-name> -- bash     # Execute command

# Deployments
kubectl get deployments
kubectl create deployment <name> --image=<image>
kubectl scale deployment <name> --replicas=3
kubectl set image deployment/<name> <container>=<image>:<tag>
kubectl rollout history deployment/<name>
kubectl rollout undo deployment/<name>
kubectl rollout status deployment/<name>

# Services
kubectl get svc
kubectl expose deployment <name> --type=NodePort --port=80 --target-port=8080
kubectl port-forward svc/<name> 8080:80

# ConfigMap & Secrets
kubectl create configmap <name> --from-literal=KEY=VALUE
kubectl create secret generic <name> --from-literal=KEY=VALUE
kubectl get configmap
kubectl get secret

# Namespaces
kubectl create namespace <name>
kubectl get namespaces
kubectl config set-context --current --namespace=<namespace>

# Apply YAML
kubectl apply -f deployment.yaml
kubectl apply -f . -R                    # Apply directory recursively
kubectl delete -f deployment.yaml
kubectl delete pod <pod-name>

# Other
kubectl get all                          # All resources
kubectl describe <resource> <name>
kubectl edit <resource> <name>
kubectl patch <resource> <name> -p '{"spec":{"replicas":3}}'
kubectl top nodes                        # Resource usage
kubectl top pods
```

---

# 🎓 Learning Path

```
Week 1-2: Fundamentals
├─ Install Minikube/Docker Desktop
├─ Learn kubectl basics
├─ Understand Pods, Deployments, Services
└─ Deploy first application

Week 3-4: Core Concepts
├─ ConfigMaps & Secrets
├─ Storage (PVC, PV)
├─ Namespaces
├─ Services & Networking
└─ Ingress

Week 5-6: Advanced
├─ StatefulSets & DaemonSets
├─ RBAC & Security
├─ Resource Management
├─ Monitoring & Logging
└─ Helm

Week 7-8: Production
├─ Production deployment tools (kops, kubeadm)
├─ High Availability
├─ Disaster Recovery
├─ GitOps
└─ Best Practices
```

---

# 🔗 Resources

```
Official Kubernetes:
├─ Kubernetes.io (official docs)
├─ Play with Kubernetes (online)
└─ Kubernetes Academy (courses)

Tools:
├─ kubectl cheatsheet
├─ Helm Hub (charts)
├─ Kustomize documentation
└─ CNCF Landscape

Learning:
├─ Linux Academy
├─ A Cloud Guru
├─ Udemy Kubernetes courses
├─ YouTube (KodeKloud, etc.)
└─ Kubernetes by Example
```

---

## 🎉 Summary

```
Kubernetes Deployment Tools at a Glance:

kind:     Fast, lightweight, testing
Minikube: Easy, local development
kops:     Production AWS clusters
kubeadm:  Manual, any infrastructure

Kubernetes Objects Hierarchy:

Cluster
├─ Namespace
│  ├─ Deployment
│  │  └─ ReplicaSet
│  │     └─ Pod
│  │        ├─ Container
│  │        └─ Volume
│  ├─ Service (network endpoint)
│  ├─ Ingress (HTTP routing)
│  ├─ ConfigMap (config)
│  ├─ Secret (sensitive data)
│  ├─ PersistentVolume (storage)
│  └─ PersistentVolumeClaim (storage request)

Key Concepts:

✅ Declarative (describe desired state)
✅ Self-healing (restarts failed containers)
✅ Scalable (horizontal & vertical)
✅ Flexible networking (service discovery)
✅ Storage abstraction (multiple backends)
✅ Advanced features (operators, CRDs)
```

---

**Now you have a complete Kubernetes guide from beginner to advanced!** 🚀

#Kubernetes #DevOps #Containers #Orchestration #CloudNative