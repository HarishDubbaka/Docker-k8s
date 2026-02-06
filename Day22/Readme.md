# Kubernetes Taints & Tolerations

Organizations and teams often need **multi-tenant, heterogeneous Kubernetes clusters** to meet diverse application requirements. In many cases, clusters must also enforce **special scheduling constraints**—for example:

* Pods that require special hardware (GPU, SSD, high memory)
* Pods that must be isolated from others
* Pods that must colocate with specific workloads

One effective way to achieve this is through **taints and tolerations**.

This document explains **what taints and tolerations are**, **why they’re useful**, and **how to use them**, with a hands-on example using a Kubernetes cluster.

---

## What Are Taints and Tolerations?

**Taints** and **tolerations** work together to control **where pods can be scheduled**.

* **Taints** are applied to **nodes**
* **Tolerations** are defined in **pod specifications**

When a node is tainted, it **repels all pods** except those that explicitly tolerate the taint.

A node can have **multiple taints**.

---

## Why Kubernetes Uses Taints by Default

Most Kubernetes distributions **automatically taint control-plane nodes** so that:

* Only system control-plane pods run there
* User workloads are scheduled on worker nodes instead

This ensures that critical components like `etcd`, `kube-apiserver`, and `controller-manager` are protected from resource contention.

---

## Taint Effects

A taint can have one of the following effects:

### `NoSchedule`

* Pods **without tolerations will not be scheduled**
* Existing pods are not affected

### `PreferNoSchedule`

* Scheduler **tries to avoid placing pods**
* Not a hard requirement

### `NoExecute`

* Pods **without tolerations are evicted**
* New pods without tolerations are not scheduled

---

## 🔑 Key Use Cases

### 1. Dedicated Nodes for Critical Workloads

* **Scenario:** Reserve nodes for system-critical workloads
* **Taint:** `critical=true:NoSchedule`
* **Benefit:** Prevents application pods from using critical nodes

---

### 2. Isolating Special Hardware

* **Scenario:** GPU-only workloads
* **Taint:** `gpu=true:NoSchedule`
* **Benefit:** Prevents non-GPU workloads from using GPU nodes

---

### 3. Multi-Tenant Cluster Management

* **Scenario:** Shared cluster across teams
* **Taint:** `team=A:NoSchedule`
* **Benefit:** Enforces workload isolation

---

### 4. Handling Node Conditions

* **Scenario:** Node becomes unreachable or unhealthy
* **Taint:** `node.kubernetes.io/unreachable`
* **Benefit:** Improves resilience and availability

---

### 5. Preemptive Resource Control

* **Scenario:** Reserve nodes for future workloads
* **Taint:** `reserved=true:NoSchedule`
* **Benefit:** Guarantees capacity when needed

---

## 📊 Comparison Table

| Use Case               | Taint Example                    | Pod Requirement          | Benefit                 |
| ---------------------- | -------------------------------- | ------------------------ | ----------------------- |
| Critical workloads     | `critical=true:NoSchedule`       | Matching toleration      | Protects system pods    |
| GPU isolation          | `gpu=true:NoSchedule`            | GPU toleration           | Prevents GPU waste      |
| Multi-tenant isolation | `team=A:NoSchedule`              | Team-specific toleration | Prevents interference   |
| Node health handling   | `node.kubernetes.io/unreachable` | Auto-managed             | Improves resilience     |
| Reserved resources     | `reserved=true:NoSchedule`       | Explicit toleration      | Guarantees availability |

---

## Hands-On Example: Using Taints and Tolerations

### Cluster Overview

```bash
kubectl get nodes
```

```
NAME                          STATUS   ROLES           AGE     VERSION
cka-qacluster-control-plane   Ready    control-plane   3d23h   v1.31.0
cka-qacluster-worker          Ready    <none>          3d23h   v1.31.0
cka-qacluster-worker2         Ready    <none>          3d23h   v1.31.0
cka-qacluster-worker3         Ready    <none>          3d23h   v1.31.0
```

Check existing taints:

```bash
kubectl get nodes -o custom-columns=NAME:.metadata.name,TAINTS:.spec.taints
```

```
cka-qacluster-control-plane   [node-role.kubernetes.io/control-plane:NoSchedule]
cka-qacluster-worker          <none>
cka-qacluster-worker2         <none>
cka-qacluster-worker3         <none>
```

➡️ Control-plane node is already tainted to block user workloads.

---

## Taint a Worker Node

We want **only front-end pods** to run on `cka-qacluster-worker`.

```bash
kubectl taint node cka-qacluster-worker app=frontend:NoSchedule
```

Verify:

```bash
kubectl get nodes -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{range .spec.taints[*]}{.key}={.value}:{.effect}{" "}{end}{"\n"}{end}'
```

```
cka-qacluster-control-plane   node-role.kubernetes.io/control-plane=:NoSchedule
cka-qacluster-worker          app=frontend:NoSchedule
cka-qacluster-worker2
cka-qacluster-worker3
```

---

## Deploy a Pod WITHOUT Toleration

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod-no-toleration
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
```

Apply it:

```bash
kubectl apply -f withouttoleration.yaml
kubectl get pods
```

➡️ The pod runs **only on untainted nodes** (`worker2`, `worker3`).

### Why?

* Pods without tolerations are **repelled** by tainted nodes
* Kubernetes schedules pods wherever possible unless blocked

---

## Deploy a Pod WITH Toleration

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod-with-toleration
spec:
  tolerations:
  - key: "app"
    operator: "Equal"
    value: "frontend"
    effect: "NoSchedule"
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
```

Apply it:

```bash
kubectl apply -f withtoleration.yml
```

---

## Verify Pod Placement

### Find the node

```bash
kubectl get pod test-pod-with-toleration -o wide
```

```
NODE
cka-qacluster-worker
```

### Verify taint on the node

```bash
kubectl describe node cka-qacluster-worker | grep -i taints
```

```
Taints: app=frontend:NoSchedule
```

### One-liner (CKA-friendly)

```bash
kubectl get pod test-pod-with-toleration -o wide && \
kubectl describe node $(kubectl get pod test-pod-with-toleration -o jsonpath='{.spec.nodeName}') | grep -i taints
```

---

## Important Note: Taints vs Node Selection

* **Taints & tolerations** allow pods to run on tainted nodes
* They **do NOT guarantee placement**
* To ensure pods land only on specific nodes:

  * Add **labels** to nodes (e.g., `app=frontend`)
  * Use **nodeSelector / affinity** in pod specs

---

## ⚠️ Risks & Considerations

* Overusing taints can make pods **unschedulable**
* Tolerations **don’t force scheduling**
* Complex taint strategies require **clear documentation**

---

## Conclusion

Taints and tolerations provide **powerful, flexible scheduling control** in Kubernetes. They are especially useful for:

* Dedicated nodes
* Hardware-specific workloads
* Multi-tenant isolation
* Node health management

When combined with **labels and node selectors**, they form a robust and exam-relevant scheduling strategy—essential knowledge for **CKA** and real-world clusters.

---

If you want, I can also:

* Add **CKA exam shortcuts**
* Convert this into a **blog-ready article**
* Add **nodeSelector + affinity examples**
* Review it as if it were **exam documentation**
