InKubernetes, ReplicationController and ReplicaSet are essential components used for managing and maintaining the desired number of pod replicas to ensure high availability and fault tolerance of applications. They enable automatic scaling, self-healing, and load distribution within a Kubernetes cluster. In this document, we will explore ReplicationController and ReplicaSet concepts along with practical examples demonstrating each step involved.


Have you ever wondered what is responsible for supervising and managing just the exact number of pods running inside the Kubernetes cluster? 
Kubernetes can do this in multiple ways, but one common approach is using ReplicationController (rc). 


A ReplicationController is responsible for managing the pod lifecycle and ensuring that the specified number of pods required are running at any given time. On the other hand, it is not responsible for the advanced cluster capabilities like performing auto-scaling, readiness and liveliness probes, and other advanced replication capabilities. Other components within the Kubernetes cluster better perform those capabilities.
If there are more than the desired number, the ReplicationController removes the excess ones and ensures the same number of pods exist even in the event of node failure or pod termination


Replication – It can replicate or scale one pod to many
Controller – It would control the desire number of pods with actual number of pod.

A ReplicationController is a Kubernetes controller that ensures that a specified number of pod replicas are running at any one time. In other words, a ReplicationController makes sure that a pod or a homogeneous set of pods is always up and available.

ReplicationControllers are useful for running stateful applications, such as databases and message queues, which need to be highly available. They can also be used to run stateless applications, such as web servers, which need to be scalable.

Once you have created a ReplicationController, it will start creating and managing pod replicas. If a pod replica fails, the ReplicationController will create a new replica to replace it.

ReplicationControllers are a powerful tool for managing Kubernetes pods. They can help to ensure that your applications are always up and available, and that they can scale to meet demand.

Here are some examples of how ReplicationControllers can be used:

A production team can use a ReplicationController to ensure that their production pods are always up and available.
A development team can use a ReplicationController to scale their development pods up or down as needed.
A company can use a ReplicationController to run a distributed database cluster.
A website can use a ReplicationController to run a scalable web server farm


Here’s a detailed step-by-step tutorial on using Kubernetes ReplicationController (RC), covering your sample YAML, key commands, scaling, and how RC manages Pods. This will walk you through the complete workflow, with explanations for every step.


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



Run the example job by downloading the example file and then running this command:

kubectl apply -f https://k8s.io/examples/controllers/replication.yaml

The output is similar to this:

replicationcontroller/nginx created


Check on the status of the ReplicationController using this command:

kubectl describe replicationcontrollers/nginx


$ kubectl describe rc/nginx
Name:         nginx
Namespace:    default
Selector:     app=nginx
Labels:       app=nginx
Annotations:  <none>
Replicas:     3 current / 3 desired
Pods Status:  3 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=nginx
  Containers:
   nginx:
    Image:         nginx
    Port:          80/TCP
    Host Port:     0/TCP
    Environment:   <none>
    Mounts:        <none>
  Volumes:         <none>
  Node-Selectors:  <none>
  Tolerations:     <none>
Events:
  Type    Reason            Age    From                    Message
  ----    ------            ----   ----                    -------
  Normal  SuccessfulCreate  3m46s  replication-controller  Created pod: nginx-g7vhn
  Normal  SuccessfulCreate  3m46s  replication-controller  Created pod: nginx-dnslf
  Normal  SuccessfulCreate  3m46s  replication-controller  Created pod: nginx-hzjkt
  Normal  SuccessfulCreate  105s   replication-controller  Created pod: nginx-jpclq
 

To list all the pods that belong to the ReplicationController in a machine readable form, you can use a command like this:

pods=$(kubectl get pods --selector=app=nginx --output=jsonpath={.items..metadata.name})
echo $pods

$ pods=$(kubectl get pods --selector=app=nginx --output=jsonpath={.items..metadata.name})
echo $pods
nginx-g7vhn nginx-hzjkt nginx-jpclq
 
lets delete (nginx-g7vhn) which is running on node cka-qacluster-worker.then rc will create based upon the replicas we provided in the rc.yml(3 replicase we deinfed in rc.yml)

$ kubectl get pods -o wide
NAME                 READY   STATUS    RESTARTS        AGE     IP           NODE                    NOMINATED NODE   READINESS GATES
app-db-sidecar-pod   3/3     Running   3 (7m58s ago)   18h     10.244.2.2   cka-qacluster-worker2   <none>           <none>
devweb-server-pod    0/1     Unknown   0               23h     <none>       cka-qacluster-worker    <none>           <none>
nginx-g7vhn          1/1     Running   0               5m28s   10.244.1.3   cka-qacluster-worker    <none>           <none>
nginx-hzjkt          1/1     Running   0               5m28s   10.244.1.2   cka-qacluster-worker    <none>           <none>
nginx-jpclq          1/1     Running   0               3m27s   10.244.2.5   cka-qacluster-worker2   <none>           <none>
web-server-pod       1/1     Running   1 (7m58s ago)   24h     10.244.2.3   cka-qacluster-worker2   <none>           <none>

