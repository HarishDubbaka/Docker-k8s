# 🚑 Kubernetes Control Plane Failure Troubleshooting Guide

Troubleshooting a **Kubernetes Control Plane failure** is a core skill for platform engineers and DevOps teams. When the control plane is down, the cluster cannot schedule or manage workloads.

This guide provides a **practical,troubleshooting workflow**.

---

# 1️⃣ Check Control Plane Node Status

First, confirm whether the control plane node is healthy.

```bash
kubectl get nodes
```

If the control plane is unhealthy, you may see:

- `NotReady`
- `Unknown`

If the API server is completely down:

```
The connection to the server <IP>:6443 was refused
```

In that case, SSH into the **control plane node** directly.

---

# 2️⃣ Check Control Plane Components

Verify the control plane components under:

```
/etc/kubernetes/manifests/
```

Expected components:

- kube-apiserver
- kube-controller-manager
- kube-scheduler
- etcd

⚠️ Even a **small spelling mistake in manifest files** can break the control plane.

Check running containers:

```bash
crictl ps | grep kube
```

or

```bash
docker ps | grep kube
```

---

# 📄 When is `admin.conf` created?

The file:

```
/etc/kubernetes/admin.conf
```

is created during execution of:

```
kubeadm init
```

on the **control plane node**.

### During `kubeadm init`, Kubernetes performs:

- Generates cluster certificates
- Generates kubeconfig files
- Creates the **admin kubeconfig**
- Writes it to:

```
/etc/kubernetes/admin.conf
```

---

# 📁 Default kubeconfig Explained

The default kubeconfig is the configuration file used by `kubectl` to connect to a Kubernetes cluster.

### Default Location

For every user on Linux:

```
$HOME/.kube/config
```

`kubectl` automatically reads this file when you run commands like:

```bash
kubectl get nodes
kubectl get pods -A
```

---

# 📌 Node Type vs admin.conf

| Node Type | Should `admin.conf` exist? | Reason |
|-----------|----------------------------|--------|
| Control Plane | ✅ Yes | Created during `kubeadm init` |
| Worker Node | ❌ No | Workers only get `kubelet.conf` |

---

# ☁️ Cloud Kubernetes (EKS / AKS / GKE)

Cloud-managed clusters **do not use `/etc/kubernetes/admin.conf`**.

Instead, kubeconfig is generated using cloud CLI tools.

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

- Create `$HOME/.kube/config`
- Merge cluster contexts
- Fetch authentication credentials from the cloud control plane

---

# 3️⃣ Check kubelet Status

If **kubelet is down**, control plane static pods will not run.

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

---

# 4️⃣ Check Static Pod Manifests

Control plane components run as **static pods** managed by kubelet.

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

If any file is **missing or corrupted**, the component will fail.

---

# 🚨 Pod Stuck in Pending & No Events → Scheduler Not Running

When a Pod is stuck in **Pending** and:

- `kubectl describe pod` shows **no node assigned**
- No scheduling events
- No `FailedScheduling` warnings

➡️ This usually means the **Kubernetes Scheduler is not running**.

---

# 🔎 Troubleshooting Scheduler Issues

### 1. Check scheduler pod

```bash
kubectl get pods -n kube-system
```

Expected:

```
kube-scheduler-master   Running
```

If status is:

- `Missing`
- `CrashLoopBackOff`
- `Not Running`

then the scheduler is down.

---

### 2. Check container runtime

```bash
crictl ps | grep scheduler
```

or

```bash
docker ps | grep scheduler
```

If no container exists → scheduler is not running.

---

### 3. Verify static manifest

```
/etc/kubernetes/manifests/kube-scheduler.yaml
```

If missing or corrupted → scheduler cannot start.

---

### 4. Check scheduler config

```bash
ls -l /etc/kubernetes/scheduler.conf
```

If missing → scheduler fails.

---

### 5. Check kubelet logs

```bash
journalctl -u kubelet -f
```

Common errors:

- invalid scheduler.yaml
- certificate problems
- API server unreachable
- port conflicts

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

This occurs when:

- ReplicaSet creates a pod
- Scheduler must assign a node

If scheduler is down:

❌ Pod stays **Pending**  
❌ No node assigned  
❌ No events generated

Fix the scheduler → pods start creating again.

---

# ⚠️ Other Reasons Pods Stay Pending

Even if scheduler is working, these issues may block pods.

---

## 1️⃣ Node Resource Exhaustion

Check:

```bash
kubectl describe pod <pod> | grep -i insufficient
```

Possible errors:

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

Pods need **tolerations** to run.

---

## 3️⃣ Node Selector Mismatch

Example:

```yaml
nodeSelector:
  env: prod
```

If no node has label → pod never schedules.

Check:

```bash
kubectl get nodes --show-labels
```

---

## 4️⃣ Affinity / Anti-Affinity Rules

Strict scheduling rules can block pods.

Example:

```
requiredDuringSchedulingIgnoredDuringExecution
```

---

## 5️⃣ PVC Not Bound

Check storage:

```bash
kubectl get pvc
kubectl describe pvc <name>
```

Unbound PVC → pod stays Pending.

---

## 6️⃣ CNI Plugin Failure

If CNI fails, nodes become `NotReady`.

Check:

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

Check quotas:

```bash
kubectl get quota -A
```

Quotas may block pod creation.

---

## 9️⃣ Deployment Rollout Stuck

Check rollout:

```bash
kubectl rollout status deployment/<name>
```

---

# 5️⃣ Verify etcd Health

Control plane heavily depends on **etcd**.

Check health:

```bash
ETCDCTL_API=3 etcdctl endpoint health
```

List members:

```bash
etcdctl member list
```

Common issues:

- Disk full
- Certificate expiration
- Network partition

---

# 6️⃣ Check API Server Logs

```bash
crictl logs <apiserver-container-id>
```

Typical failures:

- certificate errors
- etcd connection issues
- port conflicts

---

# 7️⃣ Check Disk & Memory

Disk usage:

```bash
df -h
```

Memory:

```bash
free -m
```

Look for:

- Disk full
- OOMKilled processes

---

# 8️⃣ Check Certificate Expiration

```bash
kubeadm certs check-expiration
```

Renew certificates:

```bash
kubeadm certs renew all
systemctl restart kubelet
```

---

# 9️⃣ Network & Port Validation

| Component | Port |
|-----------|------|
| kube-apiserver | 6443 |
| etcd | 2379–2380 |
| kubelet | 10250 |
| controller-manager | 10257 |
| scheduler | 10259 |

Check ports:

```bash
netstat -tulnp | grep 6443
```

---

# 🔟 Check Cluster Events

If API server is reachable:

```bash
kubectl get events -A
```

Look for:

- Scheduling failures
- Node communication errors

---

# ⚡ Real-World Troubleshooting Order (Fast)

1. SSH to control plane node  
2. Check `kubelet` service  
3. Verify `/etc/kubernetes/manifests`  
4. Verify `etcd` health  
5. Inspect API server logs  
6. Check disk and memory  
7. Validate certificates  

---

# 💡 Interview Tip

If asked:

### **How do you troubleshoot Kubernetes control plane failure?**

Answer in this order:

1. Check kubelet service  
2. Verify static pod manifests  
3. Check etcd health  
4. Inspect API server logs  
5. Validate certificates and node resources  

---

# 🚀 Want More?

I can also provide:

- **25 real Kubernetes control plane failure scenarios**
- **SRE troubleshooting playbook**
- **Production debugging checklist**

Perfect for **DevOps interviews and real cluster operations**.
