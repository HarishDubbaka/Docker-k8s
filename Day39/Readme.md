# Kubernetes Cluster Setup with Kubeadm

Kubeadm is an excellent tool to set up a working Kubernetes cluster in less time. It does all the heavy lifting in terms of setting up all Kubernetes cluster components and follows best practices for cluster configuration.

---

## 📌 What is Kubeadm?

`kubeadm` is a tool to bootstrap a minimum viable Kubernetes cluster without complex configuration. It simplifies the process by running a series of prechecks to ensure the server has all the essential components and configurations to run Kubernetes.

It is developed and maintained by the official Kubernetes community.  
Other options like **Minikube** or **Kind** are easier to set up and are good for lightweight testing. However, if you want to experiment with cluster components or test utilities related to cluster administration, **Kubeadm** is the best option. It allows you to create a production‑like cluster locally or on cloud VMs for development and testing.

---

## ⚙️ Control Plane Components Installed by Kubeadm

- **API Server**
- **ETCD**
- **Controller Manager**
- **Scheduler**

Along with these, it helps install CLI tools such as:
- `kubeadm`
- `kubelet`
- `kubectl`

---

## 🚀 Ways to Install Kubernetes

![Kubernetes Installation Options](https://github.com/user-attachments/assets/5391de53-36bc-4574-81cf-4b1fefbda9e3)

> **Note:** In this demo, we will be installing Kubernetes on **Azure cloud VMs (self‑managed)**.

---

## 🛠️ Steps to Set Up the Kubernetes Cluster

![Cluster Setup Steps](https://github.com/user-attachments/assets/e0943ad5-2d13-4128-8147-1ef644c62955)


## 🖥️ Prerequisites for Azure VM Setup

Before bootstrapping Kubernetes with Kubeadm, ensure the following:

- **Operating System**: Ubuntu 22.04 LTS (Jammy) recommended
- **Master Node**: 2 CPUs, 4 GB RAM minimum
- **Worker Nodes**: 1 CPU, 2 GB RAM minimum
- **Networking**:
  - All nodes must be on the same virtual network
  - Ensure DNS resolution between nodes
- **Swap**: Disabled on all nodes
- **Firewall / NSG Rules**:
  - Allow inbound traffic on required Kubernetes ports:
    - `6443/tcp` – Kubernetes API server
    - `10250/tcp` – Kubelet API
    - `10251/tcp` – kube-scheduler
    - `10252/tcp` – kube-controller-manager
    - `10255/tcp` – Read-only Kubelet API
    - `30000–32767/tcp` – NodePort Services

---

## 📋 Next Steps

1. Provision Azure VMs and configure networking.
2. Install container runtime (`containerd`) on all nodes.
3. Install Kubernetes tools (`kubeadm`, `kubelet`, `kubectl`).
4. Initialize the master node with `kubeadm init`.
5. Install a CNI plugin (e.g., Calico).
6. Join worker nodes using the `kubeadm join` command.
7. Verify cluster health with `kubectl get nodes`.

---
