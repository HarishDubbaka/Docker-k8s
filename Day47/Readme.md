# Kubernetes Operators

Kubernetes offers limited initial functionality to ensure flexibility and scalability. **K8s Operators** are software extensions that make use of Kubernetes APIs to extend behavior.

This document explains how Operators work and their benefits.

---

# Introduction

Kubernetes has an ace up its sleeve that makes it even more useful, powerful, and flexible. That ace is called an **Operator**.

Operators exist because Kubernetes was designed from the beginning for **automation**, which is built directly into the very heart of the software.

Kubernetes allows users to automate:

* Deployment of workloads
* Execution of workloads
* Cluster behavior and operations

Operators enable **complex clusters and systems to operate automatically** based on patterns and principles.

In short:

> Operators are patterns that extend the behavior of the cluster without changing Kubernetes core code.

They act as **custom resource controllers using Kubernetes APIs**.

---

# Why Kubernetes Needs Operators

Kubernetes follows two main principles:

1. **Simplicity and flexibility**
2. **Automation**

These strengths make Kubernetes scalable and usable across many environments.

However, Kubernetes initially exposes only a limited set of commands and APIs. To perform more complex operations, additional automation is required.

This is where **Operators** come in.

Operators:

* Manage application logic
* Work as controllers in the Kubernetes control plane
* Continuously compare:

  * **Actual cluster state**
  * **Desired cluster state**

If the states differ, the Operator **reconciles them automatically**.

---

# What is a Kubernetes Operator?

The **Operator concept was introduced in 2016** by the CoreOS Linux development team.

The Kubernetes project defines Operators as:

> "Operators are software extensions that use custom resources to manage applications and their components."

Operators automate both:

### Day-1 Operations

* Installation
* Configuration
* Initial deployment

### Day-2 Operations

* Reconfiguration
* Scaling
* Upgrades
* Backups
* Failover
* Recovery

Operators run **inside the Kubernetes cluster** and integrate with Kubernetes APIs.

---

# How Operators Work

Operators use:

### Custom Resource Definitions (CRDs)

CRDs extend the Kubernetes API to define new resource types.

### Custom Resources (CR)

Custom resources define the **desired state of an application**.

Example components:

```
Custom Resource Definition (CRD)
        ↓
Custom Resource (CR)
        ↓
Operator Controller
        ↓
Cluster State Reconciliation
```

The Operator continuously runs a **control loop**:

1. Watches CR objects
2. Checks actual state
3. Reconciles differences
4. Creates/updates Kubernetes resources

Possible actions:

* Deploy applications
* Scale workloads
* Restart failed components
* Update configurations

---

# Purpose and Function

Operators behave like **automated system administrators**.

You define the **desired state**, and the Operator ensures the system always remains in that state.

Operators follow the **declarative model**:

* User declares desired configuration
* Operator ensures the cluster matches that configuration

---

# Common Tasks Automated by Operators

Operators can automate many operations including:

* Deploy applications on demand
* Backup application state
* Restore from backups
* Perform rolling upgrades
* Manage dependencies
* Update configuration and databases
* Expose services for non-Kubernetes applications

---

# Kubernetes Operator Examples

There are two major platforms to discover operators:

### Artifact Hub

CNCF project that hosts:

* Operators
* Helm Charts

### Operator Hub

Red Hat project dedicated to Kubernetes Operators.

---

# Example: Prometheus Operator

One of the most popular operators is the **Prometheus Operator**, used for monitoring Kubernetes clusters.

OperatorHub link:

