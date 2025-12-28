# 📚 Kubernetes Installation Script Explained

I'll break down every line of this script with **why**, **how**, and **what** it does:

---

## 🎯 Script Overview

```
Purpose: Automated Kubernetes v1.33.0 single-node cluster setup
├─ Install Kubernetes components (kubectl, kubeadm, kubelet)
├─ Configure system for Kubernetes
├─ Setup container runtime (containerd)
├─ Initialize the cluster
├─ Install networking (Flannel)
└─ Remove taints to schedule pods on control plane
```

---

## 📖 Line-by-Line Explanation

### **Header & Error Handling**

````bash
#!/bin/bash
# WHAT: Shebang line
# WHY: Tells OS to execute this file using bash interpreter
# HOW: When you run ./script.sh, bash interprets the commands

set -e
# WHAT: Exit on error flag
# WHY: If any command fails, stop execution (don't continue)
# HOW: Prevents cascading failures from incomplete setup
# EXAMPLE: If kubeadm install fails, script stops (doesn't initialize cluster)
````

---

### **Step 1: Install Kubernetes Components**

#### Part 1A: Create Directories & Install Dependencies

````bash
echo "Step 1: Install kubectl, kubeadm, and kubelet v1.33.0"
# WHAT: Print status message
# WHY: User knows what's happening
# HOW: Displays in terminal

sudo mkdir -p /etc/apt/keyrings
# WHAT: Create directory for GPG keys
# WHY: APT uses this to verify package signatures
# HOW: -p flag creates parent directories if they don't exist
# RESULT: /etc/apt/keyrings/ exists and ready for keys

sudo apt-get install -y apt-transport-https ca-certificates curl gpg
# WHAT: Install 4 essential packages
# WHY: Needed for secure package downloads and GPG verification
# HOW: -y flag auto-answers "yes" to prompts
# PACKAGES:
#   ├─ apt-transport-https: HTTPS support for APT
#   ├─ ca-certificates: SSL/TLS certificate validation
#   ├─ curl: Download files from internet
#   └─ gpg: Verify digital signatures
````

#### Part 1B: Add Kubernetes Repository

````bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.33/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
# WHAT: Download Kubernetes GPG key and convert it
# WHY: APT needs this key to verify Kubernetes packages are authentic
# HOW BROKEN DOWN:
#   curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.33/deb/Release.key
#   ├─ -f: Fail silently on HTTP errors
#   ├─ -s: Silent mode (no progress bar)
#   ├─ -S: Show errors even in silent mode
#   ├─ -L: Follow redirects
#   └─ Downloads the GPG key as text
#
#   | (pipe): Send output to next command
#
#   sudo gpg --dearmor
#   ├─ Convert ASCII GPG key to binary format
#   ├─ (--dearmor converts ASCII-armored to binary)
#   └─ APT requires binary format
#
#   -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
#   └─ Save binary key to this location

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.33/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
# WHAT: Add Kubernetes repository to APT sources
# WHY: Tell APT where to download Kubernetes packages from
# HOW BROKEN DOWN:
#   echo 'deb [signed-by=...] https://...'
#   └─ Create repository line with:
#      ├─ deb: Debian package repository
#      ├─ signed-by: Use our GPG key to verify
#      └─ https://pkgs.k8s.io/...: Official Kubernetes repo
#
#   | sudo tee /etc/apt/sources.list.d/kubernetes.list
#   ├─ tee: Write to file AND display on terminal
#   ├─ /etc/apt/sources.list.d/: Directory for .list files
#   └─ kubernetes.list: File APT checks for repos

# RESULT: APT now knows about Kubernetes v1.33 packages
````

#### Part 1C: Install Kubernetes Tools

````bash
sudo apt-get update -y
# WHAT: Refresh APT package list
# WHY: APT needs to know about newly added Kubernetes repo
# HOW: Reads all .list files in /etc/apt/sources.list.d/
# RESULT: APT index updated with Kubernetes packages

