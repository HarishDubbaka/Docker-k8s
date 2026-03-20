# 🚀 Highly Available Kubernetes Cluster (kubeadm + HAProxy)

## 📌 Overview

A single control plane node in a Kubernetes cluster creates a **single point of failure**. If it goes down:

* ❌ Cluster administration stops
* ✅ Existing workloads may continue temporarily

To overcome this, we build a **Highly Available Kubernetes Cluster** with:

* Multiple control plane nodes
* Multiple worker nodes
* Load balancer (HAProxy)
* Stacked ETCD setup

---

## 🏗️ Architecture

* **Control Plane Nodes:** 2–3 (HA)
* **Worker Nodes:** 2+
* **Load Balancer:** HAProxy
* **Networking:** Azure VNet (private communication)

---

# 🌐 Step 1: Create Resource Group

* **Name:** `multik8s-rg`
* **Region:** Central India

---

# 🌐 Step 2: Create Virtual Network

### VNet Configuration

* **Name:** `multik8s-vnet`
* **Address Space:** `10.0.0.0/16`

### Subnets

| Subnet Name   | Address Range | Purpose             |
| ------------- | ------------- | ------------------- |
| master-subnet | 10.0.1.0/24   | Control Plane Nodes |
| worker-subnet | 10.0.2.0/24   | Worker Nodes        |

---

# 🔓 Step 3: NSG Rules (Security)

## Master Node Ports

| Port      | Purpose            |
| --------- | ------------------ |
| 22        | SSH                |
| 6443      | Kubernetes API     |
| 2379-2380 | ETCD               |
| 10250     | Kubelet            |
| 10257     | Controller Manager |
| 10259     | Scheduler          |

---

## 🔐 Worker Node Ports

| Port        | Purpose           |
| ----------- | ----------------- |
| 22          | SSH               |
| 10250       | Kubelet           |
| 30000-32767 | NodePort Services |

---

## ⚠️ Important Notes

* Only expose **22 & 6443 publicly**
* Keep ETCD ports **internal**
* Use **private IPs inside VNet**

---

# 🖥️ Step 4: Create VMs

* 2 Control Plane Nodes
* 2 Worker Nodes
* 1 Load Balancer VM

---

# ⚖️ Step 5: Setup HAProxy (Load Balancer VM)

## 🔐 Switch to Root User

```bash
sudo -i
```

## 🔄 Update System Packages

```bash
sudo apt-get update && sudo apt-get upgrade -y
```

---

## 📦 Install HAProxy

```bash
sudo apt update
sudo apt install haproxy -y
```

---

## ⚙️ Configure HAProxy

Open the configuration file:

```bash
sudo vi /etc/haproxy/haproxy.cfg
```

### 🔧 Add/Update Configuration

```cfg
frontend kubernetes-api
    bind *:6443
    mode tcp
    option tcplog
    default_backend kube-master-nodes

backend kube-master-nodes
    mode tcp
    balance roundrobin
    option tcp-check

    server master1 10.0.0.5:6443 check
    server master2 10.0.0.6:6443 check
```

> ⚠️ **Important:** Replace `10.0.0.5` and `10.0.0.6` with the **private IP addresses of your control plane nodes**.

---

## 🔁 Restart & Enable HAProxy

```bash
sudo systemctl restart haproxy
sudo systemctl enable haproxy
```

---

## 🧪 Verify HAProxy

Check if HAProxy is listening on port **6443**:

```bash
nc -v localhost 6443
```

### ✅ Expected Output

```bash
Connection to localhost (127.0.0.1) 6443 port [tcp/*] succeeded!
```
---

Here is your section polished and structured cleanly for a **professional README.md** 👇

---

# ⚙️ Step 6: Setup Control Plane (Master 1)

## 🔐 Disable Swap

Kubernetes requires swap to be disabled on all nodes:

```bash
swapoff -a
sed -i '/ swap / s/^/#/' /etc/fstab
```

---

## 🌐 Enable Required Networking

### Load Kernel Modules

```bash
cat <<EOF | tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

modprobe overlay
modprobe br_netfilter
```

### Configure Sysctl Parameters

