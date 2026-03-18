# Day 58 – Mastering Kubernetes Output with JSONPath (with Example)

Yesterday, we explored how **kubectl** works behind the scenes — fetching data from the Kubernetes API server in **JSON format**.

But by default, we only see a **human-readable table**.

👉 So how do we extract exactly what we need?
**Answer: JSONPath**

---

## 🔍 Real-Time Example

Let’s say you run:

```bash
kubectl get pods -n dev
```

👉 Output:

```
NAME        READY   STATUS    RESTARTS   AGE
nginx-1     1/1     Running   0          2d
redis-1     1/1     Pending   0          1d
api-1       1/1     Running   1          3d
```

🎯 Requirement: Get only **Running pods**

---

## ⚡ Using JSONPath to Filter Data

```bash
kubectl get pods -n dev -o=jsonpath='{range .items[?(@.status.phase=="Running")]}{.metadata.name}{"\n"}{end}'
```

👉 Output:

```
nginx-1
api-1
```

✅ No manual filtering — **instant results**

---

## 🛠️ Practical Examples

### 🎯 Pod → Node Mapping

```bash
kubectl get pods -o=jsonpath='{range .items[*]}{.metadata.name} {"-->"} {.spec.nodeName}{"\n"}{end}'
```

👉 Output:

```
nginx-1 --> node01
redis-1 --> node02
api-1 --> node01
```

---

### 📊 Extract Container Image

```bash
kubectl get pod nginx-1 -o=jsonpath='{.spec.containers[*].image}'
```

👉 Output:

```
nginx:1.21
```

---

## 🖥️ Node-Level Examples

### 🔹 Get Node Architecture

```bash
kubectl get nodes -o=jsonpath='{.items[*].status.nodeInfo.architecture}'
```

👉 Output:

```
amd64 amd64
```

---

### 🔹 Get CPU Capacity

```bash
kubectl get nodes -o=jsonpath='{.items[*].status.capacity.cpu}'
```

👉 Output:

```
4 4
```

---

### 🔹 Combine Multiple Fields

```bash
kubectl get nodes -o=jsonpath='{.items[*].metadata.name}{"\n"}{.items[*].status.capacity.cpu}'
```

👉 Output:

```
master node01
4 4
```

---

## 📊 Advanced Formatting

### 🔹 Table Format using Loop

```bash
kubectl get nodes -o=jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.capacity.cpu}{"\n"}{end}'
```

👉 Output:

```
master   4
node01   4
```

---

### 🔹 Using Custom Columns (Simpler Way)

```bash
kubectl get nodes -o=custom-columns=NODE:.metadata.name,CPU:.status.capacity.cpu
```

---

### 🔹 Sorting Output

```bash
kubectl get nodes --sort-by=.metadata.name
kubectl get nodes --sort-by=.status.capacity.cpu
```

---

## 📌 Basic Syntax

```
kubectl get <resource> -o jsonpath='{expression}'
```

Example:

```
kubectl get pods -o jsonpath='{.items[*].metadata.name}'
```

👉 Outputs only pod names

---

## ⚡ Most Useful JSONPath Expressions

* **Get Pod Names**

  `kubectl get pods -o jsonpath='{.items[*].metadata.name}'`

* **Get Pod + Node Mapping**

  `kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{" -> "}{.spec.nodeName}{"\n"}{end}'`

* **Get Pod IPs**

  `kubectl get pods -o jsonpath='{.items[*].status.podIP}'`

* **Get Container Images**

  `kubectl get pods -o jsonpath='{.items[*].spec.containers[*].image}'`

* **Filter by Condition (Advanced)**

  `kubectl get pods -o jsonpath='{.items[?(@.status.phase=="Running")].metadata.name}'`

* **Get Labels**

  `kubectl get pods -o jsonpath='{.items[*].metadata.labels}'`

---

## 🔁 Looping (Very Important)

```
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\n"}{end}'
```

👉 `range` works like a loop

---

## 🚀 Advanced kubectl Commands

* **Custom Columns** (Better than JSONPath sometimes)

  `kubectl get pods -o custom-columns=NAME:.metadata.name,NODE:.spec.nodeName,IP:.status.podIP`

* **Sort Output**

  `kubectl get pods --sort-by=.metadata.creationTimestamp`

* **Describe Only Needed Fields**

  `kubectl get pod <pod-name> -o jsonpath='{.status.containerStatuses[*].restartCount}'`

* **Get Restart Count of All Pods**

  `kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{" -> "}{.status.containerStatuses[*].restartCount}{"\n"}{end}'`

* **Check Node Capacity**

  `kubectl get nodes -o jsonpath='{range .items[*]}{.metadata.name}{" CPU:"}{.status.capacity.cpu}{"\n"}{end}'`

* **Combine with watch**

  `watch kubectl get pods -n <namespace> -o custom-columns=NAME:.metadata.name,STATUS:.status.phase`

* **Extract Secrets (Base64 decode)**

  `kubectl get secret my-secret -o jsonpath='{.data.password}' | base64 --decode`

* **Debug Pod Quickly**

  `kubectl exec -it <pod> -- /bin/bash`

* **Logs of Multiple Pods**

  `kubectl logs -l app=myapp --all-containers=true`

* **Port Forward**

  `kubectl port-forward pod/<pod-name> 8080:80`

---

## 🧠 Pro Tips (Advanced Usage)

* ✅ Combine JSONPath + grep for quick filtering

  `kubectl get pods -o json | jq '.items[].metadata.name'`

* ✅ Use `jq` when JSONPath falls short — it's more powerful and expressive

* ✅ Find Non-Running Pods

  `kubectl get pods --field-selector=status.phase!=Running`

* ✅ Get Events Sorted

  `kubectl get events --sort-by=.metadata.creationTimestamp`

---

## ⚠️ Common Mistakes

* Missing quotes `{}`
* Using wrong field path
* Forgetting `.items[*]` for lists

---

## ✅ One-line Summary

Instead of dumping full YAML/JSON, pick only what you need — JSONPath lets you extract the exact fields you want.

---

## ✅ Conclusion

## 💡Why JSONPath is Essential
JSONPath simplifies the management of Kubernetes resources, especially when dealing with large clusters. By using the power of JSONPath queries, you can precisely target the data you need, making troubleshooting, reporting, and monitoring much easier.
Kubernetes
Kubectl
Jsonpath
Json

## 💡 Why This Matters (Real Scenario)

In production environments:

* 🖥️ Hundreds of nodes
* 🚀 Thousands of pods
* ⚙️ Multiple deployments

❌ Manual checking is slow
✅ JSONPath makes it automated and precise

👉 Common use cases:

* Get **failed pods instantly**
* Extract **node capacity details**
* Build **monitoring scripts**
* Automate **health checks**

---

## 🧠 Key Takeaway

> Don’t read everything.
> Extract only what you need using JSONPath.

---

## 📌 What’s Next?

👉 **Next – Real-time Project**

