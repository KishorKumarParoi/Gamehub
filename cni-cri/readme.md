# CNI and CRI in Kubernetes: Short Guide

I'll explain both with focus on your KIND cluster setup.

---

## **WHAT is CNI?**

**CNI = Container Network Interface**

The plugin that handles **pod-to-pod networking**.

```
Without CNI:
  Pod A (IP: 10.1.0.1)
    ❌ Cannot talk to Pod B (IP: 10.2.0.1)
    ❌ Network doesn't exist

With CNI:
  Pod A (IP: 10.1.0.1)
    ✅ Can reach Pod B (IP: 10.2.0.1)
    ✅ CNI created virtual network
```

### **How CNI Works:**

```
1. Pod created → Kubernetes tells CNI: "Create network interface"
2. CNI assigns IP address to pod
3. CNI connects pod to network
4. Pod can now communicate with other pods
5. Pod deleted → CNI cleans up network interface
```

### **Common CNI Plugins:**

| CNI | Use Case | Network Model |
|-----|----------|---------------|
| **Flannel** | Simple, default for KIND | Overlay network |
| **Calico** | Production, network policies | BGP routing |
| **Weave** | Multi-cloud | Mesh network |
| **Cilium** | Advanced, eBPF-based | High performance |
| **AWS VPC CNI** | AWS EKS | Direct VPC integration |

### **KIND Default CNI:**

```bash
# KIND uses Kindnetd by default
# It's a simple CNI built into KIND for local development

# Verify CNI running
kubectl get pods -n kube-system | grep kindnetd

# Output:
# kindnet-abc12    1/1     Running
```

---

## **WHAT is CRI?**

**CRI = Container Runtime Interface**

The component that **runs containers** (pulls images, starts/stops containers).

```
Without CRI:
  Pod spec: "Run nginx image"
    ❌ Nothing knows how to run containers
    ❌ Pod can't start

With CRI:
  Pod spec: "Run nginx image"
    ✅ CRI pulls image from registry
    ✅ CRI starts container process
    ✅ Pod is now running
```

### **How CRI Works:**

```
1. Kubelet says: "Start container with image: nginx:latest"
2. CRI pulls image from Docker registry
3. CRI creates container from image
4. CRI starts container process
5. Container runs
6. When pod deleted → CRI stops and removes container
```

### **Common CRI Implementations:**

| CRI | Technology | Use Case |
|-----|-----------|----------|
| **Docker** | containerd + runc | Default, universal |
| **containerd** | OCI-compliant | Lightweight, production |
| **CRI-O** | OCI-focused | Kubernetes-native |
| **runc** | Low-level runtime | Container process |
| **gVisor** | Lightweight VM | Isolation/security |

### **KIND Default CRI:**

```bash
# KIND uses containerd by default
# Located inside KIND nodes

# Check CRI in KIND node
docker exec tws-kind-cluster-control-plane \
  crictl ps

# Output shows running containers inside the KIND node
```

---

## **CNI vs CRI - Key Difference**

```
CNI (Container Network Interface):
  ↓ Handles networking
  ↓ "How do pods communicate?"
  └─ Answer: By connecting to network via CNI plugins

CRI (Container Runtime Interface):
  ↓ Handles container execution
  ↓ "How do containers run?"
  └─ Answer: By using container runtime (Docker, containerd, etc)

Visual:
┌─────────────────────────────────────┐
│ Kubernetes Cluster                  │
│                                     │
│ ┌─────────────────────────────────┐ │
│ │ Node 1                          │ │
│ │                                 │ │
│ │ ┌──────────┐    ┌───────────┐   │ │
│ │ │Pod (CRI) │    │Pod (CRI)  │   │ │
│ │ └──────────┘    └───────────┘   │ │
│ │       ↓              ↓            │ │
│ │ ┌──────────────────────────────┐ │ │
│ │ │ CRI (containerd)             │ │ │
│ │ │ - Pulls images               │ │ │
│ │ │ - Runs containers            │ │ │
│ │ │ - Manages lifecycle          │ │ │
│ │ └──────────────────────────────┘ │ │
│ │       ↑              ↑            │ │
│ │ ┌──────────────────────────────┐ │ │
│ │ │ CNI (kindnetd in KIND)       │ │ │
│ │ │ - Assigns IP addresses       │ │ │
│ │ │ - Creates network interfaces │ │ │
│ │ │ - Pod-to-pod communication   │ │ │
│ │ └──────────────────────────────┘ │ │
│ └─────────────────────────────────┘ │
└─────────────────────────────────────┘
```

---

## **In Your KIND Cluster**

### **CNI Setup (Already Done):**

```bash
# KIND automatically deploys kindnetd CNI
# Check it:
kubectl get pods -n kube-system

# Output:
# NAME                                      READY   STATUS
# coredns-...                              1/1     Running
# etcd-tws-kind-cluster-control-plane      1/1     Running
# kindnet-abc123 (CNI plugin)              1/1     Running
# kube-apiserver-...                       1/1     Running
# kube-controller-manager-...              1/1     Running
# kube-proxy-...                           1/1     Running
# kube-scheduler-...                       1/1     Running
```

### **CRI Setup (Already Done):**

```bash
# KIND uses containerd as CRI
# Check it:
docker exec tws-kind-cluster-control-plane \
  crictl version

# Output shows containerd version

# List running containers
docker exec tws-kind-cluster-control-plane \
  crictl ps

# Output shows all containers running via CRI
```

---

## **How to Switch CNI (If Needed)**

### **Replace kindnetd with Calico:**

```bash
# 1. Delete default CNI
kubectl delete daemonset kindnet -n kube-system

# 2. Install Calico
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.0/manifests/tigera-operator.yaml

# 3. Verify
kubectl get pods -n calico-system

# Now pods use Calico for networking instead of kindnetd
```

### **Replace kindnetd with Flannel:**

```bash
# 1. Delete default CNI
kubectl delete daemonset kindnet -n kube-system

# 2. Install Flannel
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

# 3. Verify
kubectl get pods -n kube-flannel

# Now pods use Flannel for networking
```

---

## **Quick Reference**

```yaml
CNI (Networking):
  What: Connects pods to network
  Who manages: CNI plugins (Calico, Flannel, Weave)
  Configuration: Network policies
  In KIND: kindnetd (default)
  Command to check:
    kubectl get pods -n kube-system | grep kindnet

CRI (Container Runtime):
  What: Runs containers
  Who manages: Container runtime (Docker, containerd, CRI-O)
  Configuration: Image registries, runtime options
  In KIND: containerd
  Command to check:
    docker exec kind-control-plane crictl ps
```

---

## **For Your KIND Cluster Setup**

```bash
# 1. Check CNI is working
kubectl get pods --all-namespaces | grep kindnet

# 2. Check CRI is working
docker ps | grep kind

# 3. Test networking (CNI)
kubectl run -it --rm debug --image=busybox --restart=Never -- sh
# Inside pod: ping <other-pod-ip>

# 4. Test container runtime (CRI)
kubectl run nginx --image=nginx
# Check: CRI pulled image and started container
```

---

**Summary:**
- **CNI** = How pods **talk to each other** (networking)
- **CRI** = How containers **run and execute** (runtime)

Both work together to make Kubernetes function! 🚀