# Kubernetes Resource Requests and Limits

Imagine workloads fighting for limited CPU or memory during peak times, or idle resources sitting unused while costs continue to rise. Mismanaging these resources can lead to cascading failures, reduced performance, and inflated bills.

This is where **Kubernetes resource requests and limits** come in. By defining the minimum (requests) and maximum (limits) resources a container can consume, you can allocate resources intelligently, ensuring that your cluster remains **stable, efficient, and cost-effective**.

---

## Why Requests and Limits Matter

Properly configured requests and limits:

* **Prevent Service Outages:** Guarantee resources for critical applications while keeping runaway workloads in check.
* **Optimize Costs:** Avoid paying for unused resources while maintaining performance.
* **Enable Smart Scheduling:** Help the Kubernetes scheduler distribute workloads efficiently across nodes.

**Misconfigured requests and limits can cause:**

* Requests set too high → underutilized resources and higher costs.
* Limits set too low → applications crash or are throttled under load.
* Omitting requests → unpredictable scheduling and resource contention.

---

## Understanding Requests and Limits

### Requests

A request specifies the **minimum resources Kubernetes guarantees** for a container. The scheduler uses this to ensure the container can always operate within its reserved capacity.

**Example:**

```yaml
resources:
  requests:
    cpu: "500m"   # 0.5 CPU cores reserved
```

### Limits

A limit sets the **maximum resources** a container can consume. Kubernetes enforces this using CPU throttling or memory termination.

**Example:**

```yaml
resources:
  limits:
    memory: "1Gi"   # Container is terminated if it exceeds 1 GiB
```

---

## Importance of Requests and Limits

Requests and limits help:

* Prevent resource hogging and starvation.
* Improve scheduling decisions for better performance.
* Protect cluster health by preventing any single container from consuming excessive resources.

Together, they provide the framework for **multi-tenant resource management**.

---

## Setting Up Kubernetes Limits

1. **Define a Pod manifest**: Specify `requests` and `limits` under the `resources` section of each container.
2. **Specify CPU and memory**:

   * CPU: in CPU units (1 = 1 CPU core)
   * Memory: in bytes, using `Mi` or `Gi` for readability

**Example: Pod within limits (`withinlimit.yml`)**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: memory-demo
  namespace: mem-example
spec:
  containers:
    - name: memory-demo-ctr
      image: polinux/stress
      resources:
        requests:
          memory: "100Mi"
        limits:
          memory: "200Mi"
      command: ["stress"]
      args: ["--vm", "1", "--vm-bytes", "150M", "--vm-hang", "1"]
```

Apply the pod:

```bash
kubectl apply -f withinlimit.yml
kubectl get pods -n mem-example
kubectl top pod -n mem-example
```

**Command & Args Explanation:**

* `command: ["stress"]` → Runs the stress tool
* `args: ["--vm", "1", "--vm-bytes", "150M", "--vm-hang", "1"]`

  * `--vm 1` → Start 1 memory worker
  * `--vm-bytes 150M` → Allocate 150 MB memory
  * `--vm-hang 1` → Keep worker running

---

## Example: Exceeding Memory Limit

**Pod YAML (`exceedlimit.yml`)**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: memory-exceed-limit
  namespace: mem-example
spec:
  containers:
    - name: memory-demo-ctr
      image: busybox
      resources:
        requests:
          memory: "50Mi"
        limits:
          memory: "100Mi"
      command: ["sh", "-c"]
      args:
        - "echo 'Allocating memory to exceed limit'; dd if=/dev/zero of=/tmp/mem bs=1M count=150; sleep 3600"
```

**Behavior:**

* Requests 50Mi but tries to allocate 150Mi → exceeds 100Mi limit
* Kubernetes kills the pod → **OOMKilled**

---

## Common Mistakes

### 1. Over-Provisioning

Allocating excessive resources leads to high costs.

```yaml
resources:
  requests:
    cpu: "1"
    memory: "512Mi"
  limits:
    cpu: "2"
    memory: "1Gi"
```

**Avoid by:** Using monitoring tools and actual usage data.

---

### 2. Under-Provisioning

Requests set too low can cause:

* Pod evictions
* CPU throttling and degraded performance

```yaml
resources:
  requests:
    memory: "300Mi"
    cpu: "250m"
  limits:
    memory: "600Mi"
    cpu: "500m"
```

**Avoid by:** Setting requests based on baseline usage.

---

### 3. Ignoring Requests

Failing to set requests leads to unpredictable scheduling.

```yaml
resources:
  requests:
    memory: "256Mi"
    cpu: "250m"
  limits:
    memory: "512Mi"
    cpu: "500m"
```

---

## Best Practices

1. **Use Resource Quotas**:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: team-quota
  namespace: dev-team
spec:
  hard:
    requests.cpu: "4"
    requests.memory: "8Gi"
    limits.cpu: "8"
    limits.memory: "16Gi"
```

2. **Regularly Profile Workloads**
   Use Prometheus, Grafana, or Goldilocks for monitoring and optimization.

3. **Automate Scaling**

   * **HPA** → scales replicas based on CPU/memory
   * **VPA** → adjusts pod requests/limits automatically

**Example HPA:**

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: example-hpa
  namespace: default
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: example-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 75
```

**Example VPA:**

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: example-vpa
  namespace: default
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: example-app
  updatePolicy:
    updateMode: "Auto"
```

4. **Periodic Reviews**
   Regularly profile and adjust resources as workloads evolve.

---

## Final Thoughts

Kubernetes resource requests and limits are essential for **efficient, stable, and cost-effective clusters**.

* Set thoughtful requests and limits
* Monitor usage
* Adjust over time

…to create a cluster that supports applications **without waste or instability**.

**Remember:** Kubernetes is flexible — use resource management wisely to keep workloads performant and costs under control.

---

