# Kubernetes Scheduling Deep Dive  

### Static Pods, Scheduler Behavior, Manual Scheduling, Labels & Annotations

## Introduction

In Kubernetes, **pod scheduling** plays a crucial role in ensuring applications run smoothly across a cluster.  
This document explains three important concepts:

- **Static Pods**
- **Manual Scheduling**
- **Labels & Selectors (and Annotations)**

Understanding these concepts will significantly improve your ability to **operate, troubleshoot, and reason about Kubernetes clusters**, especially in real-world and CKA-style scenarios.

---

## Kubernetes Scheduler Overview

In a Kubernetes cluster, the **scheduler** is a control plane component responsible for deciding **which node a pod should run on**.

### Scheduling Flow

1. A client submits a Pod request.
2. The **API Server** stores the Pod object in **etcd**.
3. The **scheduler** watches for Pods that do not have a node assigned.
4. Using scheduling algorithms, it selects the best node.
5. The scheduler updates the Pod object with the chosen node.
6. The **kubelet** on that node creates and runs the Pod.

### Key Responsibility
> The scheduler’s primary role is to assign pods to nodes while ensuring efficient resource utilization.

---

## Pods and High Availability

- Pods are the **smallest deployable units** in Kubernetes.
- Kubernetes is designed to tolerate failures:
  - **ReplicaSets / Deployments** maintain multiple replicas.
  - If one Pod fails, another replaces it.
  - Controllers automatically reschedule Pods onto healthy nodes.

### When Does the System Go Down?

The cluster is impacted only if:
- All replicas of a critical service fail simultaneously
- The **control plane** (API Server, Scheduler, etc.) fails
- There is a cluster-wide failure with no redundancy

---

## Test Setup: Kind Cluster

We are using a **Kind (Kubernetes in Docker)** cluster with:

- **1 control-plane node**
- **3 worker nodes**

```bash
kubectl get nodes
````

```
NAME                          STATUS   ROLES           AGE     VERSION
cka-qacluster-control-plane   Ready    control-plane   2d23h   v1.31.0
cka-qacluster-worker          Ready    <none>          2d23h   v1.31.0
cka-qacluster-worker2         Ready    <none>          2d23h   v1.31.0
cka-qacluster-worker3         Ready    <none>          2d23h   v1.31.0
```

---

## Scheduler as a Static Pod

The scheduler runs as a **static pod** on the control-plane node.

```bash
kubectl get pods -n kube-system | grep kube-scheduler
```

```
kube-scheduler-cka-qacluster-control-plane   1/1   Running
```

---

## What Happens If the Scheduler Goes Down?

* Existing Pods **continue running**
* New Pods **cannot be scheduled**
* Pods remain in **Pending** state
* Cluster is **degraded**, not down
* HA clusters usually have multiple scheduler instances

---

## Static Pods Explained

### What Are Static Pods?

Static Pods are:

* Managed **directly by the kubelet**
* Not scheduled by the Kubernetes scheduler
* Defined as YAML files on the node filesystem

### Important Notes

* The **kubelet is not part of the control plane**
* It runs on **every node**
* It watches a directory for static pod manifests

### Default Static Pod Location

```text
/etc/kubernetes/manifests
```

This directory usually contains:

* `etcd.yaml`
* `kube-apiserver.yaml`
* `kube-controller-manager.yaml`
* `kube-scheduler.yaml`

---

## Static Pods in Managed vs Self-Managed Clusters

### Managed Kubernetes (EKS, GKE, AKS, OpenShift)

* Control plane nodes are hidden
* You cannot access `/etc/kubernetes/manifests`
* Provider manages control plane components

### Self-Managed / Kind Cluster

* Control plane is accessible
* Static pod manifests are visible
* You can inspect and modify them

---

## Inspecting Static Pods in Kind

Access the control-plane container:

```bash
docker exec -it cka-qacluster-control-plane sh
```

```bash
cd /etc/kubernetes/manifests
ls -ltr
```

```
etcd.yaml
kube-apiserver.yaml
kube-scheduler.yaml
kube-controller-manager.yaml
```

---

## Simulating Scheduler Failure

Move the scheduler manifest out of the directory:

```bash
mv kube-scheduler.yaml /tmp
```

### Result

* Kubelet stops the scheduler pod
* Scheduler is no longer running

---

## Create a Pod While Scheduler Is Down

```bash
kubectl run nginxharish --image=nginx
```

```bash
kubectl get pods
```

```
nginxharish   0/1   Pending
```

### Why?

* No scheduler is running
* Pod has no `nodeName`
* No scheduling decision can be made

---

## Key Takeaway

* Existing Pods keep running
* New Pods stay **Pending**
* Cluster is **partially degraded**

👉 **Existing workloads are safe; new scheduling is blocked**

---

## Restoring the Scheduler

Move the manifest back:

```bash
mv /tmp/kube-scheduler.yaml /etc/kubernetes/manifests
```

### What Happens Next?

* Kubelet detects the file immediately
* Scheduler pod restarts
* Pending Pods get scheduled

```bash
kubectl get pods | grep nginxharish
```

```
nginxharish   1/1   Running
```

---

## Manual Scheduling (Bypassing the Scheduler)

Kubernetes allows **manual scheduling** using `nodeName`.

### How It Works

* Scheduler **ignores** Pods with `nodeName`
* Kubelet directly starts the Pod on the specified node
* Works **even if the scheduler is down**

### Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  nodeName: cka-qacluster-worker2
  containers:
    - name: my-container
      image: nginx
```

