# Cert-Manager Operator Setup using OLM

This guide explains how to install and configure the **cert-manager Operator** using **Operator Lifecycle Manager (OLM)** in Kubernetes.

Cert-manager automates the management and issuance of TLS certificates inside Kubernetes clusters.

---

# Step 1: Install Operator Lifecycle Manager (OLM)

First install **Operator Lifecycle Manager**, which manages Kubernetes Operators.

OLM will create the `olm` namespace and deploy required components.

## Install OLM

```bash
curl -sL https://github.com/operator-framework/operator-lifecycle-manager/releases/download/v0.32.0/install.sh | bash -s v0.32.0
````

## Install cert-manager Operator

```bash
kubectl create -f https://operatorhub.io/install/cert-manager.yaml
```

## Verify OLM Pods

```bash
kubectl get pods -n olm
```

You should see pods like:

* `olm-operator`
* `catalog-operator`
* `packageserver`

All should be in **Running** state.

## Verify Operator Installation

```bash
kubectl get csv -n operators
```

Expected output:

```
NAME                          DISPLAY          VERSION   PHASE
cert-manager.vX.X.X           cert-manager     X.X.X     Succeeded
```

---

# Step 2: Verify Operator Components

After installation, OLM deploys the **cert-manager Operator**, which then deploys the actual cert-manager components.

## Check Operator Pods

```bash
kubectl get pods -n operators
```

You should see:

* `cert-manager`
* `cert-manager-cainjector`
* `cert-manager-webhook`
* `cert-manager-operator`

---

## Verify Custom Resource Definitions (CRDs)

Cert-manager introduces several CRDs.

```bash
kubectl get crds | grep cert-manager
```

Expected CRDs:

```
certificates.cert-manager.io
issuers.cert-manager.io
clusterissuers.cert-manager.io
certificaterequests.cert-manager.io
```

---

# Step 3: Prepare cert-manager Configuration

## Create OperatorGroup

An **OperatorGroup** defines which namespaces the operator watches.

Create file:

`cert-manager-operatorgroup.yaml`

```yaml
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: cert-manager-operatorgroup
  namespace: operators
spec:
  targetNamespaces:
  - operators
```

Apply it:

```bash
kubectl apply -f cert-manager-operatorgroup.yaml
```

---

## Check Custom Resources (Initial State)

At this stage, no certificates exist yet.

```bash
kubectl get issuer -n default
kubectl get certificate -n default
```

Expected output:

```
No resources found
```

---

# Step 4: Create Issuer and Certificate

## Create Self-Signed Issuer

This issuer will generate certificates inside the `default` namespace.

Create file:

`selfsigned-issuer.yaml`

```yaml
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: selfsigned-issuer
  namespace: default
spec:
  selfSigned: {}
```

Apply:

```bash
kubectl apply -f selfsigned-issuer.yaml
```

Verify issuer:

```bash
kubectl get issuer selfsigned-issuer -n default
```

Expected:

```
NAME               READY
selfsigned-issuer  True
```

---

## Create Certificate

Create file:

`my-app-certificate.yaml`

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: my-app-certificate
  namespace: default
spec:
  secretName: my-app-tls
  dnsNames:
  - my-app.example.com
  issuerRef:
    name: selfsigned-issuer
    kind: Issuer
```

Apply:

```bash
kubectl apply -f my-app-certificate.yaml
```

---

# Step 5: Observe Operator Reconciliation

The cert-manager Operator will process the certificate request.

## Check Certificate Status

```bash
kubectl get certificate my-app-certificate -n default
```

Initially:

```
READY False
```

After certificate creation:

```
READY True
```

---

## Verify TLS Secret

Cert-manager automatically creates a TLS secret.

```bash
kubectl get secret my-app-tls -n default
```

Expected:

```
TYPE: kubernetes.io/tls
```

Inspect secret:

```bash
kubectl describe secret my-app-tls -n default
```

The certificate and private key are stored in **base64 encoded format**.

---

## View cert-manager Logs (Optional)

You can inspect operator logs.

```bash
kubectl logs -f -n operators $(kubectl get pods -n operators -l app.kubernetes.io/component=controller -o jsonpath='{.items[0].metadata.name}')
```

Logs will show:

* Certificate processing
* CertificateRequest creation
* TLS Secret generation

This demonstrates the **Operator controller reconciliation loop**.

---

# Step 6: Clean Up

After completing the lab, remove resources.

## Delete Custom Resources

```bash
kubectl delete -f my-app-certificate.yaml
kubectl delete -f selfsigned-issuer.yaml
```

---

## Delete Operator Resources

```bash
kubectl delete -f cert-manager-subscription.yaml
kubectl delete -f cert-manager-operatorgroup.yaml
```

---

## Delete Namespaces

```bash
kubectl delete namespace operators
kubectl delete namespace olm
```

---

# Final Architecture

```
User creates Certificate CR
        │
        ▼
cert-manager Operator
        │
        ▼
Creates CertificateRequest
        │
        ▼
Issuer signs certificate
        │
        ▼
TLS Secret created
        │
        ▼
Application uses TLS Secret
```

---

