## ☸️ What is Kubernetes (K8s)?

**Kubernetes**, also called **K8s**, is an **open-source platform** that automates the **deployment, scaling, and management** of containerized applications.

🔹 It groups related containers into **logical units**
🔹 Makes applications easy to **manage, discover, and scale**
🔹 Built on **15+ years of Google’s production experience**
🔹 Backed by a strong **open-source community**

Because Kubernetes is **open source**, you can run it:

* 🏢 On-premises
* ☁️ In the cloud
* 🔀 In hybrid or multi-cloud environments

This gives you **freedom and portability** to move workloads anywhere.

---

## ❗ Why Containers Become a Problem at Scale

If everything is working, there’s **no issue** 👍
But problems start when **containers fail**.

### 🚨 Container Failures

A failed container (frontend, backend, or database) directly impacts users.

Manual fixes don’t scale because of:

* 🌍 24/7 global support needs
* 💸 High operational costs
* ⏱️ Slow response during off-hours
* 🔥 Difficulty handling multiple failures at once

---

## 📈 Scale & Complexity Challenges

As applications grow:

* Managing **hundreds or thousands** of containers manually becomes impossible
* Multiple failures need **fast, coordinated recovery**
* 💥 VM or host failures can crash entire apps
* 🔄 Updating many containers becomes complex and risky

👉 This is exactly **where Kubernetes helps**.

---

## 🤖 Why Kubernetes?

Kubernetes was designed to solve these exact problems.

### ✅ Key Benefits

1. **Scalability 📈** – Automatically scale up/down
2. **High Availability 💪** – Apps stay online
3. **Automated Rollouts & Rollbacks 🔄** – Safe deployments
4. **Service Discovery & Load Balancing ⚖️**
5. **Resource Management 🧠** – Efficient CPU & memory usage
6. **Self-Healing ❤️‍🩹** – Restarts failed containers automatically
7. **Extensibility 🔌** – Easy integrations
8. **Portability 🌍** – Run anywhere
9. **Strong Community 🌟** – Tools, plugins, support

---

## 🎨 Fun Kubernetes Facts

### ⚓ Why does the Kubernetes logo have 7 spokes?

Google ran an internal project called **“Seven”**, and the logo reflects that emotional and symbolic connection.

### 🔢 Why is it called K8s?

It’s a **numeronym**:

* **K** + **8 letters** + **s**
  Similar to **i18n** (internationalization).

---

# Kubernetes Architecture

Kubernetes follows a **distributed client–server architecture** designed to run containerized applications reliably and at scale.

A **Kubernetes cluster** consists of:

* A **Control Plane** (management layer)
* A set of **Worker Nodes** (execution layer)

Every cluster requires **at least one worker node** to run Pods.

In **production environments**, both the control plane and worker nodes typically run across **multiple machines**, ensuring:

* High availability
* Fault tolerance
* Scalability

---

## Cluster Overview

* **Control Plane**
  Manages the cluster, makes global decisions, and maintains the desired state.

* **Worker Nodes**
  Run application workloads inside Pods.

---

## Control Plane

The **control plane** is responsible for:

* Scheduling Pods
* Monitoring cluster state
* Responding to failures
* Managing scaling and updates

A cluster can have **one or more control plane nodes**.

### Control Plane Components

* `kube-apiserver`
* `etcd`
* `kube-scheduler`
* `kube-controller-manager`
* `cloud-controller-manager`

---

## 1. kube-apiserver

The **kube-apiserver** is the front end of the Kubernetes control plane.

It exposes the **Kubernetes API** and acts as the central communication hub for all components.

### Key Responsibilities

* Exposes cluster API endpoints
* Handles all API requests
* Supports multiple API versions
* Authentication (certificates, tokens, basic auth)
* Authorization (RBAC, ABAC)
* Validation and mutation via admission controllers
* Coordinates communication between control plane and worker nodes
* Provides a watch mechanism for real-time updates
* Includes an aggregation layer for custom APIs (CRDs)
* Contains a built-in API server proxy for accessing ClusterIP services externally

### Architecture Notes

* All components (scheduler, controllers, kubelets) communicate **through the API server**
* The **only component the API server directly connects to is etcd**
* Designed to **scale horizontally** by running multiple instances behind a load balancer