```bash
kubectl apply -f manualpod.yml
```

```bash
kubectl get pods -o wide
```

```
my-pod   ContainerCreating   cka-qacluster-worker2
```

---

## Kubernetes Labels

### What Are Labels?

Labels are **key-value pairs** attached to Kubernetes objects.

Example:

```yaml
labels:
  app: my-app
  tier: frontend
```

### Why Labels Matter

* Organize resources
* Enable Service routing
* Support deployment strategies
* Improve monitoring and logging

---

## Label Selectors

Selectors are used to **find Pods by labels**.

### Example Deployment and Service

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: my-app
      tier: frontend
  template:
    metadata:
      labels:
        app: my-app
        tier: frontend
    spec:
      containers:
        - name: nginx
          image: nginx
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-app
    tier: frontend
  ports:
    - port: 80
      targetPort: 80
```

---

## Labels vs Selectors (Simple Explanation)

* **Labels** = identity tags on Pods
* **Selectors** = filters used to find Pods

👉 Labels describe Pods, selectors find Pods.

---

## Listing Labels and Selectors

```bash
kubectl get pods --show-labels
kubectl describe pod my-pod
kubectl describe service my-service
kubectl describe deployment my-deployment
```

---

## Label Best Practices

* Use meaningful keys (`app`, `tier`, `env`)
* Separate environments (`env=dev`, `env=prod`)
* Track versions (`version=v1`)
* Standardize naming across teams

---

## Labels vs Annotations

| Feature           | Labels               | Annotations            |
| ----------------- | -------------------- | ---------------------- |
| Purpose           | Grouping & selection | Metadata & information |
| Used by selectors | Yes                  | No                     |
| Example           | `app=web`            | `buildVersion=v1.2.3`  |

---

## Annotations in Kubernetes

Annotations store **non-identifying metadata** such as:

* Build version
* Git commit
* Owner
* Documentation links

They do **not** affect scheduling or selection.

### Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
    app: my-app
  annotations:
    buildVersion: "v1.2.3"
    gitCommit: "abc123"
    owner: "team-devops"
spec:
  containers:
    - name: nginx
      image: nginx
```

👉 Annotations are extremely useful during **production changes, debugging, audits, and automation**.

---

## Final Summary

* Static Pods are managed by **kubelet**
* Scheduler failure causes **Pending Pods**, not outages
* Manual scheduling bypasses the scheduler
* Labels identify workloads
* Selectors connect workloads
* Annotations store operational metadata

✅ Mastering these concepts is essential for **real-world Kubernetes operations and CKA success**


