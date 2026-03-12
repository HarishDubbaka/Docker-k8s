In real production environments and in the **Certified Kubernetes Administrator (CKA)**, troubleshooting should start with **understanding the problem clearly before running commands**.

Here is a **structured troubleshooting approach used by Kubernetes admins**.

---

# 1️⃣ Understand the Exact Nature of the Problem

First determine **what exactly is failing**.

Ask questions like:

* Is the **application not starting**?
* Is the **pod crashing**?
* Is the **service unreachable**?
* Is the **application slow or timing out**?

Example checks:

```bash
kubectl get pods
kubectl get svc
kubectl get nodes
```

Goal: **Identify the symptom clearly.**

---

# 2️⃣ Identify Which Components Are Affected

Determine **which Kubernetes components are involved**.

Possible components:

| Component  | Example Issue          |
| ---------- | ---------------------- |
| Pod        | CrashLoopBackOff       |
| Deployment | Wrong image            |
| Service    | No endpoints           |
| Node       | NotReady               |
| Network    | NetworkPolicy blocking |
| Storage    | PVC not mounted        |

Example command:

```bash
kubectl describe pod <pod-name>
```

This shows events and affected components.

---

# 3️⃣ Check if the Application Worked Before

This helps determine if the issue is **new or long-standing**.

Questions to ask:

* Did this workload **ever run successfully**?
* When did the failure start?
* Is the issue **intermittent or constant**?

Example investigation:

```bash
kubectl rollout history deployment <deployment-name>
```

This shows previous deployment revisions.

---

# 4️⃣ Check for Recent Configuration Changes

Many production incidents happen because of **recent configuration changes**.

Things to verify:

* Deployment updates
* ConfigMap changes
* Secret updates
* NetworkPolicy changes
* Resource limit modifications

Commands:

```bash
kubectl describe deployment <deployment-name>
kubectl get events --sort-by=.metadata.creationTimestamp
```

You may see events like:

```
ScalingReplicaSet
Updated deployment
FailedMount
```

---

# 5️⃣ Check Logs and Events

Logs and events provide **direct clues about failures**.

Logs:

```bash
kubectl logs <pod>
```

Events:

```bash
kubectl get events
```

Events often show:

* Failed scheduling
* Failed image pull
* Volume mount errors
* Network issues

---

# 6️⃣ Validate Dependencies

Applications usually depend on other services.

Examples:

* database
* cache
* external API
* storage

Check if dependent services are healthy.

Example:

```bash
kubectl get svc
kubectl get endpoints
```

---

# 7️⃣ Verify Cluster Health

Finally check the **cluster infrastructure**.

Commands:

```bash
kubectl get nodes
kubectl top nodes
kubectl top pods
```

Look for:

* CPU exhaustion
* memory pressure
* disk pressure
* node not ready

---

> “Understand the problem first, then run commands.”

Blindly executing commands without understanding the issue can make troubleshooting slower.

When debugging application connectivity or failures in **Kubernetes**  the main checks:

* Labels
* Endpoints
* Port mismatch
* NetworkPolicy

Below are **additional important checks** you should include in a **complete troubleshooting checklist**.

---

# Complete Kubernetes Service / App Troubleshooting Checklist

## 1️⃣ Check Pod Status

First confirm pods are running.

```bash
kubectl get pods -o wide
```

Look for:

* `Running`
* `CrashLoopBackOff`
* `Pending`
* `ContainerCreating`

If not running → fix pod issue first.

---

# 2️⃣ Check Pod Labels

Services route traffic using labels.

```bash
kubectl get pods --show-labels
```

Verify they match the service selector.

---

# 3️⃣ Check Service Selector

```bash
kubectl describe svc <service-name>
```

Example:

```
Selector: app=vote
```

If selectors don’t match pod labels → **service cannot find pods**.

---

# 4️⃣ Check Endpoints

```bash
kubectl get endpoints <service-name>
```

Working example:

```
vote-service   10.244.1.10:8080
```

Broken case:

```
vote-service   <none>
```

Means service cannot find matching pods.

---

# 5️⃣ Check Port Mapping

Compare **service port and container port**.

Service:

```bash
kubectl describe svc vote-service
```

Pod:

```bash
kubectl describe pod vote-pod
```

Example mismatch:

```
Service targetPort: 80
Container port: 8080
```

Fix:

```
targetPort: 8080
```

---

# 6️⃣ Check NetworkPolicy

Network policies may block traffic.

```bash
kubectl get networkpolicy
```

Inspect:

```bash
kubectl describe networkpolicy
```

Verify:

* podSelector labels
* ingress/egress rules

Labels must match correctly.

---

# 7️⃣ Check DNS Resolution

Sometimes the service exists but DNS fails.

Test inside a pod:

```bash
kubectl exec -it <pod> -- nslookup vote-service
```

If DNS fails, check **CoreDNS**.

```bash
kubectl get pods -n kube-system
```

Look for:

```
coredns Running
```

---

# 8️⃣ Test Service from Inside Cluster

Use a temporary pod.

```bash
kubectl run test --rm -it --image=busybox -- sh
```

Then test:

```bash
wget -O- vote-service
```

or

```bash
nc -zv vote-service 80
```

---

# 9️⃣ Check Container Logs

Application itself might be failing.

```bash
kubectl logs <pod-name>
```

If multiple containers:

```bash
kubectl logs <pod-name> -c <container-name>
```

---

# 🔟 Check Resource Limits

Pod may fail due to resource issues.

```bash
kubectl describe pod <pod>
```

Look for:

```
OOMKilled
```

---

# 1️⃣1️⃣ Check Events

Events often reveal the exact problem.

```bash
kubectl get events --sort-by=.metadata.creationTimestamp
```

Example:

```
FailedCreatePodSandBox
FailedMount
ImagePullBackOff
```

---

# 1️⃣2️⃣ Check Node Health

```bash
kubectl get nodes
```

If node is:

```
NotReady
```

Pods cannot communicate properly.

---

# 🧠 Recommended Debugging Order (CKA Style)

When **service not reachable**, check in this order:

```
Pods running?
↓
Pod labels
↓
Service selector
↓
Endpoints
↓
Port mapping
↓
NetworkPolicy
↓
DNS (CoreDNS)
↓
Logs
↓
Node health
↓
Cluster events
```

---

Effective Kubernetes troubleshooting means understanding the problem first, then systematically checking pods, services, network, and cluster health.
