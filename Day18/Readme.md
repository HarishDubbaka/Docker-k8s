# Kubernetes Services: A Practical Guide ⚓️

One of Kubernetes’ core concepts is the **Service**, which abstracts access to a group of Pods. Services provide a way to expose applications running on a set of Pods to other Pods, clients, or external users. Let’s explore the various types of Kubernetes Services and how to deploy them. 🌐

---

## What is a Kubernetes Service? 🧩

A **Service** defines a logical set of Pods and a policy for accessing them. Since Pods have dynamic IPs, Services provide a **stable IP address and DNS name**, allowing clients to connect reliably to a consistent endpoint.

Services enable:

* 🔄 Pod-to-Pod communication
* 🌍 External access to applications
* ⚖️ Load balancing across Pods

---

## Types of Kubernetes Services 🛠️

There are four primary types of Services:

1. **ClusterIP** – Internal, cluster-only access 🏠
2. **NodePort** – Exposes the Service on a static port on each node 🔌
3. **LoadBalancer** – Exposes the Service externally via cloud provider load balancer ☁️
4. **ExternalName** – Maps a Service to an external DNS name 🌐

---

## Deploy a Sample App 🐳

We’ll use an NGINX web server for demonstration. Save the following manifest as `nginxapp.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginxapp
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
          - containerPort: 80
```

Deploy the app:

```bash
kubectl apply -f nginxapp.yaml
```

Check that all Pods are ready:

```bash
kubectl get deployments
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
nginxapp   3/3     3            3           1m
```

---

## 1️⃣ ClusterIP Service 🏠

**ClusterIP** exposes the Service **only within the cluster**. Ideal for internal communication between Pods.

### Manifest (`clusterip.yaml`):

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-clusterip
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
    - port: 8080
      targetPort: 80
```

Deploy it:

```bash
kubectl apply -f clusterip.yaml
kubectl get svc
```

Example output:

```
NAME              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
nginx-clusterip   ClusterIP   10.96.233.215  <none>        8080/TCP   1m
```

Access the Service from inside the cluster:

```bash
kubectl exec deployment/nginxapp -- curl 10.96.233.215:8080
```

You should see the NGINX welcome page. 🎉

---

### 🔍 Understanding ClusterIP behavior

* Curling the ClusterIP hits the **Service**, not a specific Pod
* Kubernetes forwards requests to one of the Pods via **kube-proxy** (round-robin load balancing) 🔄
* To see which Pods serve traffic:

```bash
kubectl get endpoints nginx-clusterip
```

Example:

```
NAME              ENDPOINTS
nginx-clusterip   10.244.1.16:80,10.244.1.17:80,10.244.2.14:80
```

✅ Endpoints always belong to **Services**, not Deployments.

---

## 2️⃣ NodePort Service 🔌

**NodePort** exposes your app on a static port on each Node, allowing external access.

### Manifest (`nodeport.yaml`):

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - port: 80
      nodePort: 32000
```

Deploy it:

```bash
kubectl apply -f nodeport.yaml
```

Access the Service:

```bash
curl http://localhost:32000
```

> In a local kind cluster, NodePort is accessible via `localhost:<nodePort>`.

---

## 3️⃣ LoadBalancer Service ☁️

**LoadBalancer** creates an external load balancer (cloud provider dependent) to route traffic to Pods.

### Example:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-server-loadbalancer
spec:
  selector:
    app: web
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: LoadBalancer
```

Access the Service via the **external IP** assigned by the cloud provider. 🌐

---

## 4️⃣ ExternalName Service 🌐

**ExternalName** maps a Kubernetes Service to an external DNS name.

### Example:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-database
spec:
  type: ExternalName
  externalName: externaldb.example.com
```

Use Case: Access external services without creating additional DNS records. ✅

---

## ✅ Summary 🏁

| Service Type    | Use Case                                          |
| --------------- | ------------------------------------------------- |
| ClusterIP 🏠    | Internal communication within the cluster         |
| NodePort 🔌     | External access via Node IP and port              |
| LoadBalancer ☁️ | Production-grade internet access through cloud LB |
| ExternalName 🌐 | Map Service to an external DNS name               |

💡 **Key Points**

* Deployments create **Pods**; Services create **Endpoints**
* Check endpoints with `kubectl get endpoints <service-name>` to see which Pods receive traffic
* Choose Service type based on **application needs and exposure**

---