delted the pod nginx-g7vhn
Dubbakas@Dubbakas MINGW64 /d/Docker & k8s 2026
$ kubectl delete pod nginx-g7vhn
pod "nginx-g7vhn" deleted


let verfiy we have new pod or not 
a new pod is created with nginx-45qcx  in the same node where we deleted it 

Dubbakas@Dubbakas MINGW64 /d/Docker & k8s 2026
$  pods=$(kubectl get pods --selector=app=nginx --output=jsonpath={.items..metadata.name})
echo $pods
nginx-45qcx nginx-hzjkt nginx-jpclq

Dubbakas@Dubbakas MINGW64 /d/Docker & k8s 2026
$ kubectl get pods -o wide
NAME                 READY   STATUS    RESTARTS      AGE     IP           NODE                    NOMINATED NODE   READINESS GATES
app-db-sidecar-pod   3/3     Running   3 (14m ago)   18h     10.244.2.2   cka-qacluster-worker2   <none>           <none>
devweb-server-pod    0/1     Unknown   0             23h     <none>       cka-qacluster-worker    <none>           <none>
nginx-45qcx          1/1     Running   0             86s     10.244.1.4   cka-qacluster-worker    <none>           <none>
nginx-hzjkt          1/1     Running   0             11m     10.244.1.2   cka-qacluster-worker    <none>           <none>
nginx-jpclq          1/1     Running   0             9m43s   10.244.2.5   cka-qacluster-worker2   <none>           <none>
web-server-pod       1/1     Running   1 (14m ago)   24h     10.244.2.3   cka-qacluster-worker2   <none>           <none>


this show the replication will work for high avilabilty apods exist even in the event of node failure or pod termination

suppose if we delete the replication controller then all the pods mettioned it will be delted 

Dubbakas@Dubbakas MINGW64 /d/Docker & k8s 2026
$ kubectl get all
NAME                     READY   STATUS    RESTARTS        AGE
pod/app-db-sidecar-pod   3/3     Running   3 (3h13m ago)   21h
pod/devweb-server-pod    0/1     Unknown   0               26h
pod/nginx-45qcx          1/1     Running   0               3h
pod/nginx-hzjkt          1/1     Running   0               3h10m
pod/nginx-jpclq          1/1     Running   0               3h8m
pod/web-server-pod       1/1     Running   1 (3h13m ago)   27h

NAME                          DESIRED   CURRENT   READY   AGE
replicationcontroller/nginx   3         3         3       3h10m

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   2d2h

Dubbakas@Dubbakas MINGW64 /d/Docker & k8s 2026
$ kubectl delete replicationcontroller/nginx
replicationcontroller "nginx" deleted

Dubbakas@Dubbakas MINGW64 /d/Docker & k8s 2026
$ kubectl get pods -o wide
NAME                 READY   STATUS    RESTARTS        AGE   IP           NODE                    NOMINATED NODE   READINESS GATES
app-db-sidecar-pod   3/3     Running   3 (3h14m ago)   21h   10.244.2.2   cka-qacluster-worker2   <none>           <none>
devweb-server-pod    0/1     Unknown   0               26h   <none>       cka-qacluster-worker    <none>           <none>
web-server-pod       1/1     Running   1 (3h14m ago)   27h   10.244.2.3   cka-qacluster-worker2   <none>           <none>

Dubbakas@Dubbakas MINGW64 /d/Docker & k8s 2026
$ 



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

## Overview
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



In Kubernetes, a ReplicaSet (RS) is a core component designed to ensure the availability and consistency of Pods in a cluster. It controls the number of pod replicas running at any given time, maintaining desired states and replacing pods as needed if they fail or are deleted. 

1. What is a ReplicaSet?
A ReplicaSet is a controller that helps you define and maintain a specific number of pod instances in a Kubernetes cluster. This controller ensures that if a pod goes down or is deleted, it is quickly replaced to maintain the desired state defined by the user. This self-healing mechanism provides high availability and redundancy for applications.

Key Concepts:
Desired number of replicas: The number of pod replicas the ReplicaSet will attempt to maintain.
Labels: Used to select which pods are managed by the ReplicaSet.
Self-healing: Automatically replaces pods if they are deleted or fail.

