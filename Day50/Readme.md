# Kubernetes Cluster Upgrade (v1.29 → v1.30) using kubeadm

This guide documents the step-by-step process to upgrade a Kubernetes cluster created using **kubeadm**.

Cluster topology used:

| Node    | Role          |
| ------- | ------------- |
| master  | Control Plane |
| worker1 | Worker Node   |
| worker2 | Worker Node   |

Initial Version:

```
v1.29.15
```

Target Version:

```
v1.30.14
```

Official documentation reference:

[https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)

---

# Cluster Status Before Upgrade

```bash
kubectl get nodes
```

```
NAME      STATUS   ROLES           AGE   VERSION
master    Ready    control-plane   27m   v1.29.15
worker1   Ready    <none>          26m   v1.29.15
worker2   Ready    <none>          26m   v1.29.15
```

---

# Step 1 — Check kubeadm Version (Control Plane)

Login to the **control plane node**

```bash
kubeadm version -o json
```

Example output:

```json
{
  "clientVersion": {
    "major": "1",
    "minor": "29",
    "gitVersion": "v1.29.15"
  }
}
```

---

# Step 2 — Verify Kubernetes Package Repository


Check repository configuration.

```bash
cat /etc/apt/sources.list.d/kubernetes.list
```

Example:

```
deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] \
https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /
```

To upgrade to **v1.30**, change:

```
v1.29 → v1.30
```

```
deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] \
https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /
```
Official documentation reference:

