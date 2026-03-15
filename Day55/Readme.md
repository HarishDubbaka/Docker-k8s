# 🚑 Kubernetes Control Plane Failure Troubleshooting Guide

Troubleshooting a **Kubernetes Control Plane failure** is a critical skill for **DevOps engineers, platform engineers, and SREs**.

When the control plane is unavailable:

- The cluster **cannot schedule pods**
- Kubernetes **cannot manage workloads**
- `kubectl` commands may fail
- Cluster state **cannot be updated**

This guide provides a **clear,troubleshooting workflow used in real environments**.

---

# 1️⃣ Check Control Plane Node Status

Start by verifying whether the control plane node is reachable and healthy.

```bash
kubectl get nodes
```

Possible outputs:

| Status | Meaning |
|------|------|
| Ready | Node is healthy |
| NotReady | Node cannot communicate properly |
| Unknown | Control plane cannot reach node |

Example failure:

```
The connection to the server <IP>:6443 was refused
```

This means the **API Server is not reachable**.

➡️ SSH directly into the **control plane node** to continue troubleshooting.

---

# 2️⃣ Verify Control Plane Components

Control plane components run as **static pods** managed by `kubelet`.

Their manifest files are stored in:

```
/etc/kubernetes/manifests/
```

Expected files:

```
kube-apiserver.yaml
kube-controller-manager.yaml
kube-scheduler.yaml
etcd.yaml
```

Check if containers are running:

```bash
crictl ps | grep kube
```

or (if Docker runtime is used)

```bash
docker ps | grep kube
```

⚠️ **Important**

Even a **small syntax or spelling error** in a manifest file can prevent the component from starting.

---

# 📄 When is `admin.conf` Created?

The file:

```
/etc/kubernetes/admin.conf
```

is created during execution of:

```bash
kubeadm init
```

on the **control plane node**.

During `kubeadm init`, Kubernetes performs several actions:

- Generates cluster certificates
- Generates kubeconfig files
- Creates the **admin kubeconfig**
- Saves it to:

```
/etc/kubernetes/admin.conf
```

---

# 📁 Default kubeconfig Explained

The kubeconfig file is used by **kubectl** to connect to the cluster.

Default location:

```
$HOME/.kube/config
```

Typical commands using this configuration:

```bash
kubectl get nodes
kubectl get pods -A
```

---

# 📌 Node Type vs admin.conf

| Node Type | admin.conf Exists? | Reason |
|------|------|------|
| Control Plane | ✅ Yes | Created during `kubeadm init` |
| Worker Node | ❌ No | Worker nodes only receive `kubelet.conf` |

---

# ☁️ Managed Kubernetes (EKS / AKS / GKE)

Managed Kubernetes services **do not use `/etc/kubernetes/admin.conf`**.

Instead, kubeconfig files are generated using cloud CLI tools.

### AWS EKS

```bash
aws eks update-kubeconfig --name cluster-name
```

### Azure AKS

```bash
az aks get-credentials --resource-group myRG --name myCluster
```

### Google GKE

```bash
gcloud container clusters get-credentials cluster-name --zone us-central1-a
```

These commands:

- Generate `$HOME/.kube/config`
- Merge cluster contexts
- Retrieve authentication credentials

---

# 3️⃣ Check kubelet Service

`kubelet` is responsible for running **static control plane pods**.

If kubelet is down, the **entire control plane stops working**.

Check status:

```bash
systemctl status kubelet
```

Restart kubelet:

```bash
systemctl restart kubelet
```

View logs:

```bash
journalctl -u kubelet -f
```

Common errors:

- Certificate problems
- Invalid static pod manifest
- API server connection failures

---

# 4️⃣ Validate Static Pod Manifests

Control plane components are defined as **static pods**.

Location:

```
/etc/kubernetes/manifests/
```

Expected files:

```
kube-apiserver.yaml
kube-controller-manager.yaml
kube-scheduler.yaml
etcd.yaml
```

If any file is:

- Missing
- Corrupted
- Misconfigured

➡️ The component **will not start**.

---

# 🚨 Pod Stuck in Pending (No Events)

Scenario:

- Pod remains **Pending**
- `kubectl describe pod` shows **no node assigned**
- No scheduling events

This usually means the **Kubernetes Scheduler is not running**.

---

# 🔎 Troubleshooting Scheduler Issues

### 1. Check scheduler pod

```bash
kubectl get pods -n kube-system
```

Expected:

```
kube-scheduler-<control-plane>   Running
```

Failure states:

- Missing
- CrashLoopBackOff
- Pending

---

### 2. Check container runtime

```bash
crictl ps | grep scheduler
```

or

```bash
docker ps | grep scheduler
```

If no container exists, the scheduler is not running.

---

### 3. Verify scheduler manifest

```
/etc/kubernetes/manifests/kube-scheduler.yaml
```

