# 🚀 Kubernetes Pods & Multi-Container Pods

## 📦 Understanding Pods
Understand Pods, the smallest deployable compute object in Kubernetes, and the higher-level abstractions that help you to run them.  
A workload is an application running on Kubernetes. Whether your workload is a single component or several that work together, on Kubernetes you run it inside a set of pods. In Kubernetes, a Pod represents a set of running containers on your cluster.

A container as we all know, is a self-contained environment where we package applications and their dependencies. Typically, a container runs a single process (Although there are ways to run multiple processes). Each container gets an IP address and can attach volumes and control CPU and memory resources, among other things. All these happen via the concepts of namespaces and control groups.

Kubernetes is a container orchestration system for deploying, scaling, and managing containerized applications, and it has its own way of running containers. We call it a pod. A pod is the smallest deployable unit in Kubernetes that represents a single instance of an application.

### ❓ So how does it differ from a container?

A container is a single unit. However, a pod can contain more than one container. You can think of pods as a box that can hold one or more containers together.

Pod provides a higher level of abstraction that allows you to manage multiple containers as a single unit. Here instead of each container getting an IP address, the pod gets a single unique IP address and containers running inside the pod use localhost to connect to each other on different ports.

---

## 🛠️ Creating Pod (Practical Examples)

You can create a pod in two ways:

- **Using the kubectl imperative command**: Primarily used for learning and testing purposes. The imperative command comes with its own limitations.
- **Declarative approach**: Using YAML manifest. When working on projects, the YAML manifest is used to deploy pods.

> ⚠️ **Note:** Kubectl imperative commands are very important when you appear for Kubernetes Certifications, as they save time compared to declarative approaches.

kubectl Imperative Command

## ⚙️ Imperative Commands
Imperative commands are primarily used for:
- **Learning**: Quick experimentation with Kubernetes resources.
- **Testing**: Rapid prototyping without writing manifests.
- **One-off tasks**: Simple operations that don’t need to be repeated.

🚧 Limitations of Imperative Commands
While convenient, imperative commands come with constraints:
- Not Repeatable: Hard to reproduce consistently across environments.
- No Version Control: Commands aren’t easily tracked in Git or CI/CD pipelines.
- Harder Collaboration: Teams can’t easily share or review commands.
- Limited Complexity: Managing advanced configurations (labels, annotations, probes) is cumbersome.
- State Drift: Manual commands can lead to inconsistencies between clusters.

