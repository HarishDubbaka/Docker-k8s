# Day 45 – Migrate Ingress to Gateway API | Kubernetes

Most Kubernetes workflows still use the standard **Ingress** resource for traffic routing. However, Ingress has several limitations in advanced routing scenarios.

Additionally, NGINX Ingress Controller (Ingress-NGINX) is planned to be fully retired in **March 2026** and will no longer receive bug fixes or security updates. Because of this, many teams are migrating to a modern and supported solution — the **Gateway API**.

The **Gateway API** provides advanced traffic routing features and better separation of concerns compared to Ingress.

---

## Kubernetes Ingress

Before jumping into migration, let's briefly understand Ingress.

Ingress in a Kubernetes cluster helps route traffic to applications based on:

* Domain name
* Path

### Limitations of Ingress

* Difficult to configure header-based routing
* No clean method-based routing
* No separation between TCP/UDP and HTTP/HTTPS routing
* Limited flexibility for advanced traffic policies

---

## Kubernetes Gateway API

The **Gateway API** is the modern replacement for Ingress with more advanced and extensible traffic routing capabilities.

### Core Components Comparison

| Ingress Concept | Gateway API Equivalent |
| --------------- | ---------------------- |
| IngressClass    | GatewayClass           |
| Ingress         | Gateway                |
| Ingress Rules   | HTTPRoute              |

### Key Components

* **GatewayClass** – Defines the controller (like IngressClass)
* **Gateway** – Entry point for traffic
* **HTTPRoute** – Defines HTTP routing rules

---

## Prerequisites

* Helm v3.17.0 or higher
* Kubernetes v1.30 or higher
* Kubectl v1.30 or higher
* Configured kubeconfig to access the cluster

---

# Step 1: Deploy Sample Web Application

### Create Namespace

```bash
kubectl create namespace web-app
```

### Deployment and Service

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-service
  namespace: web-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web-service
  template:
    metadata:
      labels:
        app: web-service
    spec:
      containers:
      - name: web
        image: nginx:latest
        ports:
        - containerPort: 80
        volumeMounts:
        - name: web-config
          mountPath: /usr/share/nginx/html
      volumes:
      - name: web-config
        configMap:
          name: web-content
---
apiVersion: v1
kind: Service
metadata:
  name: web-service
  namespace: web-app
spec:
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: web-service
```

```bash
kubectl apply -f webdeployment-service.yml
```

### Create ConfigMap with HTML Content

```bash

apiVersion: v1
kind: ConfigMap
metadata:
  name: web-content
  namespace: web-app
