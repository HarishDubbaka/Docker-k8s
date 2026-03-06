# Day 48 – Kubernetes Admission Controllers

Kubernetes is a fantastic orchestration platform, but as your clusters grow and the need for policies and validations increases, you’ll start encountering a common question:

> **How do I control or validate what gets deployed into my cluster?**

The answer lies in **Admission Controllers**.

In this guide, we will explore:

* What Admission Controllers are
* Why they matter
* How they work
* How to enable them
* Practical hands-on examples

By the end of this article, you’ll understand how to control deployments and enforce policies in your Kubernetes cluster.

---

# What are Kubernetes Admission Controllers?

A **Kubernetes Admission Controller** is code that intercepts requests sent to the **Kubernetes API Server**.

The request flow inside Kubernetes:

```
User Request
     │
Authentication
     │
Authorization
     │
Admission Controllers
     │
Object Stored in etcd
```

Admission controllers run **after authentication and authorization but before the object is persisted in etcd**.

They allow cluster administrators to:

* Enforce security policies
* Modify incoming requests
* Reject invalid configurations
* Maintain cluster integrity

---

# Types of Admission Controllers

Kubernetes supports two main types of admission controllers.

## 1️⃣ Mutating Admission Controllers

These controllers **modify requests before they are stored**.

Example use cases:

* Inject sidecar containers
* Add default resource limits
* Modify container images
* Add labels or annotations

Example controller:

```
MutatingAdmissionWebhook
```

---

## 2️⃣ Validating Admission Controllers

These controllers **validate requests and either allow or reject them**.

Example use cases:

* Prevent deployments using untrusted images
* Enforce required labels
* Block privileged containers
* Enforce security policies

Example controller:

```
ValidatingAdmissionWebhook
```

---

# Why Use Admission Controllers?

Admission Controllers give you **fine-grained control over your Kubernetes cluster**.

Common use cases:

### Security Enforcement

Ensure all pods:

* run as non-root
* have security contexts
* use approved container registries

### Automatic Mutation

Automatically inject:

* service mesh sidecars
* default labels
* resource limits

### Custom Validation

Reject:

* containers using latest tag
* deployments without resource limits
* pods running in restricted namespaces

### Cost Management

Prevent pods without resource limits to avoid cluster overconsumption.

---

# Kubernetes Cluster Setup

Lab environment used in this exercise.

```
kubectl get nodes
```

Output:

```
NAME           STATUS   ROLES           AGE     VERSION
controlplane   Ready    control-plane   6h13m   v1.30.0
node01         Ready    <none>          6h13m   v1.30.0
```

---

Check system pods.

```
kubectl get pods -n kube-system
```

Output:

```
coredns
etcd
kube-apiserver
kube-controller-manager
kube-scheduler
kube-proxy
```

---

# Checking Enabled Admission Controllers

Method 1: Check API Server Pod

```
kubectl get pod kube-apiserver-controlplane \
-n kube-system -o yaml | grep -i admission
```

Output:

```
--enable-admission-plugins=NodeRestriction
```

---

# Method 2: Check API Server Manifest

Navigate to Kubernetes manifest directory.

```
cd /etc/kubernetes/manifests
```

List files:

```
ls -ltr
```

```
kube-apiserver.yaml
kube-controller-manager.yaml
kube-scheduler.yaml
etcd.yaml
```

Open the API server manifest.

```
cat kube-apiserver.yaml
```

Inside the command section:

```
--enable-admission-plugins=NodeRestriction
```

---

# Namespace Creation Test

Check existing namespaces.

```
kubectl get ns
```

Output:

```
default
kube-system
kube-public
kube-node-lease
```

---

Try creating a pod in a namespace that does not exist.

```
kubectl run harishnginx1 --image=nginx -n harish
```

Error:

```
Error from server (NotFound): namespaces "harish" not found
```

Reason:

The namespace **does not exist**.

---

If we do not specify a namespace, the pod will be created in **default namespace**.

