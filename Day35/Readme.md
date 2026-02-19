# Kubernetes Authentication & User Certificate Creation (kind Cluster)

Kubernetes is a powerful container orchestration system, but securing access to its resources is crucial. Kubernetes enforces security through three key mechanisms:

- **Authentication (AuthN)** — Identifies who is making the request.
- **Authorization (AuthZ)** — Determines what actions the authenticated user can perform.
- **Admission Controllers** — Enforce policies and validations before executing requests.

In this guide, we will:
- Understand Kubernetes authentication
- Create a new user (`sree`)
- Generate and approve a client certificate
- Provide secure access in a **kind cluster**

We are using a **kind cluster** running inside Docker:

---

# Authentication vs Authorization

### Authentication (AuthN)
Verifies identity  
👉 **“Who are you?”**

### Authorization (AuthZ)
Verifies permissions  
👉 **“What can you do?”**

This guide focuses mainly on **authentication**, but both are essential.

---

# How Authentication Works in Kubernetes

All requests go through the **Kubernetes API Server**.

Flow:

1. Authentication  
2. Authorization  
3. Admission Control  
4. Execute Request  

Kubernetes does **not** store users internally like traditional systems.  
Instead, it validates identities using external authentication mechanisms.

---

# Authentication Methods

Kubernetes supports:

- **Client Certificates**
- **Bearer Tokens**
- **OIDC (OpenID Connect)**
- **Webhook Token Authentication**

In this guide, we use **Client Certificates**.

---

# Authorization Methods

- **RBAC (Role-Based Access Control)** ✅ (Most Common)
- **ABAC**
- **Webhook Authorization**

---

# kind Cluster Notes

Since we are using **kind**, Docker containers act as VM nodes.

```bash
docker exec -it cka-qacluster-control-plane //bin/bash
```

Important paths inside control-plane container:

```
Kubeconfig:
/etc/kubernetes/admin.conf

PKI Directory:
/etc/kubernetes/pki/
```

---

# Scenario

We have a new engineer **sree** joining the team.

We want:
- Secure authentication
- Certificate-based access
- Admin approval workflow

Reference:  
https://kubernetes.io/docs/tasks/tls/certificate-issue-client-csr/

---

# Step 1: Create Private Key (User Side)

> ⚠️ This must be done by the user.  
> The private key must NEVER be shared.

```bash
openssl genrsa -out sree.key 3072
```

---

# Step 2: Create Certificate Signing Request (CSR)

* CN = Username  
* O = Group (optional, for RBAC)

```bash
openssl req -new -key sree.key -out sree.csr -subj "/CN=sree"
```

Generated files:

```
sree.key   (PRIVATE – DO NOT SHARE)
sree.csr   (Safe to share)
```

---

# Step 3: Encode CSR

Admin encodes the CSR:

```bash
cat sree.csr | base64 | tr -d "\n"
```

---

# Step 4: Create Kubernetes CSR Object

Create `csr.yml`:

```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: sree
spec:
  request: <base64-encoded-csr>
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 86400
  usages:
    - client auth
```

Apply:

```bash
kubectl apply -f csr.yml
```

---

# Step 5: Check CSR Status

```bash
kubectl get csr
```

---

# Step 6: Approve CSR (Admin Action)

```bash
kubectl certificate approve sree
```

---

# Step 7: Extract Issued Certificate

```bash
kubectl get csr sree -o yaml > issuecert.yaml
```

---

# Step 8: Decode Certificate

```bash
echo "<base64-certificate>" | base64 -d > sree.crt
```

---

# Step 9: Provide Certificate to User

Admin provides:

* ✅ `sree.crt`

User already has:

* ✅ `sree.key`

Configure kubeconfig using both.

---

# Important Security Rules

✔️ Private key must never be shared  
✔️ Only admin approves CSRs  
✔️ Use RBAC to restrict access  
✔️ Certificates can have expiration  

---

# Summary

We successfully:

* Created a private key  
* Generated CSR  
* Submitted CSR to Kubernetes  
* Approved request  
* Issued certificate  
* Delivered certificate securely  

---

# 🔐 How CSR Works in Cloud Kubernetes

In managed cloud clusters like:

* Amazon EKS
* Google Kubernetes Engine
* Azure Kubernetes Service

you **do not directly control the control-plane PKI** like you do in kind.

---

# ☁️ Now: What Happens in Cloud Kubernetes?

## 🚨 Important Difference

In cloud-managed clusters:

> You **do NOT have access to the cluster CA private key**

So:

* You cannot manually sign certificates using cluster CA  
* You usually don’t create users using CSR for human access  

---

# 👤 How Authentication Works in Cloud Clusters

Cloud providers use **their own identity systems instead of Kubernetes client certificates**.

---

## 🟡 In Amazon EKS

Authentication uses:

* AWS IAM  
* `aws-auth` ConfigMap  
* AWS STS tokens  

https://docs.aws.amazon.com/eks/latest/userguide/auth-configmap.html
---

## 🔵 In Google GKE

Authentication uses:

* Google IAM  
* gcloud auth  
* OIDC  

https://docs.cloud.google.com/kubernetes-engine/docs/authentication
---

## 🔷 In Azure AKS

Authentication uses:

* Azure AD  
* Entra ID  
* OIDC  

https://learn.microsoft.com/en-us/azure/aks/enable-authentication-microsoft-entra-id
---

# 🤔 So When Is CSR Used in Cloud Kubernetes?

CSR is still used for:

1. Node Certificates  
2. Service Mesh  
3. Custom Controllers  
4. Internal mTLS  

---

# 📊 Summary Comparison

| Feature               | kind / Self-Managed | Cloud Kubernetes |
| --------------------- | ------------------- | ---------------- |
| Access to CA key      | ✅ Yes               | ❌ No             |
| Manual CSR for users  | ✅ Common            | ❌ Rare           |
| IAM integration       | ❌                   | ✅ Yes            |
| Recommended user auth | Client cert         | IAM/OIDC         |

---

# 🎯 What Should You Use in Cloud?

For new engineer (like sree):

- **In kind:** Use CSR (what you did).  
- **In EKS:** Create IAM user → map to RBAC.  
- **In GKE:** Grant IAM role.  
- **In AKS:** Add user to Azure AD group → bind RBAC.  

Cloud Kubernetes shifts authentication responsibility to cloud IAM systems.
```
