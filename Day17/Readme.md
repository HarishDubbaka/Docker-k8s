# 🐳 Kubernetes Controllers Guide – RC, RS & Deployment

This guide covers the three main Kubernetes controllers used to manage Pods:

- **ReplicationController (RC)** – legacy controller for maintaining Pod replicas  
- **ReplicaSet (RS)** – modern replacement for RC, often managed by Deployments  
- **Deployment** – high-level controller for scaling, rolling updates, and rollback  

Learn how Kubernetes achieves **self-healing**, **scaling**, and **high availability** using these controllers.

---

# Kubernetes ReplicationController (RC)  

In Kubernetes, **ReplicationController (RC)** and **ReplicaSet** are essential components used for managing and maintaining the desired number of pod replicas to ensure **high availability** and **fault tolerance** of applications. They enable **self-healing**, **load distribution**, and **basic scaling** within a Kubernetes cluster.

This document focuses on **ReplicationController**, explaining its concepts and demonstrating a complete hands-on workflow with practical examples.

---

## What Manages the Number of Pods in Kubernetes?

Have you ever wondered what is responsible for supervising and maintaining the exact number of Pods running inside a Kubernetes cluster?

Kubernetes can do this in multiple ways, and one traditional approach is using a **ReplicationController (RC)**.

---

## What is a ReplicationController?

A **ReplicationController** is responsible for managing the Pod lifecycle and ensuring that a specified number of Pods are running at any given time.

### Key Responsibilities
- Ensures the **desired number of Pods** are always running
- Recreates Pods automatically if they are deleted or fail
- Maintains availability during node failure or Pod termination

### Limitations
ReplicationController **does not handle advanced features** such as:
- Auto-scaling based on metrics
- Readiness and liveness probes
- Rolling updates

These capabilities are better handled by newer Kubernetes resources like **Deployments** and **ReplicaSets**.

---

## Understanding the Term “ReplicationController”

- **Replication** – Scales one Pod into many
- **Controller** – Controls the desired number of Pods versus the actual number of Pods

In simple terms, a ReplicationController ensures that a **homogeneous set of Pods is always running and available**.

---

## Use Cases of ReplicationController

ReplicationControllers can be used for both **stateful** and **stateless** applications:

### Examples
- Production teams ensuring critical Pods are always available
- Development teams scaling Pods up or down
- Running distributed database clusters
- Hosting scalable web server farms

---

## ReplicationController Example (Step-by-Step)

### ReplicationController YAML Definition

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    app: nginx
  template:
    metadata:
      name: nginx
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx
          ports:
            - containerPort: 80
````

---

## Step 1: Create the ReplicationController

Apply the ReplicationController using the official Kubernetes example:

```bash
kubectl apply -f nginx.yaml
```

### Output

```text
replicationcontroller/nginx created
```

---

## Step 2: Verify ReplicationController Status

```bash
kubectl describe rc/nginx
```

### Sample Output

```text
Name:         nginx
Namespace:    default
Selector:     app=nginx
Replicas:     3 current / 3 desired
Pods Status:  3 Running / 0 Waiting / 0 Succeeded / 0 Failed
```

This confirms that **3 Pods are running as desired**.

---

## Step 3: List Pods Managed by the ReplicationController

```bash
pods=$(kubectl get pods --selector=app=nginx --output=jsonpath={.items..metadata.name})
echo $pods
```

### Example Output

```text
nginx-g7vhn nginx-hzjkt nginx-jpclq
```

---

## Step 4: Test Self-Healing (Delete a Pod)

Check Pod placement:

```bash
kubectl get pods -o wide
```

Delete one Pod manually:

```bash
kubectl delete pod nginx-g7vhn
```

### Output

```text
pod "nginx-g7vhn" deleted
```

---

## Step 5: Verify Pod Recreation

List the Pods again:

```bash
pods=$(kubectl get pods --selector=app=nginx --output=jsonpath={.items..metadata.name})
echo $pods
```

### Output

```text
nginx-45qcx nginx-hzjkt nginx-jpclq
```

Check detailed status:

```bash
kubectl get pods -o wide
```

