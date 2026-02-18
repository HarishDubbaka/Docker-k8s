# 📘 Understanding Kustomize

With **Kustomize**, you don’t need to apply each YAML file individually.  
Instead, you list all your resource files inside a single `kustomization.yaml` file and deploy them together using:

```
kubectl apply -k <directory>
```

Kustomize allows you to manage and customize Kubernetes configurations in a clean, structured, and reusable way.

kustomization.yamlfile

Base and Overlays

Transformers

Patches

Let's take a look at each concept.

---

## 🔑 Key Concepts in Kustomize

### 1. **kustomization.yaml**
- The **entry point** for Kustomize.
- Defines what resources to include and how to customize them.
- Think of it as the **recipe card** that tells Kustomize what ingredients (resources) to use and what adjustments (patches/transformers) to apply.

---

### 2. **Base and Overlays**
- **Base**: A reusable set of manifests (e.g., Deployment, Service).  
  → Example: a generic app definition.
- **Overlay**: Environment-specific customization layered on top of the base.  
  → Example: dev overlay sets replicas=1, prod overlay sets replicas=5.
- Analogy: Base = “standard pizza dough,” Overlays = “different toppings for each customer.”

---

### 3. **Transformers**
- Declarative modifiers that apply **bulk changes** across resources.
- Examples:
  - Add a namespace to all resources.
  - Add labels or annotations consistently.
  - Prefix names for uniqueness.
- Analogy: Transformers = “seasoning shakers” that sprinkle labels, prefixes, or namespaces across all manifests.

---

### 4. **Patches**
- Fine-grained edits to specific resources.
- Two types:
  - **Strategic Merge Patch**: Merge YAML fields into existing manifests.
  - **JSON Patch (6902)**: Apply add/replace/remove operations.
- Analogy: Patches = “precise stitches” to fix or alter details in one resource without touching others.

---

### Clear Takeaway
- **Base** = reusable foundation.  
- **Overlay** = environment-specific tweaks.  
- **Transformers** = broad, consistent adjustments.  
- **Patches** = targeted, surgical changes.  
- **kustomization.yaml** = the conductor orchestrating all of this.

---

# 🚀 Deploy Application Using Kustomize

This guide demonstrates how **Kustomize** can be used to deploy an application across multiple environments, such as **dev** and **prod**.  

> **Note:** For simplicity, this example uses only two environments. In real-world projects, manifests may be more complex and involve multiple objects and additional environments.

---

## 📌 Scenario

We need to deploy an **Nginx web server** in two environments:

- **Dev Environment**
  - Deployment with **2 replicas**
  - **NodePort** service
  - Lower CPU and memory resources

- **Prod Environment**
  - Deployment with **4 replicas**
  - Different CPU and memory limits
  - **RollingUpdate** strategy
  - Service without NodePort

Kustomize allows us to achieve this by using a **Base + Overlay + Patch** approach, avoiding duplication and keeping environment-specific configurations separate.

---

## 📂 GitHub Repository

All the manifests used in this guide are hosted in the **Kustomize GitHub Repo**:

```bash
git clone https://github.com/techiescamp/kustomize.git
````

---

## 📁 Directory Structure

After cloning, the project directory structure looks like this:

```
├── kustomize
│   ├── base
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   └── kustomization.yaml
│   └── overlays
│       ├── dev
│       │   ├── deployment-dev.yaml
│       │   ├── service-dev.yaml
|       |   ├── index-dev.html
│       │   └── kustomization.yaml
│       └── prod
│           ├── deployment-prod.yaml
│           ├── service-prod.yaml
│           └── kustomization.yaml
```

* **base/** → Contains reusable manifests common to all environments
* **overlays/dev/** → Dev-specific changes and patches
* **overlays/prod/** → Production-specific changes and patches

---

This setup allows Kustomize to build environment-specific manifests efficiently without duplicating the full YAML files.

```
```

## 🏗️ Base Folder

The **base folder** contains the common deployment and service YAMLs along with a `kustomization.yaml`.  
These files define the shared configuration across environments.

### base/deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

### base/service.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  selector:
    app: web
  ports:
  - name: http
    port: 80
```

### base/kustomization.yaml
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
- deployment.yaml
- service.yaml
```

---

# 🚀 Dev Overlay Configuration

In this section, we define the **Dev overlay** for our application.

The goal of the Dev environment is:

* Increase replicas from 1 → 2
* Reduce CPU and memory limits
* Change Service type to `NodePort`
* Generate a dev-specific ConfigMap

Instead of modifying the base files directly, we apply **patches** using Kustomize.

---

# 📁 Dev Overlay Structure

```
overlays/dev/
├── deployment-dev.yaml
├── service-dev.yaml
├── index-dev.html
└── kustomization.yaml
```

# 📦 Deployment Patch (deployment-dev.yaml)

We only define the fields that need to change.

Kustomize compares this file with the base Deployment and applies the modifications.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deployment
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: nginx
        resources:
          limits:
            cpu: "200"
            memory: "256Mi"
          requests:
            cpu: "100"
            memory: "128Mi"
```

