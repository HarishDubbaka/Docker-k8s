Here’s a polished **`README.md`** version of your Helm guide, formatted for clarity, step-by-step learning, and ready to use in a project:

---

# Writing and Deploying Your Own Helm Chart

Helm is a package manager for Kubernetes that simplifies deploying, managing, and upgrading applications. This guide walks you through creating a custom Helm chart, deploying it, and managing releases.

---

## Table of Contents

1. [Create a Helm Chart](#create-a-helm-chart)
2. [Modify `values.yaml`](#modify-valuesyaml)
3. [Customize Templates](#customize-templates)
4. [Package the Helm Chart](#package-the-helm-chart)
5. [Install the Chart](#install-the-chart)
6. [Verify Deployment](#verify-deployment)
7. [Customizing Helm Releases](#customizing-helm-releases)
8. [Upgrade and Rollback](#upgrade-and-rollback)
9. [Uninstall Helm Release](#uninstall-helm-release)
10. [Publish Chart to Repository](#publish-chart-to-repository)

---

## Create a Helm Chart

Generate a new Helm chart named `laxmi`:

```bash
helm create laxmi
```

Navigate inside the chart directory:

```bash
cd laxmi
ls
```

You’ll see:

```
Chart.yaml
values.yaml
charts/
templates/
.helmignore
```

View key files:

```bash
cat Chart.yaml   # Chart metadata
cat values.yaml  # Default configuration
```

---

## Modify `values.yaml`

Open `values.yaml` and update with custom values to deploy NGINX:

```yaml
replicaCount: 3

image:
  repository: nginx
  tag: "latest"
  pullPolicy: IfNotPresent

imagePullSecrets: []
nameOverride: ""
fullnameOverride: ""

serviceAccount:
  create: true
  automount: true
  annotations: {}
  name: ""

podAnnotations: {}
podLabels: {}

podSecurityContext: {}
securityContext: {}

service:
  type: LoadBalancer
  port: 80

ingress:
  enabled: false
  className: ""
  annotations: {}
  hosts:
    - host: chart-example.local
      paths:
        - path: /
          pathType: ImplementationSpecific
  tls: []

resources: {}

autoscaling:
  enabled: false
  minReplicas: 1
  maxReplicas: 5
  targetCPUUtilizationPercentage: 80

volumes: []
volumeMounts: []

nodeSelector: {}
tolerations: []
affinity: {}
```

**Key values:**

* `replicaCount` – Number of deployment replicas
* `image` – Container image configuration
* `service` – Service type and port

---

## Customize Templates

### Deployment

Edit `templates/deployment.yaml` to use Helm templating:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-nginx
  labels:
    app: nginx
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - containerPort: {{ .Values.service.port }}
```

### Service

Edit `templates/service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}-nginx
spec:
  type: {{ .Values.service.type }}
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: {{ .Values.service.port }}
      targetPort: {{ .Values.service.port }}
```

---

## Package the Helm Chart

Create a `.tgz` archive:

```bash
helm package laxmi
```

This generates:

```
laxmi-0.1.0.tgz
```

---

## Install the Chart

Deploy to your cluster:

```bash
helm install laxminginx ./laxmi
```

Helm parses templates, substitutes values from `values.yaml`, and applies resources.

---

## Verify Deployment

Check resources:

```bash
helm list
kubectl get pods
kubectl get svc
```

Example status:

```
NAME           NAMESPACE  REVISION  STATUS    CHART       APP VERSION
laxminginx     default    1         deployed  laxmi-0.1.0 1.16.0
```

---

## Customizing Helm Releases

### Using `--set` Flag

Override values:

```bash
helm install shivapodnginx ./laxmi --set replicaCount=4
```

### Using a Custom Values File

Create `my-values.yaml`:

```yaml
replicaCount: 4
service:
  type: LoadBalancer
  port: 8080
```

Deploy with:

```bash
helm install my-nginx ./laxmi -f my-values.yaml
```

---

## Upgrade and Rollback

Upgrade release:

```bash
helm upgrade shivapodnginx ./laxmi --set replicaCount=5
```

Check history:

```bash
helm history shivapodnginx
```

Rollback to revision 1:

```bash
helm rollback shivapodnginx 1
```

---

## Uninstall Helm Release

Remove deployment:

```bash
helm uninstall laxminginx
```

Confirm deletion:

```bash
helm list
kubectl get all
```

---

## Publish Chart to Repository

### Option 1 — GitHub Pages

1. Create repo `helm-charts`
2. Generate `index.yaml`:

```bash
helm repo index . --url https://<username>.github.io/helm-charts
```

3. Commit chart `.tgz` and `index.yaml`
4. Enable GitHub Pages
5. Add repo in Helm:

```bash
helm repo add laxmi-repo https://<username>.github.io/helm-charts
helm repo update
```

Install chart from repo:

```bash
helm install myapp laxmi-repo/laxmi
```

---

### Option 2 — OCI Registry (Modern)

1. Login:

```bash
helm registry login registry-1.docker.io
```

2. Push chart:

```bash
helm push laxmi-0.1.0.tgz oci://registry-1.docker.io/<username>
```

3. Install:

```bash
helm install myapp oci://registry-1.docker.io/<username>/laxmi --version 0.1.0
```

---

**Congratulations!** 🎉 You’ve successfully created, customized, deployed, and managed a Helm chart. You can now:

* Package and distribute charts
* Upgrade and rollback releases
* Publish charts to repositories

---

Do you want me to also **include a sample diagram showing Helm chart structure** for this README? It makes it much easier to visualize for beginners.

