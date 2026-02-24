# Kubernetes Cluster Setup with Kubeadm

Kubeadm is an excellent tool to set up a working Kubernetes cluster in less time. It handles the heavy lifting of configuring Kubernetes components while following best practices for cluster setup.

---

## 📌 What is Kubeadm?

`kubeadm` is a tool used to bootstrap a minimum viable Kubernetes cluster without complex configuration.

It:

- Runs preflight checks
- Installs control plane components
- Configures cluster certificates
- Sets up secure communication between nodes

It is developed and maintained by the official Kubernetes community.

Other options like **Minikube** or **Kind** are easier for lightweight testing.  
However, if you want to experiment with cluster components or test administration utilities, **Kubeadm** is the best choice. It allows you to build a production-like cluster locally or on cloud VMs.

---

## ⚙️ Control Plane Components Installed by Kubeadm

- API Server
- ETCD
- Controller Manager
- Scheduler

CLI tools installed:

- kubeadm
- kubelet
- kubectl

---

## 🚀 Ways to Install Kubernetes

![Kubernetes Installation Options](https://github.com/user-attachments/assets/5391de53-36bc-4574-81cf-4b1fefbda9e3)

> **Note:** In this guide, Kubernetes is installed on Azure Cloud VMs (self-managed).

---

# Setting Up a Kubernetes Cluster with Kubeadm on Azure Virtual Machines

## 🏗️ Cluster Architecture

- 1 Master Node → `harish-master`
- 2 Worker Nodes → `bantu-worker1`, `chintu-worker2`
- OS → Ubuntu 22.04 LTS
- Kubernetes Version → 1.29.6
- Container Runtime → containerd
- CNI Plugin → Calico
- Networking → Azure VNet (10.0.0.0/16)

---

# Part 1: Creating Virtual Machines in Azure

## Step 1: Login to Azure Portal

Visit:

```

[https://portal.azure.com](https://portal.azure.com)

````

Sign in with your Azure account.

---

## Step 2: Create a Resource Group

1. Go to **Resource Groups**
2. Click **+ Create**
3. Configure:
   - Resource Group Name: `K8s-RG`
   - Region: Choose nearest region
4. Click **Review + Create → Create**

---

## Step 3: Create Virtual Network (VNet)

1. Go to **Virtual Networks**
2. Click **+ Create**
3. Configure:
   - Name: `harish-vnet`
   - Address Space: `10.0.0.0/16`
   - Subnet: `10.0.0.0/24`

---

## Step 4: Create Master Node VM

- Name: `harish-master`
- Image: Ubuntu Server 22.04 LTS
- Size: Standard_D2s_v3 (2 vCPUs, 8GB RAM)
- Authentication: Password
- Allow SSH (Port 22)
- Attach to `harish-vnet`
- Tag: `Role=master`

Click **Review + Create**

---

## Step 5: Create Worker Nodes

Repeat the same process:

- `bantu-worker1`
- `chintu-worker2`
- Tag: `Role=worker`

---

## Step 6: Open Required Ports (NSG)

Allow:

- TCP 6443 (Kubernetes API Server)
- Internal VNet traffic: `10.0.0.0/16`

---

## Step 7: Note Down Public IPs

Used for SSH access.

Example:

- harish-master → 40.xx.xx.xx
- bantu-worker1 → 40.xx.xx.xx
- chintu-worker2 → 40.xx.xx.xx

---

# Master Node Setup

SSH into master:

```bash
ssh <username>@<master-public-ip>
````

---

## 1️⃣ Disable Swap

```bash
swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

Kubernetes requires swap to be disabled.

---

## 2️⃣ Enable Kernel Modules

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
```

---

## 3️⃣ Configure Sysctl

```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system
```

---

## 4️⃣ Install Container Runtime (containerd)

Verify containerd:

```bash
systemctl status containerd
```

Ensure:

```
SystemdCgroup = true
```

in `/etc/containerd/config.toml`.

---

## 5️⃣ Install runc

```bash
curl -LO https://github.com/opencontainers/runc/releases/download/v1.1.12/runc.amd64
sudo install -m 755 runc.amd64 /usr/local/sbin/runc
```

---

## 6️⃣ Install CNI Plugins

```bash
curl -LO https://github.com/containernetworking/plugins/releases/download/v1.5.0/cni-plugins-linux-amd64-v1.5.0.tgz
sudo mkdir -p /opt/cni/bin
sudo tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.5.0.tgz
```

---

## 7️⃣ Install Kubernetes Binaries (v1.29.6)

```bash
sudo apt-get update
sudo apt-get install -y kubelet=1.29.6-1.1 kubeadm=1.29.6-1.1 kubectl=1.29.6-1.1
sudo apt-mark hold kubelet kubeadm kubectl
```

Verify:

```bash
kubeadm version
kubelet --version
kubectl version --client
```

---

## 8️⃣ Configure crictl

```bash
sudo crictl config runtime-endpoint unix:///var/run/containerd/containerd.sock
```

---

## 9️⃣ Initialize Control Plane

Check IP:

```bash
ip addr show eth0
```

Example:

```
10.0.0.4
```

Initialize:

```bash
sudo kubeadm init \
  --pod-network-cidr=192.168.0.0/16 \
  --apiserver-advertise-address=10.0.0.4 \
  --node-name master
```

⚠️ Do NOT use `127.0.0.1`.

Copy the generated `kubeadm join` command.

---

## 🔟 Configure kubeconfig

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

---

## 1️⃣1️⃣ Install Calico CNI

```bash
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/tigera-operator.yaml

curl -O https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/custom-resources.yaml

kubectl apply -f custom-resources.yaml
```

Master node is now ready.

---

# Worker Node Setup

SSH:

```bash
ssh <username>@<worker-public-ip>
```

Repeat:

* Disable swap
* Enable kernel modules
* Configure sysctl
* Install containerd
* Install runc
* Install CNI plugins
* Install kubeadm, kubelet, kubectl
* Configure crictl

---

## Join Worker Nodes to Cluster

```bash
sudo kubeadm join 10.0.0.4:6443 --token <token> \
  --discovery-token-ca-cert-hash sha256:<hash>
```

If forgotten:

```bash
kubeadm token create --print-join-command
```

---

# ✅ Verify Cluster

On master:

```bash
kubectl get nodes
```

Expected:

* master → Ready
* bantu-worker1 → Ready
* chintu-worker2 → Ready

Check pods:

```bash
kubectl get pods -A
```

---

# 🚀 Test with NGINX

```bash
kubectl create deployment nginx --image=nginx
kubectl expose deployment nginx --port=80 --type=NodePort
kubectl get svc nginx
```

Access:

```
http://[WORKER_PUBLIC_IP]:[NODE_PORT]
```

---

# 📈 Scaling the Deployment

```bash
kubectl scale deployment nginx --replicas=2
```

Results:

* Multiple pods created
* Distributed across worker nodes
* NodePort load balances traffic
* High availability achieved

---

# 🔎 Cluster Validation Checklist

* kubectl get nodes → All Ready
* kubectl get pods -A → All Running
* kubectl get svc → NodePort accessible
* Pods distributed across nodes
* Calico pods running in kube-system namespace

---

🎉 Your Kubernetes cluster on Azure VMs using kubeadm is now fully operational.

```