sudo apt-get install -y kubelet=1.33.0-1.1 kubeadm=1.33.0-1.1 kubectl=1.33.0-1.1 vim git curl wget
# WHAT: Install Kubernetes components + tools
# WHY: These are required for cluster setup
# HOW: APT downloads and installs exact versions
# PACKAGES:
#   ├─ kubelet=1.33.0-1.1: Node agent (runs containers)
#   ├─ kubeadm=1.33.0-1.1: Cluster initialization tool
#   ├─ kubectl=1.33.0-1.1: Kubernetes CLI (control cluster)
#   ├─ vim: Text editor (for config files)
#   ├─ git: Version control
#   ├─ curl: Download files
#   └─ wget: Download files (backup option)

sudo apt-mark hold kubelet kubeadm kubectl
# WHAT: Prevent automatic updates to Kubernetes tools
# WHY: Automatic upgrades could break cluster compatibility
# HOW: APT won't upgrade these packages in future updates
# EXAMPLE: If you run apt-get upgrade, these stay at v1.33.0
````

---

### **Step 2: Swap & Kernel Setup**

#### Part 2A: Disable Swap

````bash
echo "Step 2: Swap Off and Kernel Modules Setup"

sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
# WHAT: Comment out all swap lines in /etc/fstab
# WHY: Kubernetes requires swap disabled (performance requirement)
# HOW BROKEN DOWN:
#   sed -i: In-place file editing
#   '/ swap /': Find lines containing "swap"
#   s/^\(.*\)$/#\1/g: Replace with commented version (#)
#   /etc/fstab: System startup config (disks to mount)
#
# EXAMPLE:
#   Before: /dev/sda2    none    swap    sw    0    0
#   After:  #/dev/sda2   none    swap    sw    0    0

sudo swapoff -a
# WHAT: Immediately disable all swap
# WHY: Remove swap from current session (don't wait for reboot)
# HOW: Unallocates swap memory
# RESULT: Swap is off now AND will stay off after reboot (fstab change)
````

#### Part 2B: Load Kernel Modules

````bash
sudo modprobe overlay
# WHAT: Load "overlay" kernel module
# WHY: Container runtimes use overlay filesystem for layering
# HOW: Kernel module enables OverlayFS capability
# EXAMPLE: Docker/containerd layers: base → runtime → container

sudo modprobe br_netfilter
# WHAT: Load "bridge netfilter" kernel module
# WHY: Enables network filtering on bridge interfaces
# HOW: Required for Kubernetes networking (CNI plugins)
# RESULT: Linux can bridge container networks

cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF
# WHAT: Create file with modules to load at boot
# WHY: Modules disappear after reboot, this ensures they reload
# HOW BROKEN DOWN:
#   cat <<EOF ... EOF: Here-document (multi-line input)
#   List modules to load:
#   ├─ overlay: For container layering
#   └─ br_netfilter: For network filtering
#   | sudo tee /etc/modules-load.d/containerd.conf
#   └─ Write to config file
#
# RESULT: Modules load automatically on every boot
````

#### Part 2C: Configure Kernel Networking

````bash
cat <<EOF | sudo tee /etc/sysctl.d/kubernetes.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
# WHAT: Create kernel parameter config file
# WHY: Enable networking features required by Kubernetes
# HOW: sysctl.d/ files are applied at boot
# PARAMETERS:
#   ├─ net.bridge.bridge-nf-call-ip6tables = 1
#   │  WHY: Route IPv6 traffic through iptables
#   │  RESULT: Container IPv6 networking works
#   │
#   ├─ net.bridge.bridge-nf-call-iptables = 1
#   │  WHY: Route IPv4 traffic through iptables
#   │  RESULT: Container IPv4 networking works
#   │
#   └─ net.ipv4.ip_forward = 1
#      WHY: Enable IP forwarding between interfaces
#      RESULT: Packets can be routed between containers

sudo sysctl --system
# WHAT: Apply all kernel parameter changes immediately
# WHY: Don't wait for reboot
# HOW: Reads all /etc/sysctl.d/*.conf files and applies them
# RESULT: Networking features are active NOW
````

---

### **Step 3: Install & Configure containerd**

````bash
echo "Step 3: Install and Configure Containerd"