```bash
kubectl run devweb-server-pod \
  --image=nginx:1.14.2 \
  --restart=Never \
  --port=80 \
  --labels=app=web-server,environment=production \
  --annotations description="This pod runs the devweb server"
````

Here the pod gets deployed in the default namespace. You can get the status of the deployed pod using kubectl.

```bash
kubectl get pods
kubectl get pods -w
```


![Image Alt](https://github.com/HarishDubbaka/Docker-k8s/blob/fdb1bb81d8fbae2857e88829a0e2bbf0f4fdf589/Day16/imperative%20getpods.png)

---

## 🔍 Describe a Pod

If you want to know all the details of the running pod, you can describe the pod using kubectl.

```bash
kubectl describe pod devweb-server-pod
```
![Image Alt](https://github.com/HarishDubbaka/Docker-k8s/blob/fdb1bb81d8fbae2857e88829a0e2bbf0f4fdf589/Day16/imperative%20getpods.png)

Look for the field:

```
Node: <node-name>/<node-IP>
```

👉 This tells you exactly which node (control-plane or worker) the Pod is scheduled on.

To see which cluster node the pod is running on:

```bash
kubectl get pods -o wide
```

![Image Alt](https://github.com/HarishDubbaka/Docker-k8s/blob/fdb1bb81d8fbae2857e88829a0e2bbf0f4fdf589/Day16/imperative%20getpods.png)

---

## 🧾 Pod YAML

Now that we have a basic understanding of a Pod, let's have a look at how we define a Pod. Pod is a native Kubernetes Object and if you want to create a pod, you need to declare the pod requirements in YAML format.
You can also create a pod using the kubectl imperative command. Which we will see in a post this topic.

Here is an example Pod YAML that creates an Nginx web server pod. This YAML is nothing but a declarative desired state of a pod

Let's understand this pod YAML. Once you understand the basic YAML it will be easier for you to work with pods and associated objects like deployment, daemonset, statefulset, etc.

As we discussed in the Kubernetes Object blog, every Kubernetes object has some common set of parameters. The values change as per the kind of object we are creating.

Let's take a look at the Kubernetes pod object.

Parameter	Description
# 📘 Kubernetes Pod Parameters and Descriptions

| Parameter   | Description                                                                 |
|-------------|-----------------------------------------------------------------------------|
| **apiVersion** | The API version of the Pod. For Pods, it is always `v1`.                   |
| **kind**       | Defines the type of Kubernetes object. In this case, it is `Pod`.          |
| **metadata**   | Metadata uniquely identifies and describes the Pod. Includes:              |
|                | • **labels** → Key-value pairs used to group and organize Pods (similar to tagging in cloud environments). |
|                | • **name** → The name of the Pod.                                          |
|                | • **namespace** → The namespace in which the Pod resides.                  |
|                | • **annotations** → Additional non-identifying metadata in key-value format (e.g., documentation, hints). |
| **spec**       | Declares the desired state of the Pod. Defines specifications for the containers to run inside the Pod. |
| **containers** | A list of containers that run inside the Pod. Each container definition includes: |
|                | • **name** → Identifier for the container.                                |
|                | • **image** → The container image to run.                                 |
|                | • **ports** → Ports exposed by the container.                             |
|                | • Other optional fields like resources, environment variables, and probes. |

We have now looked at a basic Pod YAML manifest. It's important to note that this manifest supports many parameters. We will gradually explore these additional parameters with a hands-on, practical approach.

Here is an example Pod YAML that creates an Nginx web server pod. This YAML is nothing but a declarative desired state of a pod.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-server-pod
  labels:
    app: web-server
    environment: production
  annotations:
    description: This pod runs the web server
spec:
  containers:
  - name: web-server
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

Deploy the manifest:

```bash
kubectl create -f nginx.yaml
```

---

## 🧪 Using --dry-run to Generate YAML

```bash
kubectl run nginx-pod --image=nginx:1.14.2 --dry-run=client -o yaml
kubectl run nginx-pod --image=nginx:1.14.2 --dry-run=client -o yaml > nginx-pod.yaml
```

✅ **Answer**
No — `--dry-run=client` will not create the Pod.
It only prepares the YAML manifest so you can review or save it.

To actually create the Pod:

```bash
kubectl apply -f nginx-pod.yaml
```

---

## 🌐 Access Application Running In a Pod

```bash
kubectl port-forward pod/web-server-pod 8080:80
```

Now access:
👉 [http://localhost:8080](http://localhost:8080)

> ⚠️ `kubectl port-forward` is mainly a debugging utility. For real access, use a Kubernetes **Service**.

---

## 🖥️ Access Pod Shell

```bash
kubectl exec -it web-server-pod -- /bin/sh
```

> 📝 Note: Container images are usually minimal, so not all Linux commands may be available.

---

## 📖 Why People Get Confused

The confusion comes from the fact that both kubectl create and kubectl apply can be used to bring resources into existence, but they behave differently once the resource already exists.

# 🔍 Comparison: `kubectl create` vs `kubectl apply`

## 📖 Overview
Kubernetes provides multiple ways to manage resources using `kubectl`.  
Two commonly used commands are **`create`** and **`apply`**, but they serve different purposes and often confuse beginners.

---

## 🔍 Comparison Table

| Aspect            | `kubectl create`                          | `kubectl apply`                           |
|-------------------|-------------------------------------------|-------------------------------------------|
| **Purpose**       | Creates a new resource only               | Creates or updates resources               |
| **Style**         | Imperative (one-time command)             | Declarative (desired state via YAML/JSON)  |
| **Repeatability** | Not repeatable – errors if resource exists| Idempotent – safe to run multiple times    |
| **Updates**       | Cannot update existing resources          | Updates resources if manifest changes      |
| **Version Control** | Commands are hard to track               | YAML manifests can be stored in Git        |
| **Collaboration** | Difficult to share commands               | Easy to share and review manifests         |
| **Best Use Case** | Quick tests, learning, temporary fixes    | Production workloads, CI/CD, GitOps        |

---

✅ Best Practice

* Use kubectl create for quick experiments or temporary resources.
* Use kubectl apply for production workloads, automation, and long-term cluster management.

👉 Remember: create is one-time and imperative, while apply is repeatable and declarative.

---

## 🔄 Pod Lifecycle

Another important concept you should know about a pod is its lifecycle.

A pod is typically managed by a controller like ReplicaSet Controller, Deployment controller, etc. When you create a single pod using YAML, it is not managed by any controller. In both cases, a pod goes through different lifecycle phases.

Pod lifecycle phases:

* **Pending** ⏳
* **Running** ▶️
* **Succeeded** ✅
* **Failed** ❌
* **Unknown** ❓

Following are the pod lifecycle phases.

* **Pending**: It means the pod creation request is successful, however, the scheduling is in process. For example, it is in the process of downloading the container image.  
* **Running**: The pod is successfully running and operating as expected. For example, the pod is service client requests.  
* **Succeeded**: All containers inside the pod have been successfully terminated. For example, the successful completion of a CronJob object.  
* **Failed**: All pods are terminated but at least one container has terminated in failure. For example, the application running inside the pod is unable to start due to a config issue and the container exits with a non-zero exit code.  
* **Unknown**: Unknown status of the pod. For example, the cluster is unable to monitor the status of the pod.

If you describe the pod, you can view the phase of the pod. Here is an example.  
pod lifecyle phases - pending,Running,Succeeded,Failed,Unknown pod status

![Image Alt](image_url) podstatus

---

## 🧩 Multi-Container Pods in Kubernetes

Multi-Container Pods in Kubernetes: Managing Multiple Containers within a Single Pod  
In Kubernetes, pods are the smallest deployable units that can contain one or more containers. While it's common to have a single container per pod, there are scenarios where running multiple containers in the same pod is beneficial. Multi-container pods allow you to group related containers that share the same network namespace and storage, and can be managed together as a unit.

### What is a Multi-Container Pod?

A multi-container pod in Kubernetes is a pod that encapsulates more than one container, which run together on the same node. These containers share the same network IP, which means they can communicate with each other via localhost. Additionally, they share storage volumes, so they can also exchange data through shared file systems.

While most pods are designed to run a single container, multi-container pods are useful when the containers have tightly coupled functionality and need to be managed together. This can be useful for scenarios such as:

- Supporting a main application with auxiliary services, like logging or monitoring.
- Sidecar containers that augment or enhance the functionality of the main container.
- Ambassador containers that proxy requests to the main application.
- Adapter containers that help transform data between formats.

### Why Use Multi-Container Pods?

There are several reasons why you might want to run multiple containers in a single pod, rather than deploying them as separate pods:

**Shared Network:**  
All containers in a pod share the same IP address and port space. This allows containers to communicate with each other using localhost or the internal pod DNS name, without needing to expose external network ports.

**Shared Storage Volumes:**  
Containers in the same pod can mount the same volumes, making it easy to share files between containers. For example, one container might write logs to a shared volume, and another container might aggregate and process those logs.

**Atomic Management:**  
Kubernetes treats all containers in a pod as a unit. This means that when you scale, deploy, or delete the pod, all containers are treated together. This is important when the containers have tightly coupled functionality, such as a main application and a logging sidecar.

**Tight Coupling of Containers:**  
Multi-container pods are ideal when multiple containers need to operate in close coordination. For example, a main application may need a sidecar container to handle specific tasks like logging, monitoring, or data processing.

---

## 🧠 Multi-Container Pod Patterns

Types of Multi-Container Pod Patterns  
Kubernetes supports several patterns for running multiple containers in a single pod. The most common patterns are:

### 1. Sidecar Pattern

The sidecar pattern is one of the most popular use cases for multi-container pods. In this pattern, one container in the pod is the "main" application container, and one or more additional containers (sidecars) augment its functionality.

**Example Use Case:**

A web server container is the main application, and a sidecar container could handle logging, proxying, or monitoring.  
The sidecar containers run alongside the main container and share the same network and storage resources, but they handle different tasks. They work together to provide complementary services for the main application.

**Example:**

A nginx web server with a sidecar container running a log shipping agent (e.g., fluentd) that collects logs and sends them to a centralized logging service.

### 2. Ambassador Pattern

In the ambassador pattern, a container acts as a proxy to facilitate communication between the pod's main container and external services. This pattern is used when your application needs to interact with other services outside the pod, but you want to encapsulate the interaction within the pod.

**Example Use Case:**

A container that handles communication between a microservice and an external database or an API, while the main application container performs the business logic.  
The ambassador container acts as a gateway or proxy for the main container, simplifying external communication.

### 3. Adapter Pattern

The adapter pattern is used when you need a container to modify the data before it is passed to or after it is received by the main application. The adapter container adapts or transforms the data format or protocol to fit the needs of the main container.

**Example Use Case:**

A main container running an application that consumes data in JSON format, while the adapter container transforms incoming data from XML to JSON.  
This pattern can also be used when integrating legacy systems with newer services that may use different data formats or protocols.

### 4. Init Containers

Init containers are special containers that run before the main application container(s) in a pod start. Init containers allow you to perform initialization tasks, such as setting up configuration files or performing health checks, before the main application containers run.

While init containers are not strictly sidecars, they are often used in multi-container pods for initial setup tasks.

**Example Use Case:**

An init container could download configuration files from an external source before the main application container starts.

---

**🧩 Multi-Container Pod Example (App + DB + Log Sidecar):**

📄 Pod YAML (App + DB + Sidecar)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-db-sidecar-pod
spec:
  containers:
  # 🧠 Web Application
  - name: web-app
    image: nginx:1.14.2
    ports:
    - containerPort: 80
    env:
    - name: DB_HOST
      value: localhost
    - name: DB_PORT
      value: "3306"

  # 🗄️ Database Container
  - name: mysql-db
    image: mysql:5.7
    env:
    - name: MYSQL_ROOT_PASSWORD
      value: rootpassword
    ports:
    - containerPort: 3306
    volumeMounts:
    - name: db-data
      mountPath: /var/lib/mysql

  # 📜 Sidecar Container (Log watcher)
  - name: db-log-sidecar
    image: busybox
    #command: ["sh", "-c", "tail -f /var/log/mysql.log"]
	command: ["sh", "-c", "while true; do sleep 3600; done"]
    volumeMounts:
    - name: db-data
      mountPath: /var/lib/mysql

  # 💾 Shared Volume
  volumes:
  - name: db-data
    emptyDir: {}

Perfect! Let’s access your **multi-container Pod** step by step. Since your Pod has **3 containers** (web app, DB, sidecar), you can pick which container to enter. 🖥️
```
---

