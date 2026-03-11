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

Check  All Services

```bash
$ kubectl get all -n monitoring
NAME                                                         READY   STATUS    RESTARTS        AGE
pod/alertmanager-monitoring-kube-prometheus-alertmanager-0   2/2     Running   0               4h21m
pod/monitoring-grafana-544b666bdf-dgss2                      3/3     Running   0               4h22m
pod/monitoring-kube-prometheus-operator-c946f467d-w2kpb      1/1     Running   11 (40m ago)    4h22m
pod/monitoring-kube-state-metrics-57f5f46ddb-5v869           1/1     Running   7 (2m17s ago)   4h22m
pod/monitoring-prometheus-node-exporter-2nmwp                1/1     Running   8               4h22m
pod/monitoring-prometheus-node-exporter-5pf8f                1/1     Running   2               4h22m
pod/monitoring-prometheus-node-exporter-fq8pz                1/1     Running   1 (72m ago)     4h22m
pod/prometheus-monitoring-kube-prometheus-prometheus-0       2/2     Running   0               4h21m

NAME                                              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
service/alertmanager-operated                     ClusterIP   None            <none>        9093/TCP,9094/TCP,9094/UDP   4h21m
service/monitoring-grafana                        NodePort    10.96.19.233    <none>        3000:31807/TCP               4h22m
service/monitoring-kube-prometheus-alertmanager   ClusterIP   10.96.112.250   <none>        9093/TCP,8080/TCP            4h22m
service/monitoring-kube-prometheus-operator       ClusterIP   10.96.84.71     <none>        443/TCP                      4h22m
service/monitoring-kube-prometheus-prometheus     ClusterIP   10.96.15.126    <none>        9090/TCP,8080/TCP            4h22m
service/monitoring-kube-state-metrics             ClusterIP   10.96.190.152   <none>        8080/TCP                     4h22m
service/monitoring-prometheus-node-exporter       ClusterIP   10.96.229.71    <none>        9100/TCP                     4h22m
service/prometheus-operated                       ClusterIP   None            <none>        9090/TCP                     4h21m

NAME                                                 DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/monitoring-prometheus-node-exporter   3         3         3       3            3           kubernetes.io/os=linux   4h22m

NAME                                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/monitoring-grafana                    1/1     1            1           4h22m
deployment.apps/monitoring-kube-prometheus-operator   1/1     1            1           4h22m
deployment.apps/monitoring-kube-state-metrics         1/1     1            1           4h22m

NAME                                                            DESIRED   CURRENT   READY   AGE
replicaset.apps/monitoring-grafana-544b666bdf                   1         1         1       4h22m
replicaset.apps/monitoring-kube-prometheus-operator-c946f467d   1         1         1       4h22m
replicaset.apps/monitoring-kube-state-metrics-57f5f46ddb        1         1         1       4h22m

NAME                                                                    READY   AGE
statefulset.apps/alertmanager-monitoring-kube-prometheus-alertmanager   1/1     4h22m
statefulset.apps/prometheus-monitoring-kube-prometheus-prometheus       1/1     4h21m

Dubbakas@Dubbakas MINGW64 /d/Docker & k8s 2026
$

```

## 3️⃣ Port-Forward Services Locally

### Prometheus

```bash
kubectl port-forward svc/monitoring-kube-prometheus-prometheus 9090:9090 -n monitoring
```

Open in browser: [http://localhost:9090](http://localhost:9090)
Query metrics with **PromQL**.

Then navigate to the **Graph** tab and run the PromQL queries.

---

## 🔎 Common PromQL Queries

| Purpose                        | Query                                                                                       | Notes                                        |
| ------------------------------ | ------------------------------------------------------------------------------------------- | -------------------------------------------- |
| **CPU usage per node**         | `rate(node_cpu_seconds_total{mode!="idle"}[5m])`                                            | Shows CPU usage excluding idle time.         |
| **Memory usage**               | `node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes`                               | Displays current memory consumption.         |
| **Disk usage**                 | `node_filesystem_size_bytes{fstype!="tmpfs"} - node_filesystem_free_bytes{fstype!="tmpfs"}` | Tracks disk space used on nodes.             |
| **Pod restarts**               | `rate(kube_pod_container_status_restarts_total[5m])`                                        | Helps detect unstable pods or crash loops.   |
| **HTTP request rate**          | `rate(http_requests_total[5m])`                                                             | Requests per second over the last 5 minutes. |
| **Error rate (non-2xx)**       | `rate(http_requests_total{status!~"2.."}[5m])`                                              | Filters out successful HTTP responses.       |
| **Node uptime**                | `node_time_seconds - node_boot_time_seconds`                                                | Shows how long each node has been running.   |
| **Prometheus scrape duration** | `rate(prometheus_target_interval_length_seconds_sum[5m])`                                   | Helps detect scrape performance problems.    |

## Notes

* These queries are useful for **basic cluster monitoring and troubleshooting**.
* You can also use the same PromQL queries inside **Grafana dashboards** for visualization.
* Metrics are typically collected via **node-exporter**, **kube-state-metrics**, and other monitoring components in the Kubernetes cluster.

---

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
