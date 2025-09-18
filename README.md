# <img src="https://raw.githubusercontent.com/kubernetes/kubernetes/master/logo/logo.svg" alt="Kubernetes Logo" width="40" height="40"/> Kubernetes Overview

## 1️⃣ What is Kubernetes?

**Kubernetes** (also known as **K8s**) is an open-source platform that helps you **automate the deployment, scaling, and management** of containerized applications.

Instead of managing containers manually, Kubernetes takes care of running your apps in groups called **pods**, makes sure they stay healthy, restarts them if needed, and helps them talk to each other—**across many servers** (nodes).

---

## 2️⃣ Why Use Kubernetes Instead of Docker Swarm?

While both Kubernetes and Docker Swarm can manage containers, Kubernetes is more powerful, especially for **large and complex systems**.

| Feature                | Kubernetes                            | Docker Swarm                      |
|------------------------|----------------------------------------|-----------------------------------|
| **Scalability**        | Highly scalable and enterprise-ready   | Less scalable                     |
| **Load Balancing**     | Built-in advanced load balancing       | Basic                             |
| **Community Support**  | Huge, active, growing community        | Smaller community                 |
| **Self-healing**       | Auto-restarts, rescheduling, replicas  | Basic recovery options            |
| **Declarative configs**| Uses YAML to manage everything         | Limited                           |
| **Rolling updates**    | Natively supported                     | Basic                             |

Kubernetes may be more complex at first, but it offers **powerful tools and control** for real-world production systems.

---

## 3️⃣ 🧠 Kubernetes Architecture (Made Simple)

Kubernetes is made of **two main parts**:  
→ The **Control Plane (Master)** – makes decisions  
→ The **Worker Nodes** – actually run your applications

### 🔷 1. Control Plane Components

These control everything in the cluster:

- **API Server**  
  Receives all commands (like from `kubectl`). It’s the front door of the cluster.

- **Scheduler**  
  Picks the best node to run new pods based on available resources.

- **Controller Manager**  
  Watches the cluster and makes sure things match the desired state (e.g., restarts crashed pods).

- **etcd**  
  A simple database that stores the entire state of the cluster (pods, nodes, configs, etc.).

### 🔶 2. Worker Node Components

These are the machines that actually **run your containers**:

- **Kubelet**  
  Talks to the API Server. It gets pod instructions and tells the container runtime what to do.

- **Container Runtime**  
  Actually runs your containers (e.g., Docker, containerd).

- **Kube-proxy**  
  Manages network traffic so pods can talk to each other and to the internet.

---

## 🔄 How It All Works Together

Let’s break it down step by step:

1. You write a deployment config (YAML) and apply it using `kubectl`.
2. The **API Server** receives it and saves it to **etcd**.
3. The **Scheduler** checks which worker node has space and assigns the pod.
4. The **API Server** tells that node’s **Kubelet** to create the pod.
5. The **Kubelet** starts the container using the **Container Runtime**.
6. The **Kube-proxy** sets up networking so other pods and services can talk to it.
7. The **Controller Manager** checks if the pod is running as expected. If not, it fixes it.

