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

## 1️⃣ Kubernetes ClusterIP Service 🏠

## What is a ClusterIP Service?

`ClusterIP` is the default type of Kubernetes Service. It exposes the Service **only within the cluster**. This type is typically used for **internal communication between Pods**.

### Use Case

Use `ClusterIP` when you need to make an application available only **inside the Kubernetes cluster**, such as for microservices that don’t require external exposure.

---

## Example Configuration

Here’s a YAML file to create a ClusterIP Service for a simple web server running on port 80.

> When your NGINX Deployment is running, you need a way to access it. Connecting directly to Pods does not load balance and can cause errors if a Pod becomes unhealthy. A Service allows traffic routing between replicas reliably.

### ClusterIP Service Manifest

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
````

**Key points:**

* `spec.type` is set to `ClusterIP`.
* `spec.selector` selects the NGINX Pods using the `app: nginx` label.
* `spec.ports` maps traffic from port `8080` on the Service to port `80` on the Pods.

---

## Apply the Service

Save the manifest as `clusterip.yaml` and run:

```bash
kubectl apply -f clusterip.yaml
```

Output:

```
service/nginx-clusterip created
```
---

## Discover the ClusterIP

Use the following command:

```bash
kubectl get svc
```

Example output:

```
NAME              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
kubernetes        ClusterIP   10.96.0.1       <none>        443/TCP    2d23h
nginx-clusterip   ClusterIP   10.96.233.215   <none>        8080/TCP   98s
```

In this example, the Service IP is `10.96.233.215`. You can connect to this IP **from within the cluster**, and traffic will be automatically load balanced between all Pod replicas.

---

## Test the Service

Use `kubectl exec` to curl the Service IP from inside one of the NGINX Pods:

```bash
kubectl exec deployment/nginxapp -- curl 10.96.233.215:8080
```

You should see the standard NGINX welcome page:

```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
</html>
```

---

## What Happens When You Curl a ClusterIP?

* You are **not connecting to a specific Pod**.
* You are hitting the **Service (ClusterIP)**.
* Kubernetes forwards the request to **one of the Pods** backing the Service.
* Which Pod is chosen? Kubernetes uses `kube-proxy`:

  * Round-robin or random load balancing across healthy Pods
  * You don’t know in advance which Pod will serve the request

---

## Check Service Endpoints

Option 1: Check endpoints of a Service:

```bash
kubectl get endpoints <service-name>
```

Example:

```bash
kubectl get endpoints nginx-clusterip
```

Output:

```
NAME              ENDPOINTS                                      AGE
nginx-clusterip   10.244.1.16:80,10.244.1.17:80,10.244.2.14:80   13m
```

> These IPs are the Pod IPs receiving traffic.

Option 2: Check Pod logs:

```bash
kubectl logs pod/nginxapp-xxxxx
```

Run `curl` multiple times and see which Pod logs the request.

---

## Final Takeaway

* `ClusterIP` is an **abstraction layer** — you never talk to Pods directly.
* Kubernetes **load-balances** requests for you.
* To know which Pod served traffic:

  * ✅ Check Endpoints
  * ✅ Check Pod logs
  * ✅ Or expose the Pod hostname in the response

---

## 2️⃣ NodePort Service 🔌

**NodePort** exposes your app on a static port on each Node, allowing external access.

---

# Kind + NodePort Example (Advanced)

This guide shows how to configure a **Kind** Kubernetes cluster with extra port mappings to expose a NodePort service to your host machine.

---

## 1️⃣ Create Kind Cluster with Extra Port Mappings

NodePort services only work from your host if the cluster is configured with extra port mappings. Here’s an example cluster configuration:

```yaml
# kind-config.yaml
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
```

Create the cluster:

```bash
kind create cluster --config kind-config.yaml
```

> ⚠️ Without the `extraPortMappings`, NodePort services are **unreachable** from the host machine.

---

## 2️⃣ NodePort Service

A **NodePort** service exposes a Pod to external traffic. NodePort services open ports in the range **30000–32767**.

Here’s an example manifest for an NGINX NodePort service:

```yaml
# nodeport.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - port: 80        # Target port inside the Pod
      nodePort: 32000 # Port exposed on host
```

Apply the manifest:

```bash
kubectl apply -f nodeport.yaml
```

---

## 3️⃣ Accessing the Service

1. Find the Node IP (for Kind, this is always `localhost`):

```bash
curl http://localhost:32000
```

You should see the NGINX welcome page:

```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
</html>
```

> ✅ In Kind, **NodePort services are accessed via `localhost:<nodePort>`**, not the Node IP.

---

## 4️⃣ Verify Deployment

Check your Pods:

```bash
kubectl get pods -l app=nginx
```

Check your Service:

```bash
kubectl get svc
```

Check Service Endpoints:

```bash
kubectl get endpoints nginx-nodeport
```

Example output:

```
NAME              ENDPOINTS
nginx-nodeport    10.244.0.7:80,10.244.0.8:80,10.244.0.9:80
```

> These are the actual Pod IPs serving traffic.

---

## 5️⃣ Quick Rules

| Resource   | Creates Endpoints? |
| ---------- | ------------------ |
| Deployment | ❌ (just Pods)      |
| ReplicaSet | ❌ (just Pods)      |
| Service    | ✅ (yes, Endpoints) |

💡 Endpoints always belong to a **Service**, not directly to a Deployment.

---

✅ **Takeaway**: In a Kind cluster, you must configure `extraPortMappings` to expose NodePort services to your host. Access them using `http://localhost:<nodePort>`.

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

