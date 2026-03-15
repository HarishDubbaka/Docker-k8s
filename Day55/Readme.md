# 🚑 Kubernetes Control Plane Failure Troubleshooting Guide

Troubleshooting a **Kubernetes Control Plane failure** is a core skill for platform engineers and DevOps teams. When the control plane is down, the cluster cannot schedule or manage workloads. This guide provides a **practical, production-ready troubleshooting workflow**.

***

## 1️⃣ Check Control Plane Node Status

First, confirm whether the control plane node is healthy:

```bash
kubectl get nodes
```

If the control plane is unhealthy, you may see:

*   `NotReady`
*   `Unknown`

Or, if the API server is completely down:

    The connection to the server <IP>:6443 was refused

In that case, SSH into the **control plane node** directly.

***

## 2️⃣ Check Control Plane Components

Verify the core components: under /etc/kubernetes/manifests/

*   kube-apiserver -- checkthe file coreclty if any spelling mistakes are there
*   kube-controller-manager
*   kube-scheduler
*   etcd

Check running containers:

```bash
crictl ps | grep kubeapi
```

or

```bash
docker ps | grep kubeapi
```

***



## 3️⃣ Check kubelet Status

If **kubelet** is down, control plane static pods will not run.

Check kubelet:

```bash
systemctl status kubelet
```

Restart if needed:

```bash
systemctl restart kubelet
```

View logs:

```bash
journalctl -u kubelet -f
```

***

## 4️⃣ Check Static Pod Manifests

Control plane components run as **static pods** managed by kubelet.

Location:

    /etc/kubernetes/manifests/

Expected files:

*   kube-apiserver.yaml
*   kube-controller-manager.yaml
*   kube-scheduler.yaml
*   etcd.yaml

If any file is missing or corrupted, the component will fail.

***

## 📌 Node Type vs admin.conf

| Node Type              | Should `admin.conf` exist? | Why                             |
| ---------------------- | -------------------------- | ------------------------------- |
| Control-plane (master) | ✅ Yes                      | Created by `kubeadm init`       |
| Worker Node            | ❌ No                       | Workers only get `kubelet.conf` |

***

## 5️⃣ Verify etcd Health

The control plane heavily depends on **etcd**.

Check health:

```bash
ETCDCTL_API=3 etcdctl endpoint health
```

List members:

```bash
etcdctl member list
```

Common failure causes:

*   Disk full
*   Certificate expiration
*   Network partition

***

## 6️⃣ Check API Server Logs

If the API server fails, inspect its logs:

```bash
crictl logs <apiserver-container-id>
```

Typical issues:

*   Certificate errors
*   etcd connectivity failures
*   Port conflicts

***

## 7️⃣ Check Disk & Memory

Resource exhaustion frequently causes control plane outages.

Check disk:

```bash
df -h
```

Check memory:

```bash
free -m
```

Look for:

*   Disk full
*   OOMKilled processes

***

## 8️⃣ Verify Certificates

Expired certificates can break the entire control plane.

Check expiration:

```bash
kubeadm certs check-expiration
```

Renew certificates:

```bash
kubeadm certs renew all
systemctl restart kubelet
```

***

## 9️⃣ Network & Port Validation

Verify required ports:

| Component          | Port      |
| ------------------ | --------- |
| kube-apiserver     | 6443      |
| etcd               | 2379–2380 |
| kubelet            | 10250     |
| controller-manager | 10257     |
| scheduler          | 10259     |

Check port listeners:

```bash
netstat -tulnp | grep 6443
```

***

## 🔟 Check Cluster Events

If the API server is reachable:

```bash
kubectl get events -A
```

Look for:

*   Scheduling failures
*   Node communication errors

***

# ✅ Real-World Troubleshooting Order (Fast Approach)

1.  SSH to control plane node
2.  Check `kubelet` service
3.  Validate `/etc/kubernetes/manifests`
4.  Verify etcd health
5.  Inspect kube-apiserver logs
6.  Check disk/memory
7.  Validate certificates

***

# 💡 Interview Tip

If asked:

### **“How do you troubleshoot a Kubernetes control plane failure?”**

Answer in this order:

1.  Check kubelet service
2.  Verify static pod manifests
3.  Check etcd health
4.  Inspect API server logs
5.  Validate certificates and node resources

***

## Want more?

I can provide **🔥 25 real Kubernetes control plane failure scenarios with solutions** for:

*   Interviews
*   SRE playbooks
*   Production readiness

Just tell me!
