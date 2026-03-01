# 🖥️ Creating Your Own Web App with Docker & Kubernetes (NGINX + Ingress)

# Customized OwnWeb App with Nginx Ingress Controller

---

## 🛠️ Step 1: Project Structure

Organize your files like this:

```
my-webapp/
 ├── index.html   # your customized HTML
 └── Dockerfile
```

---

## 🛠️ Step 2: Write a Dockerfile

Use **Nginx** to serve your HTML file.

```dockerfile
# Use official Nginx image
FROM nginx:alpine

# Copy your index.html into the default Nginx web directory
COPY index.html /usr/share/nginx/html/

# Expose port 80
EXPOSE 80
```

---

## 🛠️ Step 3: Build the Docker Image

Run in your project folder:

```bash
docker build -t my-laxmi-webapp .
```

---

## 🛠️ Step 4: Run the Container

```bash
docker run -p 8080:80 my-laxmi-webapp
```

Open your browser at **[http://localhost:8080](http://localhost:8080)** → you’ll see your `index.html` served by Nginx 🎉

> ⚠️ Note: Once you exit the container, the web app will stop.

---

## 🛠️ Step 5: Push Docker Image to Docker Hub

### Step 5.1: Login

```bash
docker login
```

Enter your Docker Hub username and password.

### Step 5.2: Tag Your Image

```bash
docker tag my-laxmi-webapp 970371/my-laxmiwebapp:latest
```

### Step 5.3: Push to Docker Hub

```bash
docker push 970371/my-laxmiwebapp:latest
```

### Step 5.4: Pull & Run Anywhere

```bash
docker pull 970371/my-laxmiwebapp:latest
docker run -p 8080:80 my-laxmiwebapp:latest
```

Open **[http://localhost:8080](http://localhost:8080)** → your webapp is live 🎉

✅ **Summary:**

1. `docker login`
2. `docker tag <local-image> <username>/<repo>:tag`
3. `docker push <username>/<repo>:tag`
4. `docker pull` and `docker run` anywhere

---

## 🛠️ Step 6: Kubernetes Deployment

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

Apply deployment:

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

Apply service:

```bash
kubectl apply -f service.yml
kubectl get svc
```

> ⚠️ Make sure the `selector` matches the labels in your Deployment.

---

### Accessing in Kind Cluster

Kind doesn’t expose NodePorts by default. You can:

1. Use **port-forward**:

```bash
kubectl port-forward svc/laxmi-webapp-service 8080:80
curl http://localhost:8080
```

2. Or configure **extraPortMappings** in Kind configuration.

---

## 🛠️ Step 7: Create Ingress

### Ingress YAML

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

Apply ingress:

```bash
kubectl apply -f ingress.yml
kubectl get ingress
kubectl describe ingress laxmi-webapp-ingress
```

> ✅ Key: Your backend pods should be healthy and address shows `localhost` in Kind.

---

### Access via Ingress

#### Option 1: Modify Hosts File

Add to `C:\Windows\System32\drivers\etc\hosts`:

```
127.0.0.1 laxmiwebapp.local
```

Then open:

```
http://laxmiwebapp.local:8080
```

#### Option 2: Use curl without hosts file

```bash
curl -H "Host: laxmiwebapp.local" http://localhost:8080
```

---

## 🛠️ Step 8: Install Nginx Ingress Controller for Kind

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
kubectl get pods -n ingress-nginx
kubectl get svc -n ingress-nginx
```

> ⚠️ The service may show `LoadBalancer` type with `EXTERNAL-IP: <pending>` in Kind.
> Change to `NodePort` if needed:

```bash
kubectl edit svc ingress-nginx-controller -n ingress-nginx
```

Set:

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

---

### Verify Access

```bash
kubectl port-forward -n ingress-nginx svc/ingress-nginx-controller 8080:80
curl -H "Host: laxmiwebapp.local" http://localhost:8080
```

✅ If you see your `index.html` → everything works perfectly.

---

