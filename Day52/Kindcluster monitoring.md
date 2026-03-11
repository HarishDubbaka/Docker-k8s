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

$ kubectl get pods -n monitoring
NAME                                                     READY   STATUS    RESTARTS       AGE
alertmanager-monitoring-kube-prometheus-alertmanager-0   2/2     Running   0              4h21m
monitoring-grafana-544b666bdf-dgss2                      3/3     Running   0              4h21m
monitoring-kube-prometheus-operator-c946f467d-w2kpb      1/1     Running   11 (39m ago)   4h21m
monitoring-kube-state-metrics-57f5f46ddb-5v869           1/1     Running   7 (81s ago)    4h21m
monitoring-prometheus-node-exporter-2nmwp                1/1     Running   8              4h21m
monitoring-prometheus-node-exporter-5pf8f                1/1     Running   2              4h21m
monitoring-prometheus-node-exporter-fq8pz                1/1     Running   1 (71m ago)    4h21m
prometheus-monitoring-kube-prometheus-prometheus-0       2/2     Running   0              4h20m

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


Dubbakas@Dubbakas MINGW64 /d/Docker & k8s 2026/moni+log
$ kubectl get svc -n monitoring
NAME                                      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
alertmanager-operated                     ClusterIP   None            <none>        9093/TCP,9094/TCP,9094/UDP   3h11m
monitoring-grafana                        NodePort    10.96.19.233    <none>        3000:31807/TCP               3h11m
monitoring-kube-prometheus-alertmanager   ClusterIP   10.96.112.250   <none>        9093/TCP,8080/TCP            3h11m
monitoring-kube-prometheus-operator       ClusterIP   10.96.84.71     <none>        443/TCP                      3h11m
monitoring-kube-prometheus-prometheus     ClusterIP   10.96.15.126    <none>        9090/TCP,8080/TCP            3h11m
monitoring-kube-state-metrics             ClusterIP   10.96.190.152   <none>        8080/TCP                     3h11m
monitoring-prometheus-node-exporter       ClusterIP   10.96.229.71    <none>        9100/TCP                     3h11m
prometheus-operated                       ClusterIP   None            <none>        9090/TCP                     3h11m

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
