# 🚑 Kubernetes Worker Node Failure Troubleshooting Guide

Worker nodes are responsible for running **application workloads (Pods)** in a Kubernetes cluster.
If a worker node fails, pods may stop running, become **NotReady**, or get **rescheduled to other nodes**.

This guide provides a **practical production troubleshooting workflow** used by DevOps and SRE teams.

---

# 1️⃣ Check Node Status

First verify whether the worker node is healthy.

```bash
kubectl get nodes
```

Example output:

```
NAME           STATUS     ROLES    AGE   VERSION
worker-node1   NotReady   <none>   20d   v1.28
worker-node2   Ready      <none>   20d   v1.28
```

If the node shows **NotReady**, Kubernetes cannot schedule pods on it.

Get detailed information:

```bash
kubectl describe node <node-name>
```

Check the **Conditions** section for issues such as:

* MemoryPressure
* DiskPressure
* PIDPressure
* NetworkUnavailable

---

# 2️⃣ Verify Network Plugin (CNI)

Worker nodes require a **CNI (Container Network Interface)** plugin for pod networking.

Common CNI plugins:

* Calico
* Flannel
* Cilium

If the network plugin is not available, pods may remain in **Pending** state.

From the control plane, verify system pods:

```bash
kubectl get pods -n kube-system
```

Example CNI pods:

```
calico-node
flannel
cilium
```

On the worker node, verify CNI configuration:

```bash
cd /etc/cni/net.d
ls
```

If CNI configuration is missing or corrupted, networking will fail and nodes may become **NotReady**.

---

# 3️⃣ Check Kubelet Service

The **kubelet** agent runs on every worker node and manages pods.

If kubelet stops running, the node becomes **NotReady**.

Login to the worker node and run:

```bash
systemctl status kubelet
```

If the service is stopped:

```bash
sudo systemctl restart kubelet
```

View kubelet logs:

```bash
journalctl -u kubelet
```

Helpful log navigation tips:

* `Shift + G` → jump to the last line
* `i` → informational logs
* `e` → error logs

Also verify the **kubelet configuration file** and ensure the correct **certificate paths** are configured.

Common kubelet issues:

* Container runtime not running
* Certificate expired
* Node registration failure

---

# 4️⃣ Verify Container Runtime

Kubernetes depends on a container runtime such as:

* containerd
* Docker

Check runtime status:

```bash
systemctl status containerd
```

or

```bash
systemctl status docker
```

Restart the runtime if required:

```bash
sudo systemctl restart containerd
```

---

# 5️⃣ Check Node Resources

Worker nodes may fail due to **CPU, memory, or disk exhaustion**.

Check system resources:

```bash
top
free -m
df -h
```

If disk usage is full, remove unused images.

For Docker:

```bash
docker system prune -a
```

For containerd:

```bash
crictl rmi --prune
```

---

# 6️⃣ Check Node Network Connectivity

Network issues may prevent worker nodes from communicating with the control plane.

Verify network interfaces:

```bash
ip a
```

Test connectivity to the control plane:

```bash
ping <control-plane-ip>
```

Also verify CNI pods:

```bash
kubectl get pods -n kube-system
```

If CNI pods are failing, restart or redeploy the network plugin.

---

# 7️⃣ Check Node Events

Kubernetes events help identify node failures.

```bash
kubectl get events --sort-by=.metadata.creationTimestamp
```

Common events:

* NodeNotReady
* FailedMount
* FailedScheduling
* NodePressure

---

# 8️⃣ Verify Node Capacity

Check if workloads exceed node resource limits.

```bash
kubectl describe node <node-name>
```

Look for:

```
Allocated resources
CPU Requests
Memory Requests
```

If resource requests exceed node capacity, pods remain in **Pending** state.

---

# 9️⃣ Check Image or Registry Issues

Pod failures can also occur due to image pull issues.

Check pod description:

```bash
kubectl describe pod <pod-name>
```

Common errors:

* ImagePullBackOff
* ErrImagePull

Possible fixes:

* Verify image name
* Check container registry connectivity
* Validate authentication credentials

---

# 🔟 Check Node Certificates

If kubelet certificates expire, the node cannot communicate with the API server.

Check certificate validity:

```bash
openssl x509 -in /var/lib/kubelet/pki/kubelet-client-current.pem -noout -dates
```

If expired, rotate the certificates.

---

# 1️⃣1️⃣ Restart the Node (Last Option)

If the issue persists after troubleshooting, restart the node.

```bash
sudo reboot
```

Kubernetes will automatically **reschedule pods to healthy nodes**.

---

# ⚡ Quick Worker Node Troubleshooting Checklist

| Check          | Command                           |
| -------------- | --------------------------------- |
| Node status    | `kubectl get nodes`               |
| Node details   | `kubectl describe node`           |
| Kubelet logs   | `journalctl -u kubelet`           |
| Runtime status | `systemctl status containerd`     |
| Disk usage     | `df -h`                           |
| Memory usage   | `free -m`                         |
| Node events    | `kubectl get events`              |
| System pods    | `kubectl get pods -n kube-system` |

---

# 🔥 Real DevOps Production Tip

In real production environments, the **top 5 causes of worker node failures** are:

1. Kubelet crash
2. Container runtime failure
3. Disk full
4. Network / CNI failure
5. Node resource exhaustion

---

