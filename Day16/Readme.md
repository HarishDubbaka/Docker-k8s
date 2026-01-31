# 🚀 Kubernetes Pods & Multi-Container Pods

## 📦 Understanding Pods
Understand Pods, the smallest deployable compute object in Kubernetes, and the higher-level abstractions that help you to run them.  
A workload is an application running on Kubernetes. Whether your workload is a single component or several that work together, on Kubernetes you run it inside a set of pods. In Kubernetes, a Pod represents a set of running containers on your cluster.

A container as we all know, is a self-contained environment where we package applications and their dependencies. Typically, a container runs a single process (Although there are ways to run multiple processes). Each container gets an IP address and can attach volumes and control CPU and memory resources, among other things. All these happen via the concepts of namespaces and control groups.

Kubernetes is a container orchestration system for deploying, scaling, and managing containerized applications, and it has its own way of running containers. We call it a pod. A pod is the smallest deployable unit in Kubernetes that represents a single instance of an application.

### ❓ So how does it differ from a container?

A container is a single unit. However, a pod can contain more than one container. You can think of pods as a box that can hold one or more containers together.

Pod provides a higher level of abstraction that allows you to manage multiple containers as a single unit. Here instead of each container getting an IP address, the pod gets a single unique IP address and containers running inside the pod use localhost to connect to each other on different ports.

---

## 🛠️ Creating Pod (Practical Examples)

You can create a pod in two ways:

- **Using the kubectl imperative command**: Primarily used for learning and testing purposes. The imperative command comes with its own limitations.
- **Declarative approach**: Using YAML manifest. When working on projects, the YAML manifest is used to deploy pods.

> ⚠️ **Note:** Kubectl imperative commands are very important when you appear for Kubernetes Certifications, as they save time compared to declarative approaches.

```bash
kubectl run devweb-server-pod \
  --image=nginx:1.14.2 \
  --restart=Never \
  --port=80 \
  --labels=app=web-server,environment=production \
  --annotations description="This pod runs the devweb server"
````

Here the pod gets deployed in the default namespace. You can get the status of the deployed pod using kubectl.

```bash
kubectl get pods
kubectl get pods -w
```

---

## 🔍 Describe a Pod

If you want to know all the details of the running pod, you can describe the pod using kubectl.

```bash
kubectl describe pod web-server-pod
```

Look for the field:

```
Node: <node-name>/<node-IP>
```

👉 This tells you exactly which node (control-plane or worker) the Pod is scheduled on.

To see which cluster node the pod is running on:

```bash
kubectl get pods -o wide
```

---

## 🧾 Pod YAML (Object Definition)

Now that we have a basic understanding of a Pod, let's have a look at how we define a Pod. Pod is a native Kubernetes Object and if you want to create a pod, you need to declare the pod requirements in YAML format.

Here is an example Pod YAML that creates an Nginx web server pod. This YAML is nothing but a declarative desired state of a pod.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-server-pod
  labels:
    app: web-server
    environment: production
  annotations:
    description: This pod runs the web server
spec:
  containers:
  - name: web-server
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

Deploy the manifest:

```bash
kubectl create -f nginx.yaml
```

---

## 🧪 Using --dry-run to Generate YAML

```bash
kubectl run nginx-pod --image=nginx:1.14.2 --dry-run=client -o yaml
kubectl run nginx-pod --image=nginx:1.14.2 --dry-run=client -o yaml > nginx-pod.yaml
```

✅ **Answer**
No — `--dry-run=client` will not create the Pod.
It only prepares the YAML manifest so you can review or save it.

To actually create the Pod:

```bash
kubectl apply -f nginx-pod.yaml
```

---

## 🌐 Access Application Running In a Pod

```bash
kubectl port-forward pod/web-server-pod 8080:80
```

Now access:
👉 [http://localhost:8080](http://localhost:8080)

> ⚠️ `kubectl port-forward` is mainly a debugging utility. For real access, use a Kubernetes **Service**.

---

## 🖥️ Access Pod Shell

```bash
kubectl exec -it web-server-pod -- /bin/sh
```

> 📝 Note: Container images are usually minimal, so not all Linux commands may be available.

---

## 🔄 Pod Lifecycle

Pod lifecycle phases:

* **Pending** ⏳
* **Running** ▶️
* **Succeeded** ✅
* **Failed** ❌
* **Unknown** ❓

---

## 🧩 Multi-Container Pods in Kubernetes

Multi-container pods allow you to group tightly coupled containers that:

* Share the same **network IP**
* Communicate via **localhost**
* Share **storage volumes**
* Are managed as a single unit

---

## 🧠 Multi-Container Pod Patterns

### 1️⃣ Sidecar Pattern

Enhances the main application (logging, monitoring, proxying)

### 2️⃣ Ambassador Pattern

Acts as a proxy for external communication

### 3️⃣ Adapter Pattern

Transforms data formats or protocols

### 4️⃣ Init Containers

Run setup tasks before the main container starts

---

## 🧪 Multi-Container Pod Example YAML

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod-patterns
spec:
  containers:
  - name: main-app
    image: nginx:latest
    ports:
    - containerPort: 80

  - name: log-collector
    image: busybox
    command: ["sh", "-c", "tail -n+1 -f /var/log/nginx/access.log"]
```

---

```

If you want next:
- 🎯 **CKA / CKAD exam-focused version**
- 📘 **Shortened GitHub-friendly README**
- 🧩 **One pattern per YAML (sidecar-only, ambassador-only)**

Just say the word 👍
```
