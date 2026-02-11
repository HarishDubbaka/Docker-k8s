# Vertical Pod Autoscaling (VPA)

## Overview

The **Vertical Pod Autoscaler (VPA)** in Kubernetes automatically adjusts **CPU and memory requests** for pods to ensure that resource allocation matches actual usage.

Unlike the **Horizontal Pod Autoscaler (HPA)**, which scales by adding or removing pods, VPA focuses on modifying the **resource limits of individual pods**. This is particularly useful for workloads that experience temporary spikes in resource demand or cannot be easily scaled horizontally.

---

## How VPA Works

VPA operates through **three key components**:

1. **Recommender**  
   Continuously monitors current and historical resource utilization to compute optimal CPU and memory recommendations.

2. **Updater**  
   Identifies pods running with outdated resource configurations and evicts them so updated limits can be applied.

3. **Admission Controller**  
   Uses a mutating webhook to inject recommended resource requests into pods during creation.

> Kubernetes does not support changing resource limits for running pods.  
> VPA terminates pods with outdated configurations, and when they are recreated by their controllers, the updated limits are applied.

VPA can also run in **recommendation mode**, where it suggests optimal resource values **without restarting pods**.

---

## When to Use VPA

VPA is commonly used for:

- Backend services with stable workloads  
- Applications with fluctuating CPU or memory needs  
- Environments where resource planning is complex  
- Scenarios where manual tuning is error-prone  

For teams operating across regions or clouds, VPA provides **baseline resource automation**.

---

## Key Limitations of VPA

### 1. Pod Restarts Cause Disruption
VPA applies changes by restarting pods, which may cause downtime for critical or stateful applications.

### 2. Conflicts with HPA
If HPA and VPA both scale using CPU or memory metrics, they can interfere and cause over-scaling.

### 3. Limited Metrics Scope
VPA only considers CPU and memory, ignoring I/O, network, and application-level signals.

### 4. Short Historical Window
Typically analyzes only hours to a few days of data, missing long-term or seasonal trends.

### 5. No Awareness of Cluster Architecture
VPA may recommend resources that exceed node capacity, leaving pods stuck in `Pending`.

### 6. Poor StatefulSet Support
Stateful workloads require careful orchestration, which VPA’s restart model doesn’t handle well.

### 7. Not Suitable for Real-Time Scaling
Because changes require restarts, VPA reacts slowly to sudden traffic spikes.

### 8. Complexity and Tuning Overhead
Production usage requires testing, tuning, and continuous monitoring.

> These challenges represent real engineering trade-offs.  
> Pod restarts can cause downtime, missed SLAs, and operational friction, especially at scale.

---

## Best Practices for Running VPA Effectively

- Run VPA in **recommendation mode** initially  
- Separate responsibilities:
  - VPA → CPU & memory tuning  
  - HPA → traffic or custom business metrics  
- Use Pod Disruption Budgets for availability protection  
- Set reasonable initial resource requests  
- Monitor using Prometheus and Grafana  
- Test thoroughly in staging environments  
- Use `LimitRange` and `ResourceQuota` to cap recommendations  

---

## Installing the VPA Components

Clone the official Kubernetes Autoscaler repository:

```bash
git clone https://github.com/kubernetes/autoscaler.git
cd autoscaler/vertical-pod-autoscaler
./hack/vpa-up.sh
````

This deploys:

* Recommender
* Updater
* Admission Controller

Verify deployments:

```bash
kubectl get deployments
```

Example output:

```text
NAME            READY   UP-TO-DATE   AVAILABLE   AGE
my-deployment   2/2     2            2           5d18h
php-apache      1/1     1            1           13h
```

---

## Configure VPA for Your Workloads

Create a `VerticalPodAutoscaler` resource:

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
    name: php-apache
  updatePolicy:
    updateMode: "Off"
```

### Supported `updateMode` Values (Case-Sensitive)

* **Off** – Recommendation only
* **Initial** – Applies recommendations only at pod creation
* **Recreate** – Restarts pods to apply new requests
* **InPlaceOrRecreate**
* **Auto** – Fully automated (use cautiously)

> ⚠️ Kubernetes is case-sensitive.
> Use `"Off"`, not `"off"`.

**Tip:**
Start with `updateMode: "Off"` to safely review recommendations in production environments.

Apply the VPA:

```bash
kubectl apply -f vpa.yml
```

Verify:

```bash
kubectl get vpa
```

Example output:

```text
NAME          MODE   CPU   MEM     PROVIDED   AGE
example-vpa   Off    25m   250Mi   True       8s
```

---

## Generate Test Traffic

Simulate application load:

```bash
kubectl run -it --rm load-generator2 \
  --image=busybox:1.28 \
  --restart=Never \
  -- sh -c 'while sleep 0.01; do wget -q -O- http://php-apache; done'
```

---

## Monitor VPA Recommendations

Describe the VPA resource:

```bash
kubectl describe vpa example-vpa
```

You’ll see:

* `target`
* `lowerBound`
* `upperBound`

These values show recommended CPU and memory ranges.

---

## Monitoring Commands

Watch pod restarts:

```bash
kubectl get pods -w
```

Check deployment health:

```bash
kubectl get deployment
```

Monitor live resource usage:

```bash
kubectl top pods
```

These commands help correlate **actual usage**, **VPA recommendations**, and **pod restarts**.

---

## Summary

Vertical Pod Autoscaler removes guesswork from resource tuning by continuously analyzing workload behavior. While powerful, it must be used carefully due to pod restarts, metric limitations, and potential conflicts with HPA.

When run in recommendation mode and combined with strong observability, VPA becomes a valuable tool for efficient and cost-optimized Kubernetes workloads.


