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

> **Note:** In this demo, we will be installing Kubernetes on cloud VMs (self‑managed).

---

## 🛠️ Steps to Set Up the Kubernetes Cluster

![Cluster Setup Steps](https://github.com/user-attachments/assets/e0943ad5-2d13-4128-8147-1ef644c62955)

> **Note:**  
> - On Mac Silicon chips, **Multipass** is recommended since VirtualBox has compatibility issues.  
> - Alternatively, you can spin up virtual machines on the cloud and use those.

---


-