2. Purpose of a ReplicaSet
The primary purposes of a ReplicaSet include:

Ensuring Application Availability: It makes sure the specified number of pod replicas is always running, providing high availability.
Automatic Scaling: Allows scaling up and down by adjusting the number of replicas.
Resilience and Fault Tolerance: Automatically replaces failed or terminated pods without manual intervention.

3. Components of a ReplicaSet
A ReplicaSet consists of the following components:

Selector: Specifies a label query to select the pods managed by the ReplicaSet. Only pods with matching labels will be monitored and managed.
Replicas: Defines the desired number of pod replicas.
Template: A pod template containing the specification for the pods, including container image, resources, and ports. New pods are created based on this template.

4. How ReplicaSet Works
When you define a ReplicaSet in a YAML file and apply it, Kubernetes will:

Create the specified number of pods based on the template.
Continuously monitor the pods to ensure they match the specified replicas count.
Automatically create new pods if any of the existing pods are deleted or fail, or scale down if there are too many.
5. Creating a ReplicaSet: A Step-by-Step Example
Let’s go through creating a ReplicaSet for an Nginx deployment.

ReplicaSet YAML Configuration Example
Below is an example YAML file for creating a ReplicaSet with three replicas of an Nginx pod.

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
		
Explanation of YAML File
apiVersion: The API version for ReplicaSet (in this case, apps/v1).
kind: Specifies the resource type, ReplicaSet.
metadata: Contains the name of the ReplicaSet.
spec:
replicas: Desired number of pod replicas. Here, we specify 5.
selector: Defines the label selector to match the pods managed by this ReplicaSet.
template: Contains the specification for the pod:
metadata: Labels to identify the pods.
spec: Specifies the container configuration. Here, we define an Nginx container.

Applying the ReplicaSet
Apply the ReplicaSet using the following command:

$ kubectl apply -f rs.yaml
replicaset.apps/nginx-replicaset created

Verifying the ReplicaSet
Once applied, you can verify that the ReplicaSet has created three Nginx pods.

kubectl get rs
$ kubectl get rs
NAME               DESIRED   CURRENT   READY   AGE
nginx-replicaset   5         5         5       63s


$ kubectl get pods -l app=nginx
NAME                     READY   STATUS    RESTARTS   AGE
nginx-replicaset-7hg8h   1/1     Running   0          90s
nginx-replicaset-8z9f6   1/1     Running   0          89s
nginx-replicaset-9fsw2   1/1     Running   0          89s
nginx-replicaset-c2v29   1/1     Running   0          89s
nginx-replicaset-ls6cg   1/1     Running   0          89s


Kubernetes scheduled your 5 Pods across 2 nodes: 3 on cka-qacluster-worker and 2 on cka-qacluster-worker2. This is normal—ReplicaSet ensures the Pods running,
but doesn’t guarantee perfectly even distribution. To spread them evenly, you can use Pod anti-affinity or PodTopologySpread rules. which we will discuss later

Dubbakas@Dubbakas MINGW64 /d/Docker & k8s 2026
$ kubectl get pods -o wide
NAME                     READY   STATUS    RESTARTS        AGE     IP           NODE                    NOMINATED NODE   READINESS GATES
app-db-sidecar-pod       3/3     Running   3 (3h21m ago)   22h     10.244.2.2   cka-qacluster-worker2   <none>           <none>
devweb-server-pod        0/1     Unknown   0               27h     <none>       cka-qacluster-worker    <none>           <none>
nginx-replicaset-7hg8h   1/1     Running   0               2m18s   10.244.1.7   cka-qacluster-worker    <none>           <none>
nginx-replicaset-8z9f6   1/1     Running   0               2m17s   10.244.1.5   cka-qacluster-worker    <none>           <none>
nginx-replicaset-9fsw2   1/1     Running   0               2m17s   10.244.2.7   cka-qacluster-worker2   <none>           <none>
nginx-replicaset-c2v29   1/1     Running   0               2m17s   10.244.2.6   cka-qacluster-worker2   <none>           <none>
nginx-replicaset-ls6cg   1/1     Running   0               2m17s   10.244.1.6   cka-qacluster-worker    <none>           <none>
web-server-pod           1/1     Running   1 (3h21m ago)   27h     10.244.2.3   cka-qacluster-worker2   <none>           <none>

ReplicaSet ensures the desired number of Pods are always running. Deleting a Pod triggers it to recreate a new one automatically — self-healing in action: “Systems that heal themselves never break for long.”

Dubbakas@Dubbakas MINGW64 /d/Docker & k8s 2026
$ kubectl delete pod nginx-replicaset-7hg8h
pod "nginx-replicaset-7hg8h" deleted