```bash
cat <<EOF | tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables=1
net.ipv4.ip_forward=1
EOF

sysctl --system
```

---

## 📦 Install Container Runtime (containerd)

```bash
curl -LO https://github.com/containerd/containerd/releases/download/v1.7.14/containerd-1.7.14-linux-amd64.tar.gz
tar Cxzvf /usr/local containerd-1.7.14-linux-amd64.tar.gz
```

### Enable containerd

```bash
systemctl enable --now containerd
```

---

## ☸️ Install Kubernetes Components

```bash
apt update
apt install -y kubelet kubeadm kubectl
apt-mark hold kubelet kubeadm kubectl
```

---

## 🚀 Initialize Kubernetes Cluster

### Example Configuration

```bash
kubeadm init \
  --control-plane-endpoint "10.0.0.4:6443" \
  --upload-certs \
  --pod-network-cidr=192.168.0.0/16 \
  --apiserver-advertise-address=10.0.0.5
```

### 🔁 Generic (Recommended)

```bash
kubeadm init \
  --control-plane-endpoint "<LOAD_BALANCER_PRIVATE_IP>:6443" \
  --upload-certs \
  --pod-network-cidr=192.168.0.0/16 \
  --apiserver-advertise-address=<MASTER_NODE_PRIVATE_IP>
```

> ⚠️ **Important:**
> Replace:
>
> * `<LOAD_BALANCER_PRIVATE_IP>` → HAProxy private IP
> * `<MASTER_NODE_PRIVATE_IP>` → Current master node private IP

---

## 📌 Post Initialization Setup

### For Normal User

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### For Root User

```bash
export KUBECONFIG=/etc/kubernetes/admin.conf
```

---

## 🔑 Save Join Commands (CRITICAL)

After initialization, you will get join commands.

### Control Plane Join Command

```bash
kubeadm join 10.0.0.4:6443 --token <token> \
  --discovery-token-ca-cert-hash <hash> \
  --control-plane \
  --certificate-key <certificate-key>
```

### Worker Node Join Command

```bash
kubeadm join 10.0.0.4:6443 --token <token> \
  --discovery-token-ca-cert-hash <hash>
```

> ⚠️ **Important:**
>
> * Save these commands immediately
> * Required for adding:
>
>   * Additional master nodes
>   * Worker nodes

---

## ⚠️ Initial Node Status

```bash
kubectl get nodes
```

Example:

```bash
NAME        STATUS     ROLES           AGE     VERSION
mastervm1   NotReady   control-plane   2m42s   v1.33.0
```

> ❗ **This is expected** because the CNI plugin is not installed yet.

---

# 🌐 Step 7: Install CNI (Calico)

Install Calico networking:

```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/tigera-operator.yaml

curl -O https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/custom-resources.yaml

kubectl apply -f custom-resources.yaml
```

---

## 🔍 Verify Cluster Components

```bash
kubectl get pods -A
```

### ✅ Expected Output

All pods should be in **Running** state:

* calico-apiserver
* calico-node
* coredns
* kube-apiserver
* etcd
* controller-manager
* scheduler

---

## 🎯 Final Node Status

```bash
kubectl get nodes
```

```bash
NAME        STATUS   ROLES           AGE   VERSION
mastervm1   Ready    control-plane   10m   v1.33.0
```

---

Here is your content cleaned, structured, and formatted as a professional **README.md section for Master 2 (Additional Control Plane Node)** 👇

---

# ➕ Step 8: Add Additional Control Plane Node (Master 2)

## 🔐 Connect to Master 2

```bash
ssh <user>@<master2-ip>
sudo -i
```

---

## ⚙️ Prepare the Node

### Disable Swap

```bash
swapoff -a
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

---

## 🌐 Enable Networking

### Load Kernel Modules

```bash
cat <<EOF | tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

modprobe overlay
modprobe br_netfilter
```

### Configure Sysctl

```bash
cat <<EOF | tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sysctl --system
```

---

## ✅ Verify Kernel Modules

```bash
lsmod | grep br_netfilter
lsmod | grep overlay
```

## ✅ Verify Sysctl Values

```bash
sysctl net.bridge.bridge-nf-call-iptables \
       net.bridge.bridge-nf-call-ip6tables \
       net.ipv4.ip_forward
