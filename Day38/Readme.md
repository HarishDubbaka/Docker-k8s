# Kubernetes Networking & NetworkPolicy – Hands-On Guide (Kind Cluster)

## 1️⃣ Kubernetes Networking Basics

The scalability of Kubernetes allows you to run multiple applications inside a single cluster.

However, **by default Kubernetes does NOT provide network isolation** — all Pods can freely communicate with each other.

To control this communication, Kubernetes provides **NetworkPolicy** objects.

---

## 2️⃣ Example Environment – Kind Cluster

We provisioned a cluster using:

Kind (Kubernetes IN Docker)

Cluster name: `cka-qacluster`
Nodes: 1 control-plane + 3 worker nodes

```bash
kubectl get nodes
```

Output:

```
NAME                          STATUS   ROLES           AGE   VERSION
cka-qacluster-control-plane   Ready    control-plane   19d   v1.31.0
cka-qacluster-worker          Ready    <none>          19d   v1.31.0
cka-qacluster-worker2         Ready    <none>          19d   v1.31.0
cka-qacluster-worker3         Ready    <none>          19d   v1.31.0
```

---

## 3️⃣ What Happens When a Kind Cluster is Created?

When you create a Kind cluster:

### ✅ Flat Network Model

* Every Pod gets its own IP.
* Pods communicate across nodes without NAT.

### ✅ CNI Plugin Installed Automatically

Kind installs **kindnet** by default.

```bash
kubectl get pods -n kube-system | grep kindnet
```

### ✅ Service Discovery

Kubernetes Services provide stable endpoints:

* ClusterIP
* NodePort
* LoadBalancer

### ✅ Pod-to-Pod Communication

Even across 3 worker nodes, Pods can talk seamlessly.

---

## 🚨 Why This Can Be a Problem

Because Kubernetes allows all traffic by default:

* ❌ A compromised Pod can attack others
* ❌ Multi-tenant workloads may interfere
* ❌ Databases may be exposed internally
* ❌ Security/compliance risks

---

## 🏙 Simple Analogy

Think of the cluster as a city with 3 districts (nodes):

* Kubernetes builds roads automatically.
* Everyone can travel freely.
* If you want traffic rules → You create **NetworkPolicies**.

---

# 4️⃣ Creating a Cluster WITHOUT Default CNI

To control networking manually, we disable Kind’s default CNI.

`confignode.yml`

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4

nodes:
- role: control-plane
  extraPortMappings:
  - containerPort: 32000
    hostPort: 32000
- role: worker
- role: worker
- role: worker

networking:
  disableDefaultCNI: true
  podSubnet: 192.168.0.0/16
```

### Key Settings

* `disableDefaultCNI: true`
  → Do NOT install kindnet.
* `podSubnet: 192.168.0.0/16`
  → Defines Pod IP range.

Create cluster:

```bash
kind create cluster --name cka-prodcluster --config confignode.yml
```

---

# 5️⃣ Problem: Nodes Show `NotReady`

```bash
kubectl get nodes
```

Output:

```
STATUS: NotReady
```

### Why?

Because we disabled the CNI — and Kubernetes networking is not initialized.

From:

```bash
kubectl describe node <node-name>
```

You will see:

```
KubeletNotReady
NetworkPluginNotReady
cni plugin not initialized
```

---

# 6️⃣ Common Reasons for NotReady Nodes

* Docker not running properly
* Port conflicts (6443 etc.)
* CNI plugin missing
* Insufficient CPU/Memory

### Important Kubernetes Ports

* 6443 → API Server
* 10250 → Kubelet
* 30000–32767 → NodePort range

---

# 7️⃣ Installing a Network Policy Provider

To fix the issue, install a CNI plugin that supports NetworkPolicy.

Example: Weave Net

```bash
kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
```

Check:

```bash
kubectl get ds -n kube-system
```

Once `weave-net` becomes READY:

```bash
kubectl get nodes
```

Now:

```
STATUS: Ready
```

---

# 8️⃣ 3-Tier Architecture Example

Architecture:

Frontend → Backend → Database

---

## Deployment File: `laxmi3tire.yml`

Contains:

* frontend Pod + Service (nginx)
* backend Pod + Service (nginx)
* mysql Pod + Service (DB)

Apply:

```bash
kubectl apply -f laxmi3tire.yml
```

---

# 9️⃣ Testing Communication (Before NetworkPolicy)

Enter frontend Pod:

```bash
kubectl exec -it frontend -- bash
```

Test backend:

```bash
curl backend:80
```

✅ Success

Test DB:

```bash
telnet db 3306
```

✅ Connected

**This is insecure.**
Frontend should NOT directly access DB.

---

# 🔟 Creating NetworkPolicy

Goal:

Allow only **backend → mysql**
Block **frontend → mysql**

`netpolicy.yml`

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-test
spec:
  podSelector:
    matchLabels:
      name: mysql
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: backend
    ports:
    - port: 3306
```

Apply:

```bash
kubectl apply -f netpolicy.yml
```

---

# 1️⃣1️⃣ Testing After NetworkPolicy

From frontend:

```bash
telnet db 3306
```

❌ Connection timed out

Policy is working.

Now:

* Frontend ❌ cannot access DB
* Backend ✅ can access DB

This is proper 3-tier isolation.

---

# 1️⃣2️⃣ Ingress vs Egress

* **Ingress** → Incoming traffic to Pod
* **Egress** → Outgoing traffic from Pod
* NetworkPolicy can control both directions.

---

# 1️⃣3️⃣ Port Forwarding

`kubectl port-forward` creates a secure tunnel from your local machine to a Pod or Service.

Example:

```bash
kubectl port-forward pod/my-pod 8080:80
```

Access:

```
http://localhost:8080
```

Local port 8080 → Pod port 80

---

# ✅ Summary

* Kubernetes allows all Pod communication by default.
* Kind installs kindnet automatically.
* Disabling CNI causes Nodes to be `NotReady`.
* Installing a CNI like Weave Net restores networking.
* NetworkPolicies enforce isolation.
* 3-tier architecture requires restricting DB access.
* NetworkPolicy ensures only backend talks to DB.

---

# 🔐 Final Takeaway

Kubernetes builds the roads.
NetworkPolicies install the traffic rules.

Without policies → Everything talks to everything.
With policies → Secure, production-ready isolation.
