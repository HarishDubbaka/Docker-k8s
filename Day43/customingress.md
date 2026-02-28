# 🖥️ Creating Your Own Web App with Docker & Kubernetes (NGINX + Ingress)

This guide walks you through building a simple Docker web app, pushing it to Docker Hub, and deploying it on Kubernetes with an NGINX Ingress Controller.

---

## 1️⃣ Docker Web App

### Project Structure

```
my-webapp/
 ├── index.html        # Your custom HTML file
 └── Dockerfile        # Dockerfile to build the image
```

### Dockerfile

```dockerfile
# Use official Nginx image
FROM nginx:alpine

# Copy your index.html into the default Nginx web directory
COPY index.html /usr/share/nginx/html/

# Expose port 80
EXPOSE 80
```

### Build Docker Image

```bash
docker build -t my-laxmi-webapp .
```

### Run the Container

```bash
docker run -p 8080:80 my-laxmi-webapp
```

Open your browser at [http://localhost:8080](http://localhost:8080) to see your web app.

> Note: Exiting the container stops the web app.

---

### Push Image to Docker Hub

#### 1️⃣ Login to Docker Hub

```bash
docker login
```

Enter your Docker Hub username and password.

#### 2️⃣ Tag Your Image

Assume your Docker Hub username is `970371`:

```bash
docker tag my-laxmi-webapp 970371/my-laxmiwebapp:latest
```

#### 3️⃣ Push Image

```bash
docker push 970371/my-laxmiwebapp:latest
```

#### 4️⃣ Pull & Run Anywhere

```bash
docker pull 970371/my-laxmiwebapp:latest
docker run -p 8080:80 970371/my-laxmiwebapp:latest
```

---

## 2️⃣ Kubernetes Deployment

### Deployment YAML

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: laxmi-webapp
  labels:
    app: laxmi-webapp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: laxmi-webapp
  template:
    metadata:
      labels:
        app: laxmi-webapp
    spec:
      containers:
      - name: laxmi-webapp
        image: 970371/my-laxmiwebapp:latest
        ports:
        - containerPort: 80
```

```bash
kubectl apply -f deployment.yml
kubectl get deployment
kubectl get pods
```

---

### Service YAML

```yaml
apiVersion: v1
kind: Service
metadata:
  name: laxmi-webapp-service
spec:
  selector:
    app: laxmi-webapp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: NodePort
```

```bash
kubectl apply -f service.yml
kubectl get svc
```

> ⚠️ Make sure the `selector` matches the `app` label in your Deployment.

---

## 3️⃣ NGINX Ingress Controller (Kind / Local)

### Install Ingress Controller

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
```

Check pods:

```bash
kubectl get pods -n ingress-nginx
```

---

### Create Ingress Resource

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: laxmi-webapp-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: laxmiwebapp.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: laxmi-webapp-service
            port:
              number: 80
```

```bash
kubectl apply -f ingress.yml
kubectl get ingress
```

---

### NodePort Access

Check current Ingress Controller service:

```bash
kubectl get svc -n ingress-nginx
```

Change Service to NodePort if needed:

```bash
kubectl edit svc ingress-nginx-controller -n ingress-nginx
```

Update:

```yaml
spec:
  type: NodePort
  ports:
    - name: http
      port: 80
      targetPort: 80
      nodePort: 30080
    - name: https
      port: 443
      targetPort: 443
      nodePort: 30443
```

Verify:

```bash
kubectl get svc -n ingress-nginx
```

Access via browser:

```
http://<NodeIP>:30080
https://<NodeIP>:30443
```

> In Kind, the NodeIP is usually `localhost`.

---

### Port-Forward (Simplest, Guaranteed)

```bash
kubectl port-forward svc/ingress-nginx-controller -n ingress-nginx 8080:80
curl http://localhost:8080
```

✅ Works on WSL2, Docker Desktop, or any cluster type.

---

### Quick Checklist

1. `kubectl get nodes -o wide` → check Node IP.
2. Update hosts file if using custom hostnames:

```
127.0.0.1 laxmiwebapp.local
```

3. Use NodePort or port-forward to access your app locally.

---

This README gives you a full **end-to-end workflow**: build, push, deploy, expose via Service/Ingress, and test your web app locally.

---