```

---

## 📦 Install Container Runtime (containerd)

```bash
curl -LO https://github.com/containerd/containerd/releases/download/v1.7.14/containerd-1.7.14-linux-amd64.tar.gz
tar Cxzvf /usr/local containerd-1.7.14-linux-amd64.tar.gz

curl -LO https://raw.githubusercontent.com/containerd/containerd/main/containerd.service
mkdir -p /usr/local/lib/systemd/system/
mv containerd.service /usr/local/lib/systemd/system/

mkdir -p /etc/containerd
containerd config default | tee /etc/containerd/config.toml

sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml

systemctl daemon-reload
systemctl enable --now containerd
```

---

## 🔍 Verify containerd

```bash
systemctl status containerd
```

---

## ⚙️ Install runc

```bash
curl -LO https://github.com/opencontainers/runc/releases/download/v1.1.12/runc.amd64
install -m 755 runc.amd64 /usr/local/sbin/runc
```

---

## 🌐 Install CNI Plugins

```bash
curl -LO https://github.com/containernetworking/plugins/releases/download/v1.5.0/cni-plugins-linux-amd64-v1.5.0.tgz
mkdir -p /opt/cni/bin
tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.5.0.tgz
```

---

## ☸️ Install Kubernetes Components

```bash
apt-get update
apt-get install -y apt-transport-https ca-certificates curl gpg
```

### Add Kubernetes Repository

```bash
mkdir -p /etc/apt/keyrings

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.33/deb/Release.key \
  | gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] \
https://pkgs.k8s.io/core:/stable:/v1.33/deb/ /" \
| tee /etc/apt/sources.list.d/kubernetes.list
```

### Install Components

```bash
apt-get update

apt-get install -y \
  kubelet=1.33.0-1.1 \
  kubeadm=1.33.0-1.1 \
  kubectl=1.33.0-1.1 \
  --allow-downgrades --allow-change-held-packages

apt-mark hold kubelet kubeadm kubectl
```

---

## 🔍 Verify Installation

```bash
kubeadm version
kubelet --version
kubectl version --client
```

---

## 🔗 Join as Control Plane Node

Run the **join command generated on Master 1**:

```bash
kubeadm join 10.0.0.4:6443 \
  --token <token> \
  --discovery-token-ca-cert-hash <hash> \
  --control-plane \
  --certificate-key <certificate-key>
```

> ⚠️ Replace placeholders with actual values from Master 1 output.

---

## ✅ Verify Cluster

```bash
kubectl get nodes
```

### 🎯 Expected Output

```bash
NAME        STATUS   ROLES           AGE   VERSION
mastervm1   Ready    control-plane   18m   v1.33.0
mastervm2   Ready    control-plane   38s   v1.33.0
```

---

# 👷 Step 9: Add Worker Nodes to Cluster

## 🔐 Connect to Worker Node

```bash
ssh <user>@<worker-node-ip>
sudo -i
```

---

## ⚙️ Prepare the Worker Node

### Disable Swap

```bash
swapoff -a
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

---

## 🌐 Enable Networking

### Load Kernel Modules

```bash
cat <<EOF | tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

modprobe overlay
modprobe br_netfilter
```

### Configure Sysctl Parameters

```bash
cat <<EOF | tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sysctl --system
```

---

## ✅ Verify Configuration

### Check Kernel Modules

```bash
lsmod | grep br_netfilter
lsmod | grep overlay
```

### Check Sysctl Values

```bash
sysctl net.bridge.bridge-nf-call-iptables \
       net.bridge.bridge-nf-call-ip6tables \
       net.ipv4.ip_forward
```

---

## 📦 Install Container Runtime (containerd)

```bash
curl -LO https://github.com/containerd/containerd/releases/download/v1.7.14/containerd-1.7.14-linux-amd64.tar.gz
tar Cxzvf /usr/local containerd-1.7.14-linux-amd64.tar.gz

curl -LO https://raw.githubusercontent.com/containerd/containerd/main/containerd.service
mkdir -p /usr/local/lib/systemd/system/
mv containerd.service /usr/local/lib/systemd/system/

mkdir -p /etc/containerd
containerd config default | tee /etc/containerd/config.toml

sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml

systemctl daemon-reload
systemctl enable --now containerd
```