### ✅ What Changed?

* Replicas increased
* CPU and memory limits reduced for dev
* Resource requests adjusted

> Only the modified fields are defined — the rest comes from the Base.

---

# 🌐 Service Patch (service-dev.yaml)

In Dev, we expose the service using `NodePort`.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  type: NodePort
```

### ✅ What Changed?

* Service type updated to `NodePort`

---

# 📝 Dev HTML ConfigMap (index-dev.html)

This file is used in the Dev overlay to generate a ConfigMap containing a simple HTML page for the Dev environment.

```html
<html>
  <body>
    <h1>Welcome to Dev Environment</h1>
  </body>
</html>
```

### ✅ What Changed?
Added a Dev-specific welcome page

Will be included in the ConfigMap index-html-configmap

# ⚙️ Dev kustomization.yaml

This file ties everything together.

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
- ../../base

patches:
- path: deployment-dev.yaml
- path: service-dev.yaml

generatorOptions:
  labels:
    fruit: apple

configMapGenerator:
- name: index-html-configmap
  behavior: replace
  files:
  - index-dev.html
```

---

## 🔎 Explanation

### `resources`

Points to the Base directory.

```
resources:
- ../../base
```

This tells Kustomize:

> “Start from the base configuration.”

---

### `patches`

Applies Strategic Merge Patches.

```
patches:
- path: deployment-dev.yaml
- path: service-dev.yaml
```

Kustomize:

* Finds matching resources in base
* Merges only the changed fields

---

### `configMapGenerator`

Generates a ConfigMap from a file:

```
configMapGenerator:
- name: index-html-configmap
  behavior: replace
  files:
  - index-dev.html
```

* Creates or replaces a ConfigMap
* Uses contents from `index-dev.html`

---

# 🔍 Review the Rendered Output

Before applying, review the generated manifests.

### If inside `overlays/dev`:

```
kubectl kustomize .
```

### If at project root:

```
kubectl kustomize overlays/dev
```

This renders the final Kubernetes manifests after applying:

* Base configuration
* Deployment patch
* Service patch
* Generated ConfigMap

---

# 🚀 Deploy the Dev Environment

### If inside `overlays/dev`:

```
kubectl apply -k .
```

### If at project root:

```
kubectl apply -k overlays/dev
```

---


# 🚀 Prod Overlay Configuration

In this section, we define the **Production overlay** for our application.

The goal of the Production environment is:

* Increase replicas to 4
* Set higher CPU and memory limits
* Add a RollingUpdate deployment strategy
* Modify the Service type
* Keep the Base configuration reusable

Instead of modifying the base manifests directly, we apply **Strategic Merge Patches**.

---

# 📁 Prod Overlay Structure

```
overlays/prod/
├── deployment-prod.yaml
├── service-prod.yaml
└── kustomization.yaml
```

---

# 📦 Deployment Patch (deployment-prod.yaml)

This patch modifies:

* Replica count
* CPU and memory resources
* Deployment update strategy

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deployment
spec:
  replicas: 4
  template:
    spec:
      containers:
      - name: nginx
        resources:
          limits:
            cpu: "1"
            memory: "1Gi"
          requests:
            cpu: "500m"
            memory: "512Mi"
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
```

---

## ✅ What Changed?

* Replicas increased to **4**
* CPU and memory limits increased
* RollingUpdate strategy added
* Ensures zero-downtime deployments

> Only modified fields are defined — remaining configuration comes from the Base.

---

# 🌐 Service Patch (service-prod.yaml)

We modify the Service type for production.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  type: NodePort
```

---

## ✅ What Changed?

* Service type updated as required for production exposure.

---

# ⚙️ Prod kustomization.yaml

This file connects the base configuration with production patches.

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
- ../../base

patches:
- path: deployment-prod.yaml
- path: service-prod.yaml
```

---

## 🔎 Explanation

### `resources`

```
resources:
- ../../base
```

This tells Kustomize:

> “Start from the base manifests.”

---

### `patches`

```
patches:
- path: deployment-prod.yaml
- path: service-prod.yaml
```

Kustomize:

* Matches resources by name
* Merges only the changed fields
* Keeps everything else from the base

---

# 🔍 Review the Rendered Output

Before applying changes, always review the generated manifests.

### Using Kustomize CLI:

```
kustomize build overlays/prod
```

### Using kubectl:

```
kubectl kustomize overlays/prod
```

---

# 🚀 Deploy the Prod Environment

### Option 1 – Using Kustomize CLI

```
kustomize build overlays/prod | kubectl apply -f -
```

### Option 2 – Using kubectl (Recommended)

```
kubectl apply -k overlays/prod
```

---

## 📂 Path Usage Reference

### If inside `~/kustomize/overlays/prod`:

```
kubectl kustomize .
kubectl apply -k .
```

### If inside project root `~/kustomize`:

```
kubectl kustomize overlays/prod
kubectl apply -k overlays/prod
```

---

# ✅ Verify Deployment

After successful deployment, verify resources:

```
kubectl get deployments
kubectl get services
kubectl get pods
```

Or view everything:

```
kubectl get all
```

---

## 📊 Expected Output (Example)

```
NAME                                 READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/web-deployment       4/4     4            4           22s