if ! command -v containerd &> /dev/null
# WHAT: Check if containerd is installed
# WHY: Don't reinstall if already present
# HOW BROKEN DOWN:
#   if ! command -v containerd: "Is containerd in PATH?"
#   &> /dev/null: Suppress output (don't show on terminal)
#   then: If NOT installed, do next steps
then
    echo "Containerd not found, installing..."
    
    sudo mkdir -p /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker-archive-keyring.gpg
    # WHAT: Same as Kubernetes GPG key, but for Docker repo
    # WHY: Docker repo also needs GPG verification
    
    echo \
    "deb [arch=amd64 signed-by=/etc/apt/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" \
    | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    # WHAT: Add Docker repository to APT sources
    # WHY: containerd.io is in Docker's official repo
    # HOW:
    #   arch=amd64: Only x86_64 packages
    #   $(lsb_release -cs): Get Ubuntu codename (focal, jammy, etc.)
    #   stable: Use stable channel, not beta
    
    sudo apt-get update -y
    # WHAT: Refresh APT index with Docker repo
    
    sudo DEBIAN_FRONTEND=noninteractive apt-get install -y \
      -o Dpkg::Options::="--force-confold" \
      --allow-downgrades --allow-change-held-packages containerd.io
    # WHAT: Install containerd.io with special options
    # WHY: Need specific flags to handle conflicts/upgrades
    # HOW BROKEN DOWN:
    #   DEBIAN_FRONTEND=noninteractive: No interactive prompts
    #   -o Dpkg::Options::="--force-confold": Keep old config files
    #   --allow-downgrades: Permit version downgrades
    #   --allow-change-held-packages: Allow held packages to change
    #   containerd.io: The container runtime package
else
    echo "Containerd is already installed, skipping installation."
    # WHAT: If containerd exists, skip installation
    # WHY: Avoid duplicate/conflicting installs
fi

# ALWAYS configure containerd (even if already installed)
sudo mkdir -p /etc/containerd
# WHAT: Create containerd config directory
# WHY: Ensure directory exists for config

containerd config default | sudo tee /etc/containerd/config.toml > /dev/null
# WHAT: Generate default containerd config and save it
# HOW BROKEN DOWN:
#   containerd config default: Print default configuration
#   | sudo tee /etc/containerd/config.toml: Write to config file
#   > /dev/null: Don't display output on terminal
# RESULT: /etc/containerd/config.toml created with defaults

sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
# WHAT: Change containerd cgroup driver to systemd
# WHY: Kubernetes recommends systemd cgroup driver (better integration)
# HOW BROKEN DOWN:
#   sed -i: In-place edit
#   s/SystemdCgroup = false/SystemdCgroup = true/: Find and replace
#   /etc/containerd/config.toml: The config file
# BEFORE: SystemdCgroup = false (uses cgroupfs)
# AFTER:  SystemdCgroup = true  (uses systemd)
# RESULT: containerd uses systemd for cgroup management

sudo systemctl restart containerd
# WHAT: Restart containerd service to apply config
# WHY: Config changes don't take effect without restart
# HOW: systemctl stops and starts the service

sudo systemctl enable containerd
# WHAT: Auto-start containerd on boot
# WHY: Container runtime needs to be ready when system boots
# HOW: Creates startup symlink in systemd
````

---

### **Step 4: Kubernetes Initialization**

````bash
sudo systemctl enable kubelet
# WHAT: Auto-start kubelet on boot
# WHY: kubelet must run for cluster to function
# HOW: Creates startup symlink in systemd
# NOTE: kubelet won't start until kubeadm init is run

echo "Step 4: Pull Kubernetes images and init cluster"

sudo kubeadm config images pull --cri-socket unix:///run/containerd/containerd.sock --kubernetes-version v1.33.0
# WHAT: Pre-download Kubernetes component images
# WHY: Speeds up cluster initialization
# HOW BROKEN DOWN:
#   kubeadm config images pull: Download container images
#   --cri-socket unix:///run/containerd/containerd.sock
#   └─ Use containerd as container runtime (not Docker)
#   --kubernetes-version v1.33.0
#   └─ Download images for v1.33.0
# IMAGES DOWNLOADED:
#   ├─ kube-apiserver:v1.33.0
#   ├─ kube-controller-manager:v1.33.0
#   ├─ kube-scheduler:v1.33.0
#   ├─ kube-proxy:v1.33.0
#   ├─ pause:3.x
#   ├─ etcd:x.x.x
#   └─ coredns:x.x.x