> ✅ Kubernetes always works to match your **desired state** (what you want) with the **actual state** (what's running), and automatically makes corrections when needed.

---

## 📡 Communication Between Components

- All communication goes through the **API Server**.
- Control Plane tells Worker Nodes what to do.
- **Pods inside nodes** use **Kube-proxy** to talk to each other across nodes.
- **etcd** is the memory of the cluster – storing everything.

---

Here's a clean and professional version of the **Minikube Installation Guide** that you can include in your `README.md` file:

---

## 🚀 Minikube Installation Guide

Minikube makes it easy to run Kubernetes locally. All you need is Docker (or a compatible container/VM manager), and Kubernetes is just a single command away:

```bash
minikube start
```

---

### 🛠️ What You’ll Need

- 2 CPUs or more  
- 2GB of free memory  
- 20GB of free disk space  
- Internet connection  
- Container or VM manager, such as:
  - Docker - QEMU - Hyperkit - Hyper-V - KVM  - Parallels - Podman - VirtualBox - VMware Fusion/Workstation
---

### 🐧 Installation (Linux-based VM)

1. **Download Minikube:**
```bash
curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-amd64
```

2. **Install it:**
```bash
sudo install minikube-linux-amd64 /usr/local/bin/minikube
rm minikube-linux-amd64
```

---

### 🚀 Start Your Cluster

From a terminal with administrator access (but not logged in as `root`):

```bash
minikube start
```

---

### 🧑‍💻 Install `kubectl` (Kubernetes CLI)

To interact with your Minikube cluster, you’ll need `kubectl`.

#### Option 1: Install the actual `kubectl` binary

```bash
curl -LO "https://dl.k8s.io/release/$(curl -Ls https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```

#### Option 2: Use `minikube kubectl`

You can also run:

```bash
minikube kubectl -- get po -A
```

To always use `kubectl` instead of `minikube kubectl`, you can create a symbolic link:

```bash
sudo ln -s $(which minikube) /usr/local/bin/kubectl
```

> ⚠️ This will alias the `minikube` binary as `kubectl`, which is **not recommended** for production use. Prefer installing the real `kubectl` binary instead.

---

### ✅ Verify Your Setup

```bash
kubectl get nodes
kubectl get componentstatus
kubectl get pods
```

---

---

# 🚀 Kubernetes Installation on EC2 (Real-Time Guide)

This guide explains how to install a **Kubernetes cluster** on Amazon EC2 using **kubeadm** with **Docker as container runtime** (`cri-dockerd`).
It covers **Master Node setup**, **Worker Node joining**, and **reset steps**.



## 🔹 Example Architecture Diagram (ASCII Style)



```
             ┌───────────────────────────┐
             │   Control Plane (Master)   │
             │ ───────────────────────── │
             │  • kube-apiserver          │
             │  • etcd                    │
             │  • controller-manager      │
             │  • scheduler               │
             └─────────────┬─────────────┘
                           │
              Flannel CNI (Pod Network)
                           │
   ┌───────────────────────┴───────────────────────┐
   │                                               │
┌──▼──────────────┐                        ┌───────▼──────────────┐
│ Worker Node #1  │                        │ Worker Node #2       │
│  • kubelet      │                        │  • kubelet           │
│  • kube-proxy   │                        │  • kube-proxy        │
│  • Pods         │                        │  • Pods              │
└─────────────────┘                        └──────────────────────┘
```
---

## 📑 Table of Contents

* [🔧 Master Node Installation](#-master-node-installation)

  * [Step 1: Update and disable swap](#step-1-update-and-disable-swap)
  * [Step 2: Install Docker](#step-2-install-docker)
  * [Step 3: Add Kubernetes repository](#step-3-add-kubernetes-repository)
  * [Step 4: Install Kubernetes components](#step-4-install-kubernetes-components)
  * [Step 5: Initialize Kubernetes (Master)](#step-5-initialize-kubernetes-master)
  * [Step 6: Configure kubectl](#step-6-configure-kubectl)
  * [Step 7: Grant ec2-user kube access](#step-7-grant-ec2-user-kube-access)
  * [Step 8: Install Flannel CNI](#step-8-install-flannel-cni-pod-network)
  * [Step 9: Verify nodes](#step-9-verify-nodes)
* [🖥 Worker Node Installation](#-worker-node-installation)

  * [Step 1: Update and disable swap](#step-1-update-and-disable-swap-1)
  * [Step 2: Install Docker](#step-2-install-docker-1)
  * [Step 3: Add Kubernetes repository](#step-3-add-kubernetes-repository-1)
  * [Step 4: Install Kubernetes components](#step-4-install-kubernetes-components-1)
  * [Step 5: Join Worker Node to Master](#step-5-join-worker-node-to-master)
* [🔄 Reset Master Node (if needed)](#-reset-master-node-if-needed)

---

## 🔧 Master Node Installation

### Step 1: Update and disable swap

```bash
sudo yum update -y
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

### Step 2: Install Docker

```bash
sudo yum install -y docker
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker ec2-user
```

### Step 3: Add Kubernetes repository

```bash
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.29/rpm/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.29/rpm/repodata/repomd.xml.key
EOF
```

✅ Verify repository:

```bash
cat /etc/yum.repos.d/kubernetes.repo
```

### Step 4: Install Kubernetes components

```bash
sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
sudo systemctl enable kubelet
```

### Step 5: Initialize Kubernetes (Master)

👉 For Docker runtime:

```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --cri-socket unix:///var/run/cri-dockerd.sock
```

### Step 6: Configure kubectl

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
```

### Step 7: Grant ec2-user kube access

```bash
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### Step 8: Install Flannel CNI (Pod Network)

```bash
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

### Step 9: Verify nodes

```bash
kubectl get nodes
```

---

## 🖥 Worker Node Installation

### Step 1: Update and disable swap

```bash
sudo yum update -y
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

### Step 2: Install Docker

```bash
sudo yum install docker -y
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker ec2-user
```

### Step 3: Add Kubernetes repository

```bash
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.29/rpm/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.29/rpm/repodata/repomd.xml.key
EOF
```

### Step 4: Install Kubernetes components

```bash
sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
sudo systemctl enable kubelet
```

### Step 5: Join Worker Node to Master

> ⚠️ Replace `<CONTROL-PLANE-IP>`, `<TOKEN>`, and `<HASH>` with the values from your `kubeadm init` output.

## On the master (control plane) node, you can always generate the worker node join command with this:

```bash
kubeadm token create --print-join-command
```
# Note: use this command at the end of join join command("-cri-socket unix:///var/run/cri-dockerd.sock") to run docker as container run. By default K8 use containerd as conatiner run time from v1.24.
### Below join token is used to join worker node to master after running all above commands( you join N no of worker node to master like this using join token)
sudo kubeadm join 172.31.39.190:6443 --token b6ujsf.7bod2eanho1mc0rj --discovery-token-ca-cert-hash sha256:0bafe7570e932da737eadaf75e1d323a676f22d7482b37ad1a200f04e86b4a67 --cri-socket unix:///var/run/cri-dockerd.sock

---

## 🔄 Reset Master Node (if needed)

```bash
sudo kubeadm reset
```

---

✨ That’s it! Your Kubernetes cluster is ready 🎉

---

### 🔹 Example Screenshot Sections

````markdown
### ✅ Verify Cluster
```bash
kubectl get nodes -o wide
````

**Output:**

```
NAME                                         STATUS   ROLES           AGE   VERSION
ip-10-0-1-186.ap-south-1.compute.internal    Ready    control-plane   10m   v1.29.15
ip-10-0-1-167.ap-south-1.compute.internal    Ready    <none>          5m    v1.29.15
```

Let me know if you want this formatted for a GitHub README (with badges or markdown enhancements), or if you want to add a section for Windows/macOS too.

## 📚 Want to Learn More?

- [Kubernetes Official Docs](https://kubernetes.io/docs/)
- [Play with Kubernetes](https://labs.play-with-k8s.com/)
- [Awesome Kubernetes GitHub](https://github.com/ramitsurana/awesome-kubernetes)