data:
  index.html: |
    <!DOCTYPE html>
    <html lang="en">
    <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>Harish | Introduction</title>
      <style>
        body {
          font-family: Arial, sans-serif;
          margin: 0; padding: 0;
          background: #fdfdfd; color: #111;
        }
        .hero {
          background: #000; color: #fff;
          text-align: center; padding: 5rem 2rem;
        }
        .hero img {
          width: 150px; height: 150px;
          border-radius: 50%; border: 4px solid #fff;
          margin-bottom: 1rem;
        }
        .hero h1 { font-size: 2.5rem; margin-bottom: 0.5rem; }
        .hero p { font-size: 1.2rem; opacity: 0.85; }

        nav {
          background: #111; display: flex;
          justify-content: center; padding: 1rem;
        }
        nav a {
          color: #fff; margin: 0 15px;
          text-decoration: none; font-weight: bold;
        }
        nav a:hover { color: #aaa; }

        main { max-width: 900px; margin: auto; padding: 2rem; }
        h2 { border-bottom: 2px solid #000; margin-bottom: 1rem; }
        .card {
          background: #fff; border-radius: 6px;
          box-shadow: 0 4px 8px rgba(0,0,0,0.1);
          padding: 1.5rem; margin-bottom: 1.5rem;
        }
        footer {
          background: #000; color: #fff;
          text-align: center; padding: 1rem;
        }
      </style>
    </head>
    <body>

      <div class="hero">
        <img src="https://via.placeholder.com/150" alt="Profile Photo">
        <h1>👋 Hi, I'm Harish</h1>
        <p>SAP BASIS Administrator | Aspiring DevOps Engineer</p>
      </div>

      <nav>
        <a href="#about">About</a>
        <a href="#projects">Projects</a>
        <a href="#contact">Contact</a>
      </nav>

      <main>
        <section id="about">
          <h2>About Me</h2>
          <div class="card">
            <p>I specialize in SAP BASIS administration, S/4HANA migrations, and enterprise cloud operations.
            Currently focused on DevOps automation with Docker, Kubernetes, Terraform, and Azure DevOps.</p>
          </div>
        </section>

        <section id="projects">
          <h2>Projects</h2>
          <div class="card">
            <h3>🚀 Kubernetes Ingress Setup</h3>
            <p>Documented reproducible ingress setup with Nginx for kind clusters.</p>
          </div>
          <div class="card">
            <h3>⚡ Terraform Challenge</h3>
            <p>Completed a 30-day Terraform challenge, publishing guides and reusable modules on GitHub.</p>
          </div>
        </section>

        <section id="contact">
          <h2>Contact</h2>
          <div class="card">
            <p>📧 Email: harishdubbaka450@gmail.com</p>
            <p>🌐 GitHub: github.com/HarishDubbaka</p>
            <p>💼 LinkedIn: linkedin.com/in/harish-dubbaka-2841481a4/</p>
          </div>
        </section>
      </main>

      <footer>
        &copy; 2026 Harish. All rights reserved.
      </footer>

    </body>
    </html>
	
```bash
kubectl apply -f config+html.yml
```

---

# Step 2: Create TLS Secret

### Generate Self-Signed Certificate

```bash
MSYS_NO_PATHCONV=1 openssl req -x509 -nodes -days 365 \
  -newkey rsa:2048 \
  -keyout tls.key \
  -out tls.crt \
  -subj "/CN=gateway.web.k8s.local"
```

### Create TLS Secret

```bash
kubectl create secret tls web-tls-secret \
  --cert=tls.crt \
  --key=tls.key \
  --namespace=web-app
```

Verify:

```bash
kubectl get secret web-tls-secret -n web-app
```

---

# Step 3: Install Ingress Controller

Install Helm (if required), then install Ingress-NGINX:

```bash
helm install ingress-nginx \
    --set controller.service.type=NodePort \
    --set controller.service.nodePorts.http=30082 \
    --set controller.service.nodePorts.https=30443 \
    --repo https://kubernetes.github.io/ingress-nginx \
    ingress-nginx
```

---

## Create Ingress Resource

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web
  namespace: web-app
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - gateway.web.k8s.local
    secretName: web-tls-secret
  rules:
  - host: gateway.web.k8s.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
```

```bash
kubectl apply -f ingress.yml
```

Verify:

```bash
kubectl get ingress -n web-app
```

---

# Step 4: Test Ingress

Add to `/etc/hosts`:

```
<NODE_IP> gateway.web.k8s.local
```

Test:

```bash
curl -k http://gateway.web.k8s.local/
curl -k https://gateway.web.k8s.local/
```

Ingress should work correctly before migration.

---

# Step 5: Inspect Existing Ingress

```bash
kubectl get ingress web -n web-app -o yaml
```

---

# Step 6: Install Gateway API CRDs

```bash
kubectl kustomize "https://github.com/nginx/nginx-gateway-fabric/config/crd/gateway-api/standard?ref=v1.5.1" | kubectl apply -f -
```

Verify:

```bash
kubectl get crd | grep gateway
```

---

# Step 7: Install NGINX Gateway Fabric

We will use NGINX Gateway Fabric as the Gateway controller.

### Deploy CRDs

```bash
kubectl apply -f https://raw.githubusercontent.com/nginx/nginx-gateway-fabric/v1.6.1/deploy/crds.yaml
```

### Deploy Gateway Controller

```bash
kubectl apply -f https://raw.githubusercontent.com/nginx/nginx-gateway-fabric/v1.6.1/deploy/nodeport/deploy.yaml
```

Verify:

```bash
kubectl get pods -n nginx-gateway
```

---

### Patch NodePorts

```bash
kubectl patch svc nginx-gateway -n nginx-gateway --type='json' -p='[
  {"op": "replace", "path": "/spec/ports/0/nodePort", "value": 30080},
  {"op": "replace", "path": "/spec/ports/1/nodePort", "value": 30081}
]'
```

Verify:

```bash
kubectl get svc -n nginx-gateway nginx-gateway
```

---

# Step 8: Create GatewayClass

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
  name: nginx
spec:
  controllerName: gateway.nginx.org/nginx-gateway-controller
```

Verify:

```bash
kubectl get gc
```

---

# Step 9: Create Gateway Resource

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: nginx-gateway
  namespace: nginx-gateway
spec:
  gatewayClassName: nginx
  listeners:
  - name: https
    port: 443
    protocol: HTTPS
    hostname: gateway.web.k8s.local
    tls:
      mode: Terminate
      certificateRefs:
      - kind: Secret
        name: web-tls-secret
        namespace: web-app
    allowedRoutes:
      namespaces:
        from: All
  - name: http
    port: 80
    protocol: HTTP
    hostname: gateway.web.k8s.local
    allowedRoutes:
      namespaces:
        from: All
```

Verify:

```bash
kubectl get gateway -n nginx-gateway
```

---

# Step 10: Create HTTPRoute

### HTTP Route

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: web-route
  namespace: web-app
spec:
  parentRefs:
  - name: nginx-gateway
    namespace: nginx-gateway
    sectionName: http
  hostnames:
  - gateway.web.k8s.local
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /
    backendRefs:
    - name: web-service
      port: 80
```

### HTTPS Route

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: web-route-https
  namespace: web-app
spec:
  parentRefs:
  - name: nginx-gateway
    namespace: nginx-gateway
    sectionName: https
  hostnames:
  - gateway.web.k8s.local
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /
    backendRefs:
    - name: web-service
      port: 80
```

Verify:

```bash
kubectl get httproute -n web-app
```

---

# Step 11: Verify Gateway Configuration

```bash
kubectl describe gateway nginx-gateway -n nginx-gateway
kubectl describe httproute web-route -n web-app
```

Check Accepted condition:

```bash
kubectl get httproute web-route -n web-app -o jsonpath='{.status.parents[0].conditions[?(@.type=="Accepted")].status}'
```

If you face TLS secret namespace issues, create a **ReferenceGrant** in the `web-app` namespace.

---

# Step 12: Test Gateway API

Add to `/etc/hosts`:

```
<NODE_IP> gateway.web.k8s.local
```

Test HTTP:

```bash
curl -v -H "Host: gateway.web.k8s.local" http://<NODE_IP>:30080/
```

Test HTTPS:

```bash
curl -v -k https://gateway.web.k8s.local:30081/
```

---

# Step 13: Remove Old Ingress

After verifying Gateway works:

```bash
kubectl delete ingress web -n web-app
```

---

# Key Difference

With Ingress, we only configured HTTP routing.

With Gateway API, we explicitly defined:

* HTTP listener
* HTTPS listener
* TLS termination
* Separate HTTPRoute resources

This provides better flexibility, scalability, and separation of concerns.

---

## Lab Environment Used

Instead of a Kind cluster (which requires port-forwarding), a multi-node playground lab was used for easier NodePort testing.

---

# Conclusion

Migrating from Ingress to Gateway API:

* Provides advanced routing capabilities
* Enables better separation of traffic policies
* Offers future-proof architecture
* Aligns with modern Kubernetes networking standards

Gateway API is the future of Kubernetes traffic management.