A **new Pod (`nginx-45qcx`) is automatically created**, proving that the ReplicationController maintains the desired replica count.

---

## High Availability Demonstration

This behavior confirms:

* Pods are recreated automatically
* High availability is maintained
* Applications continue running even after Pod failure

---

## Step 6: Delete the ReplicationController

Check existing resources:

```bash
kubectl get all
```

Delete the ReplicationController:

```bash
kubectl delete replicationcontroller/nginx
```

### Output

```text
replicationcontroller "nginx" deleted
```

Verify Pods:

```bash
kubectl get pods -o wide
```

### Result

All Pods managed by the ReplicationController are **deleted automatically**.

---

## Key Takeaways

* ReplicationController ensures **desired Pod count**
* Provides **self-healing** and **fault tolerance**
* Automatically recreates failed or deleted Pods
* Deleting the RC deletes all associated Pods
* Consider **Deployments / ReplicaSets** for modern workloads

---

## Conclusion

ReplicationController is a foundational Kubernetes concept that demonstrates how Kubernetes achieves **high availability** and **resilience**. While it has largely been replaced by **Deployments**, understanding RCs is essential for mastering Kubernetes internals and legacy workloads.

---

# Stateful vs Stateless Applications in Kubernetes

Understanding whether an application is **stateful** or **stateless** is critical when deploying workloads on Kubernetes. This distinction determines how Pods are managed, scaled, and connected to storage.

---

## 📦 Stateless Applications

- **Definition:** Each request is independent; the application does not retain memory of past interactions.
- **Deployment:** Typically managed using **Deployments** or **ReplicaSets**.
- **Scaling:** Easy to scale horizontally — Pods are interchangeable and can be added/removed without issues.
- **Storage:** No persistent storage required; data is ephemeral and lost when a Pod restarts.
- **Networking:** Pods share a common Service endpoint.
- **Examples:** REST APIs, web frontends, microservices.

---

## 🌀 Stateful Applications

- **Definition:** Applications that need to maintain state across sessions or requests.
- **Deployment:** Managed using **StatefulSets**.
- **Scaling:** More complex — Pods have stable identities and ordered startup/shutdown.
- **Storage:** Uses **PersistentVolume (PV)** and **PersistentVolumeClaim (PVC)** to ensure data survives Pod restarts.
- **Networking:** Each Pod gets a stable DNS name (e.g., `pod-0`, `pod-1`).
- **Examples:** Databases (MySQL, PostgreSQL), message queues (Kafka, RabbitMQ).

---

## 🔑 Key Differences in Kubernetes

| Feature        | Stateless (Deployment) | Stateful (StatefulSet) |
|----------------|-------------------------|-------------------------|
| **Pod Identity** | Random, interchangeable | Stable, ordered (pod-0, pod-1, …) |
| **Storage**     | Ephemeral               | Persistent via PV/PVC |
| **Scaling**     | Easy, elastic           | Complex, requires careful planning |
| **Networking**  | Shared Service          | Stable DNS per Pod |
| **Use Case**    | Web servers, APIs       | Databases, distributed systems |

---

## ⚖️ Why It Matters

- **Stateless apps** are cloud-native friendly: resilient, easy to scale, and simple to manage.
- **Stateful apps** require Kubernetes features like **StatefulSets, persistent volumes, and headless services** to ensure reliability and data consistency.

---


# Why ReplicationController (RC) Is Replaced by ReplicaSet (RS)

In Kubernetes, **ReplicationController (RC)** was one of the earliest resources used to ensure that a specified number of Pods are running at all times.  
As Kubernetes evolved, **ReplicaSet (RS)** was introduced as a more powerful and flexible replacement.

Today, **ReplicaSets are used by Deployments**, and ReplicationControllers are considered legacy.

---

## What ReplicationController (RC) Does
ReplicationController:
- Ensures a fixed number of Pods are running
- Recreates Pods if they fail or are deleted
- Uses **simple label selectors**

Example:
```

app = myapp

```

---

## Why ReplicationController Is Limited
ReplicationController has these limitations:
- Supports only **equality-based label selectors**
- Cannot manage multiple versions of Pods efficiently
- Not suitable for rolling updates and rollbacks
- No longer receives new features

