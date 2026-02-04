# Kubernetes DaemonSets ☸️

Kubernetes is a **distributed system**, and platform administrators often need to run **platform-specific applications on all nodes** in a cluster.

For example:
- Running a **logging agent** on every node
- Running a **monitoring agent** on every node
- Running **networking or security components** on every node

This is where **DaemonSets** come into the picture.

A **DaemonSet** is a native Kubernetes object designed to run **system daemons**.

---

## 📌 The Importance of DaemonSets

In earlier Kubernetes concepts like **ReplicaSets** and **Deployments**, we define a `replicas` field to control how many Pods run.

Example:
```yaml
replicas: 3
````

Kubernetes might schedule:

* 3 NGINX Pods
* Each Pod potentially on a different node

### How DaemonSets are different

DaemonSets ensure that:

* **One copy of a Pod runs on every node**
* When a **new node is added**, a Pod is automatically created
* When a **node is removed**, the Pod is automatically deleted

This makes DaemonSets ideal for **node-level services** like logging and monitoring.

---

## 🧠 DaemonSet Examples ☸️

A **DaemonSet** ensures that **one Pod runs on every node** in the cluster —
similar to installing a required tool on **every computer in an office**.

---

### 📋 Common DaemonSet Use Cases

| Use Case           | Example Tool             | Layman Explanation                                                                                         |
| ------------------ | ------------------------ | ---------------------------------------------------------------------------------------------------------- |
| Log Collection 📝  | Fluentd, Filebeat        | Every server generates logs. A DaemonSet ensures **each node has a log collector**, so no logs are missed. |
| Monitoring 📊      | Prometheus Node Exporter | To monitor CPU, memory, and disk usage, **each node needs a monitoring agent**.                            |
| Security 🔐        | Falco, security scanners | Like antivirus software — DaemonSet ensures **every node is protected**.                                   |
| Networking 🌐      | Calico, Flannel (CNI)    | Pods need networking. DaemonSet runs networking software **on every node**.                                |
| Storage Helpers 💾 | CSI node plugins         | Each node needs help attaching disks. DaemonSet ensures **storage works everywhere**.                      |

---

## 🧩 Multiple DaemonSets

Based on requirements, we can deploy **multiple DaemonSets for the same type of daemon**, for example:

* Different CPU/memory requests
* Different flags
* Different hardware requirements

---

## 🚀 Deployment vs DaemonSet (In Short) ☸️

### 🔹 Deployment

* Runs a **fixed number of Pod replicas**
* Pods can run on **any node**
* Used for **applications and services**

**Best for:** Web apps, APIs, microservices

---

### 🔹 DaemonSet

* Runs **one Pod on every node**
* Automatically adds Pods when **new nodes join**
* Used for **node-level services**

**Best for:** Logging, monitoring, networking agents

---

### 🧠 Easy Memory Trick

* **Deployment** → App replicas
* **DaemonSet** → Pod on every node

---

## ⚙️ DaemonSet Behavior

* A DaemonSet ensures **one Pod per worker node**
* You **cannot manually scale** DaemonSet Pods
* If a DaemonSet Pod is deleted, the controller **automatically recreates it**

---

## 🧪 Example: Fluentd DaemonSet

We will deploy a **Fluentd logging agent** as a DaemonSet on all worker nodes.

### 📄 `daemonset.yaml`

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
  namespace: kube-system
spec:
  selector:
    matchLabels:
      name: fluentd
  template:
    metadata:
      labels:
        name: fluentd
    spec:
      containers:
      - name: fluentd
        image: quay.io/fluentd_elasticsearch/fluentd:latest
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
```

📌 No `replicas` field is provided because DaemonSet Pod count depends on **number of nodes**.

---

## 🚀 Deploy the DaemonSet

```bash
kubectl apply -f daemonset.yaml
```

---

## 🤔 Why Isn’t My DaemonSet in the `default` Namespace?

DaemonSets are **namespace-scoped resources**, just like Deployments.

If your YAML includes:

```yaml
namespace: kube-system
```

Then running:

```bash
kubectl get daemonset
```

will NOT show it.

### ✅ Correct command

```bash
kubectl get daemonset -A
```

---

## 🧩 What Is Running in the Cluster?

