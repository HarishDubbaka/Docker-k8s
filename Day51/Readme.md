# Kubernetes ETCD Backup & Restore Guide

In Kubernetes architecture, **etcd** is an integral part of the cluster. All cluster objects and their state are stored in etcd.

This guide explains:
- What etcd is
- Why backups are important
- How to take an etcd snapshot backup
- How to restore etcd from a snapshot

---

# What is ETCD in Kubernetes?

ETCD is the **primary datastore for Kubernetes**.

Key points:

- It is a **consistent, distributed, and secure key-value store**
- Uses the **Raft protocol** for consensus
- Supports **high availability architecture**
- Stores all **Kubernetes cluster data**

### What ETCD Stores

ETCD stores:

- Kubernetes cluster configuration
- All API objects
- Object states
- Service discovery information

---

# Raft Protocol

**Raft** is the algorithm that keeps distributed systems consistent.

It works by:

- Electing a **leader node**
- Replicating logs across nodes
- Maintaining **cluster consistency**

---

# Kubernetes Backup Strategy

Whether it is:

- Managed Kubernetes (EKS, AKS, GKE)
- Custom Kubernetes (kubeadm)

**Backup is critical.**

In Kubernetes, **backup mainly means backing up ETCD**.

You should design:

- Automated backup strategy
- Reliable restore mechanism
- Periodic backup validation

Another option is exporting Kubernetes objects as **JSON/YAML dumps**, which can later be restored into the same or a different cluster.

---

# Kubernetes ETCD Backup Using etcdctl

ETCD provides a built-in **snapshot mechanism**.

The CLI tool used is:

```

etcdctl

````

---

# Step 1 — Login to Control Plane

SSH into the control plane node.

```bash
ssh master-node
````

---

# Step 2 — Install Required Tools

We need two tools:

* `etcdctl`
* `etcdutl`

If they are not installed:

```bash
etcdctl
```

Output:

```
Command 'etcdctl' not found, but can be installed with:

sudo snap install etcd
sudo apt install etcd-client
```

Install using:

```bash
sudo apt install etcd-client
```

---

# Step 3 — Verify Installation

```bash
etcdctl version
etcdutl version
```

---

# Step 4 — Gather Required Information

To create a snapshot, we need:

* ETCD endpoint
* CA certificate
* Server certificate
* Server key

Parameters used:

```
--endpoints
--cacert
--cert
--key
```

---

# Method 1 — From ETCD Static Pod Manifest

Location:

```
/etc/kubernetes/manifests/etcd.yaml
```

Example:

```bash
Harish@master:/etc/kubernetes/manifests$ sudo cat etcd.yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/etcd.advertise-client-urls: https://10.0.0.4:2379
  creationTimestamp: null
  labels:
    component: etcd
    tier: control-plane
  name: etcd
  namespace: kube-system
spec:
  containers:
  - command:
    - etcd
    - --advertise-client-urls=https://10.0.0.4:2379
    - --cert-file=/etc/kubernetes/pki/etcd/server.crt
    - --client-cert-auth=true
    - --data-dir=/var/lib/etcd
    - --experimental-initial-corrupt-check=true
    - --experimental-watch-progress-notify-interval=5s
    - --initial-advertise-peer-urls=https://10.0.0.4:2380
    - --initial-cluster=master=https://10.0.0.4:2380
    - --key-file=/etc/kubernetes/pki/etcd/server.key
    - --listen-client-urls=https://127.0.0.1:2379,https://10.0.0.4:2379
    - --listen-metrics-urls=http://127.0.0.1:2381
    - --listen-peer-urls=https://10.0.0.4:2380
    - --name=master
    - --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
    - --peer-client-cert-auth=true
    - --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
    - --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    - --snapshot-count=10000
    - --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    image: registry.k8s.io/etcd:3.5.16-0
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 127.0.0.1
        path: /health?exclude=NOSPACE&serializable=true
        port: 2381
        scheme: HTTP
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    name: etcd
    resources:
      requests:
        cpu: 100m
        memory: 100Mi
    startupProbe:
      failureThreshold: 24
      httpGet:
        host: 127.0.0.1
        path: /health?serializable=false
        port: 2381
        scheme: HTTP
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    volumeMounts:
    - mountPath: /var/lib/etcd
      name: etcd-data
    - mountPath: /etc/kubernetes/pki/etcd
      name: etcd-certs
  hostNetwork: true
  priority: 2000001000
  priorityClassName: system-node-critical
  securityContext:
    seccompProfile:
      type: RuntimeDefault
  volumes:
  - hostPath:
      path: /etc/kubernetes/pki/etcd
      type: DirectoryOrCreate
    name: etcd-certs
  - hostPath:
      path: /var/lib/etcd
      type: DirectoryOrCreate
    name: etcd-data
status: {}
Harish@master:/etc/kubernetes/manifests$
```

