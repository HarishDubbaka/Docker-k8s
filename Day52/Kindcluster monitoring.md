# Kind Cluster Monitoring Setup (Prometheus + Grafana + Loki)

This guide shows how to set up a **Kubernetes monitoring stack** on a local **kind** cluster using `kube-prometheus-stack` (Prometheus + Grafana) and Loki for logs.

---

## 1️⃣ Create a KIND Cluster

Create a `kind-monitoringconfig.yaml`:

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
    extraPortMappings:
      - containerPort: 30000
        hostPort: 30000
        protocol: TCP
  - role: worker
  - role: worker
````

Create the cluster:

```bash
kind create cluster --config kind-monitoringconfig.yaml --name monitoring-cluster
```

---

## 2️⃣ Install Monitoring (Prometheus + Grafana)

### Step A: Create Namespace

Create `monitoring-namespace.yaml`:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: monitoring
```

Apply it:

```bash
kubectl apply -f monitoring-namespace.yaml
```

---

### Step B: Render and Apply Prometheus Manifests

Use Helm to render YAML manifests:

```bash
helm template monitoring prometheus-community/kube-prometheus-stack \
  -n monitoring \
  -f kube-prometheus-values.yaml > manifests.yaml
```

Apply manifests:

```bash
kubectl apply -f manifests.yaml
```

---

### Step C: Verify Installation

Check pods:

```bash
kubectl get pods -n monitoring
```

You should see components like:

* **Prometheus server**
* **Grafana**
* **Alertmanager**
* **Node Exporters**
* **kube-state-metrics**

Check services:

```bash
kubectl get svc -n monitoring
```

Example output:

| Name                                  | Type      | Cluster-IP   | Port(s)        |
| ------------------------------------- | --------- | ------------ | -------------- |
| monitoring-grafana                    | NodePort  | 10.96.19.233 | 3000:31807/TCP |
| monitoring-kube-prometheus-prometheus | ClusterIP | 10.96.15.126 | 9090/TCP       |
| prometheus-operated                   | ClusterIP | None         | 9090/TCP       |

---

## 3️⃣ Port-Forward Services Locally

### Prometheus

```bash
kubectl port-forward svc/monitoring-kube-prometheus-prometheus 9090:9090 -n monitoring
```

Open in browser: [http://localhost:9090](http://localhost:9090)
Query metrics with **PromQL**.

### Grafana

```bash
kubectl port-forward svc/monitoring-grafana 3000:3000 -n monitoring
```

Open in browser: [http://localhost:3000](http://localhost:3000)

Login credentials:

```
username: admin
password: prom-operator
```

### Loki (Logs)

```bash
kubectl port-forward svc/loki 3100:3100 -n logging
```

Open in browser: [http://localhost:3100](http://localhost:3100)

---

## 4️⃣ Add Data Sources in Grafana

### Prometheus Data Source

* URL: `http://monitoring-kube-prometheus-prometheus.monitoring.svc.cluster.local:9090`
* Test & Save

### Loki Data Source

* URL: `http://loki.logging.svc.cluster.local:3100`
* Test & Save

---

## 5️⃣ Access Dashboards

| Service    | Port-forward command                                                                     | Local URL                                      |
| ---------- | ---------------------------------------------------------------------------------------- | ---------------------------------------------- |
| Prometheus | `kubectl port-forward svc/monitoring-kube-prometheus-prometheus 9090:9090 -n monitoring` | [http://localhost:9090](http://localhost:9090) |
| Grafana    | `kubectl port-forward svc/monitoring-grafana 3000:3000 -n monitoring`                    | [http://localhost:3000](http://localhost:3000) |
| Loki       | `kubectl port-forward svc/loki 3100:3100 -n logging`                                     | [http://localhost:3100](http://localhost:3100) |

---

💡 **Tip:**
You can now create dashboards in Grafana to monitor CPU, memory, pod status, and logs from Loki, all in one place.

---