### 1️⃣ Fluentd DaemonSet Status

```bash
kubectl get ds -A
```

Example output:

```
NAMESPACE     NAME     DESIRED   CURRENT   READY
kube-system   fluentd  3         3         3
```

### Meaning:

* **DESIRED = 3** → 3 nodes
* **CURRENT = 3** → 3 Pods created
* **READY = 3** → All Pods are healthy

✅ Perfect DaemonSet state

---

## 🖥️ Proof: Pods Are Node-Specific

```bash
kubectl get pods -n kube-system -o wide | grep fluentd
```

Example:

```
fluentd-p9khx   Running   cka-qacluster-worker2
fluentd-tv4h6   Running   cka-qacluster-worker
fluentd-zh9hd   Running   cka-qacluster-worker3
```

👉 Exactly **one Fluentd Pod per node**

---

## 💥 Self-Healing in Action

Delete one Pod:

```bash
kubectl delete pod fluentd-p9khx -n kube-system
```

Check DaemonSet:

```bash
kubectl get ds -A
```

```
DESIRED=3  CURRENT=3  READY=2
```

### What happened?

* Kubernetes detected a missing Pod
* A new Pod was created
* Pod was not ready yet

---

## ⏳ After a Few Seconds

```bash
kubectl get ds -A -w
```

```
DESIRED=3  CURRENT=3  READY=3
```

✅ DaemonSet restored **one Pod per node**

---

## 🧠 The BIG Idea

### DaemonSet Rule

> **Every eligible node must always have exactly one Pod**

* Pod deleted ❌ → recreated 🔁
* Node added ➕ → new Pod created
* Node removed ➖ → Pod deleted

---

## 🔍 Why This Is Powerful

DaemonSets are perfect for:

* Logging agents
* Monitoring agents
* Networking components
* Security agents

Because:

* You never want a node without them
* Manual deletion does not break coverage

---

## 🧠 Final Mental Model

* **Deployment** → cares about number of Pods
* **DaemonSet** → cares about node coverage


☸️ **DaemonSet = One Pod per Node, Always**

---


# Why Is a DaemonSet Running Only on Worker Nodes?

🤔 Ever noticed that your DaemonSet Pods are running on worker nodes but **not** on the master (control-plane) node?

### Short Answer

👉 The **control-plane node is tainted**  
👉 Your **DaemonSet does NOT tolerate that taint**

Because of this, Kubernetes intentionally prevents Pods from being scheduled on the control-plane node.

---

## Understanding the Root Cause

By default, Kubernetes applies a taint to the control-plane node to protect it from running regular workloads.

This taint looks like:

```

node-role.kubernetes.io/control-plane:NoSchedule

````

Unless a Pod explicitly **tolerates** this taint, it will not be scheduled on the control-plane node.

---

## Inspect Node Taints

To see taints applied to all nodes in your cluster, run:

```bash
kubectl get nodes -o custom-columns=NAME:.metadata.name,TAINTS:.spec.taints
````

### Example Output

```bash
NAME                          TAINTS
cka-qacluster-control-plane   [map[effect:NoSchedule key:node-role.kubernetes.io/control-plane]]
cka-qacluster-worker          <none>
cka-qacluster-worker2         <none>
cka-qacluster-worker3         <none>
```

🔍 As you can see:

* The **control-plane node** has a `NoSchedule` taint
* **Worker nodes** have no taints, so Pods can run freely on them

---

## Why Kubernetes Does This

This behavior is **intentional** and helps ensure:

* Stability of the control-plane
* Protection from resource-heavy workloads
* Better separation between system components and user workloads

---

## Advantages of Using DaemonSets

DaemonSets are ideal for workloads that must run on **every node** (or a specific subset of nodes).

### Key Benefits

### 🚀 Automatic Scaling

DaemonSets automatically add or remove Pods as nodes join or leave the cluster.

### 🛠 Centralized Management

Easily monitor and manage Pods using standard Kubernetes tools like `kubectl`.

### 🔄 Resilience

If a Pod is deleted or a node fails, Kubernetes automatically recreates the Pod.

---

## When to Use DaemonSets

DaemonSets are perfect for system-level and node-level services such as:

* Log collectors
* Monitoring agents
* Networking components
* Security agents