Important flags inside the manifest:

```
--advertise-client-urls
--cert-file
--key-file
--trusted-ca-file
```

Example snippet:

```yaml
- --cert-file=/etc/kubernetes/pki/etcd/server.crt
- --key-file=/etc/kubernetes/pki/etcd/server.key
- --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
```

---

# Method 2 — Describe ETCD Pod

```bash
kubectl get pods -n kube-system
```

Then:

```bash
kubectl describe pod etcd-master -n kube-system
```

---

# Method 3 — Using Process Command

```bash
ps -aux | grep etcd
```

---

# Step 5 — Create Backup Directory

```bash
sudo mkdir -p /opt/backup
```

---

# Step 6 — Take ETCD Snapshot Backup

Command:

```bash
ETCDCTL_API=3 etcdctl \
--endpoints=https://127.0.0.1:2379 \
--cacert=<ca-cert> \
--cert=<server-cert> \
--key=<server-key> \
snapshot save <backup-file>
```

Example:

```bash
ETCDCTL_API=3 etcdctl \
--endpoints=https://127.0.0.1:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
snapshot save /opt/etcd_backup_080326.db
```

Successful output:

```
Harish@master:~$ sudo ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379   --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key   snapshot save /opt/etcd_backup_080326.db
{"level":"info","ts":1772950362.0642352,"caller":"snapshot/v3_snapshot.go:119","msg":"created temporary db file","path":"/opt/etcd_backup_080326.db.part"}
{"level":"info","ts":"2026-03-08T06:12:42.071081Z","caller":"clientv3/maintenance.go:212","msg":"opened snapshot stream; downloading"}
{"level":"info","ts":1772950362.071139,"caller":"snapshot/v3_snapshot.go:127","msg":"fetching snapshot","endpoint":"https://127.0.0.1:2379"}
{"level":"info","ts":"2026-03-08T06:12:42.178567Z","caller":"clientv3/maintenance.go:220","msg":"**completed snapshot** read; closing"}
{"level":"info","ts":1772950362.196413,"caller":"snapshot/v3_snapshot.go:142","msg":"fetched snapshot","endpoint":"https://127.0.0.1:2379","size":"7.2 MB","took":0.132092391}
{"level":"info","ts":1772950362.1965554,"caller":"snapshot/v3_snapshot.go:152","msg":"saved","path":"/opt/etcd_backup_080326.db"}
Snapshot saved at /opt/etcd_backup_080326.db
Harish@master:~$
```

---

# Verify Backup

Navigate to the backup location:

```bash
cd /opt
ls -ltr
```

Example output:

```
-rw------- 1 root root 7213088 etcd_backup_080326.db
```

---

# Check Snapshot Status

```bash
sudo etcdctl --write-out=table snapshot status etcd_backup_080326.db
```

Example output:

```
+----------+----------+------------+------------+
| HASH     | REVISION | TOTAL KEYS | TOTAL SIZE |
+----------+----------+------------+------------+
| 68894dcd | 14078    | 1252       | 7.2 MB     |
+----------+----------+------------+------------+
```

