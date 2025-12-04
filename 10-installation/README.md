# Kubernetes Installation Guide

This guide covers various methods to install and set up a Kubernetes cluster, ranging from local development environments to production-ready bare metal setups.

## Prerequisites

Before you begin, ensure you have the following tools installed:

*   **Docker**: Container runtime. [Install Docker](https://docs.docker.com/get-docker/)
*   **kubectl**: Kubernetes command-line tool. [Install kubectl](https://kubernetes.io/docs/tasks/tools/)

Verify installation:
```bash
docker --version
kubectl version --client
```

---

## 1. Local Development Environments

For learning and local development, lightweight clusters are recommended.

### Option A: Kind (Kubernetes in Docker)

Kind runs Kubernetes nodes as Docker containers. It's fast and great for CI/CD.

**Installation:**
```bash
# macOS
brew install kind

# Windows
choco install kind

# Linux
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```

**Create a Cluster:**
```bash
kind create cluster --name my-cluster
```

**Interact:**
```bash
kubectl cluster-info --context kind-my-cluster
```

### Option B: Minikube

Minikube creates a VM (or container) to run a single-node cluster. It supports many add-ons.

**Installation:**
```bash
# macOS
brew install minikube

# Windows
choco install minikube

# Linux
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

**Start Cluster:**
```bash
minikube start
```

**Interact:**
```bash
minikube kubectl -- get pods -A
```

---

## 2. Production / Bare Metal

For production environments or when you need to manage the underlying infrastructure manually.

### Kubeadm

Kubeadm is a tool built to provide `kubeadm init` and `kubeadm join` as best-practice "fast paths" for creating Kubernetes clusters.

**[👉 Click here for the full Kubeadm Production Installation Guide](./kubeadm-production.md)**

This guide covers:
1.  System Preparation (Swap, Networking)
2.  Installing Container Runtime (containerd)
3.  Installing Kubeadm, Kubelet, Kubectl
4.  Initializing the Control Plane
5.  Setting up Calico Networking
6.  Joining Worker Nodes

For the official documentation, visit: [Bootstrapping clusters with kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)
