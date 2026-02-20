# 🔐 Kubernetes RBAC – Complete Explanation (README Style)

## 📌 Scenario

Yesterday, we created a user **sree** and issued a client certificate.
However, the user was **not able to list pods**.

### Verification

```bash
kubectl auth can-i get pods
yes

kubectl auth whoami
Username: kubernetes-admin
Groups: [kubeadm:cluster-admins system:authenticated]

kubectl auth can-i get pods --as sree
no
```

### ✅ What This Means

* `kubernetes-admin` → Has full permissions (cluster-admin)
* `sree` → Authenticated successfully
* ❌ But `sree` has NO authorization

👉 To allow `sree` to list pods, we must configure **RBAC (Role-Based Access Control)**.

---

# 🔐 What is RBAC?

In Kubernetes:

* **Authentication** → Who are you?
* **Authorization (RBAC)** → What are you allowed to do?

Without RBAC:

> The user is identified but cannot perform any action.

RBAC connects:

```
User ➝ Permissions ➝ Resources
```

---

# 🏗 Kubernetes API Groups

## 🔹 Core API Group (`v1`)

![Image](https://miro.medium.com/1%2ANuQkJd-hactvC0j2OSmw5Q.png)

![Image](https://miro.medium.com/1%2AZlQffqUWhkxqbKI--ZUitg.png)

![Image](https://miro.medium.com/1%2AHfn62d4gyGgmbwOHOIZROw.png)

![Image](https://miro.medium.com/1%2Ad3ypfOPzh-6dDSpFHidtAw.jpeg)

![Image](https://stevenschwenke.de/images/kubernetes-icons-set-for-kubernetes-architecture-diagrams.png)

* No explicit group name
* Uses: `apiVersion: v1`
* Includes foundational resources:

  * Pod
  * Service
  * ConfigMap
  * Secret
  * PersistentVolumeClaim (PVC)

Example:

```yaml
apiVersion: v1
kind: Pod
```

---

## 🔸 Named API Groups

![Image](https://stevenschwenke.de/images/k8s-diagram-with-community-icons.png)

![Image](https://miro.medium.com/0%2AT7C4hJigWVn4LlhK)

![Image](https://miro.medium.com/v2/resize%3Afit%3A1400/1%2Ad7TXtoe4gwq8Rx1BXydyxw.png)

![Image](https://gitlab.eqipe.ch/uploads/-/system/project/avatar/141/cronjob-256.png)

* Explicit group name
* Format: `apiVersion: <group>/<version>`

Examples:

| API Group                      | Resources                 |
| ------------------------------ | ------------------------- |
| `apps/v1`                      | Deployments, StatefulSets |
| `batch/v1`                     | Jobs, CronJobs            |
| `rbac.authorization.k8s.io/v1` | Roles, RoleBindings       |

---

## 📊 Quick Comparison

| Aspect     | Core Group      | Named Group       |
| ---------- | --------------- | ----------------- |
| Group Name | None            | Explicit          |
| Example    | `v1`            | `apps/v1`         |
| Purpose    | Basic resources | Advanced features |

---

# 🔺 The Three Pillars of RBAC

RBAC works like a triangle:

1. **Subject** → WHO (User, Group, ServiceAccount)
2. **Role / ClusterRole** → WHAT (Permissions)
3. **RoleBinding / ClusterRoleBinding** → CONNECTOR

⚠️ You cannot directly give permissions to a user.
You must bind a Role.

---

# 📝 Step 1: Create a Role (Namespace Level)

We create a Role allowing pod read access.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

Apply:

```bash
kubectl apply -f role.yml
kubectl get roles
kubectl describe role pod-reader
```

🔎 Important:
Creating a Role does NOT give access.
It only defines permissions.

---

# 🔗 Step 2: Create RoleBinding

Bind the Role to user `sree`.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: sree
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

Apply:

```bash
kubectl apply -f rolebinding.yml
```

Test again:

```bash
kubectl auth can-i get pods --as sree
```

Output:

```
yes
```

✅ Now `sree` can list pods in the `default` namespace.

---

# 🌍 Namespace vs Cluster Scope

Some resources are:

### Namespace-scoped

* Pods
* Services
* Deployments

### Cluster-scoped

* Nodes
* PersistentVolumes
* Namespaces

Test:

```bash
kubectl auth can-i get nodes --as sree
```

Output:

```
no
```

Because:
Role works only inside a namespace.

---

# 🌎 ClusterRole & ClusterRoleBinding

## Create ClusterRole

```bash
kubectl create clusterrole node-reader \
  --verb=get --verb=list --verb=watch \
  --resource=nodes,services
```

## Bind ClusterRole

```bash
kubectl create clusterrolebinding node-reader-binding \
  --clusterrole=node-reader \
  --user=sree
```

Test:

```bash
kubectl auth can-i get nodes --as sree
```

Output:

```
yes
```

✅ Now `sree` can access cluster-level resources.

---

# 🔑 Configure kubeconfig for User sree

Kubernetes does NOT store users internally.
Access depends on **kubeconfig**.

---

## Verify Certificate Files

```bash
ls /d/Docker\ &\ k8s\ 2026/certificates/
```

Ensure:

* `sree.crt`
* `sree.key`

---

## Add Credentials

```bash
kubectl config set-credentials sree \
  --client-certificate="/d/Docker & k8s 2026/certificates/sree.crt" \
  --client-key="/d/Docker & k8s 2026/certificates/sree.key" \
  --embed-certs=true
```

---

## Create Context

```bash
kubectl config set-context sree \
  --cluster=kind-cka-qacluster \
  --user=sree
```

Switch:

```bash
kubectl config use-context sree
```

Verify:

```bash
kubectl auth whoami
```

Output:

```
Username: sree
Groups: [system:authenticated]
```

---

# 🧪 Final Validation

```bash
kubectl get pods     ✅ Works
kubectl get nodes    ✅ Works
kubectl get svc      ✅ Works (after binding)
```

---

# ⚠ Common Issues Checklist

* ❌ Role created but no RoleBinding
* ❌ Wrong namespace
* ❌ Wrong kubeconfig context
* ❌ Missing certificate files
* ❌ Expired certificate
* ❌ Trying cluster resource with only Role

Check expiry:

```bash
openssl x509 -in sree.crt -text -noout | grep "Not After"
```

---

# 📁 kubeconfig Location

| OS          | Path                               |
| ----------- | ---------------------------------- |
| Linux/macOS | `~/.kube/config`                   |
| Windows     | `%USERPROFILE%\.kube\config`       |
| Git Bash    | `/c/Users/<username>/.kube/config` |

---

# 🧠 Final Summary

| Component          | Scope     | Purpose                          |
| ------------------ | --------- | -------------------------------- |
| Role               | Namespace | Defines permissions              |
| RoleBinding        | Namespace | Grants role to user              |
| ClusterRole        | Cluster   | Defines cluster-wide permissions |
| ClusterRoleBinding | Cluster   | Grants cluster role to user      |

---

# 🎯 Key Takeaway

Authentication = Who you are
Authorization = What you can do

Without RoleBinding:
✔ User exists
❌ No access

With RoleBinding:
✔ User exists
✔ Access granted

---