---

## What ReplicaSet (RS) Improves
ReplicaSet does everything RC does **plus more**.

Key improvements:
- Supports **set-based label selectors** (`in`, `notin`, etc.)
- Can manage multiple Pod versions
- Designed to work with **Deployments**
- Enables rolling updates and rollbacks

Example selectors:
```

app in (v1, v2)
tier notin (test)

```

---

## Why Deployments Use ReplicaSets
Deployments provide:
- Rolling updates
- Rollbacks
- Version history

To achieve this, Kubernetes needs:
- Multiple ReplicaSets running simultaneously
- Smooth scaling of old and new Pods

ReplicaSet makes this possible.  
ReplicationController cannot.

Deployment hierarchy:
```

Deployment
└── ReplicaSet
└── Pods

```

---

## Current Best Practice
- ❌ Do NOT use ReplicationController
- ⚠️ Rarely create ReplicaSet directly
- ✅ Use Deployment to manage Pods

ReplicaSets are usually created and managed **automatically by Deployments**.

---

# Kubernetes ReplicaSet (RS)

In Kubernetes, a **ReplicaSet (RS)** is a core component designed to ensure the **availability**, **consistency**, and **resilience** of Pods in a cluster. It maintains the desired number of pod replicas and automatically replaces Pods if they fail or are deleted.

---

## 1. What is a ReplicaSet?

A **ReplicaSet** is a Kubernetes controller that ensures a specified number of identical Pods are running at all times.

If a Pod goes down, crashes, or is deleted, the ReplicaSet **automatically creates a new Pod** to maintain the desired state. This self-healing capability provides **high availability** and **redundancy** for applications.

### Key Concepts
- **Desired replicas**: Number of Pods ReplicaSet tries to maintain
- **Labels**: Used to identify and manage Pods
- **Self-healing**: Automatically replaces failed or deleted Pods

---

## 2. Purpose of a ReplicaSet

ReplicaSets serve the following purposes:

- **Ensuring Application Availability**  
  Keeps the specified number of Pods running at all times

- **Automatic Scaling**  
  Scale up or down by changing the replicas count

- **Resilience & Fault Tolerance**  
  Automatically recovers from Pod failures without manual intervention

---

## 3. Components of a ReplicaSet

A ReplicaSet consists of:

### Selector
Defines a label query to identify which Pods the ReplicaSet manages.

### Replicas
Specifies the desired number of Pod replicas.

### Template
A Pod template used to create new Pods, including:
- Container image
- Ports
- Resource settings

---

## 4. How ReplicaSet Works

When a ReplicaSet is applied:

1. Kubernetes creates the specified number of Pods
2. Continuously monitors the Pods
3. Recreates Pods if any fail or are deleted
4. Removes extra Pods if the count exceeds desired replicas

---

## 5. Creating a ReplicaSet – Step-by-Step Example

### ReplicaSet YAML Configuration

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-replicaset
spec:
  replicas: 5
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
````

### YAML Explanation

* **apiVersion**: apps/v1 (ReplicaSet API)
* **kind**: ReplicaSet
* **metadata.name**: Name of the ReplicaSet
* **replicas**: Desired Pod count (5)
* **selector**: Matches Pods with label `app=nginx`
* **template**: Defines Pod configuration

---

## 6. Applying the ReplicaSet

```bash
kubectl apply -f rs.yaml
```

### Output

```text
replicaset.apps/nginx-replicaset created
```

---

## 7. Verifying the ReplicaSet

### Check ReplicaSet Status

```bash
kubectl get rs
```

```text
NAME               DESIRED   CURRENT   READY   AGE
nginx-replicaset   5         5         5       63s
```

### Check Pods

```bash
kubectl get pods -l app=nginx
```

```text
nginx-replicaset-7hg8h
nginx-replicaset-8z9f6
nginx-replicaset-9fsw2
nginx-replicaset-c2v29
nginx-replicaset-ls6cg
```

---

## 8. Pod Distribution Across Nodes

```bash
kubectl get pods -o wide
```

Pods may be distributed unevenly across nodes. ReplicaSet **does not guarantee even placement**.