If missing or invalid, the scheduler cannot start.

---

### 4. Verify scheduler config

```bash
ls -l /etc/kubernetes/scheduler.conf
```

If this file is missing, the scheduler will fail.

---

### 5. Check kubelet logs

```bash
journalctl -u kubelet -f
```

Possible errors:

- Invalid scheduler configuration
- Certificate issues
- API server connectivity problems
- Port conflicts

---

### 6. Restart kubelet after fixing

```bash
systemctl restart kubelet
```

---

# 🔁 Deployment Replicas Not Recreated

Scenario:

- Deployment has **2 replicas**
- One pod deleted
- New pod **not created**

Reason:

1. ReplicaSet creates a new pod
2. Scheduler must assign a node
3. If scheduler is down → scheduling fails

Result:

❌ Pod remains **Pending**  
❌ No node assigned  
❌ No events generated

Fix the **scheduler**, and pod creation resumes.

---

# ⚠️ Other Reasons Pods Stay Pending

Even if the scheduler is running, these issues may block pods.

---

## 1️⃣ Node Resource Exhaustion

Check for resource shortage:

```bash
kubectl describe pod <pod>
```

Common errors:

- `Insufficient cpu`
- `Insufficient memory`

---

## 2️⃣ Node Taints

Check taints:

```bash
kubectl describe node <node> | grep Taints
```

Example:

```
node-role.kubernetes.io/control-plane:NoSchedule
```

Pods require **tolerations** to run on such nodes.

---

## 3️⃣ Node Selector Mismatch

Example:

```yaml
nodeSelector:
  env: prod
```

Check node labels:

```bash
kubectl get nodes --show-labels
```

If no node matches the selector → pod cannot schedule.

---

## 4️⃣ Affinity / Anti-Affinity Rules

Strict scheduling rules like:

```
requiredDuringSchedulingIgnoredDuringExecution
```

may block scheduling.

---

## 5️⃣ PVC Not Bound

Check persistent volume claims:

```bash
kubectl get pvc
kubectl describe pvc <name>
```

If PVC is not bound → pod remains Pending.

---

## 6️⃣ CNI Plugin Failure

If networking fails, nodes may show `NotReady`.

Check system pods:

```bash
kubectl get pods -n kube-system
```

Common CNI plugins:

- Calico
- Flannel
- Weave
- Cilium

---

## 7️⃣ API Server Issues

Check component health:

```bash
kubectl get componentstatuses
```

---

## 8️⃣ Resource Quotas

Namespace quotas may block pod creation.

```bash
kubectl get quota -A
```

---

## 9️⃣ Deployment Rollout Stuck

Check rollout status:

```bash
kubectl rollout status deployment/<name>
```

---

# 5️⃣ Verify etcd Health

The Kubernetes control plane **stores all cluster state in etcd**.

Check etcd health:

```bash
ETCDCTL_API=3 etcdctl endpoint health
```

List etcd members:

```bash
etcdctl member list
```

Common issues:

- Disk full
- Certificate expiration
- Network partition
- Corrupted data directory

---

# 6️⃣ Check API Server Logs

Find API server container:

```bash
crictl ps | grep apiserver
```

View logs:

```bash
crictl logs <container-id>
```

Typical failures:

- etcd connection issues
- Certificate errors
- Port conflicts

---

# 7️⃣ Check System Resources

Disk usage:

```bash
df -h
```

Memory usage:

```bash
free -m
```

Common problems:

- Disk full
- OOMKilled processes

---

# 8️⃣ Check Certificate Expiration

Verify certificate status:

```bash
kubeadm certs check-expiration
```

Renew certificates if expired:

```bash
kubeadm certs renew all
systemctl restart kubelet
```

---

# 9️⃣ Validate Required Ports

| Component | Port |
|------|------|
| kube-apiserver | 6443 |
| etcd | 2379–2380 |
| kubelet | 10250 |
| controller-manager | 10257 |
| scheduler | 10259 |

Check port usage:

```bash
netstat -tulnp | grep 6443
```

---

# 🔟 Check Cluster Events

If the API server is reachable, check cluster events:

```bash
kubectl get events -A
```

Look for:

- Scheduling failures
- Node communication errors
- Resource constraints

---

# ⚡ Real-World Troubleshooting Order (Fast Method)

When troubleshooting in production, engineers usually follow this order:

1. SSH into the control plane node
2. Check `kubelet` service
3. Verify `/etc/kubernetes/manifests`
4. Validate `etcd` health
5. Inspect API server logs
6. Check disk and memory usage
7. Verify certificate expiration
8. Validate networking and ports

---

# 🧠 Key Takeaway

If the **control plane fails**, always remember:

```
kubelet → Static Pods → API Server → etcd → Scheduler
```

Fixing the issue usually means **restoring this chain**.

---