---

## 🔍 Verify containerd

```bash
systemctl status containerd
```

---

## ⚙️ Install runc

```bash
curl -LO https://github.com/opencontainers/runc/releases/download/v1.1.12/runc.amd64
install -m 755 runc.amd64 /usr/local/sbin/runc
```

---

## 🌐 Install CNI Plugins

```bash
curl -LO https://github.com/containernetworking/plugins/releases/download/v1.5.0/cni-plugins-linux-amd64-v1.5.0.tgz
mkdir -p /opt/cni/bin
tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.5.0.tgz
```

---

## ☸️ Install Kubernetes Components

```bash
apt-get update
apt-get install -y apt-transport-https ca-certificates curl gpg
```

### Add Kubernetes Repository

```bash
mkdir -p /etc/apt/keyrings

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.33/deb/Release.key \
  | gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] \
https://pkgs.k8s.io/core:/stable:/v1.33/deb/ /" \
| tee /etc/apt/sources.list.d/kubernetes.list
```

### Install Components

```bash
apt-get update

apt-get install -y \
  kubelet=1.33.0-1.1 \
  kubeadm=1.33.0-1.1 \
  kubectl=1.33.0-1.1 \
  --allow-downgrades --allow-change-held-packages

apt-mark hold kubelet kubeadm kubectl
```

---

## 🔍 Verify Installation

```bash
kubeadm version
kubelet --version
kubectl version --client
systemctl status kubelet
```

---

## 🔗 Join Worker Node to Cluster

Run the **join command generated on Master 1**:

```bash
kubeadm join 10.0.0.4:6443 \
  --token <token> \
  --discovery-token-ca-cert-hash <hash>
```

> ⚠️ Replace `<token>` and `<hash>` with actual values from Master node.

---

## ✅ Verify Worker Node

On Master Node:

```bash
kubectl get nodes
```

### 🎯 Expected Output

```bash
NAME        STATUS   ROLES           AGE   VERSION
mastervm1   Ready    control-plane   XXm   v1.33.0
mastervm2   Ready    control-plane   XXm   v1.33.0
worker1     Ready    <none>          XXm   v1.33.0

```

# 👷 Step 10: Add Additional Worker Node (Worker 2)

> ⚠️ This step is identical to Worker 1 setup. Repeat the same process for each additional worker node.

---

## 🔐 Connect to Worker Node

```bash id="9x0k8v"
ssh <user>@<worker2-ip>
sudo -i
```

---

## ⚙️ Prepare the Node

### Disable Swap

```bash id="yovk8a"
swapoff -a
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

---

## 🌐 Enable Networking

### Load Kernel Modules

```bash id="w9jzcm"
cat <<EOF | tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

modprobe overlay
modprobe br_netfilter
```

### Configure Sysctl

```bash id="6x7k7m"
cat <<EOF | tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sysctl --system
```

---

## ✅ Verify Configuration

```bash id="pj2bsp"
lsmod | grep br_netfilter
lsmod | grep overlay

sysctl net.bridge.bridge-nf-call-iptables \
       net.bridge.bridge-nf-call-ip6tables \
       net.ipv4.ip_forward
```

---

## 📦 Install Container Runtime (containerd)

```bash id="r4w6u5"
curl -LO https://github.com/containerd/containerd/releases/download/v1.7.14/containerd-1.7.14-linux-amd64.tar.gz
tar Cxzvf /usr/local containerd-1.7.14-linux-amd64.tar.gz

curl -LO https://raw.githubusercontent.com/containerd/containerd/main/containerd.service
mkdir -p /usr/local/lib/systemd/system/
mv containerd.service /usr/local/lib/systemd/system/

mkdir -p /etc/containerd
containerd config default | tee /etc/containerd/config.toml

sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml

