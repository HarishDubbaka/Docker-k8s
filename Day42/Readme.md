# Kubernetes Networking Overview

Networking is a **core component of Kubernetes** that enables seamless communication between applications (Pods) in a cluster. In microservices architectures, services must exchange data, request resources, and coordinate actions.

Kubernetes networking provides:

* **Intra-cluster communication:** Pod-to-Pod and Pod-to-Service.
* **External communication:** Access from outside the cluster.
* **High availability & load balancing:** Distributes traffic across service instances.
* **Security:** Network policies enforce allowed traffic between services.

---

## Kubernetes Network Model

Kubernetes networking ensures:

1. **Container-to-container communication:**
   Containers in the same Pod share a network namespace, using `localhost` or the Pod IP.

2. **Pod-to-Pod communication:**
   Each Pod gets a unique ephemeral IP from the Node’s Pod CIDR. Cross-node Pod communication relies on the CNI plugin.

3. **Pod-to-Service communication:**
   Services provide a stable DNS name and load balance across Pods.

4. **Node-to-Node communication:**
   Nodes exchange data and coordinate workloads using Node IP addresses.

---

## How a Pod Gets an IP Address

1. **Container Runtime:** Starts containers in a Pod; containers share a network namespace.
2. **CNI Plugins:** Assign IPs and configure networking (`Calico`, `Flannel`, `Weave`, `Cilium`).
3. **IPAM (IP Address Management):** Allocates Pod IPs and handles cleanup.
4. **Kubelet + CNI:** Kubelet requests CNI to assign IPs, set up routes, and enforce network policies.

Check cluster Pod CIDR:

```bash
kubectl cluster-info dump | grep cluster-cidr
```

---

## Key Networking Concepts

* **Pod Networking:** Each Pod has its own IP; no NAT needed.
* **Service Networking:** Virtual IP for load balancing across Pods.
* **Cluster Networking:** All nodes and Pods can communicate seamlessly.

---

## Popular CNI Plugins

| Plugin          | Features                                        |
| --------------- | ----------------------------------------------- |
| **Calico**      | Networking + network policies, security-focused |
| **Flannel**     | Simple overlay network                          |
| **Weave Net**   | Encryption + isolation                          |
| **Cilium**      | eBPF-based security and observability           |
| **Kube-Router** | Networking, routing, and policies               |

---

## How CNI Works

1. **Pod Creation:** Kubelet requests CNI to configure networking.
2. **IP Assignment:** CNI assigns IP from cluster pool.
3. **Network Setup:** CNI configures network namespace and routes.
4. **Policy Enforcement:** Optional network policies applied.

---

## Common Issues in WSL2 / Kind / kubeadm Labs

* Nodes are actually containers → Overlay traffic (VXLAN/IPIP) may fail.
* Linux kernel tunneling modules may not behave fully.
* Calico may show "Running" but **cross-node Pod communication fails**.

✅ Quick Test:

1. Force Pods onto the **same node**:

```bash
kubectl delete pod banpod nginx1
kubectl run nginx1 --image=nginx --overrides='{"spec":{"nodeName":"cka-prodcluster-worker2"}}'
kubectl run banpod --image=nginx --overrides='{"spec":{"nodeName":"cka-prodcluster-worker2"}}'
```

2. Test Pod-to-Pod connectivity:

```bash
kubectl exec -it banpod -- curl <nginx-pod-ip>
```

---

## Working with Multi-Container Pods

Pod YAML example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: shared-namespace
spec:
  containers:
    - name: p1
      image: busybox
      command: ['/bin/sh', '-c', 'sleep 10000']
    - name: p2
      image: nginx
```

Access containers:

```bash
kubectl exec -it shared-namespace -c p1 -- sh  # enter busybox
kubectl exec -it shared-namespace -c p2 -- sh  # enter nginx
```

`-c` selects container, `--` separates kubectl command from container command.

---

## Inspecting Network Namespaces and veth Pairs

1. List namespaces on a node:

```bash
ip netns list
```

2. Inspect interfaces in a namespace:

```bash
sudo ip netns exec <namespace> ip link
```

3. List veth pairs on host:

```bash
ip link show | grep cali
```

4. Check veth statistics:

```bash
sudo ethtool -S <veth-interface>
```

This shows the network interfaces and veth pairs created by the CNI plugin for Pods.

---

## Verifying CNI Functionality

Check CNI pods:

```bash
kubectl get pods -n kube-system
```

All `calico-node` pods should be **Running**.

✅ If cross-node Pod-to-Pod communication fails, it's likely due to **WSL2 or overlay networking limitations**.

---

If time permits try the below example on the https://killercoda.com/playgrounds/scenario/kubernetes


# What happens when a pod runs - namespace 

```
apiVersion: v1
kind: Pod
metadata:
  name: shared-namespace
spec:
  containers:
    - name: p1
      image: busybox
      command: ['/bin/sh', '-c', 'sleep 10000']
    - name: p2
      image: nginx
```
Commands 

```
ip netns list

```
### how to check pause container 
```
kubectl run nginx --image=nginx
lsns | grep nginx
```
copy the process IP from above and run 
```
lsns -p <pid>

```

### check the network namespace (this gives list of all network namespaces)
`ls -lt /var/run/netns`

### exec into the namespace or into the pod to see the IP links

```
ip netns exec <namespace> ip link
kubectl exec -it shared-namespace -- ip addr #run this on controlplane
```
Now you will see `eth@9` -> after `@` there will be a number and you can then search its corresponding link on the node using 
`ip link | grep -A1 ^9`
you will be able to see the same network namespace after link
These are the veth pairs or based on the CNI 





===========================


1. **List all network interfaces on the host**:

    ```bash
    ip link show
    ```

2. **List network namespaces**:

    ```bash
    sudo ip netns list
    ```

3. **Inspect network interfaces within a specific namespace**:

    Replace `<namespace>` with the actual namespace ID (e.g., `cni-6bbf75c3-3284-407a-5869-90772afb5472`).

    ```bash
    sudo ip netns exec <namespace> ip link
    ```

4. **Find veth pairs on the host**:

    ```bash
    ip link show | grep cali
    ```

5. **Verify the veth pair connection**:

    Replace `<interface>` with the actual veth interface name (e.g., `caliede2c6f02d9`).

    ```bash
    sudo ethtool -S <interface>
    ```

### Example Workflow

1. **List all interfaces**:

    ```bash
    ip link show
    ```

2. **List network namespaces**:

    ```bash
    sudo ip netns list
    ```

3. **Inspect network interfaces within the namespace `cni-6bbf75c3-3284-407a-5869-90772afb5472`**:

    ```bash
    sudo ip netns exec cni-6bbf75c3-3284-407a-5869-90772afb5472 ip link
    ```

4. **Find veth pairs on the host**:

    ```bash
    ip link show | grep cali
    ```

5. **Verify the veth pair connection for interface `caliede2c6f02d9`**:

    ```bash
    sudo ethtool -S caliede2c6f02d9
    ```

These commands will help you identify the network interfaces and veth pairs created by the CNI plugin, and inspect the network configuration for your Pods.
