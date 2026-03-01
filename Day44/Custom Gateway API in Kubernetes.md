# Day44 – Setting Up Gateway API in Kubernetes (NGINX Gateway Fabric)

This guide walks through the complete setup of the **Kubernetes Gateway API** using **NGINX Gateway Fabric**.

The setup consists of three major sections:

1. Gateway API CRD Installation  
2. Gateway API Controller Installation  
3. Gateway API Object Creation & Traffic Validation  

---

# 📌 Step 1: Install Gateway API CRDs

Gateway API objects are not native Kubernetes resources.  
We must install the Custom Resource Definitions (CRDs).

## Install Gateway API CRDs

```bash
kubectl apply -f https://raw.githubusercontent.com/nginxinc/nginx-gateway-fabric/v1.6.1/deploy/crds.yaml
````

## Verify Installation

```bash
kubectl get crd | grep gateway
```

You should see resources like:

* gatewayclasses.gateway.networking.k8s.io
* gateways.gateway.networking.k8s.io
* httproutes.gateway.networking.k8s.io
* grpcroutes.gateway.networking.k8s.io
* referencegrants.gateway.networking.k8s.io
* nginxgateway CRDs

## Check API Resources

```bash
kubectl api-resources --api-group=gateway.networking.k8s.io
```

Expected output:

```
NAME              APIVERSION                          KIND
gatewayclasses    gateway.networking.k8s.io/v1        GatewayClass
gateways          gateway.networking.k8s.io/v1        Gateway
grpcroutes        gateway.networking.k8s.io/v1        GRPCRoute
httproutes        gateway.networking.k8s.io/v1        HTTPRoute
referencegrants   gateway.networking.k8s.io/v1beta1   ReferenceGrant
```

---

# 📌 Step 2: Install Gateway API Controller

We will use **NGINX Gateway Fabric** as the Gateway API controller.

## Install CRDs (if not already installed)

```bash
kubectl apply -f https://raw.githubusercontent.com/nginx/nginx-gateway-fabric/v1.6.1/deploy/crds.yaml
```

## Deploy NGINX Gateway Fabric (NodePort mode)

```bash
kubectl apply -f https://raw.githubusercontent.com/nginx/nginx-gateway-fabric/v1.6.1/deploy/nodeport/deploy.yaml
```

## Verify Deployment

```bash
kubectl get pods -n nginx-gateway
```

Expected:

```
nginx-gateway-xxxxx   2/2   Running
```

Check all resources:

```bash
kubectl -n nginx-gateway get all
```

---

## Expose Specific NodePorts (Optional)

By default, random NodePorts are assigned.

Patch service to fixed ports:

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

Expected:

```
80:30080/TCP
443:30081/TCP
```

---

# 📌 Step 3: Create GatewayClass and Gateway

Create a file named `gateway-resources.yaml`

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
  name: nginx
spec:
  controllerName: gateway.nginx.org/nginx-gateway-controller
---
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: nginx-gateway
  namespace: nginx-gateway
spec:
  gatewayClassName: nginx
  listeners:
  - name: http
    port: 80
    protocol: HTTP
    allowedRoutes:
      namespaces:
        from: All
```

Apply:

```bash
kubectl apply -f gateway-resources.yaml
```

Verify:

```bash
kubectl get gatewayclass
kubectl get gateway -n nginx-gateway
```

Ensure:

```
ACCEPTED: True
PROGRAMMED: True
```

---

# 🚨 Gateway Troubleshooting Guide

If `PROGRAMMED` is empty:

## 1️⃣ Check CNI

```bash
kubectl get pods -n kube-system
```

If Calico or CNI pods are failing → fix networking first.

## 2️⃣ Check Controller Pod

```bash
kubectl get pods -n nginx-gateway
kubectl logs <pod-name> -n nginx-gateway
```

## 3️⃣ Check CRDs

```bash
kubectl get crd | grep gateway.nginx.org
```

## 4️⃣ Check GatewayClass

```bash
kubectl describe gatewayclass nginx
```

Must show:

```
Accepted: True
```

---

# 📌 Step 4: Deploy Frontend Application

Create `frontend-pod.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend-app
  labels:
    app: frontend-app
spec:
  containers:
  - name: frontend-container
    image: 970371/my-laxmiwebapp:latest
    ports:
    - containerPort: 80
```

Apply:

```bash
kubectl apply -f frontend-pod.yaml
```

---

# 📌 Step 5: Create Service

Create `frontend-service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  selector:
    app: frontend-app
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
```

Apply:

```bash
kubectl apply -f frontend-service.yaml
```

Verify:

```bash
kubectl get pods
kubectl get svc
```

---

# 📌 Step 6: Create HTTPRoute

Create `frontend-route.yaml`

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: frontend-route
  namespace: default
spec:
  parentRefs:
  - name: nginx-gateway
    namespace: nginx-gateway
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /
    backendRefs:
    - name: frontend-service
      port: 80
```

Apply:

```bash
kubectl apply -f frontend-route.yaml
```

Verify:

```bash
kubectl get httproute
kubectl describe httproute frontend-route
```

Ensure it shows:

```
Accepted: True
```

---

# 📌 Step 7: Test in kind Cluster

Since kind runs inside Docker:

## Option 1 – Access via NodePort

Your gateway service:

```
80:30080
```

Access:

```
http://localhost:30080
```

---

## Option 2 – Port Forward (Recommended for Testing)

```bash
kubectl port-forward svc/frontend-service 8080:80
```

Open:

```
http://localhost:8080
```

---

# ✅ Final Architecture Flow

User → NodePort (30080) → NGINX Gateway → HTTPRoute → Service → Pod

---

# 🎯 Summary

You successfully:

* Installed Gateway API CRDs
* Installed NGINX Gateway Fabric
* Created GatewayClass & Gateway
* Deployed frontend app
* Created Service
* Created HTTPRoute
* Validated traffic routing

Gateway API provides modern, flexible, multi-protocol traffic management in Kubernetes.

---

# 🚀 Day44 Complete