💡 To improve distribution, use:

* Pod Anti-Affinity
* PodTopologySpreadConstraints

---

## 9. Self-Healing in Action (Delete a Pod)

```bash
kubectl delete pod nginx-replicaset-7hg8h
```

Check Pods again:

```bash
kubectl get pods
```

A **new Pod is automatically created**, maintaining the desired replicas.

> “Systems that heal themselves never break for long.”

---

## 10. Scaling a ReplicaSet

### Current State

* Replicas = 5
* Pods running = 5

### Desired State

* Replicas = 10

### What Happens

* Kubernetes detects mismatch
* ReplicaSet creates 5 new Pods
* Desired state is achieved

### Scale Using kubectl

```bash
kubectl scale replicaset nginx-replicaset --replicas=10
```

```text
replicaset.apps/nginx-replicaset scaled
```

### Verify Scaling

```bash
kubectl get rs
```

```text
NAME               DESIRED   CURRENT   READY   AGE
nginx-replicaset   10        10        10      45m
```

---

## 11. Export Scaled ReplicaSet YAML

```bash
kubectl get rs nginx-replicaset -o yaml > rs-scaled.yaml
```

* `-o yaml` → Outputs full YAML
* `>` → Saves to file

---

## 12. Rollback Limitations in ReplicaSet

ReplicaSets **do NOT maintain history**.

* No automatic rollback
* Must manually reset replicas

```bash
kubectl scale rs nginx-replicaset --replicas=5
```

ReplicaSets are self-healing, **not version-aware**.

---

## 13. Deleting a ReplicaSet

### Delete ReplicaSet and Pods

```bash
kubectl delete replicaset nginx-replicaset
```

### Delete ReplicaSet but Keep Pods (Orphan Pods)

```bash
kubectl delete rs nginx-replicaset --cascade=orphan
```

* ReplicaSet is removed
* Pods continue running
* Pods are no longer managed

> Orphaned Pods are “free agents”

---

# Why Choose Deployment Over ReplicaSet?

Both **ReplicaSet** and **Deployment** manage Pods, but Deployments are more powerful.

---

## ReplicaSet

* Maintains Pod count ✅
* Self-healing ✅
* No version history ❌
* No rollback ❌
* Manual scaling only

```bash
kubectl scale rs <replicaset-name> --replicas=5
```
---

## Deployment

* Manages ReplicaSets automatically ✅
* Tracks history ✅
* Supports rollbacks ✅
* Rolling updates ✅
* Version control ✅

### Rollback a Deployment

```bash
kubectl rollout undo deployment myapp
```
---

## Conclusion

ReplicaSets are fundamental to Kubernetes and provide **self-healing** and **scaling** capabilities. However, for production workloads, **Deployments are the preferred choice** due to built-in update strategies, rollback support, and version tracking.

Understanding ReplicaSets gives you strong insight into how Kubernetes maintains application reliability under the hood.

---

# Kubernetes Deployment 

A **Kubernetes Deployment** is a high-level resource used to manage, scale, and update applications while ensuring they remain in the **desired state**. It provides a **declarative** way to define how many Pods should run, which container images they should use, and how updates should be applied.

Think of a Deployment as both a **blueprint** and a **controller** for Pods that greatly simplifies application lifecycle management in Kubernetes.

---

## What Can You Do with a Deployment?

With a Kubernetes Deployment, you can:

- Scale applications up or down based on demand
- Maintain reliability by ensuring the desired number of Pods are always running
- Perform **rolling updates** without downtime
- Easily **roll back** if an update causes issues
- Automate self-healing when Pods fail or are deleted

---

## Kubernetes Deployment Components

A Deployment consists of three main components:

### 1. Metadata
Contains identifying information such as:
- Name of the Deployment
- Labels used to connect Deployments with Services

### 2. Specification (spec)
Defines the **desired state**, including:
- Number of replicas
- Label selectors
- Pod template (blueprint for Pods)

The Pod template includes:
- Container images
- Container names
- Ports
- Resource definitions

### 3. Status
Automatically generated by Kubernetes.
- Tracks the **current state**
- Enables **self-healing**
- If actual state ≠ desired state, Kubernetes reconciles automatically

