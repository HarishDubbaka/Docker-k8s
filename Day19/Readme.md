# Kubernetes Namespaces ☸️📂

As Kubernetes applications scale, managing **resources, access control, and environment segregation** becomes complex.  
This is where **Kubernetes Namespaces** come in — they provide a way to **organize, isolate, and manage resources** efficiently within the same cluster.

This document explores:
- What namespaces are
- Namespace-scoped vs cluster-scoped resources
- Default namespaces
- Creating and managing namespaces
- Practical examples
- Best practices

---

## Why Kubernetes Namespaces? 🎯

Namespaces enable better control over **access, security, and resource allocation** by allowing administrators to define:

- Role-Based Access Control (RBAC)
- Resource quotas
- Network policies

at the **namespace level**.

This is especially useful in **multi-tenant environments**, where multiple teams or projects share the same Kubernetes cluster but need isolation to prevent conflicts.

---

## Namespace-Scoped vs Cluster-Scoped Resources ☸️

In Kubernetes, resources live in **two different scopes**:

- **Namespace-scoped** → belong to a namespace
- **Cluster-scoped** → belong to the entire cluster

---

## 📂 Namespace-Scoped Resources

These resources **exist inside a namespace**.

### Characteristics:
- You must specify a namespace (or use `default`)
- The same resource name can exist in different namespaces
- Used mainly for **applications and workloads**

### Examples:
- Pods
- Deployments
- ReplicaSets
- Services
- ConfigMaps
- Secrets
- Jobs / CronJobs
- Ingress

### Example:
```bash
kubectl get pods -n development
````

🧠 Think of namespaces like **folders**:

```
dev/app
prod/app
```

Same app name, different folder.

---

## 🌐 Cluster-Scoped Resources

These resources **do NOT belong to any namespace**.

### Characteristics:

* Apply to the entire cluster
* Names must be globally unique
* Used for **cluster configuration and infrastructure**

### Examples:

* Nodes
* Namespaces
* PersistentVolumes (PV)
* StorageClasses
* ClusterRoles / ClusterRoleBindings
* CustomResourceDefinitions (CRDs)

### Example:

```bash
kubectl get nodes
```

❌ This will NOT work:

```bash
kubectl get nodes -n dev
```

👉 Because **nodes are cluster-scoped**.

---

## Benefits of Using Namespaces 📊

| Feature            | Benefit                       |
| ------------------ | ----------------------------- |
| RBAC per namespace | Enhanced security 🔐          |
| Resource isolation | Improved performance ⚡        |
| Resource quotas    | Better capacity management 📈 |
| Name scoping       | Efficient organization 🗂️    |


By default, every Kubernetes cluster has a default namespace for resources that don’t specify one. Other system namespaces like kube-system and kube-public are used for Kubernetes components and shared information.

---

## Important Namespace Concepts ⚠️

* Resources inside namespaces are **logically separated**
* Pods in different namespaces **can still communicate** unless restricted
* Namespaces **do not provide true multi-tenancy**
* Every cluster has a **default namespace**
* You can create **multiple namespaces**
* ❌ Namespaces **cannot be nested**

Namespaces are optional for small clusters but **essential in large-scale deployments**.

---

## Default Kubernetes Namespaces 📦

Kubernetes comes with several built-in namespaces:

### `default`

* Used when no namespace is specified
* Suitable for simple or small environments

### `kube-system`

* Contains Kubernetes core components
* Example components:

  * kube-apiserver
  * controller-manager
  * scheduler

### `kube-public`

* Accessible across the cluster
* Used for shared cluster information

### `kube-node-lease`

* Stores node lease objects
* Helps track node health and heartbeats

---

## Creating and Managing Namespaces 🛠️

### Create a Namespace (YAML)

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: development
```

Apply it:

```bash
kubectl apply -f development-namespace.yaml
```

---

### Create a Namespace (CLI)

```bash
kubectl create namespace development
```

---

### List Namespaces

```bash
kubectl get namespaces
```

---

### Delete a Namespace ⚠️

```bash
kubectl delete namespace development
```

> ⚠️ Deleting a namespace deletes **all resources inside it**.

---

## Working With Resources in Namespaces 🔧

### Create Resources in a Namespace

```bash
kubectl create deployment my-app --image=nginx -n development
```

---

### View Resources in a Namespace

```bash
kubectl get pods -n development
```

---

## Switching the Default Namespace 🔄

To avoid repeatedly using `-n`, set a default namespace for your session:

```bash
kubectl config set-context --current --namespace=development
```

### What This Does:

* `kubectl config` → works with kubeconfig settings
* `set-context` → updates context configuration
* `--current` → applies to the active context
* `--namespace=development` → sets default namespace

✅ After this, all kubectl commands use the `development` namespace automatically.

---

## Examples of Using Kubernetes Namespaces 📘

### Example 1: Separating Environments

Create namespaces:

```bash
kubectl create namespace development
kubectl create namespace staging
kubectl create namespace production
```

Deploy applications:

```bash
kubectl create deployment dev-app --image=nginx -n development
kubectl create deployment stage-app --image=nginx -n staging
kubectl create deployment prod-app --image=nginx -n production
```

**In short:**

* Creates a Deployment
* Runs the NGINX container
* Deploys into the specified namespace
* Kubernetes creates a ReplicaSet and Pod automatically

---

### Example 2: Resource Quotas per Namespace 📊

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-quota
  namespace: development
spec:
  hard:
    requests.cpu: "1"
    requests.memory: "1Gi"
    limits.cpu: "2"
    limits.memory: "2Gi"
```

Apply the quota:

```bash
kubectl apply -f dev-quota.yaml
```

This ensures applications in the `development` namespace do not exceed defined CPU and memory limits.

---

## Namespace Best Practices ✅

* Use consistent naming (`dev`, `staging`, `prod`)
* Avoid overusing namespaces
* Always set resource quotas
* Apply RBAC per namespace
* Restrict production access
* Monitor resource usage (Prometheus, Grafana)

---

## Conclusion 🏁

Kubernetes Namespaces are a powerful mechanism to **organize, isolate, and secure** workloads in a cluster.

When combined with **RBAC, ResourceQuotas, and NetworkPolicies**, namespaces help create a **scalable, secure, and well-managed Kubernetes environment**.

With the concepts, examples, and best practices covered here, you are well-equipped to use namespaces effectively in real-world Kubernetes deployments.

---

## 🧠 Namespace-Scoped vs Cluster-Scoped — Quick Reminder ☸️

When working with Kubernetes, understanding **resource scope** saves time and prevents mistakes.

---

### ✅ Source of Truth
**`kubectl api-resources`** is the **ultimate source of truth** for knowing whether a resource is:
- Namespace-scoped  
- Cluster-scoped  

---

### 🔑 Golden Rule
- **If it runs your app → Namespace-Scoped**
- **If it defines the cluster → Cluster-Scoped**

---

### 🤣 Easy Debug Trick
- If `-n <namespace>` **works** → Namespace-scoped  (Pods, Services, Deployments)
- If `-n <namespace>` **does NOT work** → Cluster-scoped  (Nodes, PVs, CRDs)

> 🤣 *If `-n` doesn’t work… the resource is global.*

---

