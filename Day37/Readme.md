# Kubernetes Service Accounts & API Access Guide

---

## 🧠 Kubernetes: Humans vs Agents

In Kubernetes, humans initiate commands, but the real power lies in the agents that continuously enforce and automate those commands.

### 🔹 Breaking It Down

### 👤 Human Role
- Admins or developers run `kubectl` commands.
- They define the **desired state**.
- Example:
  > “I want 3 replicas of this Pod.”

### 🤖 Agent Role
- Kubernetes controllers, scheduler, and operators act as agents.
- They continuously:
  - Watch cluster state
  - Compare desired vs actual state
  - Reconcile differences

### 🔁 Example
If one Pod crashes, the ReplicaSet controller automatically creates a new one — **no human intervention needed**.

This is called the **Reconciliation Loop**.

---

# 🔐 What is a Service Account in Kubernetes?

A **Service Account** is a special Kubernetes account used by applications running inside Pods to authenticate with the Kubernetes API.

- 👤 User Accounts → Humans
- 🤖 Service Accounts → Applications / Workloads

Service Accounts:
- Provide identity to Pods
- Enable secure API communication
- Work with RBAC for fine-grained permissions

---

# ⚙️ How a Service Account Works

1. You create a ServiceAccount.
2. Kubernetes generates a token.
3. The token is mounted inside the Pod.
4. The application uses the token to authenticate to the API.
5. RBAC rules determine what actions are allowed.

Service Accounts act as **identity for workloads**.

---

# 🚀 Step-by-Step: Setup Service Account & API Access

---

## ✅ Step 1: Create a Service Account

### Option 1: Using YAML

Create `service-account.yaml`

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: promethusagent1
```

Apply it:

```bash
kubectl apply -f service-account.yaml
```

### Option 2: Using Command

```bash
kubectl create serviceaccount promethusagent
```

Check service accounts:

```bash
kubectl get serviceaccount
```

Example output:

```
NAME                  SECRETS   AGE
default               0         18d
promethusagent        0         2m
promethusagent1       0         24s
```

---

## ❓ Why SECRETS: 0?

In newer Kubernetes versions:

- ServiceAccounts no longer auto-create long-lived secrets.
- Kubernetes now uses **Projected Service Account Tokens**.
- Tokens are:
  - Short-lived
  - More secure
  - Auto-rotated
  - Mounted directly into Pods

That’s why you see `SECRETS: 0`.

This is expected behavior.

---

# 🔐 Setup Kubernetes API Access Using Service Account Token

---

## 🏷 Step 1: Create Namespace

```bash
kubectl create namespace monitoring-tools
```

Verify:

```bash
kubectl get ns -A
```

---

## 👤 Step 2: Create Service Account in Namespace

```bash
kubectl create serviceaccount api-service-account -n monitoring-tools
```

Or YAML:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: api-service-account
  namespace: monitoring-tools
```

Apply:

```bash
kubectl apply -f apiserver.yml
```

---

## 🛡 Step 3: Create a ClusterRole

Create `api-cluster-role`:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: api-cluster-role
rules:
  - apiGroups:
      - ""
      - apps
      - autoscaling
      - batch
      - extensions
      - policy
      - rbac.authorization.k8s.io
    resources:
      - pods
      - componentstatuses
      - configmaps
      - daemonsets
      - deployments
      - events
      - endpoints
      - horizontalpodautoscalers
      - ingress
      - jobs
      - limitranges
      - namespaces
      - nodes
      - persistentvolumes
      - persistentvolumeclaims
      - resourcequotas
      - replicasets
      - replicationcontrollers
      - serviceaccounts
      - services
    verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
```

Apply:

```bash
kubectl apply -f api-serverrole.yml
```

⚠️ Best Practice:  
Avoid giving full cluster access unless required.

List available resources:

```bash
kubectl api-resources
```

---

## 🔗 Step 4: Create ClusterRoleBinding

Bind Service Account to ClusterRole.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: api-cluster-role-binding
subjects:
  - kind: ServiceAccount
    name: api-service-account
    namespace: monitoring-tools
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: api-cluster-role
```

Apply:

```bash
kubectl apply -f clusterrolebindingapiserver.yml
```

---

## ✅ Step 5: Validate Access Using kubectl

Check permissions:

```bash
kubectl auth can-i get pods \
--as=system:serviceaccount:monitoring-tools:api-service-account
```

Output:

```
yes
```

Check delete deployments:

```bash
kubectl auth can-i delete deployments \
--as=system:serviceaccount:monitoring-tools:api-service-account
```

---

## 🔑 Step 6: Create Long-Lived Token (Manual Secret)

Create `sa-token.yaml`:

```yaml
apiVersion: v1
kind: Secret
type: kubernetes.io/service-account-token
metadata:
  name: api-service-account-token
  namespace: monitoring-tools
  annotations:
    kubernetes.io/service-account.name: api-service-account
```

Apply:

```bash
kubectl apply -f secret.yml
```

Extract token:

```bash
kubectl get secret api-service-account-token \
-o=jsonpath='{.data.token}' \
-n monitoring-tools | base64 --decode
```

---

## 🌐 Step 7: Get API Server Endpoint

Option 1:

```bash
kubectl cluster-info
```

Option 2:

```bash
kubectl get endpoints | grep kubernetes
```

Example:

```
kubernetes   172.18.0.5:6443
```

---

## 🌍 Step 8: Validate Access Using CURL

```bash
curl -k https://172.18.0.5:6443/api/v1/namespaces \
-H "Authorization: Bearer <YOUR_TOKEN>"
```

You can also test using Postman.

![Image Alt](image_url)

---

# 🔐 Authentication vs Authorization

| Component | Purpose |
|------------|----------|
| Token | Authentication (Who are you?) |
| ClusterRole | Authorization rules (What can you do?) |
| ClusterRoleBinding | Connects identity to permissions |

---

# 🎯 Key Takeaways

- Humans define desired state.
- Controllers enforce it automatically.
- Service Accounts provide identity to workloads.
- RBAC controls permissions.
- Tokens allow secure API access.
- Kubernetes follows a continuous reconciliation model.

---

# 📌 Summary

This guide covered:

- Kubernetes control loop concept
- Service Account fundamentals
- RBAC setup (ClusterRole + Binding)
- Manual token creation
- Direct API access using curl
- Permission validation with `kubectl auth can-i`

---