sudo kubeadm init \
  --pod-network-cidr=10.244.0.0/16 \
  --upload-certs \
  --kubernetes-version=v1.33.0 \
  --control-plane-endpoint="$(hostname)" \
  --ignore-preflight-errors=all \
  --cri-socket unix:///run/containerd/containerd.sock
# WHAT: Initialize Kubernetes control plane (master)
# WHY: Creates the cluster and generates certificates
# HOW: kubeadm handles all setup complexity
# FLAGS EXPLAINED:
#
#   --pod-network-cidr=10.244.0.0/16
#   ├─ IP range for pods (container IPs)
#   ├─ WHY: Each pod needs unique IP
#   └─ RANGE: 10.244.0.0 to 10.244.255.255 (65,536 IPs)
#
#   --upload-certs
#   ├─ Save certificates for adding control planes later
#   ├─ WHY: Needed if you add more control plane nodes
#   └─ RESULT: Certs stored in etcd for retrieval
#
#   --kubernetes-version=v1.33.0
#   ├─ Specify cluster version
#   ├─ WHY: Match component versions
#   └─ RESULT: All components run v1.33.0
#
#   --control-plane-endpoint="$(hostname)"
#   ├─ Hostname or IP of control plane
#   ├─ WHY: Used by worker nodes to find master
#   ├─ $(hostname): Get current machine's hostname
#   └─ RESULT: Workers can join using this hostname
#
#   --ignore-preflight-errors=all
#   ├─ Skip pre-flight checks
#   ├─ WHY: Avoid blocking on non-critical warnings
#   ├─ ⚠️ CAUTION: Only use in dev/test!
#   └─ RESULT: Installation proceeds despite warnings
#
#   --cri-socket unix:///run/containerd/containerd.sock
#   ├─ Container runtime socket location
#   ├─ WHY: Tell kubeadm where to find containerd
#   └─ RESULT: kubelet uses containerd for containers

# WHAT HAPPENS DURING kubeadm init:
# 1. Validates system setup
# 2. Generates certificates & signing keys
# 3. Creates config files
# 4. Starts static pods (API server, controller, scheduler)
# 5. Waits for API server to be healthy
# 6. Creates kubeconfig file
# 7. Prints join command for worker nodes
````

#### Setup kubeconfig

````bash
mkdir -p $HOME/.kube
# WHAT: Create .kube directory in home folder
# WHY: kubectl looks here for config by default
# HOW: -p flag creates parent dirs if needed

sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
# WHAT: Copy cluster admin config to user's home
# WHY: Gives kubectl credentials to control cluster
# HOW BROKEN DOWN:
#   sudo cp: Copy as root (file is owned by root)
#   -i: Interactive (ask before overwriting)
#   /etc/kubernetes/admin.conf: Kubernetes config (admin token)
#   $HOME/.kube/config: Your user's kubectl config
# RESULT: You can run kubectl commands

sudo chown $(id -u):$(id -g) $HOME/.kube/config
# WHAT: Change config file owner to current user
# WHY: You need to read/write the file
# HOW BROKEN DOWN:
#   $(id -u): Get current user's ID
#   $(id -g): Get current group's ID
#   chown: Change ownership
# RESULT: Config file is now yours to manage

export KUBECONFIG=$HOME/.kube/config
# WHAT: Set environment variable pointing to config
# WHY: kubectl uses this to find cluster credentials
# HOW: Tells kubectl "use this config file"
# RESULT: kubectl commands now work
````

---

### **Step 5: Network Setup**

````bash
echo "Step 5: Apply Flannel Network"

kubectl apply -f https://github.com/coreos/flannel/raw/master/Documentation/kube-flannel.yml
# WHAT: Install Flannel CNI (Container Network Interface)
# WHY: Provides networking between pods across nodes
# HOW BROKEN DOWN:
#   kubectl apply: Create Kubernetes resources
#   -f: From file (URL in this case)
#   kube-flannel.yml: YAML manifest file
# WHAT FLANNEL DOES:
#   ├─ Creates virtual network for all pods
#   ├─ Assigns IP to each pod
#   ├─ Routes packets between pods
#   └─ Handles pod-to-pod communication
# RESOURCES CREATED:
#   ├─ DaemonSet (runs on every node)
#   ├─ ConfigMap (Flannel configuration)
#   ├─ ServiceAccount (permissions)
#   ├─ ClusterRole (permissions)
#   └─ Network policies