[https://operatorhub.io/operator/prometheus](https://operatorhub.io/operator/prometheus)

The **kube-prometheus stack** includes:

* Prometheus
* Grafana dashboards
* Alert rules
* Kubernetes manifests

It provides **end-to-end monitoring for Kubernetes clusters**.

---

# Install Operator Lifecycle Manager (OLM)

OLM manages the lifecycle of Operators.

Install OLM:

```bash
curl -sL https://github.com/operator-framework/operator-lifecycle-manager/releases/download/v0.41.0/install.sh | bash -s v0.41.0
```

This will:

* Create namespace `olm`
* Deploy OLM components

---

# Install Prometheus Operator

```bash
kubectl create -f https://operatorhub.io/install/prometheus.yaml
```

The operator will be installed in the **operators namespace**.

Check installation:

```bash
kubectl get csv -n operators
```

Example output:

```
NAME                         DISPLAY               VERSION   PHASE
prometheusoperator.v0.70.0   Prometheus Operator   0.70.0    Succeeded
```

---

# Verify OLM Components

Check running pods:

```bash
kubectl get pods -n olm
```

Ensure these pods are running:

* olm-operator
* catalog-operator
* packageserver
* operatorhub catalog

---

# Verify CRDs Installed

```bash
kubectl get crd | grep monitoring.coreos.com
```

Example:

```
alertmanagerconfigs.monitoring.coreos.com
alertmanagers.monitoring.coreos.com
podmonitors.monitoring.coreos.com
prometheuses.monitoring.coreos.com
servicemonitors.monitoring.coreos.com
```

These CRDs were installed by the **Prometheus Operator**.

---

# Step 1: Create a Prometheus Custom Resource

Create `prometheus-cr.yaml`

```yaml
apiVersion: monitoring.coreos.com/v1
kind: Prometheus
metadata:
  name: example-prometheus
spec:
  replicas: 1
  serviceMonitorSelector: {}
  resources:
    requests:
      memory: 400Mi
```

Apply:

```bash
kubectl apply -f prometheus-cr.yaml
```

Operator will automatically:

* Deploy Prometheus
* Configure it
* Manage the instance

Verify:

```bash
kubectl get pods
```

Example:

```
prometheus-example-prometheus-0   Running
```

---

# Step 2: Deploy a Sample Application

Create `app_service.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: demo-app
  template:
    metadata:
      labels:
        app: demo-app
    spec:
      containers:
      - name: demo-app
        image: 970371/my-laxmiwebapp
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: demo-app
  labels:
    app: demo-app
spec:
  selector:
    app: demo-app
  ports:
  - name: http
    port: 8080
    targetPort: 8080
  type: ClusterIP
```

Apply:

```bash
kubectl apply -f app_service.yaml
```

---

# Step 3: Create ServiceMonitor

Prometheus Operator uses **ServiceMonitor** to discover services.

Create `servicemonitor.yaml`

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: demo-app-monitor
  labels:
    release: prometheus
spec:
  selector:
    matchLabels:
      app: demo-app
  endpoints:
  - port: http
    path: /metrics
    interval: 15s
```

Apply:

```bash
kubectl apply -f servicemonitor.yaml
```

Verify:

```bash
kubectl get servicemonitor
```

---

# Step 4: Access Prometheus UI

Port forward:

```bash
kubectl port-forward svc/prometheus-operated 9090
```

Open browser:

```
http://localhost:9090
```

Check:

```
Status → Targets
```

You should see:

```
demo-app-monitor   UP
```

This means Prometheus is scraping metrics.

---

# Step 5: Query Metrics

Example query:

```
kube_pod_status_phase
```

Filter running pods:

```
kube_pod_status_phase{phase="Running"}
```

---

# Step 6: Create Alert Rule

Create `alert-rule.yaml`

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: pod-alerts
spec:
  groups:
  - name: pod.rules
    rules:
    - alert: PodDown
      expr: kube_pod_status_phase{phase="Running"} == 0
      for: 1m
      labels:
        severity: critical
      annotations:
        description: Pod is not running
        summary: Pod failure detected
```

Apply:

```bash
kubectl apply -f alert-rule.yaml
```

Verify:

```bash
kubectl get prometheusrules
```

---

# Step 7: Install Alertmanager (Optional)

Alertmanager sends alerts to:

* Slack
* Email
* PagerDuty
* Microsoft Teams

Alert flow:

```
Pod Down
   ↓
Prometheus Alert
   ↓
Alertmanager
   ↓
Slack / Email Notification
```

---

# Final Architecture

```
Application Pod
      │
      ▼
Service
      │
      ▼
ServiceMonitor
      │
      ▼
Prometheus Operator
      │
      ▼
Prometheus
      │
      ▼
Alert Rules
      │
      ▼
Alertmanager
```

---

