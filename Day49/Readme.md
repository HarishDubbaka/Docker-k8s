# 🛠 Kubernetes Multi-Node Cluster Upgrade – Clear Explanation

---

## 1️⃣ **Understanding Kubernetes Versions**

Kubernetes uses a **Major.Minor.Patch** versioning scheme:

```
Major.Minor.Patch
Example: 1.30.2
```

* **Major:** Big changes, breaking APIs
* **Minor:** New features, backward-compatible
* **Patch:** Bug fixes and security updates

💡 **Rule:** Always upgrade to the **latest patch** within a supported minor version. This keeps your cluster secure.

---

## 2️⃣ **High-Level Upgrade Steps**

When upgrading a cluster, the order matters:

1. **Primary control plane node (Master)** – first node that runs the API server.
2. **Additional control plane nodes** – if HA setup exists.
3. **Worker nodes** – where your applications run.

> Why this order? Control plane components manage the cluster. You need a healthy control plane before upgrading worker nodes.

---

## 3️⃣ **Pre-Upgrade Requirements**

Before touching the cluster:

* **Read release notes** for your version.
* **Backup important workloads** (databases, stateful apps).
* Cluster should run **static control plane pods** or **external etcd**.
* **Swap must be disabled** on all nodes.
* Drain nodes **before minor upgrades**.
* Ensure **kubelet and kubeadm versions match**, or kubelet can be slightly older within supported skew.

---

## 4️⃣ **Draining Nodes – What Happens**

Draining a node means **preparing it for maintenance or upgrade**:

1. **Stop scheduling new pods** on the node.
2. **Evict existing pods** from the node.

**Pods behavior:**

* **Managed by Deployment/ReplicaSet:** Rescheduled automatically to other nodes.
* **Standalone pods (no controller):** Deleted permanently. They **won’t come back** after node uncordon.

**Control plane node:**

* Draining usually **doesn’t affect workloads** running on worker nodes.
* But if a worker fails during this time, diagnosing is harder.
* **Recommendation:** Use HA (High Availability) control plane in production.

---

## 5️⃣ **Worker Node Upgrade Strategies**

### **Strategy 1: All at Once**

* Upgrade **all worker nodes simultaneously**.
* Pros: Fastest.
* Cons: High risk → workloads may face downtime.

### **Strategy 2: Rolling Upgrade (Recommended)**

* Upgrade **one node at a time**:

  1. Drain the node → `kubectl drain <node> --ignore-daemonsets`
  2. Upgrade kubeadm, kubelet, kubectl
  3. Uncordon the node → `kubectl uncordon <node>`
* Pros: Maintains service availability.
* Cons: Slightly slower.

### **Strategy 3: Blue-Green Upgrade (Safest)**

* Provision **new upgraded nodes (Green)**.
* Keep old nodes (Blue) running.
* Gradually move workloads to Green nodes.
* Remove old nodes once stable.
* Pros: Minimal risk, easy rollback.
* Cons: Requires extra resources.

---

## 6️⃣ **Determine the Version to Upgrade**

* Find **latest patch for your minor version**.
* **Ubuntu/Debian example:**

```bash
sudo apt update
sudo apt-cache madison kubeadm
```

* Make sure **Kubernetes repo** is configured correctly.

---

## 7️⃣ **Upgrade Notes**

* `kubeadm upgrade` **only updates Kubernetes components** (not workloads).
* All containers **restart** because the container spec hash changes.
* Verify kubelet restart:

```bash
systemctl status kubelet
journalctl -xeu kubelet
```

* Use `kubeadm upgrade --config` to customize upgrade settings.

---

## 8️⃣ **Summary – Safe Upgrade Flow**

1. Backup important workloads.
2. Drain **primary control plane node** → Upgrade → Uncordon.
3. Upgrade **additional control plane nodes** (if any).
4. Upgrade **worker nodes** using **rolling or blue-green strategy**.
5. Verify cluster status:

```bash
kubectl get nodes
kubectl get pods -A
```

**Key Advice:**

* HA control plane recommended for production.
* Keep kubeadm, kubelet, kubectl versions **compatible**.
* Always test in **staging environment** before production.

---

✅ **Takeaway:**

Upgrading Kubernetes is **not just a version bump**. It’s about **safe orchestration**, ensuring workloads stay running while control plane and nodes are upgraded sequentially.

---

Tomorrow we’ll do the hands-on, step-by-step upgrade on your kubeadm cluster. You’ll see exactly how to:

Drain nodes safely

Upgrade control plane and worker nodes

Use rolling or blue-green strategies

Verify everything is running smoothly
