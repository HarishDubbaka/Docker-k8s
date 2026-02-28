# 🖥️ Kubernetes Ingress Guide

When deploying applications on Kubernetes, one of the most common challenges is managing how external traffic reaches services inside the cluster. Kubernetes provides several ways to expose services such as **NodePort**, **LoadBalancer**, and **ClusterIP**, but each has limitations.

**Ingress** is a more advanced and flexible solution for managing HTTP and HTTPS traffic, providing features that make it the preferred option for most use cases.

---

## Why Ingress Matters (Controller-Agnostic)

### 1. Centralized Management of External Access

* NodePort → exposes services on unique ports across nodes, messy at scale.
* LoadBalancer → one per service, costly in cloud environments.
* Ingress → consolidates routing rules into a single resource for easier management.

---

### 2. Path-Based and Host-Based Routing

* **Path-based routing:** Different services under one domain via URL paths.

  * `example.com/api` → API service
  * `example.com/app` → App service
* **Host-based routing:** Different services under different subdomains.

  * `api.example.com` → API service
  * `app.example.com` → App service

---

### 3. SSL/TLS Termination

* Ingress can centrally handle HTTPS traffic.
* Certificates managed in one place, simplifying encryption for backend services.

---

### 4. Load Balancing

* Distributes traffic across Pods automatically.
* Supports scaling and health checks, ensuring requests reach healthy Pods only.

---

### 5. Cost Efficiency

* One entry point for the cluster reduces cloud load balancer costs.

---

### 6. Advanced Traffic Routing

* URL rewriting, redirection, rate limiting, and traffic splitting (A/B testing, canary releases).

---

### 7. Security Features

* Authentication, IP whitelisting/blacklisting, and TLS encryption between Ingress and backend services.

---

### 8. External DNS Integration

* Works with tools like **ExternalDNS** to automatically update DNS records.

---

### 9. Flexibility

* Kubernetes-native, not tied to a single implementation.
* Supports a wide range of routing, security, and traffic management needs.

---

### 10. Declarative Configuration

* YAML-based, version-controlled, GitOps-friendly.
* Fits naturally into Kubernetes’ declarative deployment model.

---

## Kubernetes Ingress Overview

Ingress is a native Kubernetes resource, similar to Pods or Deployments, used to manage external access to services.

**Ingress = traffic that enters the cluster**
**Egress = traffic that exits the cluster**

The **Ingress Controller** does the actual routing by reading rules from Ingress objects stored in `etcd`.

### High-Level Example

* **Without Ingress:** Each service uses NodePort or LoadBalancer.

![Image Alt](https://github.com/HarishDubbaka/Docker-k8s/blob/e5abe19035cc24c45f672833802a4f0a035b012c/Day43/withoutingress.png).

* **With Ingress:** Reverse proxy layer (Ingress Controller) routes traffic to services.

![Image Alt](https://github.com/HarishDubbaka/Docker-k8s/blob/e5abe19035cc24c45f672833802a4f0a035b012c/Day43/withingress%20controller.png).

---

## Before Kubernetes Ingress

### 1. Custom Reverse Proxies

* Deploy Nginx/HAProxy as Pods with a LoadBalancer.
* Routes stored in ConfigMaps; manual updates required.

### 2. Manual Service Discovery

* External tools like Consul updated DNS or proxy configs automatically.
* Reduced downtime but complex setup.

### 3. OpenShift Router

* Early approach using HAProxy-based Router.
* Operators define routes via YAML; Router manages external exposure.

---

## Transition to Kubernetes Ingress

* Ingress formalized routing into a native resource.
* Rules defined as Ingress objects, picked up dynamically by the Ingress Controller.
* Eliminates external scripts or manual reloads.

**Key Difference:**

* Before → ad hoc, proxy-based routing
* After → declarative, Kubernetes-native, dynamic

---

## How Kubernetes Ingress Works

1. **Ingress Resource:** Stores DNS routing rules in the cluster.
2. **Ingress Controller:** Reads the rules and routes traffic accordingly.

---

## How an Ingress Controller Works (Nginx Example)

* The **nginx.conf** inside the controller pod is a template that dynamically updates routes from Ingress resources.
* For each Ingress object:

  * Configuration is generated under `/etc/nginx/conf.d/`
  * Main `/etc/nginx/nginx.conf` references these files
  * Updates to Ingress objects trigger a graceful reload

---

## Common Kubernetes Ingress Controllers

* Nginx Ingress Controller (Community & Nginx Inc)
* Traefik
* HAProxy
* Contour
* GKE Ingress Controller
* AWS ALB Ingress Controller
* Azure Application Gateway Ingress Controller

*Reference:* Learnk8s Ingress Controller Comparison

---

## Deploy Your First Ingress Controller

We will use the **Kubernetes community Nginx Ingress Controller**.

### Prerequisites

* Kubernetes cluster
* `kubectl` installed and authenticated
* Admin access to the cluster
* Optional: Valid domain pointing to LoadBalancer IP

---

### Installation (Kind / Local)

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
kubectl get pods -n ingress-nginx
```

---

### Creating an Ingress Resource

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: myapp.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-app-service
            port:
              number: 80
```

```bash
kubectl apply -f ingress.yml
kubectl get ingress
```

---

### Accessing Services

* Check NodePort:

```bash
kubectl get svc -n ingress-nginx
```

* Port-forward (guaranteed to work locally):

```bash
kubectl port-forward svc/ingress-nginx-controller -n ingress-nginx 8080:80
curl http://localhost:8080
```

---

✅ Ingress Advantages Summary

* Centralized traffic management
* Path/host-based routing
* SSL/TLS termination
* Load balancing
* Cost efficiency
* Security and flexibility

---

*Images to include:*

1. `without-ingress.png` → Service exposure without Ingress
2. `with-ingress-controller.png` → High-level Ingress Controller flow
3. Optional: diagram of Nginx Ingress Pod routing traffic from external to Pods

---