---

## Benefits of Using Kubernetes Deployments

Kubernetes Deployments provide a structured and automated way to manage application delivery.

### Key Benefits

- **Declarative updates**  
  Define desired state in YAML; Kubernetes handles execution

- **Rolling updates**  
  Gradually replace Pods to avoid downtime

- **Automatic rollback**  
  Revert to a previous version if rollout fails

- **Self-healing**  
  Automatically recreates failed or deleted Pods

- **Consistent scaling**  
  Adjust replicas without impacting application behavior

- **Separation of concerns**  
  Configuration, lifecycle management, and runtime are loosely coupled

---

# Types of Kubernetes Deployment Strategies

The **deployment strategy** determines how Kubernetes updates applications from an old version to a new one. The choice impacts **availability, traffic flow, risk, and rollback complexity**.

Some strategies may cause downtime, while others support **zero-downtime** and **progressive delivery**.

---

## Deployment Strategy Comparison

| Deployment Strategy | Traffic Shift Model | Downtime Risk | Rollback Effort | Tools / Primitives | Ideal Use Cases | Key Caveats |
|--------------------|--------------------|--------------|----------------|-------------------|---------------|------------|
| Recreate | Stop then start | High | Very easy | Deployment (Recreate) | Labs, migrations | Full outage |
| Rolling Update | Gradual replacement | Low | Easy | Deployment (default) | Production updates | Slow error detection |
| Ramped / Slow Rollout | Throttled rolling | Very low | Easy | Rollout pause/resume | Critical workloads | Ops overhead |
| Blue-Green | Instant traffic switch | Zero | Instant | Services, Ingress | Major releases | Double infra cost |
| Best-effort Controlled | Fast rolling | Moderate | Easy | Argo Rollouts | Stateless apps | SLO risk |
| Shadow | Mirrored traffic | Zero | None | Istio, Envoy | Load testing | Double compute |
| Canary | Progressive traffic | Very low | Very easy | Argo Rollouts, Flagger | Feature launches | Requires metrics |
| A/B Testing | Rule-based routing | Very low | Moderate | Service mesh, gateways | Experiments | Analytics complexity |

---

# Kubernetes Nginx Deployment Guide

This section demonstrates a **hands-on example** of deploying Nginx using a Kubernetes Deployment.

---

## Step 1: Create a Deployment YAML File

### `nginx-deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
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
          image: nginx:1.14.2
          ports:
            - containerPort: 80
````

Save the file as:

```bash
nginx-deployment.yaml
```

---

## Step 2: Apply the Deployment

```bash
kubectl apply -f nginx-deployment.yaml
```

Expected output:

```text
deployment.apps/nginx-deployment created
```

### Verify Deployment Status

```bash
kubectl get deployments
```

Example output:

```text
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           29s
```

---

## Step 3: Expose the Deployment

Expose the application using a Service:

```bash
kubectl expose deployment nginx-deployment \
  --type=LoadBalancer \
  --port=80 \
  --target-port=80
```

### Verify Service

```bash
kubectl get services
```

---

## Step 4: Update the Deployment

Update the container image:

Earlier image: nginx:1.14.2

```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1
```

### Monitor Rollout Status

```bash
kubectl rollout status deployment/nginx-deployment
```

---

## Step 5: Scale the Deployment

Scale based on traffic demand:

```bash
kubectl scale deployment/nginx-deployment --replicas=5
```
---

## Best Practices

* Use **Rolling Updates** to minimize downtime
* Monitor Deployments using:

  ```bash
  kubectl describe deployment nginx-deployment
  ```
* Enable **Horizontal Pod Autoscaling (HPA)**:

  ```bash
  kubectl autoscale deployment nginx-deployment \
    --min=3 \
    --max=10 \
    --cpu-percent=80
  ```

---

## Conclusion

Kubernetes Deployments are the **preferred way** to run applications in production. They provide powerful features like rolling updates, rollback support, self-healing, and scaling — all while keeping application management simple and declarative.

For modern Kubernetes workloads, **Deployment > ReplicaSet**.