They ensure consistent behavior across your cluster with minimal manual effort.



Here’s your content rewritten cleanly and consistently as a **`README.md`**, polished but keeping the **same meaning and flow** 👌
Ready to copy-paste into your repo.

---

# Kubernetes Jobs ☸️

A **Job** in Kubernetes is used to run **short-lived, one-time, or batch tasks**.  
Unlike **Deployments**, which run continuously, Jobs are designed for workloads that **start, complete, and then stop**.

---

## What Is a Kubernetes Job?

A Job creates one or more Pods and ensures that a **specified number of them successfully complete**.  
If a Pod fails, Kubernetes automatically retries it until the Job succeeds or reaches its retry limit.

---

## Key Points About Jobs

### 1️⃣ Short-Lived Pods
- Jobs run Pods that perform a task and then exit.
- Common use cases include:
  - Batch data processing
  - Sending email notifications
  - Database migrations
  - One-time scripts

---

### 2️⃣ Ensures Completion
- Kubernetes tracks the Pods created by a Job.
- A Job is considered successful only when the required number of Pods complete successfully.
- If a Pod fails due to:
  - Application crash
  - Node failure  
  Kubernetes will **automatically retry** the Pod.

---

### 3️⃣ Pod Completion Behavior
- Jobs can be configured to run:
  - **A single Pod** → one-time task
  - **Multiple Pods in parallel** → parallel processing
- The Job is marked **Completed** only after all required Pods succeed.

---

### 4️⃣ Automatic Cleanup
- Deleting a Job also deletes all Pods created by it.
- Jobs can be **suspended**, which stops active Pods until the Job is resumed.

---

## How Kubernetes Jobs Work

1. You define a Job using a YAML manifest.
2. Kubernetes creates Pods based on the Job specification.
3. Pods execute the task and exit.
4. Kubernetes monitors Pod success or failure.
5. The Job is marked **Completed** when the success criteria are met.

---

## Example: Single Pod Job

### Scenario
- Runs a Job in the `kube-system` namespace.
- Uses the image `devopscube/kubernetes-job-demo:latest`.
- The container prints numbers up to `100` and then exits.
- If the Pod fails, Kubernetes retries automatically.

---

## Job Manifest Example

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: kubernetes-job-example
  namespace: kube-system   # Runs in kube-system namespace
  labels:
    jobgroup: jobexample
spec:
  template:
    metadata:
      name: kubejob
      labels:
        jobgroup: jobexample
    spec:
      containers:
      - name: c
        image: devopscube/kubernetes-job-demo:latest
        args: ["100"]
      restartPolicy: OnFailure
````

---

## Apply the Job

```bash
kubectl apply -f job.yaml
```

---

## Check Job Status (All Namespaces)

```bash
kubectl get jobs -A
```

### Example Output

```bash
NAMESPACE     NAME                     STATUS     COMPLETIONS   DURATION   AGE
kube-system   kubernetes-job-example   Complete   1/1           4m29s      4m43s
```

---

## Check Pods Created by the Job

```bash
kubectl get pods -n kube-system --selector=job-name=kubernetes-job-example
```

### Example Output

```bash
NAME                           READY   STATUS      RESTARTS   AGE
kubernetes-job-example-dmdnv   0/1     Completed   0          5m18s
```

---

## View Job Logs

```bash
kubectl logs kubernetes-job-example-dmdnv -n kube-system
```

### Sample Output

```text
This Job will echo message 100 times
1] Hey I will run till the job completes.
2] Hey I will run till the job completes.
...
100] Hey I will run till the job completes.
```

Once the task reaches `100`, the Pod exits and the Job completes successfully.

---

## Key Concepts in Kubernetes Jobs

| Concept            | Description                                           |
| ------------------ | ----------------------------------------------------- |
| **Pod Template**   | Defines the Pod that performs the actual work         |
| **Completions**    | Number of successful Pods required for Job completion |
| **Parallelism**    | Maximum number of Pods running at the same time       |
| **Restart Policy** | Defines how Pods behave on failure or completion      |

---

## Jobs vs CronJobs

* **Jobs** → One-time or batch tasks
* **CronJobs** → Scheduled and repetitive tasks (like cron)