kubectl taint nodes $(hostname) node-role.kubernetes.io/control-plane:NoSchedule-
# WHAT: Remove control plane taint
# WHY: By default, pods can't run on control plane (master)
# HOW BROKEN DOWN:
#   kubectl taint: Manage node taints
#   nodes: Apply to nodes
#   $(hostname): Current machine
#   node-role.kubernetes.io/control-plane: Taint name
#   :NoSchedule-: Remove the NoSchedule taint (- = remove)
# DEFAULT: Control plane nodes have NoSchedule taint
#   REASON: Production clusters keep master for control only
# AFTER: Pods can now be scheduled on this node
# ⚠️ NOTE: Single-node clusters NEED this (no other nodes!)

echo "Kubernetes cluster setup is complete!"
# WHAT: Success message
# WHY: Let user know script finished
````

---

## 🎨 Visual Summary

```
┌─────────────────────────────────────────────────────────┐
│ Step 1: Install Kubernetes Components                   │
├─────────────────────────────────────────────────────────┤
│ ├─ Add GPG keys (verify packages are authentic)         │
│ ├─ Add Kubernetes repository                            │
│ └─ Install: kubectl, kubeadm, kubelet v1.33.0          │
└─────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────┐
│ Step 2: System Configuration                            │
├─────────────────────────────────────────────────────────┤
│ ├─ Disable swap (Kubernetes requirement)               │
│ ├─ Load overlay module (container layering)            │
│ ├─ Load br_netfilter module (network filtering)        │
│ └─ Configure kernel networking (IP forwarding)         │
└─────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────┐
│ Step 3: Install Container Runtime                       │
├─────────────────────────────────────────────────────────┤
│ ├─ Add Docker repository (has containerd.io)           │
│ ├─ Install containerd.io                               │
│ ├─ Configure containerd (use systemd cgroups)          │
│ └─ Restart & enable containerd service                 │
└─────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────┐
│ Step 4: Initialize Kubernetes Cluster                   │
├─────────────────────────────────────────────────────────┤
│ ├─ Pre-download container images (images pull)         │
│ ├─ Initialize control plane (kubeadm init)             │
│ ├─ Copy kubeconfig to user's home                      │
│ └─ Set up kubectl credentials                          │
└─────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────┐
│ Step 5: Install Networking                              │
├─────────────────────────────────────────────────────────┤
│ ├─ Install Flannel (pod networking plugin)             │
│ └─ Remove control plane taint (allow pod scheduling)   │
└─────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────┐
│ ✅ Kubernetes v1.33.0 Cluster Ready!                    │
├─────────────────────────────────────────────────────────┤
│ Ready to deploy applications                            │
└─────────────────────────────────────────────────────────┘
```

---

## 🔍 Key Concepts Explained

| Concept | What | Why | How |
|---------|------|-----|-----|
| **GPG Keys** | Digital signatures | Verify packages are authentic | `gpg --dearmor` converts keys |
| **Swap** | Disk memory | Slows containers | `swapoff -a` disables it |
| **Modules** | Kernel features | Enable functionality | `modprobe` loads them |
| **sysctl** | Kernel parameters | Network configuration | `/etc/sysctl.d/` applies at boot |
| **containerd** | Container runtime | Manages containers | CRI socket communicates with it |
| **kubeadm init** | Cluster setup | Creates control plane | Generates certs, starts services |
| **kubeconfig** | Credentials | kubectl authentication | Stores tokens and certificates |
| **Flannel** | CNI plugin | Pod networking | Creates virtual network |
| **Taint** | Node restriction | Control pod placement | `NoSchedule` prevents scheduling |

---

**This is production-grade Kubernetes setup!** ☸️

#Kubernetes #K8s #DevOps #Containerization #Linux