NAME                 TYPE        CLUSTER-IP     PORT(S)
service/web-service  NodePort    10.101.21.45   8080:32510/TCP

NAME                                           READY   STATUS
pod/web-deployment-xxxxx                      1/1     Running
```

You should see:

* 4 running pods
* NodePort service
* RollingUpdate strategy applied

---

# 🎯 Key Takeaway

The Production overlay:

* Reuses the Base configuration
* Applies production-specific patches
* Adds scaling and update strategy
* Keeps environment separation clean
* Avoids YAML duplication

This demonstrates the power of **Base + Overlays + Patches** in Kustomize.


Here is your content formatted cleanly as a **README.md** section.

---

# ✅ Benefits of Using Kustomize

Kustomize provides a powerful and flexible way to manage Kubernetes configurations without modifying the original YAML files. Below are the key benefits:

---

## 🔹 Simplified Configuration Management

Kustomize is easy to use and helps manage Kubernetes configurations in a structured and modular way.

* Organize resources clearly
* Separate base and environment-specific configurations
* Apply changes declaratively

This improves readability and maintainability of your manifests.

---

## 🔹 Reusability

With Kustomize:

* A single **Base** configuration can be reused across multiple environments.
* Overlays define only what changes per environment.

This avoids duplication and reduces the need to create separate YAML files for every deployment.

---

## 🔹 Version Control Friendly

Kustomize works directly with standard YAML files, making it:

* Easy to track changes in Git
* Simple to review differences
* Convenient to roll back to previous configurations

---

## 🔹 Template-Free

Kustomize does not use a templating language.

* No placeholders
* No template syntax
* Pure Kubernetes YAML

It leverages the full power of the Kubernetes API without parameterizing every line (unlike Helm).

---

## 🔹 Native kubectl Support

Kustomize is built into `kubectl` (version 1.14+).

You can deploy using:

```
kubectl apply -k <directory>
```

No separate installation is required.

---

## 🔹 Built-in Transformers

Kustomize includes built-in transformers that can:

* Add labels
* Add namespaces
* Modify images
* Add prefixes or suffixes

It can also be extended via plugins.

---

## 🔹 Standalone CLI Tool

Kustomize is available as:

* A standalone CLI tool
* A Go-based package

This makes it easy to integrate into automation tools and CI/CD workflows.

---

## 🔹 Declarative Configuration Updates

Kustomize allows declarative configuration changes without modifying the original manifests.

This keeps your configuration clean, reusable, and environment-aware.

---

# 📌 Kustomize Best Practices

Following best practices ensures better organization and maintainability.

---

## 📁 1. Separate Base and Overlays

Keep:

* Base resources
* Overlays
* Patches

In separate directories to maintain clarity and separation between environments.

---

## 📌 2. Follow Kubernetes Best Practices

While using Kustomize:

* Use proper naming conventions
* Define resource limits and requests
* Use namespaces appropriately
* Keep manifests clean and readable

---

## 📌 3. Keep Common Values in the Base

Place shared configuration such as:

* Namespace
* Common labels
* Metadata
* Standard configurations

In the Base directory.

Environment-specific changes should go into overlays.

---

## 🧹 4. Format Files Before Committing

Before pushing to Git, format your files:

```
kubectl kustomize cfg fmt <file_name>
```

This ensures consistent indentation and clean YAML structure.

---

## 🔍 5. Validate Before Deployment

Always review generated manifests before applying them:

```
kubectl kustomize <overlay-directory>
```

Perform thorough testing to ensure correctness.

---

## 🔄 6. Integrate with CI/CD

Integrate Kustomize into your:

* CI/CD pipelines
* GitOps workflows
* Automated deployment systems

This ensures consistent and reliable deployments.

---

## 🛠 7. Use the `edit` Subcommand

Kustomize provides the `edit` command to update `kustomization.yaml` imperatively.

You can:

* Add labels
* Set namespaces
* Modify images
* Update replicas

This reduces manual editing errors.

---

# 🎯 Final Summary

Kustomize offers:

* Clean and structured configuration management
* Reusable base configurations
* Environment-specific overlays
* No templating complexity
* Native Kubernetes integration
* Git-friendly workflow

It is a lightweight yet powerful solution for managing Kubernetes manifests efficiently.
