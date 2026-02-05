# Kubernetes Scheduling, Static Pods, and Scheduler Failure (Kind Cluster Demo)

## Overview

In Kubernetes, pod scheduling plays a crucial role in ensuring applications run smoothly across a cluster. This document explains three key concepts:

* Kubernetes Scheduler
* Static Pods
* What happens when the scheduler goes down

We demonstrate these concepts using a **Kind (Kubernetes in Docker)** cluster with real commands and observations.

---

## Kubernetes Scheduler – How Pod Scheduling Works

In a Kubernetes cluster, the **scheduler** is a control plane component responsible for deciding **which node a pod should run on**.

### Scheduling Flow

1. A client requests a new pod (e.g., `kubectl run`).
2. The **API server** creates a pod object in **etcd**.
3. The **scheduler** watches for pods that do not have a node assigned.
4. Using scheduling algorithms, the scheduler selects the best node.
5. The scheduler updates the pod object with the chosen node.
6. The **kubelet** on that node:

   * Pulls the container image
   * Creates and starts the pod
7. The kubelet reports the pod status back to the API server.

### Key Role of the Scheduler

* Assigns pods to nodes
* Ensures efficient resource utilization
* Only involved during **initial scheduling**

---

## Pods and High Availability

* Pods are the **smallest deployable units** in Kubernetes.
* Kubernetes is designed to tolerate failures:

  * ReplicaSets / Deployments maintain multiple pod replicas
  * Controllers restart or reschedule failed pods
* The system continues running unless:

  * All replicas of a critical service fail
  * Control plane components fail
  * A cluster-wide failure occurs without redundancy

---

## Cluster Setup (Kind)

This demo uses a Kind cluster with **4 nodes**:

* 1 control-plane node
* 3 worker nodes

```bash
kubectl get nodes
```

```text
NAME                          STATUS   ROLES           AGE     VERSION
cka-qacluster-control-plane   Ready    control-plane   2d23h   v1.31.0
cka-qacluster-worker          Ready    <none>          2d23h   v1.31.0
cka-qacluster-worker2         Ready    <none>          2d23h   v1.31.0
cka-qacluster-worker3         Ready    <none>          2d23h   v1.31.0
```

---

## Verifying the Scheduler Pod

The scheduler runs as a **static pod** on the control-plane node.

```bash
kubectl get pods -n kube-system | grep kube-scheduler
```

```text
kube-scheduler-cka-qacluster-control-plane   1/1   Running
```

```bash
kubectl get pods -o wide -n kube-system | grep kube-scheduler
```

```text
kube-scheduler-cka-qacluster-control-plane   Running   cka-qacluster-control-plane
```

---

## What Happens If the Scheduler Goes Down?

* Existing pods **continue running**
* New pods **cannot be scheduled**
* The cluster is **degraded**, not down
* Pods remain in the `Pending` state
* In HA setups, another scheduler instance may take over

---

## Static Pods Explained

### What Are Static Pods?

* Static pods are **managed directly by the kubelet**
* They are **not scheduled by the scheduler**
* Defined as YAML files on a node’s filesystem

### Key Characteristics

* Created from files in:

  ```
  /etc/kubernetes/manifests
  ```
* The kubelet constantly watches this directory
* When a file appears:

  * The kubelet immediately creates the pod
* When a file is removed:

  * The kubelet stops the pod

### Kubelet Role

* Runs on **every node** (worker + control-plane)
* Not a control plane component
* Responsible for:

  * Managing static pods
  * Running containers on the node

---

## Static Pods in Cloud vs Self-Managed Clusters

### Managed Kubernetes (EKS, GKE, AKS, etc.)

* Control plane nodes are **not accessible**
* `/etc/kubernetes/manifests` is hidden
* Control plane is fully managed by the provider

### Self-Managed / Kind Clusters

* Control-plane nodes are accessible
* Static pod manifests are visible
* Control plane components run as static pods

---

## Accessing Static Pod Manifests in Kind

Each Kind node is a **Docker container**.

```bash
docker exec -it cka-qacluster-control-plane sh
```

```bash
cd /etc/kubernetes/manifests
ls -ltr
```

```text
etcd.yaml
kube-apiserver.yaml
kube-scheduler.yaml
kube-controller-manager.yaml
```

---

## Verifying Kubelet Is Running

```bash
ps -ef | grep kubelet
```

You should see the kubelet process running inside the control-plane container.

---

## Simulating Scheduler Failure

### Step 1: Remove Scheduler Manifest

```bash
mv /etc/kubernetes/manifests/kube-scheduler.yaml /tmp
```

Result:

* Kubelet stops the scheduler pod
* Scheduler is no longer running

---

### Step 2: Create a New Pod

```bash
kubectl run nginxharish --image=nginx
```

```bash
kubectl get pods | grep nginxharish
```

```text
nginxharish   0/1   Pending
```

```bash
kubectl describe pod nginxharish
```

Key observations:

* `Node: <none>`
* No scheduling events
* Pod stays in `Pending`

---

## Why the Pod Is Pending

* Scheduler is responsible for assigning nodes
* Scheduler pod is missing
* No scheduling decisions can be made

### Important Behavior

* Existing pods → **Unaffected**
* New pods → **Stuck in Pending**
* Cluster → **Degraded, not down**

---

## Restoring the Scheduler

### Step 3: Restore the Manifest

```bash
mv /tmp/kube-scheduler.yaml /etc/kubernetes/manifests/
```

What happens next:

* Kubelet detects the file immediately
* Scheduler static pod is recreated
* Scheduler starts running again

---

## Verifying Recovery

```bash
kubectl get pods | grep nginxharish
```

```text
nginxharish   1/1   Running
```

```bash
kubectl get pods -n kube-system -o wide | grep kube-scheduler
```

```text
kube-scheduler-cka-qacluster-control-plane   1/1   Running
```

---

## Final Summary

* Static pods are managed by the kubelet, not the scheduler
* Control plane components run as static pods
* If the scheduler goes down:

  * Existing pods continue running
  * New pods cannot be scheduled
* Restoring the static pod manifest immediately restores scheduling

👉 **In short:**
The cluster doesn’t go down—only **new scheduling stops** until the scheduler is back.

