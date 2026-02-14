# 🚀 Helm – Kubernetes Package Manager

Kubernetes is the most popular container orchestration platform, but it becomes significantly more powerful when combined with ecosystem tools. Among these tools, **Helm** stands out as the package manager for Kubernetes.

If you've used package managers like **YUM, DNF, or APT**, you already understand the concept.
Those tools manage RPM/DEB packages.
👉 **Helm manages Kubernetes applications using Charts.**

---

# 📦 What is Helm?

![Image](https://helm.sh/img/helm.svg)

![Image](https://devopscube.com/content/images/2025/03/helm-chart-drawio-1.png)

![Image](https://docs.bitnami.com/images/img/platforms/kubernetes/k8-tutorial-41.png)

![Image](https://miro.medium.com/1%2AAHFbMN0FwatRz5uP7_bC8A.png)

**Helm** is a package manager for Kubernetes that simplifies application deployment and lifecycle management.

Managing Kubernetes apps manually requires multiple YAML files:

* Deployments
* Services
* ConfigMaps
* Secrets
* Ingress
* RBAC

As applications grow, this becomes complex and error-prone.

Helm solves this by:

* Packaging all resources into **Helm Charts**
* Enabling version control
* Supporting rollbacks
* Managing dependencies

---

# 🤔 Why Use Helm?

## ✅ Simplified Deployments

Deploy complex applications with a single command:

```bash
helm install my-app chart-name
```

## ✅ Parameterization & Reusability

Use `values.yaml` to configure:

* Dev
* Staging
* Production

## ✅ Version Control & Rollbacks

Helm tracks releases and allows:

```bash
helm rollback <release-name> <revision>
```

## ✅ Dependency Management

Charts can depend on other charts.

## ✅ CI/CD & GitOps Integration

Works seamlessly with:

* Argo CD
* Flux
* GitHub Actions

---

# ⚙️ How Helm Works

![Image](https://developer.ibm.com/developer/default/blogs/kubernetes-helm-3/images/helm3-arch.png)

![Image](https://razorops.com/images/blog/helm-3-tree.png)

![Image](https://user-images.githubusercontent.com/9508513/177470720-e537f110-a6d8-4f1e-8310-5cc86f5ddb58.png)

![Image](https://devopscube.com/content/images/2025/03/helm-template-1.png)

Helm works using **Charts**, which contain Kubernetes resource definitions.

### 🔹 Workflow

1. User runs `helm install`
2. Helm reads:

   * `Chart.yaml`
   * `values.yaml`
   * Templates
3. Templates are rendered using values
4. Generated manifests are sent to Kubernetes API
5. Resources are deployed to cluster

---

# 📁 What is a Helm Chart?

A **Helm Chart** is a collection of files describing a related set of Kubernetes resources.

### Chart Structure

```
my-chart/
│
├── Chart.yaml        # Metadata about the chart
├── values.yaml       # Default configuration values
├── templates/        # Kubernetes manifest templates
├── charts/           # Chart dependencies
└── README.md         # Documentation
```

### 📌 Key Files

| File          | Purpose                                     |
| ------------- | ------------------------------------------- |
| `Chart.yaml`  | Chart metadata (name, version, description) |
| `values.yaml` | Default configurable values                 |
| `templates/`  | Kubernetes YAML templates                   |
| `charts/`     | Dependencies                                |
| `README.md`   | Usage instructions                          |

Helm promotes:

* Reusability
* Standardization
* Versioned deployments

---

# 📦 What is a Helm Chart Repository?

A **Helm Chart Repository** stores packaged charts.

It contains:

* Packaged `.tgz` chart files
* `index.yaml` (metadata about charts)

Repositories can be:

* Public
* Private

Popular public repos include:

* Bitnami
* Official Helm Hub

Add a repo:

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
```

---

# 🏗 Helm Architecture

![Image](https://developer.ibm.com/developer/default/blogs/kubernetes-helm-3/images/helm3-arch.png)

![Image](https://www.researchgate.net/publication/363933692/figure/fig3/AS%3A11431281087092686%401664459497019/Helm-client-connects-and-installs-the-helm-chart-to-the-Kubernetes-cluster-by-interacting.png)

![Image](https://cdn.prod.website-files.com/67f18e860e54dbf2b97a05c6/6828e94cc207ec8e35f83566_image-10.png)

![Image](https://glasskube.dev/assets/images/helm-workflow-diagram-73ec11046f99e2e990ce3cabc5b6105c.png)

Helm 3 architecture consists of:

## 1️⃣ Helm Client

CLI used to:

* Install charts
* Upgrade releases
* Rollback deployments
* Uninstall applications

## 2️⃣ Helm Library

Core engine that:

* Renders templates
* Manages releases
* Communicates with Kubernetes API

> Note: Helm 3 removed Tiller (used in Helm 2).

---

# 🛠 Installing Helm

Official installation guide:
👉 [https://helm.sh/docs/intro/install](https://helm.sh/docs/intro/install)

Helm supports:

* macOS
* Linux
* Windows

---

# 🔍 Verify Installation

```bash
helm version
```

Short version:

```bash
helm version --short
```

Example output:

```
v3.18.2+g04cad46
```

---

# ⚙️ Configuring Helm

## ➕ Add Repository

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
```

Verify:

```bash
helm repo list
```

---

## 🔄 Update Repositories

```bash
helm repo update
```

---

## 🔎 Search for Charts

```bash
helm search repo nginx
```

---

# 🚀 Install a Helm Chart

Deploy NGINX:

```bash
helm install laxmi-nginx bitnami/nginx
```

What happens:

* Downloads chart
* Renders templates
* Deploys resources
* Creates release named `laxmi-nginx`

---

# 📊 Check Deployment Status

List releases:

```bash
helm list
```

Check specific release:

```bash
helm status laxmi-nginx
```

---

# 🗑 Uninstall a Release

```bash
helm uninstall laxmi-nginx
```

This removes:

* Deployment
* Services
* ConfigMaps
* Associated resources

---

# 🎯 Summary

Helm is essential for production Kubernetes environments.

It provides:

* Simplified application deployment
* Versioned releases
* Rollbacks
* Dependency management
* CI/CD integration
* Environment-based configuration

If Kubernetes is the operating system of the cloud,
👉 **Helm is its package manager.**

---

# 📌 Next Steps

To deepen your Helm knowledge:

* Create your own chart:

  ```bash
  helm create my-chart
  ```
* Explore chart structure
* Practice upgrading and rolling back
* Integrate with GitOps tools

---

**Happy Helming! ⛵**
