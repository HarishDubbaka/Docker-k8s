# Kubernetes Node Affinity 

## What is Node Affinity?

Node Affinity is a set of rules used by Kubernetes to constrain which nodes your pods can be scheduled on, based on **labels attached to nodes**. This allows you to control pod placement to meet specific requirements such as hardware constraints, regulatory compliance, or performance optimization.

---

## Types of Node Affinity

Node Affinity is categorized into two main types:

### 1. RequiredDuringSchedulingIgnoredDuringExecution

- Pods **must** be scheduled on nodes that match the specified rules.
- If no matching nodes exist, the pod will remain in the **Pending** state.
- If node labels change after scheduling, the pod will **continue running**.

### 2. PreferredDuringSchedulingIgnoredDuringExecution

- Kubernetes **tries** to schedule the pod on matching nodes.
- If no matching nodes are available, the pod will still be scheduled on any suitable node.
- This is a **soft rule**, not mandatory.

---

## Why Use Node Affinity?

Node Affinity is useful for:

### Performance Optimization
Certain workloads require specific hardware (e.g., SSDs, GPUs, high-memory nodes). Node Affinity ensures such workloads are scheduled on appropriate nodes.

### Compliance and Security
Helps enforce rules about where sensitive workloads can run, supporting regulatory and security requirements.

### Resource Management
Keeps similar workloads together, improving efficiency and reducing resource contention.

---

## 1. RequiredDuringSchedulingIgnoredDuringExecution (Example)

### Pod Manifest

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis-required
  labels:
    run: redis
spec:
  containers:
  - name: redis
    image: redis
  tolerations:
  - key: "app"
    operator: "Equal"
    value: "frontend"
    effect: "NoSchedule"
  - key: "node-role.kubernetes.io/control-plane"
    effect: "NoSchedule"
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd
````

---

### List Node Labels

```bash
kubectl get nodes --show-labels
```

Example output:

```text
NAME                          STATUS   ROLES           AGE     VERSION   LABELS
cka-qacluster-control-plane   Ready    control-plane   4d23h   v1.31.0   node-role.kubernetes.io/control-plane=
cka-qacluster-worker          Ready    <none>          4d23h   v1.31.0   kubernetes.io/os=linux
cka-qacluster-worker2         Ready    <none>          4d23h   v1.31.0   kubernetes.io/os=linux
cka-qacluster-worker3         Ready    <none>          4d23h   v1.31.0   kubernetes.io/os=linux
```

---

### Apply the Manifest

```bash
kubectl apply -f nodeaffinity.yml
```

Check pod status:

```bash
kubectl get pods | grep redis
```

```text
redis-required   0/1   Pending   0   54s
```

---

### Describe the Pod

```bash
kubectl describe pod redis-required
```

Event output:

```text
Warning  FailedScheduling  default-scheduler
0/4 nodes are available:
1 node(s) had untolerated taint {app: frontend},
1 node(s) had untolerated taint {node-role.kubernetes.io/control-plane: },
2 node(s) didn't match Pod's node affinity/selector.
```

This means:

* Some nodes are blocked by **taints**
* Others don’t match **node affinity rules**

---

### Label the Node

```bash
kubectl label node cka-qacluster-worker disktype=ssd
```

Verify labels:

```bash
kubectl get nodes --show-labels
```

Once the label matches, the pod moves to **Running**:

```bash
kubectl get pods | grep redis
```

```text
redis-required   1/1   Running   0   107s
```

> **Note:**
> If a pod using `RequiredDuringSchedulingIgnoredDuringExecution` stays pending, always check:

* Node labels
* Taints and tolerations

---

## 2. PreferredDuringSchedulingIgnoredDuringExecution (Example)

### Pod Manifest

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis-preferred
  labels:
    run: redis
spec:
  containers:
  - name: redis
    image: redis
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: disktype
            operator: In
            values:
            - hdd
```

> **Note:** No nodes have the label `disktype=hdd`.

---

### Apply the Manifest

```bash
kubectl apply -f preferred.yml
```

Check pod status:

```bash
kubectl get pods | grep redis
```

```text
redis-preferred   0/1   ContainerCreating   0   7s
```

After a short time:

```bash
kubectl get pods | grep redis
```

```text
redis-preferred   1/1   Running   0   70s
```

Even without matching labels, Kubernetes schedules the pod because this is a **preferred**, not required, rule.

---

## Best Practices

* **Combine with Taints and Tolerations** for precise scheduling control
* **Use meaningful labels** and document them clearly
* **Review affinity rules regularly** as workloads evolve
* **Test in non-production environments** before deploying to production

---

## Conclusion

Node Affinity is a powerful Kubernetes scheduling feature that helps control where workloads run in your cluster. By using `Required` and `Preferred` affinity correctly, you can optimize performance, enforce compliance, and manage resources effectively. Mastering Node Affinity is an essential skill for Kubernetes administrators and CKA aspirants alike.


