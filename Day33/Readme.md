# Managing Kubernetes Configurations with Kustomize

Managing Kubernetes configurations across development, staging, and production frequently degenerates into an error-prone workflow. Teams duplicate YAML manifests, adjust environment-specific values manually, and rely on manual checks to catch mistakes. This copy-and-paste model inevitably leads to configuration drift, making it difficult to guarantee consistency or safely promote changes between environments.

**Kustomize** is a Kubernetes-native configuration management tool built to address this problem directly. Rather than relying on templating, it uses a **base-and-overlay model**. You define a canonical set of manifests once in a base, then layer environment-specific changes as small, declarative patches. This approach keeps shared configuration centralized while making differences explicit and reviewable.

---

## 📌 Key Takeaways

- **Manage environment configurations without duplicating code**  
  Use Kustomize's base and overlay structure to define a single source of truth for your application's manifests. Apply environment-specific patches for details like replica counts or image tags, keeping your core configuration clean and consistent.

- **Prevent configuration drift with a declarative workflow**  
  Because Kustomize is integrated directly into `kubectl`, you can enforce a declarative, GitOps-centric process. Every change is version-controlled and auditable, ensuring your live cluster state never diverges from what's defined in your repository.

- **Scale deployments across clusters**  
  While Kustomize excels at managing configurations for individual applications, you can integrate it with GitOps platforms to orchestrate deployments across multiple clusters, ensuring fleet-wide consistency.

---

## 🚀 What Is Kustomize?

Kustomize is a Kubernetes-native configuration management tool that lets you customize raw, template-free YAML across multiple environments without modifying the original manifests.

Instead of copying files or introducing a templating language, Kustomize applies environment-specific changes as declarative patches on top of a shared base configuration.

This model:

- Reduces configuration drift  
- Keeps environment differences explicit  
- Makes promotion between environments safer  

Since Kustomize is built directly into `kubectl`, you can render or apply configurations from any directory containing a `kustomization.yaml` file using:

```bash
kubectl apply -k .
````

No additional tooling required.

---

## ⚙️ Kustomize Features

* Declarative configuration (same style as Kubernetes YAML)
* Modify resources without altering original files
* Add common labels and annotations across resources
* Modify container images per environment
* Built-in `secretGenerator` and `configMapGenerator`
* Patch-based overlays for environment-specific changes

---

## 📦 Install Kustomize

> **Note:** You must have a Kubernetes cluster running and `kubectl` installed and configured.

### Option 1: Use Built-in Kubectl Kustomize

Kustomize is built into `kubectl`.

```bash
kubectl kustomize --help
```

---

### Option 2: Install Standalone Kustomize

#### Linux / macOS (Script)

```bash
curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh" | bash
```

Verify installation:

```bash
kustomize version
```

If you get `command not found`:

```bash
sudo install -o root -g root -m 0755 kustomize /usr/local/bin/kustomize
```

---

### macOS (Homebrew)

```bash
brew install kustomize
```

---

### Windows (Chocolatey)

```bash
choco install kustomize
```

---

## 🆚 Helm vs Kustomize

| Feature               | Helm                        | Kustomize                 |
| --------------------- | --------------------------- | ------------------------- |
| Approach              | Templating (Go templates)   | Patch-based customization |
| Package Manager       | Yes                         | No                        |
| Dependency Management | Supports dependencies       | No dependency management  |
| Versioning            | Supports versioned releases | No built-in versioning    |
| Learning Curve        | Moderate                    | Easy                      |
| Integration           | Helm CLI                    | Built into kubectl        |

---

## 🎯 When to Use What?

* **Use Kustomize** when:

  * You want clean, template-free YAML
  * You need environment-specific overlays
  * You prefer declarative GitOps workflows
  * You want kubectl-native tooling

* **Use Helm** when:

  * You need application packaging
  * You manage complex dependencies
  * You want versioned releases

You can also use **Helm and Kustomize together** to combine packaging with environment customization.

---

## 📁 Example Project Structure

```
my-app/
│
├── base/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── kustomization.yaml
│
├── overlays/
│   ├── dev/
│   │   └── kustomization.yaml
│   ├── staging/
│   │   └── kustomization.yaml
│   └── prod/
│       └── kustomization.yaml
```

---

## ✅ Conclusion

Kustomize simplifies Kubernetes configuration management by:

* Eliminating YAML duplication
* Preventing configuration drift
* Enabling declarative GitOps workflows
* Supporting scalable, multi-environment deployments

If you're managing multiple environments in Kubernetes, Kustomize provides a clean, maintainable, and Kubernetes-native solution.

---

