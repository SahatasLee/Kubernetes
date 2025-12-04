# Production Kubernetes Installation with Kubeadm

This guide details the steps to bootstrap a production-ready Kubernetes cluster using `kubeadm` on **Ubuntu/Debian** systems.

## Prerequisites

*   **Machines**: At least 2 machines (1 Control Plane, 1+ Workers).
*   **OS**: Ubuntu 20.04+ or Debian 11+.
*   **Hardware**: 2 CPU, 2GB RAM minimum per node.
*   **Network**: Unique hostname, MAC address, and product_uuid for every node. Full network connectivity between machines.
*   **Swap**: Must be disabled.

## 1. System Preparation (All Nodes)

Perform these steps on **all** nodes (control plane and workers).

### Disable Swap
Kubernetes requires swap to be disabled.
```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

### Enable IPv4 Forwarding & Bridging
Required for container networking.
```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# sysctl params required by setup, params persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system
```

## 2. Install Container Runtime (All Nodes)

We will use **containerd**.

### Install containerd
```bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

# Install containerd
sudo apt-get install -y containerd.io
```

### Configure containerd
Generate default config and enable SystemdCgroup.
```bash
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml

# Set SystemdCgroup = true
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml

sudo systemctl restart containerd
```

## 3. Install Kubeadm, Kubelet, Kubectl (All Nodes)

```bash
sudo apt-get update
# apt-transport-https may be a dummy package; if so, you can skip that package
sudo apt-get install -y apt-transport-https ca-certificates curl gpg

# Download the public signing key for the Kubernetes package repositories
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

# Add the appropriate Kubernetes apt repository
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

## 4. Initialize Control Plane (Master Node Only)

Run this **only** on the control plane node.

```bash
# Replace <CONTROL_PLANE_IP> with the IP of this machine
sudo kubeadm init --pod-network-cidr=192.168.0.0/16 --apiserver-advertise-address=<CONTROL_PLANE_IP>
```

**Post-Init Configuration:**
To start using your cluster, run the following as a regular user:
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

**Save the Join Command:**
The output of `kubeadm init` will give you a `kubeadm join` command. **Save this!** You will need it to join worker nodes.

## 5. Install Pod Network Add-on (Master Node Only)

Install **Calico** for networking.

```bash
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/tigera-operator.yaml
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/custom-resources.yaml
```
*Note: Ensure the CIDR in `custom-resources.yaml` matches your `--pod-network-cidr` (default is usually 192.168.0.0/16).*

Wait for pods to be ready:
```bash
kubectl get pods -n calico-system -w
```

## 6. Join Worker Nodes

Run the join command you saved from Step 4 on each **worker node**.

```bash
sudo kubeadm join <CONTROL_PLANE_IP>:6443 --token <TOKEN> --discovery-token-ca-cert-hash sha256:<HASH>
```

## 7. Verification

On the **Control Plane** node:

```bash
kubectl get nodes
```
You should see all nodes in `Ready` status.
