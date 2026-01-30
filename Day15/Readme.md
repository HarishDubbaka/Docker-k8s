#  Kubernetes Cluster Setup and Installation (Using Kind)

## Overview

Kubernetes is a powerful container orchestration platform used to deploy, manage, and scale containerized applications. A Kubernetes cluster consists of:

* **Control Plane (Master Node)** – manages the cluster state
* **Worker Nodes** – run application workloads (Pods)

For local development and learning purposes, setting up a full production Kubernetes cluster is complex. **Kind (Kubernetes IN Docker)** simplifies this by running Kubernetes clusters inside Docker containers.

This guide explains:

* Installing Kind
* Creating single-node and multi-node clusters
* Working with multiple clusters
* Context switching
* What happens when a cluster is deleted
* Basic troubleshooting

---

## Why Kind?

Kind is designed for:

* Learning Kubernetes
* Practicing CKA/CKAD
* Testing manifests locally
* Running multi-node clusters on a single machine

Kind is **not meant for production**.

---

## Prerequisites

Make sure the following tools are installed:

### 1. Docker

Kind uses Docker to create Kubernetes nodes as containers.

Docker must be **installed and running**.

Check:

```bash
docker ps
```

---

### 2. kubectl

`kubectl` is the command-line tool used to interact with the Kubernetes API server.

> ⚠️ Kind does not require kubectl to create a cluster,
> but you **cannot manage or interact with the cluster without it**.

---

### 3. Kind

Kind is the tool that creates Kubernetes clusters inside Docker.

---

## Step 1: Install Kind

### On Windows (PowerShell)

```powershell
curl.exe -Lo kind-windows-amd64.exe https://kind.sigs.k8s.io/dl/v0.31.0/kind-windows-amd64
Move-Item .\kind-windows-amd64.exe C:\some-dir-in-your-PATH\kind.exe
```

---

### On Windows (Chocolatey)

```powershell
choco install kind
```

Verify installation:

```bash
kind version
```

---

## Step 2: Create a Kubernetes Cluster

### Create a Default Cluster

```bash
kind create cluster
```

### Explanation

* Creates a single-node Kubernetes cluster
* The control-plane runs inside a Docker container
* kubeconfig is updated automatically

This setup is ideal for beginners.

---

### Create a Cluster with a Custom Name

```bash
kind create cluster --name cka-devcluster
```

![Image Alt](https://github.com/HarishDubbaka/Docker-k8s/blob/6351dd4a5f082ee3531bfa1de511e9eca61c09f9/Day15/devcluster%20i.png)


### Why use names?

* Helps manage multiple clusters
* Useful for dev, QA, and testing environments

---

## Step 3: Interact with the Cluster

### Check Cluster Nodes

```bash
kubectl get nodes
```

![Image Alt](https://github.com/HarishDubbaka/Docker-k8s/blob/d8a4f8ec3c81af4da82399b258dc1a3db1f79ab3/Day15/devnodes.png)

This confirms:

* Cluster is running
* Node status is `Ready`

---

### View Cluster Information

```bash
kubectl cluster-info --context kind-cka-devcluster
```

Example output:

```text
Kubernetes control plane is running at https://127.0.0.1:58105
CoreDNS is running at https://127.0.0.1:58105/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```

---

## Customizing Your Kind Cluster

By default, Kind creates only a control-plane node.
To simulate real-world Kubernetes behavior, you can create **multi-node clusters** using a configuration file.

---

## Step 4: Create a Multi-Node Cluster

### Configuration File (`config.yaml`)

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
```

### Explanation

* 1 control-plane node manages the cluster
* 2 worker nodes run application Pods
* This mimics a production-like environment

---

### Create the Cluster

```bash
kind create cluster --name cka-qacluster --config config.yaml
```

![Image Alt](https://github.com/HarishDubbaka/Docker-k8s/blob/097fde04922ffa36a79d4616f2f29d6ff9990266/Day15/qacluster.png)

---

### Verify the Cluster

```bash
kubectl get nodes
kubectl cluster-info --context kind-cka-qacluster
```

![Image Alt](https://github.com/HarishDubbaka/Docker-k8s/blob/d8a4f8ec3c81af4da82399b258dc1a3db1f79ab3/Day15/qavnodes.png)

---

## Working with Multiple Clusters (Contexts)

Kubernetes stores cluster access details as **contexts** in kubeconfig.

### List All Contexts

```bash
kubectl config get-contexts
```

![Image Alt](https://github.com/HarishDubbaka/Docker-k8s/blob/d8a4f8ec3c81af4da82399b258dc1a3db1f79ab3/Day15/context.png)

---

### Switch to Dev Cluster

```bash
kubectl config use-context kind-cka-devcluster
```

---

### Switch to QA Cluster

```bash
kubectl config use-context kind-cka-qacluster
```

### Why Contexts Matter

* Prevent accidental deployments to the wrong cluster
* Essential when managing multiple environments

---

## Deleting a Kind Cluster

```bash
kind delete cluster --name cka-devcluster
```

### What Happens When You Delete a Cluster?

* Entire Kubernetes cluster is destroyed
* Docker containers acting as nodes are removed
* All Pods, Services, Deployments stop immediately
* No data persistence unless volumes were explicitly mounted

👉 Kind clusters are **ephemeral by design**.

---

## Kind vs Production Kubernetes

| Feature        | Kind               | Production       |
| -------------- | ------------------ | ---------------- |
| Purpose        | Learning & Testing | Live Workloads   |
| Persistence    | No                 | Yes              |
| Node Type      | Docker Containers  | VMs / Bare Metal |
| Safe to Delete | Yes                | No               |

---

## Troubleshooting

### Docker Not Running

```bash
docker ps
```

Docker must be running before creating a Kind cluster.

---

### Insufficient Resources

Multi-node clusters require sufficient:

* CPU
* Memory

✔ Fix:

* Increase Docker resource limits
* Reduce worker node count

---

### Logs and Diagnostics

```bash
docker logs <container-id>
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl cluster-info dump
```

---

## Conclusion

Kind provides a simple, fast, and safe way to run Kubernetes locally. It is ideal for:

* Learning Kubernetes concepts
* Practicing CKA/CKAD
* Testing deployments and configurations
* Running multiple clusters on a single machine

Using Kind, you can confidently experiment with Kubernetes without impacting production systems.

---