systemctl daemon-reload
systemctl enable --now containerd
```

---

## 🔍 Verify containerd

```bash id="7v4zz8"
systemctl status containerd
```

---

## ⚙️ Install runc

```bash id="ptbg8u"
curl -LO https://github.com/opencontainers/runc/releases/download/v1.1.12/runc.amd64
install -m 755 runc.amd64 /usr/local/sbin/runc
```

---

## 🌐 Install CNI Plugins

```bash id="q0smjw"
curl -LO https://github.com/containernetworking/plugins/releases/download/v1.5.0/cni-plugins-linux-amd64-v1.5.0.tgz
mkdir -p /opt/cni/bin
tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.5.0.tgz
```

---

## ☸️ Install Kubernetes Components

```bash id="q3z9yt"
apt-get update
apt-get install -y apt-transport-https ca-certificates curl gpg
```

### Add Kubernetes Repository

```bash id="t1mf9d"
mkdir -p /etc/apt/keyrings

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.33/deb/Release.key \
  | gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] \
https://pkgs.k8s.io/core:/stable:/v1.33/deb/ /" \
| tee /etc/apt/sources.list.d/kubernetes.list
```

### Install Components

```bash id="3dfy7x"
apt-get update

apt-get install -y \
  kubelet=1.33.0-1.1 \
  kubeadm=1.33.0-1.1 \
  kubectl=1.33.0-1.1 \
  --allow-downgrades --allow-change-held-packages

apt-mark hold kubelet kubeadm kubectl
```

---

## 🔍 Verify Installation

```bash id="m4u6e3"
kubeadm version
kubelet --version
kubectl version --client
systemctl status kubelet
```

---

## 🔗 Join Worker Node to Cluster

Run the join command from Master:

```bash id="xk5q9r"
kubeadm join 10.0.0.4:6443 \
  --token <token> \
  --discovery-token-ca-cert-hash <hash>
```

---

## ✅ Verify Cluster (From Master Node)

```bash id="9x6l4d"
kubectl get nodes
```

### 🎯 Expected Output

```bash id="k6z8h1"
NAME        STATUS   ROLES           AGE     VERSION
mastervm1   Ready    control-plane   53m     v1.33.0
mastervm2   Ready    control-plane   35m     v1.33.0
worker1     Ready    <none>          5m41s   v1.33.0
worker2     Ready    <none>          2m13s   v1.33.0
```
---

## 🔍 Step 8: Verify HAProxy (Load Balancer Health Check)

After setting up HAProxy, verify that it is properly routing traffic to control plane nodes.

---

### ✅ 1. Check HAProxy Service Status

On the Load Balancer VM:

```bash
sudo systemctl status haproxy
```

✔ Ensure status is:

```
active (running)
```

---

### 📊 2. Check HAProxy Backend Status

#### Option A: Using Stats Page (if enabled)

Open in browser:

```
http://<LB-IP>:<stats-port>
```

---

#### Option B: Using CLI

```bash
echo "show stat" | sudo socat unix-connect:/run/haproxy/admin.sock stdio