```
kubectl run harishnginx1 --image=nginx
```

Output:

```
pod/harishnginx1 created
```

---

# Enabling NamespaceAutoProvision Admission Controller

This admission controller **automatically creates a namespace if it does not exist**.

---

## Step 1: Backup API Server Manifest

Navigate to manifest directory.

```
cd /etc/kubernetes/manifests
```

Create backup:

```
cp -pr kube-apiserver.yaml kube-apiserver.yaml_Bkup
```

---

## Step 2: Edit API Server Manifest

```
vi kube-apiserver.yaml
```

Modify this line:

```
--enable-admission-plugins=NodeRestriction
```

Change it to:

```
--enable-admission-plugins=NodeRestriction,NamespaceAutoProvision
```

---

## Step 3: Restart API Server

Since kube-apiserver is a **static pod**, kubelet will automatically restart it when the file changes.

You can also simulate restart by moving files temporarily.

```
mv kube-scheduler.yaml kube-apiserver.yaml /tmp
```

Then move them back.

```
mv /tmp/kube-scheduler.yaml /tmp/kube-apiserver.yaml /etc/kubernetes/manifests
```

---

Check pods again.

```
kubectl get pods -n kube-system
```

---

Verify admission plugins again.

```
kubectl get pod kube-apiserver-controlplane \
-n kube-system -o yaml | grep -i admission
```

Output:

```
--enable-admission-plugins=NodeRestriction,NamespaceAutoProvision
```

---

# Testing NamespaceAutoProvision

Now create a pod in a namespace that does not exist.

```
kubectl run harishnginx2 --image=nginx -n harish
```

Output:

```
pod/harishnginx2 created
```

Check namespaces again.

```
kubectl get ns
```

Output:

```
default
harish
kube-system
kube-public
kube-node-lease
```

The namespace **was automatically created**.

---

# Built-in Admission Controllers Examples

## ResourceQuota (Validating Controller)

ResourceQuota limits resource usage in a namespace.

### Create Resource Quota

File: `resource-quota-demo.yaml`

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: quota-demo
---
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: quota-demo
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
    pods: "4"
```

Apply configuration:

```
kubectl apply -f resource-quota-demo.yaml
```

---

### Create Pod That Exceeds Quota

File: `big-pod.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: big-pod
  namespace: quota-demo
spec:
  containers:
  - name: big-container
    image: nginx
    resources:
      requests:
        memory: "2Gi"
        cpu: "1"
```

Apply:

```
kubectl apply -f big-pod.yaml
```

The **ResourceQuota admission controller will reject this pod**.

---

# LimitRanger (Mutating Controller)

LimitRanger automatically injects default resource limits.

---

### Step 1: Create LimitRange

File: `limit-range-demo.yaml`

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-limit-range
  namespace: quota-demo
spec:
  limits:
  - default:
      memory: "512Mi"
      cpu: "200m"
    defaultRequest:
      memory: "256Mi"
      cpu: "100m"
    type: Container
```

Apply:

```
kubectl apply -f limit-range-demo.yaml
```

---

### Step 2: Create Pod Without Resources

File: `simple-pod.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-pod
  namespace: quota-demo
spec:
  containers:
  - name: simple-container
    image: nginx
```

Apply:

```
kubectl apply -f simple-pod.yaml
```

---

### Step 3: Verify Injected Resources

```
kubectl get pod simple-pod -n quota-demo -o yaml | grep -A 10 resources
```

You will see **default CPU and memory limits automatically added**.

---

# Learning Outcome

After completing this lab, you learned:

* What **Admission Controllers** are
* Difference between **Mutating vs Validating controllers**
* How to check enabled admission plugins
* How to enable **NamespaceAutoProvision**
* How **ResourceQuota** controls resource usage
* How **LimitRanger** injects default resource limits

---

# Lab Environment

Practice environment:

[https://kodekloud.com/public-playgrounds/multi-node-k8s-1-30/](https://kodekloud.com/public-playgrounds/multi-node-k8s-1-30/)