# 🐳 Accessing Multi-Container Pod with MySQL DB

This guide shows how to work with a **multi-container Pod** running a web app, MySQL database, and a logging sidecar in Kubernetes.

---

## 1️⃣ See Containers Inside the Pod

```bash
kubectl get pod app-db-sidecar-pod -o jsonpath="{.spec.containers[*].name}"
````

✅ Example output:

```
web-app mysql-db log-sidecar
```

---

## 2️⃣ Exec into a Specific Container

**Syntax:**

```bash
kubectl exec -it <pod-name> -c <container-name> -- /bin/sh
```

**Examples:**

**Web app container (nginx)**

```bash
kubectl exec -it app-db-sidecar-pod -c web-app -- /bin/sh
```

**MySQL container**

```bash
kubectl exec -it app-db-sidecar-pod -c mysql-db -- /bin/sh
```

**Sidecar container (logs)**

```bash
kubectl exec -it app-db-sidecar-pod -c log-sidecar -- /bin/sh
```

> 📝 Note: Some containers may only have `/bin/sh` or `/bin/bash` depending on the image.

---

## 3️⃣ Accessing the MySQL Database

Once inside the MySQL container, connect to the DB:

```bash
mysql -u root -p
```

Enter your root password (as defined in the container spec).

---

## 4️⃣ Access the Web App

If your web container exposes port 80:

```bash
kubectl port-forward pod/app-db-sidecar-pod 8080:80
```

Open your browser: 👉 [http://localhost:8080](http://localhost:8080)

> All containers share the same Pod IP, so the web app can connect to MySQL via `localhost:3306`.

---

## 5️⃣ Connect to MySQL from Outside the Pod

You can port-forward the MySQL container port (3306) to your local machine:

```bash
kubectl port-forward pod/app-db-sidecar-pod 3306:3306 -c mysql-db
```

Then, use any MySQL client:

```bash
mysql -h 127.0.0.1 -P 3306 -u root -p
```

✅ This allows tools like **MySQL Workbench**, **DBeaver**, or `mysql` CLI to connect to the DB running inside Kubernetes.

---

# ⚡ Key Takeaways – Kubernetes Pods & Multi-Container Pods

- 🐳 **Pods** are the unit of deployment in Kubernetes.

- 🧩 **Multi-container pods** are useful for tightly coupled containers that need to work together.

- 🌐 **Containers share network & storage**, enabling patterns like:
  - Sidecar
  - Ambassador
  - Adapter

- 🖥️ Use **`kubectl exec`** and **port-forwarding** for debugging and accessing containers.