---

# Kubernetes ETCD Restore Using Snapshot

Now we will restore ETCD using the backup.

Backup file location:

```
/opt/etcd_backup_080326.db
```

---

# Important Warning

Before restoring:

You must stop:

* ETCD
* Kubernetes API Server

If the API server is running during restore, **data corruption may occur**.

---

# Step 1 — Move Static Pod Manifests

Move manifest files temporarily.

```bash
sudo mv /etc/kubernetes/manifests/kube-apiserver.yaml /tmp/
sudo mv /etc/kubernetes/manifests/etcd.yaml /tmp/
```

This stops the running containers.

Verify:

```bash
crictl ps
```

Ensure **ETCD and API server containers are stopped**.

---

# Step 2 — Restore ETCD Snapshot

Command:

```bash
sudo etcdctl snapshot restore /opt/etcd_backup_080326.db

Harish@master:/opt$ sudo etcdctl snapshot restore /opt/etcd_backup_080326.db --data-dir /Harish/etcd-restore
{"level":"info","ts":1772951379.8833618,"caller":"snapshot/v3_snapshot.go:306","msg":"restoring snapshot","path":"/opt/etcd_backup_080326.db","wal-dir":"/Harish/etcd-restore/member/wal","data-dir":"/Harish/etcd-restore","snap-dir":"/Harish/etcd-restore/member/snap"}
{"level":"info","ts":1772951379.9379153,"caller":"mvcc/kvstore.go:388","msg":"restored last compact revision","meta-bucket-name":"meta","meta-bucket-name-key":"finishedCompactRev","restored-compact-revision":13579}
{"level":"info","ts":1772951379.951338,"caller":"membership/cluster.go:392","msg":"added member","cluster-id":"cdf818194e3a8c32","local-member-id":"0","added-peer-id":"8e9e05c52164694d","added-peer-peer-urls":["http://localhost:2380"]}
{"level":"info","ts":1772951379.9697037,"caller":"snapshot/v3_snapshot.go:326","msg":"**restored snapshot**","path":"/opt/etcd_backup_080326.db","wal-dir":"/Harish/etcd-restore/member/wal","data-dir":"/Harish/etcd-restore","snap-dir":"/Harish/etcd-restore/member/snap"}
Harish@master:/opt$ ^C
Harish@master:/opt$ ls

```

Optional: specify new data directory.

```bash
sudo etcdctl snapshot restore /opt/etcd_backup_080326.db \
--data-dir /Harish/etcd-restore
```

---

# Step 3 — Update ETCD Manifest

Edit:

```
/tmp/etcd.yaml
```

Modify the volume path:

```yaml
volumes:
  - name: etcd-data
    hostPath:
      path: /Harish/etcd-restore
      type: DirectoryOrCreate
```

---

# Step 4 — Move Manifest Files Back

```bash
sudo mv /tmp/kube-apiserver.yaml /etc/kubernetes/manifests/
sudo mv /tmp/etcd.yaml /etc/kubernetes/manifests/
```

Kubernetes will recreate the pods automatically.

Wait **10–15 seconds**.

---

# Step 5 — Verify Cluster

Check nodes:

```bash
kubectl get nodes
```

Example:

```
NAME      STATUS   ROLES           AGE   VERSION
master    Ready    control-plane   27m   v1.29.15
worker1   Ready    <none>          26m   v1.29.15
worker2   Ready    <none>          26m   v1.29.15
```

---

# Verify ETCD Health

```bash
ETCDCTL_API=3 etcdctl \
--endpoints=https://127.0.0.1:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
endpoint health
```

---

# Summary

✔ ETCD stores all Kubernetes cluster state
✔ Backup ETCD regularly using `etcdctl snapshot save`
✔ Verify snapshot before using it
✔ Stop API server before restore
✔ Restore snapshot using `etcdctl snapshot restore`

---

# References

Kubernetes Documentation:

[https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/)

---

