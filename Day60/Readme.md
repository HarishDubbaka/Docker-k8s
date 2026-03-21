## 🚀 Day 60 – Simplifying Kubernetes with K9s

Managing multiple Kubernetes clusters from the CLI can quickly become repetitive and error-prone.

### 🔹 The Problem

Switching contexts manually:

```bash
kubectl config use-context <context-name>
kubectl get pods
```

Doing this repeatedly during debugging or deployments slows you down.

---

## 📘 What is K9s?

**K9s** is an open-source, terminal-based UI that lets you interact with Kubernetes clusters in real time.

It eliminates the need to constantly type `kubectl` commands by giving you a fast, keyboard-driven interface.

---

## ⚖️ Why Use K9s?

K9s is popular among DevOps engineers because it provides:

* 📊 **Real-time monitoring** of cluster resources
* ⚡ **Fast navigation** using keyboard shortcuts
* 📄 **Live logs streaming** and filtering
* 🛠️ **Resource management** (delete, scale, edit)
* 🔄 **Easy context switching** between clusters
* 🔍 **Search & filtering** for quick debugging
* 🌐 **Built-in port forwarding**
* 🎛️ **Customizable workflows**

👉 Think of it as:
**kubectl + top + htop — all in one UI**

---

## 🧰 Prerequisites

* Kubernetes cluster (Minikube, Kind, EKS, GKE, AKS)
* `kubectl` installed
* Valid kubeconfig (`~/.kube/config`)

---

## ⚙️ Installation (Windows)

1. Download from K9s releases
2. Extract the archive
3. Move `k9s.exe` to a folder (e.g., `C:\Program Files\k9s`)
4. Add that folder to your system `PATH`
5. Verify:

```bash
k9s version
```

---

## 🚀 Getting Started

```bash
k9s
```

---

## 🧭 Core Navigation

* `:` → command mode
* `q` → quit/back
* `?` → help
* `/` → search
* `Enter` → drill down
* `Esc` → go back

---

## 📦 Common Resource Commands

| Resource    | Command   |
| ----------- | --------- |
| Pods        | `:po`     |
| Deployments | `:deploy` |
| Services    | `:svc`    |
| Nodes       | `:no`     |
| Namespaces  | `:ns`     |
| ConfigMaps  | `:cm`     |
| Secrets     | `:sec`    |

---

## 📄 Debugging Shortcuts

Inside a pod:

* `l` → logs
* `s` → describe / shell
* `d` → delete

---

## 🔄 Namespace Switching

```text
:ns
```

or directly:

```text
:ns kube-system
```

---

## 🔍 Search & Filter

Press `/` and type:

```text
nginx
```

---

## 🔁 Port Forward

* Select pod → `Shift + F`

---

## 🧪 Exec into Pod

* Select pod → `s`

---

## 📊 View YAML

* Select resource → `y`

---

## ⭐ Save Favorites

* `Ctrl + S`

---

## 🧠 Real-World Workflow

```text
k9s
→ :ns
→ :po
→ Enter
→ l
```

👉 Fast debugging in seconds.

---

## ⚠️ Common Issue

If K9s shows no resources:

```bash
kubectl get pods
```

👉 Your kubeconfig may not be set correctly.

---

## 💡 Final Tip

If you use Kubernetes daily, **K9s is not optional — it's a productivity multiplier.**

---