Dubbakas@Dubbakas MINGW64 /d/Docker & k8s 2026
$ kubectl get pods
NAME                     READY   STATUS    RESTARTS        AGE
app-db-sidecar-pod       3/3     Running   3 (3h27m ago)   22h
devweb-server-pod        0/1     Unknown   0               27h
nginx-replicaset-8z9f6   1/1     Running   0               7m45s
nginx-replicaset-9fsw2   1/1     Running   0               7m45s
nginx-replicaset-c2v29   1/1     Running   0               7m45s
nginx-replicaset-jsq5b   1/1     Running   0               9s
nginx-replicaset-ls6cg   1/1     Running   0               7m45s
web-server-pod           1/1     Running   1 (3h27m ago)   27h



6. Scaling a ReplicaSet
ReplicaSets are designed to make scaling applications straightforward. You can change the replicas count either by editing the YAML file or directly using kubectl.

1️⃣ Current state

replicas = 5 → 5 Pods running

2️⃣ Desired state

replicas = 10 → you want 10 Pods running

3⃣ What happens

Kubernetes detects current replicas (5) < desired replicas (10)

ReplicaSet creates 5 new Pods automatically

After scaling, total Pods = 10, achieving the desired state

💡 Key point:
ReplicaSet ensures self-healing and scaling. Any deleted or failed Pod will be recreated to maintain the desired replicas.

Scaling with kubectl

Dubbakas@Dubbakas MINGW64 /d/Docker & k8s 2026
$ kubectl scale replicaset nginx-replicaset --replicas=10
replicaset.apps/nginx-replicaset scaled

This command updates the replicas to 10. Kubernetes will create two more pods to meet this new count.


1️⃣ Get the ReplicaSet YAML

1️⃣ Get the ReplicaSet YAML
kubectl get rs <replicaset-name> -o yaml > rs-scaled.yaml

Dubbakas@Dubbakas MINGW64 /d/Docker & k8s 2026
$ kubectl get rs
NAME               DESIRED   CURRENT   READY   AGE
nginx-replicaset   10        10        10      45m

Dubbakas@Dubbakas MINGW64 /d/Docker & k8s 2026
$ kubectl get rs nginx-replicaset -o yaml > rs-scaled.yaml

<replicaset-name> → your ReplicaSet name

-o yaml → outputs full YAML

> rs-scaled.yaml → saves it to a file


1️⃣ Direct rollback in a ReplicaSet is manual

Unlike Deployments, ReplicaSets do not store history, so you cannot “undo” automatically.
You have to manually set the replicas back to 5. as we have scaled in the above 
kubectl scale rs <replicaset-name> --replicas=5

replicaSets are self-healing, but they don’t track history. Rollbacks must be done manually by resetting the replicas field.

8. Deleting a ReplicaSet
When you delete a ReplicaSet, all pods created by it are also deleted by default.

$ kubectl delete replicaset.apps/nginx-replicaset
replicaset.apps "nginx-replicaset" deleted

To delete the ReplicaSet but leave the pods running, you can use --cascade=orphan:

kubectl delete rs nginx-replicaset --cascade=orphan

kubectl delete replicaset.apps/nginx-replicaset --cascade=orphan

This command removes the ReplicaSet but keeps the pods. They will no longer be managed by the ReplicaSet.
Orphaned Pods are “free agents” — they run until you delete them, their node dies, or some system action evicts them.



# Why Choose Deployment Over ReplicaSet

In Kubernetes, both **ReplicaSets** and **Deployments** manage Pods, but they serve different purposes when it comes to updates and rollbacks.

---

## 1. ReplicaSet

- Ensures a specified number of Pods are running ✅
- Automatically recreates Pods if they fail ✅
- **Does NOT track history of changes** ❌
- **Cannot roll back** automatically ❌
- Scaling must be done manually

**Example:** Scaling manually
```bash
kubectl scale rs <replicaset-name> --replicas=5

## 2. Deployment

- Manages ReplicaSets under the hood ✅
- Every update creates a new ReplicaSet ✅
- Keeps old ReplicaSets for rollback ✅

Supports:

- Rolling updates
- Rollbacks
- Version history
- Self-healing and scaling built-in

**Example:* Rollback a Deployment
```bash

kubectl rollout undo deployment myapp



A Kubernetes Deployment is a high-level resource used to manage and scale applications while ensuring they remain in the desired state. It provides a declarative way to define how many Pods should run, which container images they should use, and how updates should be applied.