### Security Note

The API server must be properly secured. Publicly exposed API servers significantly increase the attack surface of a cluster.

---

## 2. etcd

**etcd** is the **backing store and brain of Kubernetes**.

It stores **all cluster data**, including:

* Configuration
* State
* Metadata of Kubernetes objects

### Characteristics

* **Strongly consistent** distributed key-value store
* Designed to run as a cluster without sacrificing consistency
* Built on `bboltDB`
* Uses the **Raft consensus algorithm** for leader election and fault tolerance

### How Kubernetes Uses etcd

* All Kubernetes objects are stored in etcd
* API server reads and writes cluster state to etcd
* Uses the `Watch()` API to track object state changes

### Example Storage Path

```
/registry/pods/default/nginx
```

⚠️ **Regular backups of etcd are critical** for disaster recovery.

---

## 3. kube-scheduler

The **kube-scheduler** assigns Pods to worker nodes.

It watches for newly created Pods that do not yet have a node assigned.

### Scheduling Factors

* CPU and memory requirements
* Node availability
* Hardware/software constraints
* Affinity and anti-affinity rules
* Data locality
* Inter-workload interference
* Deadlines and priorities

### Scheduling Process

1. **Filtering phase**

   * Identifies nodes that can run the Pod
   * Uses `percentageOfNodesToScore` to optimize performance in large clusters

2. **Scoring phase**

   * Ranks eligible nodes using scheduling plugins
   * Node with the highest score is selected
   * If scores are equal, a node is chosen randomly

3. **Binding**

   * Scheduler creates a binding event in the API server
   * Pod is assigned to the selected node

### Additional Capabilities

* High-priority Pods are scheduled first
* Supports Pod eviction and rescheduling
* Allows **custom schedulers**
* Uses a **pluggable scheduling framework**

---

## 4. kube-controller-manager

Controllers continuously monitor the cluster and ensure the **desired state matches the actual state**.

Each controller runs an infinite control loop.

### Common Controllers

* Deployment Controller
* ReplicaSet Controller
* DaemonSet Controller
* Job Controller
* CronJob Controller
* Endpoint Controller
* Namespace Controller
* ServiceAccount Controller
* Node Controller

### Responsibilities

* Manages all built-in controllers
* Ensures cluster resources stay in the desired state
* Supports extending Kubernetes using **custom controllers and CRDs**

---

## 5. cloud-controller-manager (CCM)

The **Cloud Controller Manager** integrates Kubernetes with cloud provider APIs.

It is used only in cloud environments (AWS, Azure, GCP).

### Responsibilities

* Node lifecycle management
* Cloud load balancer provisioning
* Cloud routing configuration
* Persistent volume provisioning

This separation allows Kubernetes core components to remain cloud-agnostic.

---

## Worker Node Components

Worker nodes run application workloads and provide the runtime environment.

### 1. kubelet

* Agent running on every worker node
* Ensures containers defined in PodSpecs are running and healthy
* Communicates with the API server
* Manages only Kubernetes-created containers

---

### 2. kube-proxy (Optional)

* Network proxy implementing Kubernetes Service networking
* Maintains network rules on nodes
* Enables Pod-to-Pod and external traffic routing
* Uses OS-level packet filtering when available

> Some CNI plugins can replace kube-proxy functionality.

---

### 3. Container Runtime

The container runtime is responsible for:

* Pulling images
* Creating containers
* Managing container lifecycle

Supported runtimes include:

* `containerd`
* `CRI-O`
* Any CRI-compliant implementation

---

## 💻 Running Kubernetes Locally

Even though Kubernetes is widely used in the cloud, running it **locally** is very useful:

### ✅ Benefits

1. 🧪 Try Kubernetes before production
2. 🛡️ Separate dev and production safely

### 🛠️ Popular Local Kubernetes Tools

* **Minikube 🚀** – Best for local development
* **kind 🐳** – Runs clusters inside Docker containers
* **CodeReady Containers (CRC) 🖥️** – Local OpenShift 4.x
* **Minishift 💻** – Local OpenShift 3.x (VM-based)

All are **open source** and Apache 2.0 licensed.

---


