# Kubernetes Autoscaling 

## What is Autoscaling?

Kubernetes autoscaling is a powerful feature that enables clusters to automatically adjust resources in response to changing demand, optimizing both performance and cost. It ensures applications have the right resources at the right time by dynamically scaling nodes or pods based on real-time metrics.

Kubernetes supports **three main types of autoscaling**:

1. **Horizontal Pod Autoscaler (HPA)**  
   Scales the number of pod replicas based on workload demand.

2. **Vertical Pod Autoscaler (VPA)**  
   Adjusts CPU and memory requests/limits for pods.

3. **Cluster Autoscaler**  
   Scales the number of nodes in the cluster.

Together, these mechanisms provide elastic scalability while balancing reliability and cost efficiency. Implementing autoscaling effectively requires careful tuning of metrics, thresholds, and scaling strategies.

---

## 3 Kubernetes Autoscaling Methods

### Horizontal Pod Autoscaler (HPA)

The **Horizontal Pod Autoscaler (HPA)** automatically adjusts the number of pod replicas in response to changing workload demands.

HPA continuously monitors real-time metrics such as:
- CPU utilization
- Memory utilization
- Custom or external metrics (e.g., queue length, request rate)

If a defined threshold is crossed (for example, CPU usage exceeds 70%), HPA increases the number of pods. When usage drops, it scales them down.

HPA is:
- Best suited for stateless applications
- Capable of working with stateful workloads
- Not applicable to objects like `DaemonSets`

Combined with cluster autoscaling, HPA is a key tool for handling unpredictable traffic spikes efficiently.

---

## HPA Metrics

Kubernetes HPA relies on metrics to determine when to scale.

### Default Metrics
- Average CPU utilization
- Average memory utilization

> ⚠️ **Metrics Server must be installed**  
HPA requires the Kubernetes Metrics Server to access real-time CPU and memory usage.

### Custom Metrics
HPA can also scale based on:
- Application-specific metrics
- External metrics via the Custom Metrics API

---

## How to Set Up Kubernetes HPA

This demo walks through configuring HPA for a simple application.

### Prerequisites
- Kubernetes cluster
- `kubectl` installed and configured
- Metrics Server installed

---

## 1. Create the Application

Create a Deployment and Service for the demo application.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: php-apache
spec:
  selector:
    matchLabels:
      run: php-apache
  template:
    metadata:
      labels:
        run: php-apache
    spec:
      containers:
      - name: php-apache
        image: registry.k8s.io/hpa-example
        ports:
        - containerPort: 80
        resources:
          limits:
            cpu: 500m
          requests:
            cpu: 200m
---
apiVersion: v1
kind: Service
metadata:
  name: php-apache
  labels:
    run: php-apache
spec:
  ports:
  - port: 80
  selector:
    run: php-apache
````

Apply the manifest:

```bash
kubectl apply -f deploy.yaml
```

Verify:

```bash
kubectl get deployments
kubectl get svc
```

---

## 2. Create the HorizontalPodAutoscaler

Create an HPA that:

* Maintains CPU utilization at **50%**
* Scales between **1 and 10 pods**

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: php-apache
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: php-apache
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

Apply the HPA:

```bash
kubectl apply -f hpa.yml
```

Check HPA status:

```bash
kubectl get hpa
```

---

## 3. Verify Deployment Status

```bash
kubectl get deployments
```

Example output:

```text
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
php-apache  1/1     1            1           4m
```

---

## 4. Generate Test Traffic

To simulate load, run a BusyBox pod that continuously sends requests to the service:

```bash
kubectl run -it --rm load-generator2 \
  --image=busybox:1.28 \
  --restart=Never \
  -- sh -c 'while sleep 0.01; do wget -q -O- http://php-apache; done'
```

Watch the HPA scale in real time:

```bash
kubectl get hpa -w
```

Example output:

```text
NAME         REFERENCE               TARGETS         MINPODS   MAXPODS   REPLICAS
php-apache  Deployment/php-apache   cpu: 76%/50%    1         10        5
php-apache  Deployment/php-apache   cpu: 50%/50%    1         10        6
```

Check the Deployment again:

```bash
kubectl get deployment php-apache
```

```text
READY   UP-TO-DATE   AVAILABLE
6/6     6            6
```

🎉 The application has successfully autoscaled.

Stop the load generator (`Ctrl + C`) and observe the HPA scale the pods back down.

---

## Kubernetes HPA Best Practices

* ✅ Install Metrics Server
* ✅ Set accurate CPU and memory requests/limits
* ❌ Don’t mix HPA and VPA on the same pods
* ✅ Use VPA first to identify correct resource values
* ✅ Enable Cluster Autoscaler for full elasticity
* ✅ Monitor and tune scaling thresholds regularly

---

## Conclusion

Kubernetes Horizontal Pod Autoscaler enables applications to automatically respond to traffic changes, ensuring high availability while optimizing resource usage. When combined with proper metrics, cluster autoscaling, and best practices, HPA becomes a cornerstone of modern cloud-native architecture.

---

Happy autoscaling 🚀

Just tell me 👍
```