With a Deployment, you can:
Scale applications up or down based on demand.
Maintain reliability, ensuring the desired number of Pods are always running and healthy.
Perform rolling updates to introduce new versions without downtime.
Rollback easily if an update causes issues.
Think of it as both a blueprint and controller for Pods that simplifies application management in Kubernetes.

Kubernetes Deployment Components
It mainly consists of three components:

Metadata: It consists of the name and labels for the configuration file. The labels are used for establishing the connection between deployment and services.
Specification: It consists of information regarding the number of replicas, selector labels, and template that is the blueprint for pods. The template itself is like a configuration file for pods and consists of metadata and specification for pods that stores information regarding the containers to be used in the pod, the image to be used for building the container, and the name and ports of the container.
Status: This component is automatically generated and added by Kubernetes. This is the basis of the self-healing feature of Kubernetes. If the desired status and actual status of a deployment do not match Kubernetes fixes the pod and matches it with the desired status.


What are the benefits of using the Kubernetes Deployment object?
The Kubernetes deployment object offers a structured way to manage application delivery and lifecycle using declarative configurations. It simplifies complex operations like updates, recovery, and scaling by automating them behind the scenes.

Key benefits include:

Declarative updates: Define your desired application state in YAML; Kubernetes handles the rest.
Rolling updates: Apply updates gradually to avoid downtime.
Automatic rollback: Revert to a previous version if a rollout fails.
Self-healing: Automatically replaces failed or deleted pods to maintain availability.
Consistent scaling: Increase or decrease replicas without changing application behavior.
Separation of concerns: Keeps configuration, lifecycle management, and runtime loosely coupled for better maintainability.


# Types of Kubernetes Deployment Strategies

When deploying applications to a Kubernetes (K8s) cluster, the **deployment strategy** you choose determines how your application is updated from an older version to a newer one. Different strategies impact **availability, traffic flow, rollback complexity, and risk**.

Some deployment strategies may cause downtime, while others enable **zero-downtime releases**, **progressive delivery**, and **user behavior analysis**.

---

## Basic Kubernetes Deployment Strategies

These strategies are commonly used and natively supported by Kubernetes.

## Deployment Strategy Comparison

| Deployment Strategy | Traffic Shift Model | Downtime Risk | Rollback Effort | Tools / Primitives | Ideal Use Cases | Key Caveats |
|--------------------|--------------------|--------------|----------------|-------------------|---------------|------------|
| Recreate | Stop then start | High | Very easy | Deployment (`Recreate`) | Labs, migrations | Full outage |
| Rolling Update | Gradual replacement | Low | Easy | Deployment (default) | Production updates | Slow error detection |
| Ramped / Slow Rollout | Throttled rolling | Very low | Easy | Rollout pause/resume | Critical workloads | Ops overhead |
| Blue-Green | Instant traffic switch | Zero | Instant | Services, Ingress | Major releases | Double infra cost |
| Best-effort Controlled | Fast rolling | Moderate | Easy | Argo Rollouts | Stateless apps | SLO risk |
| Shadow | Mirrored traffic | Zero | None | Istio, Envoy | Load testing | Double compute |
| Canary | Progressive traffic | Very low | Very easy | Argo Rollouts, Flagger | Feature launches | Requires metrics |
| A/B Testing | Rule-based routing | Very low | Moderate | Service mesh, gateways | Experiments | Analytics complexity |

---

````md
# Kubernetes Nginx Deployment Guide

This README explains how to deploy an **Nginx application on Kubernetes**, expose it, update it, scale it, and follow best practices.

---

## Step 1: Create a Deployment YAML File

Create a deployment configuration file for Nginx.

### nginx-deployment.yaml

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

Apply the deployment using kubectl:

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

Expose the deployment to make it accessible:

```bash
kubectl expose deployment nginx-deployment \
  --type=LoadBalancer \
  --port=80 \
  --target-port=80
```

### Check Service Details

```bash
kubectl get services
```

---

## Step 4: Update the Deployment

Update the Nginx container image:

```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1
```

### Monitor Rollout Status

```bash
kubectl rollout status deployment/nginx-deployment
```

---

## Step 5: Scale the Deployment

Scale the deployment based on traffic needs:

```bash
kubectl scale deployment/nginx-deployment --replicas=5
```

---

## Best Practices

* Use **Rolling Updates** to minimize downtime
* Monitor deployments using:

  ```bash
  kubectl describe deployment nginx-deployment
  ```
* Enable **Horizontal Pod Autoscaling (HPA)**:

  ```bash
  kubectl autoscale deployment/nginx-deployment \
    --min=3 \
    --max=10 \
    --cpu-percent=80
  ```

This approach ensures high availability, scalability, and efficient resource utilization.

```
```