📌 Kubernetes Jobs are ideal for **one-time executions, batch processing, and parallel workloads**, ensuring reliable completion even in failure scenarios.


---

# Kubernetes CronJobs ☸️

A **CronJob** in Kubernetes enables the scheduling and automation of **recurring tasks** based on a cron-like schedule.  
CronJobs provide a declarative way to define **when and how tasks should run repeatedly**.  

They are especially useful for automating:

- Periodic maintenance tasks  
- Data synchronization between systems  
- Scheduled backups  
- Reporting or aggregation tasks  

---

## Key Concepts in Kubernetes CronJobs

| Concept                   | Description                                                                 |
|---------------------------|-----------------------------------------------------------------------------|
| **Schedule**              | Specifies the time and frequency at which the CronJob should run.           |
| **Job Template**           | Defines the Pods that will perform the actual task (similar to Jobs).       |
| **Concurrency Policy**     | Controls how overlapping executions are handled (`Allow`, `Forbid`, `Replace`). |
| **Starting Deadline Seconds** | Maximum time a job can start after the scheduled time before considered missed. |

---

## Use Cases for Kubernetes CronJobs

- Scheduled backups or database snapshots  
- Regular system maintenance (log rotation, cleanup)  
- Periodic data aggregation or reporting  
- Automated scaling or resource management based on time  

---

## Jobs vs CronJobs Comparison

| Feature                  | Kubernetes Jobs                                         | Kubernetes CronJobs                                          |
|---------------------------|--------------------------------------------------------|-------------------------------------------------------------|
| **Purpose**               | Run a task once or a set number of times              | Schedule tasks to run at specific times or intervals       |
| **When It Runs**          | Runs immediately when created                          | Runs based on a specified schedule (cron syntax)           |
| **Use Case**              | One-time tasks like backups or batch processing       | Recurring tasks like daily reports or maintenance scripts  |
| **How to Define**         | `Job` resource in YAML                                  | `CronJob` resource in YAML                                  |
| **Execution Control**     | Runs until it completes or fails                        | Creates Jobs automatically according to the schedule       |
| **Scheduling**            | Not scheduled; starts immediately                       | Uses cron syntax (`*/5 * * * *`) to define the schedule   |
| **Retries**               | Can retry failed tasks a set number of times           | Jobs it creates inherit retry policies                     |
| **Concurrency**           | Single instance or multiple in parallel                | Can control overlapping jobs (`Allow`, `Forbid`, `Replace`)|
| **Pausing**               | Cannot be paused once started                           | Can pause to temporarily stop creating new jobs           |
| **Cleanup**               | Can delete Pods automatically after completion         | Can limit how many old jobs are retained                  |

---

## How to Create a Kubernetes CronJob

A CronJob defines a **schedule** and a **job template** that runs repeatedly.  

### Example: Run a Job Daily at 2 AM

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: example-cronjob
spec:
  schedule: "0 2 * * *"   # Runs daily at 2 AM
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            command: ["echo", "Hello, World!"]
          restartPolicy: OnFailure
````

---

### Deploy the CronJob

```bash
kubectl create -f cronjobs.yaml
```

---

### List CronJobs

```bash
kubectl get cronjobs
```

---

### Monitor Jobs Created by CronJob

```bash
kubectl get jobs --watch
```

---

### Check Logs

1. List Pods created by the CronJob:

```bash
kubectl get pods --selector=job-name=example-cronjob
```

2. View logs of a Pod:

```bash
kubectl logs <pod-name>
```

3. Check CronJob schedule and status:

```bash
kubectl describe cronjob example-cronjob | grep Schedule
```

---

## Summary

* **Jobs** → Run tasks **once** or a specific number of times
* **CronJobs** → Run tasks **repeatedly** according to a schedule

CronJobs are perfect for **automated maintenance, periodic backups, and scheduled reporting**, while Jobs are ideal for **one-time batch processing**.



---

## Summary

DaemonSet → Ensures a Pod runs on all (or selected) nodes. Great for system services.

Job → Runs a Pod/task once or a fixed number of times. Ensures completion and retries.

CronJob → Automates recurring Jobs using a cron schedule. Ideal for periodic tasks and maintenance.

These resources allow Kubernetes to handle node-level services, one-time tasks, and scheduled tasks efficiently.


