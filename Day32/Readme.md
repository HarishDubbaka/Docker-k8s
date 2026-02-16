# Helm Cheat Sheet

A quick reference guide to Helm for Kubernetes deployments.  
This cheat sheet covers the essential concepts, file structure, and commands to help you work efficiently with Helm charts.

---

## Table of Contents
1. [Helm Overview](#helm-overview)
2. [Chart Structure](#chart-structure)
3. [Key Files](#key-files)
4. [Common Commands](#common-commands)
5. [Install / Upgrade / Rollback Flow](#install--upgrade--rollback-flow)
6. [Values Override Hierarchy](#values-override-hierarchy)
7. [Tips & Best Practices](#tips--best-practices)

---

## Helm Overview
Helm is a package manager for Kubernetes that allows you to:
- Define, install, and upgrade Kubernetes applications
- Package applications as reusable charts
- Manage releases with version control

---

## Chart Structure
A Helm chart typically contains the following structure:

```

my-chart/
├── Chart.yaml          # Chart metadata
├── values.yaml         # Default configuration values
├── templates/          # Kubernetes manifests templates
└── charts/             # Optional dependent charts

````

---

## Key Files
- **Chart.yaml** – Contains chart metadata like name, version, description  
- **values.yaml** – Default configuration for templates  
- **templates/** – Folder with Kubernetes manifest templates (`deployment.yaml`, `service.yaml`, etc.)  
- **charts/** – Optional folder for dependent charts  

---

## Common Commands
- `helm create <chart-name>` – Create a new chart  
- `helm install <release> <chart>` – Install a chart  
- `helm upgrade <release> <chart>` – Upgrade a release  
- `helm rollback <release> <revision>` – Rollback to a previous release  
- `helm list` – List installed releases  
- `helm template <chart>` – Render templates locally  

---

## Install → Upgrade → Rollback Flow
1. **Install:** Deploy a new chart  
   ```bash
   helm install my-app ./my-chart
````

2. **Upgrade:** Apply changes to an existing release

   ```bash
   helm upgrade my-app ./my-chart
   ```
3. **Rollback:** Revert to a previous release

   ```bash
   helm rollback my-app 1
   ```

---

## Values Override Hierarchy

1. **Command-line overrides** (`--set key=value`)
2. **Custom values file** (`-f custom-values.yaml`)
3. **Default values** (`values.yaml` in chart)

---

## Tips & Best Practices

* Use meaningful release names
* Keep `values.yaml` clean and minimal
* Version control your charts
* Use templates to reuse manifests

---
