# Kubernetes Health Checks and Probes

If you've worked with Kubernetes for any length of time, you've likely experienced the frustration of applications that appear to be running but aren't actually functioning correctly.

Perhaps:

* Requests time out
* Connections fail
* Users report errors
* Pods show status **"Running"** but the app is broken

These symptoms indicate a gap between a container’s technical **running state** and its actual **operational health**.

Kubernetes solves this using **Health Checks**, implemented as **Probes**.

Probes continuously monitor your applications and allow Kubernetes to automatically detect and respond to failures.

---

# How to Check Health in Kubernetes

Pod health in Kubernetes is checked using:

* **Liveness Probes**
* **Readiness Probes**
* **Startup Probes**

These probes are defined inside Pod specs and can use:

* HTTP checks
* TCP checks
* Command (exec) checks
* gRPC checks

---

# What Is a Kubernetes Health Check (Probe)?

A Kubernetes health check (called a **probe**) automates verifying whether your containerized applications are healthy.

Kubernetes “probes” your Pods to determine their internal state.

* ✅ If a probe succeeds → container is healthy
* ❌ If a probe fails → Kubernetes takes corrective action

Probes allow applications to communicate their health to the Kubernetes control plane.

---

# Types of Kubernetes Probes

Probes are essential for high availability. Without them, Kubernetes cannot determine if a Pod that is “Running” is actually serving traffic correctly.

Kubernetes supports three probe types:

---

## 1️⃣ Liveness Probe

**Purpose:**
Checks if the container is alive.

**Behavior on Failure:**

* Kubernetes restarts the container.

Use liveness probes only when the application is in an unrecoverable state.

⚠ Do NOT use liveness probes to check external dependencies like databases. That belongs in readiness probes.

---

## 2️⃣ Readiness Probe

**Purpose:**
Checks if the container is ready to receive traffic.

**Behavior on Failure:**

* Pod marked **Not Ready**
* Removed from Service load balancing
* Container is NOT restarted

Useful when:

* Database is temporarily unavailable
* External API is down
* App is overloaded

---

## 3️⃣ Startup Probe

**Purpose:**
Checks whether the application has fully started.

Liveness and Readiness probes begin only after the startup probe succeeds.

Useful for:

* Slow-starting applications
* Applications requiring heavy initialization

---

# Probe Action Types

Each probe must define an action.

## 1. Command (exec)

Runs a command inside the container.

* Exit code `0` → Healthy
* Non-zero exit code → Unhealthy

---

## 2. HTTP (httpGet)

Makes an HTTP GET request to a container port.

* `2xx` → Healthy
* `4xx / 5xx` → Failure

Most commonly used probe type.

---

## 3. TCP (tcpSocket)

Attempts to open a TCP socket.

* Successful connection → Healthy

---

## 4. gRPC (grpc)

Uses gRPC Health Checking Protocol on a specified port.

---

# Probe Timing Options

All probes share these configuration options:

| Field                 | Description                                                         |
| --------------------- | ------------------------------------------------------------------- |
| `initialDelaySeconds` | Delay before first probe                                            |
| `periodSeconds`       | Interval between probes (default 10s)                               |
| `timeoutSeconds`      | Time before probe times out (default 1s)                            |
| `failureThreshold`    | Consecutive failures before marking unhealthy (default 3)           |
| `successThreshold`    | Consecutive successes before marking healthy again (Readiness only) |

Tuning these values is critical for avoiding:

* False positives
* Delayed failure detection
* Excessive load

---

# Example 1: Liveness Probe (Exec Command)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: demo-deployment
  template:
    metadata:
      labels:
        app: demo-deployment
    spec:
      containers:
        - name: demo
          image: nginx:latest
          livenessProbe:
            exec:
              command:
                - echo
                - "Healthy"
            periodSeconds: 5
```

### Check Liveness Status

```bash
kubectl get pods
kubectl describe pod <pod-name>
kubectl get pods -w
```

Look for:

```
Liveness: exec [echo Healthy] delay=0s timeout=1s period=5s
```

---

# Example 2: Readiness Probe (HTTP)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-readiness
spec:
  replicas: 3
  selector:
    matchLabels:
      app: demo-deployment
  template:
    metadata:
      labels:
        app: demo-deployment
    spec:
      containers:
        - name: demo
          image: nginx:latest
          readinessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 5
            periodSeconds: 10
```

### Custom Headers Example

```yaml
httpGet:
  path: /
  port: 80
  httpHeaders:
    - name: Authorization
      value: Secret-Token
```

### Describe Pod

```bash
kubectl describe pod <pod-name>
```

Look for:

```
Readiness: http-get http://:80/ delay=5s timeout=1s period=10s
```

---

# Liveness vs Readiness (Important)

| Feature       | Liveness Probe      | Readiness Probe                 |
| ------------- | ------------------- | ------------------------------- |
| Purpose       | Is container alive? | Is container ready for traffic? |
| On Failure    | Restarts container  | Removes from Service            |
| Restarts Pod? | Yes                 | No                              |

---

# Example 3: Startup Probe (TCP Socket)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-startupprobe
spec:
  replicas: 3
  selector:
    matchLabels:
      app: demo-deployment
  template:
    metadata:
      labels:
        app: demo-deployment
    spec:
      containers:
        - name: demo
          image: nginx:latest
          startupProbe:
            tcpSocket:
              port: 80
            initialDelaySeconds: 5
```

---

# Debugging Tips & Best Practices

## 1️⃣ Verify Probe Configuration

* Check command spelling
* Confirm correct port
* Ensure correct endpoint path

---

## 2️⃣ Set Appropriate Intervals

Too frequent:

* False failures
* High resource usage

Too slow:

* Delayed failure detection

---

## 3️⃣ Use Lightweight Health Endpoints

Create dedicated endpoints like:

```
/health
/ready
```

They should:

* Perform minimal checks
* Return quickly
* Return HTTP 200 when healthy

---

## 4️⃣ Enable Connection Reuse

Reuse connections where possible to:

* Reduce overhead
* Improve reliability

---

## 5️⃣ Always Use Probes in Production

Without probes:

* Kubernetes cannot detect broken apps
* Traffic may go to unhealthy pods
* Debugging becomes harder
* High availability suffers

---

# Final Recommendation

For production workloads, use:

* ✅ Liveness Probe
* ✅ Readiness Probe
* ✅ Startup Probe

Together they provide:

* Automatic restarts
* Safe traffic routing
* Proper startup handling
* Better visibility

---

# Healthy Probes = Healthy Cluster 🚀

