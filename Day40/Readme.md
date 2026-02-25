# Kubernetes Storage Provisioning with Storage Classes

---

## 🔹 Understanding Ephemeral Storage (emptyDir)

### Example Volume Configuration

```yaml
volumeMounts:
  - name: redis-storage
    mountPath: /data/redis
volumes:
  - name: redis-storage
    emptyDir: {}
```

---

## 🔹 Testing Inside the Pod

```bash
kubectl exec -it redis-pod -- bash
```

### Check Directory

```bash
ls -ltr
df -h
```

### Create a File

```bash
cd /data/redis
echo "Harish Dubbaka is an Devops Engineer" > Harish.txt
ls -ltr
cat Harish.txt
```

### Kill Main Process

```bash
kill -9 1
ps -aux
```

Even after killing the process, the file still exists:

```
Harish Dubbaka is an Devops Engineer
```

### ✅ Why?

* The **container restarted**
* The **Pod was NOT deleted**
* `emptyDir` survives container restarts
* But if the Pod is deleted → data is lost ❌

---

# Pod Lifecycle vs Container Lifecycle

| Pod Lifecycle                         | Container Lifecycle                |
| ------------------------------------- | ---------------------------------- |
| Managed by Kubernetes                 | Managed by container runtime       |
| Has phases (Pending, Running, Failed) | Has states (Running, Terminated)   |
| Pod gets new identity if recreated    | Container restarts inside same Pod |
| Has its own IP                        | Shares Pod IP                      |
| Can contain multiple containers       | Single process                     |

---

### 🎯 Simple Example

If your app crashes:

* 🔁 Container crash → Container restarts inside same Pod.
* 💥 Pod deleted or node fails → New Pod is created (new IP).

> If you delete a Pod and recreate it using the same image, the data will NOT be available.

Container lifecycle = process-level behavior
Pod lifecycle = infrastructure-level behavior

That’s where Kubernetes storage comes in — to persist data.

---

# Introduction to Storage Classes

Kubernetes manages containerized applications at scale.
For applications that need persistent storage, **Storage Classes** are essential.

Storage Classes allow dynamic provisioning of storage based on:

* Performance
* Availability
* Cost
* Storage backend

---

## What is a Storage Class?

A **Storage Class** defines how storage volumes are dynamically created.

It includes:

* Storage type (SSD, HDD)
* Backend (AWS EBS, GCP PD, NFS, Azure, etc.)
* Replication
* Encryption
* Performance characteristics

---

## Provisioner

Each StorageClass has a **provisioner** that determines which backend plugin is used.

| Volume Plugin  | Internal Provisioner |
| -------------- | -------------------- |
| AzureFile      | ✓                    |
| PortworxVolume | ✓                    |
| VsphereVolume  | ✓                    |
| NFS            | External             |
| Local          | External             |
| CephFS         | External             |

---

# Kubernetes Storage Components

## 1️⃣ Persistent Volume (PV)

* Cluster-level storage resource
* Provisioned manually or dynamically

## 2️⃣ Persistent Volume Claim (PVC)

* User/application storage request
* Defines size, access mode, storage class

## 3️⃣ Storage Class (SC)

* Defines provisioning rules
* Specifies provisioner backend

---

# Kubernetes Access Modes

Access modes define how a PV can be mounted.

| Access Mode      | Abbreviation | Description                            |
| ---------------- | ------------ | -------------------------------------- |
| ReadWriteOnce    | RWO          | Mounted as read-write by a single node |
| ReadOnlyMany     | ROX          | Mounted as read-only by many nodes     |
| ReadWriteMany    | RWX          | Mounted as read-write by many nodes    |
| ReadWriteOncePod | RWOP         | Mounted as read-write by a single Pod  |

### Backend Support

* AWS EBS / GCP PD → RWO
* NFS / CephFS → RWX

---

## Example PVC

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 5Gi
  storageClassName: nfs-storage
```

---

# Sample PV Used in Demo

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: k8s-pv-volume
  labels:
    type: local
spec:
  storageClassName: standard
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/data
    type: DirectoryOrCreate
```

Apply:

```bash
kubectl apply -f pv.yml
```

---

# Sample PVC Used in Demo

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: k8s-pv-claim
spec:
  storageClassName: standard
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

Apply:

```bash
kubectl apply -f pvc.yml
```

Check:

```bash
kubectl get pv
kubectl get pvc
```

---

# Reclaim Policy

Defines what happens when PVC is deleted.

## Retain

* PV and storage are not deleted
* Moves to Released state
* Manual cleanup required

## Delete (Default)

* PV and underlying storage deleted automatically

## Recycle (Deprecated)

* Performs basic cleanup
* Not recommended anymore

---

## Check Reclaim Policy

```bash
kubectl get pv
```

## Change Reclaim Policy

```bash
kubectl patch pv k8s-pv-volume -p '{"spec":{"persistentVolumeReclaimPolicy":"Retain"}}'
```

---

# Why PVC Stays Pending?

With `WaitForFirstConsumer`:

Kubernetes waits for:

1. A Pod that uses the PVC
2. Scheduler decision
3. Then binds the PV

This helps with:

* Node-aware storage
* Zonal storage
* Better scheduling

---

# Troubleshooting PVC

## Step 1 — Describe PVC

```bash
kubectl describe pvc k8s-pv-claim
```

Check the **Events** section.

---

## Solution — Create Pod Using PVC

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: task-pv-pod
spec:
  volumes:
    - name: task-pv-storage
      persistentVolumeClaim:
        claimName: k8s-pv-claim
  containers:
    - name: task-pv-container
      image: nginx
      volumeMounts:
        - mountPath: /usr/share/nginx/html
          name: task-pv-storage
```

Apply:

```bash
kubectl apply -f pod.yaml
```

Then check:

```bash
kubectl get pvc
kubectl get pv
```

Status should become:

```
Bound
```

---

# Safe Storage Reset Method

If storage is broken:

```bash
kubectl delete pod <pod>
kubectl delete pvc <pvc>
kubectl delete pv <pv>
```

Recreate in order:

1️⃣ PV
2️⃣ PVC
3️⃣ Pod

---

# Force Delete (If Stuck)

```bash
kubectl delete pvc k8s-pv-claim --grace-period=0 --force
kubectl delete pv k8s-pv-volume --grace-period=0 --force
```

---

# If Still Stuck — Remove Finalizers

```bash
kubectl patch pvc k8s-pv-claim -p '{"metadata":{"finalizers":null}}'
kubectl patch pv k8s-pv-volume -p '{"metadata":{"finalizers":null}}'
```

This immediately removes protection and deletes the resource.

---

# 🎯 Final Understanding

Without persistent storage:

Pod deleted → Data lost ❌

With PV + PVC:

Pod deleted → Data safe ✅

---

## 🧠 Key Takeaways

* Containers are ephemeral
* Pods are ephemeral
* Persistent storage ensures data survival
* PV = Storage resource
* PVC = Storage request
* StorageClass = Provisioning rules
* Access Modes = Mount behavior
* Reclaim Policy = Cleanup behavior

---
n