[https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/change-package-repository/#verifying-if-the-kubernetes-package-repositories-are-used]

---

# Step 3 — Check Available Versions

Determine which version to upgrade to
Find the latest patch release for Kubernetes 1.35 using the OS package manager:

```bash
# Find the latest 1.35 version in the list.
# It should look like 1.35.x-*, where x is the latest patch.
sudo apt update
sudo apt-cache madison kubeadm
```

Example:

```
kubeadm | 1.30.14-1.1
kubeadm | 1.30.13-1.1
...
```

Select latest patch version:

```
1.30.14
```

---

# Step 4 — Upgrade kubeadm

Upgrading control plane nodes
The upgrade procedure on control plane nodes should be executed one node at a time. Pick a control plane node that you wish to upgrade first. It must have the /etc/kubernetes/admin.conf file.

```bash
sudo apt-mark unhold kubeadm

sudo apt-get update

sudo apt-get install -y kubeadm=1.30.14-1.1

sudo apt-mark hold kubeadm
```

Verify:

```bash
kubeadm version
```

---

# Step 5 — Check Upgrade Plan

```bash
sudo kubeadm upgrade plan

Harish@master:~$ sudo kubeadm upgrade plan
[preflight] Running pre-flight checks.
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[upgrade] Running cluster health checks
[upgrade] Fetching available versions to upgrade to
[upgrade/versions] Cluster version: 1.29.15
[upgrade/versions] kubeadm version: v1.30.14
I0308 04:38:41.275959   51346 version.go:256] remote version is much newer: v1.35.2; falling back to: stable-1.30
[upgrade/versions] Target version: v1.30.14
[upgrade/versions] Latest version in the v1.29 series: v1.29.15

Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
COMPONENT   NODE      CURRENT    TARGET
kubelet     master    v1.29.15   v1.30.14
kubelet     worker1   v1.29.15   v1.30.14
kubelet     worker2   v1.29.15   v1.30.14

Upgrade to the latest stable version:

COMPONENT                 NODE      CURRENT    TARGET
kube-apiserver            master    v1.29.15   v1.30.14
kube-controller-manager   master    v1.29.15   v1.30.14
kube-scheduler            master    v1.29.15   v1.30.14
kube-proxy                          1.29.15    v1.30.14
CoreDNS                             v1.11.1    v1.11.3
etcd                      master    3.5.16-0   3.5.15-0

You can now apply the upgrade by executing the following command:

        kubeadm upgrade apply v1.30.14

_____________________________________________________________________


The table below shows the current state of component configs as understood by this version of kubeadm.
Configs that have a "yes" mark in the "MANUAL UPGRADE REQUIRED" column require manual config upgrade or
resetting to kubeadm defaults before a successful upgrade can be performed. The version to manually
upgrade to is denoted in the "PREFERRED VERSION" column.

API GROUP                 CURRENT VERSION   PREFERRED VERSION   MANUAL UPGRADE REQUIRED
kubeproxy.config.k8s.io   v1alpha1          v1alpha1            no
kubelet.config.k8s.io     v1beta1           v1beta1             no
_____________________________________________________________________

Harish@master:~$
```

Output shows:

* Current version
* Target version
* Components to upgrade

Example:

```
Cluster version: 1.29.15
Target version: v1.30.14
```

---

# Step 6 — Upgrade Control Plane

```bash
sudo kubeadm upgrade apply v1.30.14
```

During upgrade:

* Pulls required images
* Upgrades control plane components
* Updates configuration
* Updates CoreDNS and kube-proxy

Success message:

```
Harish@master:~$ sudo kubeadm upgrade apply v1.30.14
[preflight] Running pre-flight checks.
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[upgrade] Running cluster health checks
[upgrade/version] You have chosen to change the cluster version to "v1.30.14"
[upgrade/versions] Cluster version: v1.29.15
[upgrade/versions] kubeadm version: v1.30.14
[upgrade] Are you sure you want to proceed? [y/N]: y
[upgrade/prepull] Pulling images required for setting up a Kubernetes cluster
[upgrade/prepull] This might take a minute or two, depending on the speed of your internet connection
[upgrade/prepull] You can also perform this action in beforehand using 'kubeadm config images pull'
W0308 04:42:47.269464   54075 checks.go:844] detected that the sandbox image "registry.k8s.io/pause:3.8" of the container runtime is inconsistent with that used by kubeadm.It is recommended to use "registry.k8s.io/pause:3.9" as the CRI sandbox image.
[upgrade/apply] Upgrading your Static Pod-hosted control plane to version "v1.30.14" (timeout: 5m0s)...
[upgrade/etcd] Upgrading to TLS for etcd
[upgrade/etcd] Non fatal issue encountered during upgrade: the desired etcd version "3.5.15-0" is older than the currently installed "3.5.16-0". Skipping etcd upgrade
[upgrade/staticpods] Writing new Static Pod manifests to "/etc/kubernetes/tmp/kubeadm-upgraded-manifests4161686176"
[upgrade/staticpods] Preparing for "kube-apiserver" upgrade
[upgrade/staticpods] Renewing apiserver certificate
[upgrade/staticpods] Renewing apiserver-kubelet-client certificate
[upgrade/staticpods] Renewing front-proxy-client certificate
[upgrade/staticpods] Renewing apiserver-etcd-client certificate
[upgrade/staticpods] Moved new manifest to "/etc/kubernetes/manifests/kube-apiserver.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2026-03-08-04-43-08/kube-apiserver.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[upgrade/staticpods] This can take up to 5m0s
[apiclient] Found 1 Pods for label selector component=kube-apiserver
[upgrade/staticpods] Component "kube-apiserver" upgraded successfully!
[upgrade/staticpods] Preparing for "kube-controller-manager" upgrade
[upgrade/staticpods] Renewing controller-manager.conf certificate
[upgrade/staticpods] Moved new manifest to "/etc/kubernetes/manifests/kube-controller-manager.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2026-03-08-04-43-08/kube-controller-manager.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[upgrade/staticpods] This can take up to 5m0s
[apiclient] Found 1 Pods for label selector component=kube-controller-manager
[upgrade/staticpods] Component "kube-controller-manager" upgraded successfully!
[upgrade/staticpods] Preparing for "kube-scheduler" upgrade
[upgrade/staticpods] Renewing scheduler.conf certificate
[upgrade/staticpods] Moved new manifest to "/etc/kubernetes/manifests/kube-scheduler.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2026-03-08-04-43-08/kube-scheduler.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[upgrade/staticpods] This can take up to 5m0s
[apiclient] Found 1 Pods for label selector component=kube-scheduler
[upgrade/staticpods] Component "kube-scheduler" upgraded successfully!
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config" in namespace kube-system with the configuration for the kubelets in the cluster
[upgrade] Backing up kubelet config file to /etc/kubernetes/tmp/kubeadm-kubelet-config4294760236/config.yaml
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

[upgrade/successful] SUCCESS! Your cluster was upgraded to "v1.30.14". Enjoy!

[upgrade/kubelet] Now that your control plane is upgraded, please proceed with upgrading your kubelets if you haven't already done so.
Harish@master:~$.
```

---

# Step 7 — Verify Control Plane Image

```bash
cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep image
```

Example:

```
image: registry.k8s.io/kube-apiserver:v1.30.14
```

---

# Step 8 — Drain Control Plane Node

```bash
kubectl drain master --ignore-daemonsets
```

Result:

```
Harish@master:~$ kubectl get nodes
NAME      STATUS                     ROLES           AGE   VERSION
master    Ready,SchedulingDisabled   control-plane   75m   v1.29.15
worker1   Ready                      <none>          74m   v1.29.15
worker2   Ready                      <none>          73m   v1.29.15
Harish@master:~$
Note : oberver the master node this indicates the master is drained
```

---

# Step 9 — Upgrade kubelet and kubectl


Upgrade kubelet and kubectl

Note:On Linux nodes, the kubelet defaults to supporting only cgroups v2. For Kubernetes 1.35 the FailCgroupV1 kubelet configuration option is set to true by default.

To learn more, Official documentation reference:
https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/upgrading-linux-nodes/

Check versions:

```bash
apt-cache madison kubelet
apt-cache madison kubectl
```

Install:

```bash
sudo apt-mark unhold kubelet kubectl

sudo apt-get update

sudo apt-get install -y kubelet=1.30.14-1.1 kubectl=1.30.14-1.1

sudo apt-mark hold kubelet kubectl
```

Restart kubelet:

```bash
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

---

# Step 10 — Verify Control Plane Upgrade

```bash
kubectl get nodes
```
---

```bash
Harish@master:~$ kubectl version --client
Client Version: v1.30.14
Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3
Harish@master:~$ kubelet --version
Kubernetes v1.30.14
Harish@master:~$
Example:
```
---
```
NAME      STATUS                     ROLES           VERSION
master    Ready,SchedulingDisabled   control-plane   v1.30.14
worker1   Ready                      <none>          v1.29.15
worker2   Ready                      <none>          v1.29.15
```


---

# Step 11 — Uncordon Control Plane Node

```bash
kubectl uncordon master

Harish@master:~$ kubectl uncordon master
node/master uncordoned
Harish@master:~$
Harish@master:~$ kubectl get nodes
NAME      STATUS   ROLES           AGE   VERSION
master    Ready    control-plane   91m   v1.30.14
worker1   Ready    <none>          90m   v1.29.15
worker2   Ready    <none>          90m   v1.29.15

```
# Kubernetes Upgrade Order 

When upgrading a Kubernetes cluster using **kubeadm**, the order of upgrading components is very important.

Correct upgrade order:

1. kubeadm  
2. kubeadm upgrade (apply / node)  
3. kubelet  
4. kubectl  

# Simple Memory Trick

### ADM → LET → CTL

---

# Step 12 — Upgrade Worker Nodes

Perform upgrade **one worker at a time**.

---

# Worker Node: Update Repository

Check repository:

```bash
cat /etc/apt/sources.list.d/kubernetes.list
```

Change:

```
v1.29 → v1.30
```

---

# Step 13 — Upgrade kubeadm (Worker)

```bash
sudo apt-mark unhold kubeadm

sudo apt-get update

sudo apt-get install -y kubeadm=1.30.14-1.1

sudo apt-mark hold kubeadm
```

---

# Step 14 — Upgrade Node Configuration

```bash
sudo kubeadm upgrade node
```

---

# Step 15 — Drain Worker Node

Run from control plane:

```bash
kubectl drain worker1 --ignore-daemonsets

Harish@master:~$ kubectl drain worker1 --ignore-daemonsets
node/worker1 cordoned
Warning: ignoring DaemonSet-managed Pods: kube-system/calico-node-cf4sb, kube-system/kube-proxy-w9lz2
evicting pod kube-system/coredns-55cb58b774-q4kvv
pod/coredns-55cb58b774-q4kvv evicted
node/worker1 drained
Harish@master:~$ kubectl get nodes
NAME      STATUS                     ROLES           AGE    VERSION
master    Ready                      control-plane   107m   v1.30.14
worker1   Ready,SchedulingDisabled   <none>          105m   v1.29.15
worker2   Ready                      <none>          105m   v1.29.15
Harish@master:~$

```

---

# Step 16 — Upgrade kubelet and kubectl

```bash
sudo apt-mark unhold kubelet kubectl

sudo apt-get update

sudo apt-get install -y kubelet=1.30.14-1.1 kubectl=1.30.14-1.1

sudo apt-mark hold kubelet kubectl
```

Restart kubelet:

```bash
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

---

# Step 17 — Uncordon Worker Node

```bash
kubectl uncordon worker1

Harish@master:~$ kubectl uncordon worker1
node/worker1 uncordoned
Harish@master:~$
```

Repeat same process for **worker2**.


---

# Worker Node: Update Repository

Check repository:

```bash
cat /etc/apt/sources.list.d/kubernetes.list
```

Change:

```
v1.29 → v1.30
```

---

# Step 18 — Upgrade kubeadm (Worker)

```bash
sudo apt-mark unhold kubeadm

sudo apt-get update

sudo apt-get install -y kubeadm=1.30.14-1.1

sudo apt-mark hold kubeadm
```

---

# Step 19 — Upgrade Node Configuration

```bash
sudo kubeadm upgrade node
```

---

# Step 20 — Drain Worker Node

Run from control plane:

```bash
Harish@master:~$ kubectl drain worker2 --ignore-daemonsets
node/worker2 cordoned
Warning: ignoring DaemonSet-managed Pods: kube-system/calico-node-znz7k, kube-system/kube-proxy-s2c9w
evicting pod kube-system/coredns-55cb58b774-hslpx
evicting pod kube-system/calico-kube-controllers-787f445f84-7slxb
pod/calico-kube-controllers-787f445f84-7slxb evicted
pod/coredns-55cb58b774-hslpx evicted
node/worker2 drained

Harish@master:~$ kubectl get nodes
NAME      STATUS                     ROLES           AGE    VERSION
master    Ready                      control-plane   117m   v1.30.14
worker1   Ready                      <none>          116m   v1.30.14
worker2   Ready,SchedulingDisabled   <none>          115m   v1.29.15
Harish@master:~$

```

---

# Step 21 — Upgrade kubelet and kubectl

```bash
sudo apt-mark unhold kubelet kubectl

sudo apt-get update

sudo apt-get install -y kubelet=1.30.14-1.1 kubectl=1.30.14-1.1

sudo apt-mark hold kubelet kubectl
```

Restart kubelet:

```bash
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

---

# Step 22 — Uncordon Worker Node

```bash
kubectl uncordon worker1

Harish@master:~$ kubectl uncordon worker2
node/worker2 uncordoned
Harish@master:~$
```
---

# Final Verification

```bash
kubectl get nodes
```

Output:

```
NAME      STATUS   ROLES           VERSION
master    Ready    control-plane   v1.30.14
worker1   Ready    <none>          v1.30.14
worker2   Ready    <none>          v1.30.14
```

All nodes successfully upgraded.

---

# Verify Client & Kubelet Versions

```bash
kubectl version --client
kubelet --version
```

Example:

```
Client Version: v1.30.14
Kubernetes v1.30.14
```

---

# Upgrade Completed

Cluster successfully upgraded:

```
v1.29.15 → v1.30.14
```