Harish@lb-harish:~$ echo "show stat" | sudo socat unix-connect:/run/haproxy/admin.sock stdio
# pxname,svname,qcur,qmax,scur,smax,slim,stot,bin,bout,dreq,dresp,ereq,econ,eresp,wretr,wredis,status,weight,act,bck,chkfail,chkdown,lastchg,downtime,qlimit,pid,iid,sid,throttle,lbtot,tracked,type,rate,rate_lim,rate_max,check_status,check_code,check_duration,hrsp_1xx,hrsp_2xx,hrsp_3xx,hrsp_4xx,hrsp_5xx,hrsp_other,hanafail,req_rate,req_rate_max,req_tot,cli_abrt,srv_abrt,comp_in,comp_out,comp_byp,comp_rsp,lastsess,last_chk,last_agt,qtime,ctime,rtime,ttime,agent_status,agent_code,agent_duration,check_desc,agent_desc,check_rise,check_fall,check_health,agent_rise,agent_fall,agent_health,addr,cookie,mode,algo,conn_rate,conn_rate_max,conn_tot,intercepted,dcon,dses,wrew,connect,reuse,cache_lookups,cache_hits,srv_icur,src_ilim,qtime_max,ctime_max,rtime_max,ttime_max,eint,idle_conn_cur,safe_conn_cur,used_conn_cur,need_conn_est,uweight,agg_server_status,agg_server_check_status,agg_check_status,srid,sess_other,h1sess,h2sess,h3sess,req_other,h1req,h2req,h3req,proto,-,ssl_sess,ssl_reused_sess,ssl_failed_handshake,quic_rxbuf_full,quic_dropped_pkt,quic_dropped_pkt_bufoverrun,quic_dropped_parsing_pkt,quic_socket_full,quic_sendto_err,quic_sendto_err_unknwn,quic_sent_pkt,quic_lost_pkt,quic_too_short_dgram,quic_retry_sent,quic_retry_validated,quic_retry_error,quic_half_open_conn,quic_hdshk_fail,quic_stless_rst_sent,quic_conn_migration_done,quic_transp_err_no_error,quic_transp_err_internal_error,quic_transp_err_connection_refused,quic_transp_err_flow_control_error,quic_transp_err_stream_limit_error,quic_transp_err_stream_state_error,quic_transp_err_final_size_error,quic_transp_err_frame_encoding_error,quic_transp_err_transport_parameter_error,quic_transp_err_connection_id_limit,quic_transp_err_protocol_violation_error,quic_transp_err_invalid_token,quic_transp_err_application_error,quic_transp_err_crypto_buffer_exceeded,quic_transp_err_key_update_error,quic_transp_err_aead_limit_reached,quic_transp_err_no_viable_path,quic_transp_err_crypto_error,quic_transp_err_unknown_error,quic_data_blocked,quic_stream_data_blocked,quic_streams_blocked_bidi,quic_streams_blocked_uni,h3_data,h3_headers,h3_cancel_push,h3_push_promise,h3_max_push_id,h3_goaway,h3_settings,h3_no_error,h3_general_protocol_error,h3_internal_error,h3_stream_creation_error,h3_closed_critical_stream,h3_frame_unexpected,h3_frame_error,h3_excessive_load,h3_id_error,h3_settings_error,h3_missing_settings,h3_request_rejected,h3_request_cancelled,h3_request_incomplete,h3_message_error,h3_connect_error,h3_version_fallback,pack_decompression_failed,qpack_encoder_stream_error,qpack_decoder_stream_error,h2_headers_rcvd,h2_data_rcvd,h2_settings_rcvd,h2_rst_stream_rcvd,h2_goaway_rcvd,h2_detected_conn_protocol_errors,h2_detected_strm_protocol_errors,h2_rst_stream_resp,h2_goaway_resp,h2_open_connections,h2_backend_open_streams,h2_total_connections,h2_backend_total_streams,h1_open_connections,h1_open_streams,h1_total_connections,h1_total_streams,h1_bytes_in,h1_bytes_out,h1_spliced_bytes_in,h1_spliced_bytes_out,
kubernetes-api,FRONTEND,,,6,10,262117,110,1034581,4370901,0,0,0,,,,,OPEN,,,,,,,,,1,2,0,,,,0,0,0,18,,,,,,,,,,,0,0,0,,,0,0,0,0,,,,,,,,,,,,,,,,,,,,,tcp,,0,18,110,,0,0,0,,,,,,,,,,,0,,,,,,,,,,110,0,0,0,0,0,0,0,,-,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
kube-master-nodes,master1,0,0,5,7,250,57,926423,4031756,,0,,0,0,0,0,UP,100,1,0,1,1,4619,1251,256,1,3,1,,57,,2,0,,9,L4OK,,1,,,,,,,,,,,3,0,,,,,435,,,0,1,0,3862,,,,Layer4 check passed,,2,2,3,,,,10.0.0.5:6443,,tcp,,,,,,,,0,57,0,,,0,,0,21,0,90054,0,0,0,0,1,100,,,,0,,,,,,,,,,-,0,0,0,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,
kube-master-nodes,master2,0,0,1,4,250,37,108158,339145,,0,,0,4,0,0,UP,100,1,0,1,1,3547,2319,256,1,3,2,,37,,2,0,,9,L4OK,,0,,,,,,,,,,,4,4,,,,,435,,,0,1,0,3104,,,,Layer4 check passed,,2,2,3,,,,10.0.0.6:6443,,tcp,,,,,,,,0,37,0,,,0,,0,2,0,90068,0,0,0,0,1,100,,,,0,,,,,,,,,,-,0,0,0,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,
kube-master-nodes,BACKEND,0,0,6,10,26212,110,1034581,4370901,0,0,,16,4,0,0,UP,200,2,0,,1,4679,1247,,1,3,0,,94,,1,0,,18,,,,,,,,,,,,,,7,4,0,0,0,0,435,,,0,1,0,3564,,,,,,,,,,,,,,tcp,roundrobin,,,,,,,0,94,0,,,,,0,21,0,90068,0,,,,,200,0,0,0,,,,,,,,,,,-,0,0,0,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
Harish@lb-harish:~$ 
```

---

### 🔎 What to Verify

Look for:

* `kube-master-nodes`
* Backend servers:

  * `master1`
  * `master2`

✔ Expected:

```
Status: UP
Check: L4OK / Layer4 check passed
```

---

### 🌐 3. Test Load Balancer Connectivity

From LB or any client:

```bash
nc -vz 10.0.0.4 6443
```

✔ Expected:

```
Harish@lb-harish:~$ # Test LB -> master API
nc -vz 10.0.0.4 6443
Connection to 10.0.0.4 6443 port [tcp/*] succeeded!
Harish@lb-harish:~$

```

👉 This confirms:

* HAProxy is listening
* API server is reachable via LB

---

### 📜 4. Check HAProxy Logs (Real-time)

```bash
sudo journalctl -u haproxy -f
```

✔ Useful for:

* Debugging failures
* Watching live traffic

---

## 🧪 Step 9: Simulate Failure (High Availability Test)

This is the **most important validation step** ✅

---

### 🔻 Simulate Control Plane Failure

On one master node:

```bash
sudo systemctl stop kubelet
```

OR shut down the VM.

---

### 🔍 Verify Cluster Behavior

Run on any active master:

```bash
kubectl get nodes
kubectl get pods -A
```

---

### ✅ Expected Outcome

* Cluster **continues to respond**
* Other control plane nodes take over
* No API downtime via Load Balancer
* Workloads keep running

---

## 🏁 Final Cluster State

```bash
kubectl get nodes
```

Example:

```
NAME        STATUS   ROLES           AGE     VERSION
mastervm1   Ready    control-plane   53m     v1.33.0
mastervm2   Ready    control-plane   35m     v1.33.0
worker1     Ready    <none>          5m41s   v1.33.0
worker2     Ready    <none>          2m13s   v1.33.0
```

---

# 🎉 Conclusion

You have successfully built a:

✅ Highly Available Kubernetes Cluster
✅ Multi-Control Plane Setup
✅ HAProxy Load Balanced API Server
✅ Fault-Tolerant Infrastructure

---

# 💡 Key Takeaways

* Control plane HA removes single point of failure
* HAProxy ensures seamless API access
* ETCD replication keeps cluster state consistent
* Failure testing is critical for validation

---

# 📌 Key Notes

* Worker nodes:

  * Do **not** run control plane components
  * Only run application workloads

* Ensure:

  * Same Kubernetes version across all nodes
  * Proper network connectivity to control plane
  * Use **private IPs** for node communication

* If a node shows `NotReady`, check:

  * `kubelet` status
  * CNI plugin (Calico)
  * Firewall / NSG rules
  * Network connectivity

---



# 🚀 Final Result

✅ Multi-master HA Kubernetes cluster
✅ Multiple worker nodes successfully joined
✅ Load-balanced control plane
✅ Fault-tolerant ETCD (stacked)
✅ Fully functional and production-ready setup

---

# 📊 Summary

| Component     | Status     |
| ------------- | ---------- |
| Control Plane | HA ✅       |
| Worker Nodes  | Scalable ✅ |
| Load Balancer | Working ✅  |
| Networking    | Calico ✅   |

---


