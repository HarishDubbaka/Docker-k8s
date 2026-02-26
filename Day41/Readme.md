# 🌐 Domain Name System (DNS)

## What is DNS?
The **Domain Name System (DNS)** is like the internet’s phone book. It translates human-readable domain names (e.g., `google.com`) into machine-readable IP addresses (e.g., `142.250.191.14`).

### 🏪 Real-World Analogy: Postal System
- **Domain name:** "John's Pizza Shop, Main Street, Springfield" (human-friendly)
- **IP address:** "123 Main St, Springfield, IL 62701" (exact location)
- **DNS servers:** Post offices that know how to route your mail
- **DNS resolution:** The process of finding the exact address from the business name

Example:
```bash
You type: www.google.com
DNS translates to: 142.250.191.14
Your browser connects to: Google's web server
```

Without DNS, you’d have to memorize IP addresses for every website.

---

## 🔍 Practical Commands

### nslookup
```bash
nslookup www.google.com
```
Shows DNS server used and resolved IP addresses.

### ping
```bash
ping google.com
```
Tests connectivity and shows resolved IP (IPv4/IPv6).

### Inspect Local Config
```bash
cat /etc/hosts
cat /etc/resolv.conf
```
- `/etc/hosts`: Local static mappings of hostnames to IPs  
- `/etc/resolv.conf`: DNS resolver configuration  

---

## 🌳 Domain Name Hierarchy
Domains are read **right to left**:

Example: `mail.google.com.`  
- **Root (`.`):** Top of DNS tree (managed by IANA)  
- **TLD (`.com`):** Commercial domains (managed by VeriSign)  
- **Second-level (`google`):** Owned by Google  
- **Subdomain (`mail`):** Gmail service  

### Common TLDs
- `.com` → Commercial  
- `.org` → Organizations  
- `.net` → Networks  
- `.edu` → Education  
- `.gov` → Government  
- Country codes: `.us`, `.uk`, `.de`, `.jp`, `.ca`

---

## 🌍 Types of DNS Servers
- **Root DNS Servers:** 13 logical servers (A–M), operated globally with anycast.  
- **TLD Servers:** Manage domains within their TLD (e.g., `.com`, `.org`).  
- **Authoritative Servers:** Hold DNS records for a domain.  
- **Recursive Resolvers:** Query other servers on behalf of clients.

---

## 📊 DNS Record Types

| Record | Purpose | Example |
|--------|----------|---------|
| **A** | Maps domain → IPv4 | `example.com. IN A 192.0.2.1` |
| **AAAA** | Maps domain → IPv6 | `example.com. IN AAAA 2001:db8::1` |
| **CNAME** | Alias to another domain | `www.example.com. IN CNAME example.com.` |
| **MX** | Mail server | `example.com. IN MX 10 mail1.example.com.` |
| **TXT** | Text info (SPF, DKIM, verification) | `example.com. IN TXT "v=spf1 include:_spf.google.com ~all"` |
| **NS** | Delegates to name servers | `example.com. IN NS ns1.example.com.` |
| **SOA** | Start of Authority (zone info) | `example.com. IN SOA ns1.example.com. admin.example.com.` |
| **PTR** | Reverse DNS (IP → domain) | `1.2.0.192.in-addr.arpa. IN PTR mail.example.com.` |

---

## 🛠️ Quick Troubleshooting Checklist
1. Run `nslookup <domain>` → Verify DNS resolution.  
2. Run `ping <domain>` → Confirm connectivity.  
3. Check `/etc/resolv.conf` → Ensure correct DNS server.  
4. Check `/etc/hosts` → Rule out local overrides.  
5. Use `curl <ClusterIP>` inside Kubernetes → Validate service routing.  

---

# 📘 CoreDNS in Kubernetes 

---

# ☸️ What is CoreDNS?

**CoreDNS** is the default DNS server in a Kubernetes cluster.

It provides:

* 🔍 Service discovery
* 🌐 Name resolution for Pods and Services
* ⚡ Internal DNS inside the cluster

In Kubernetes, Pods and Services get IP addresses, but IPs are hard to remember.
CoreDNS allows communication using **names instead of IP addresses**.

---

## 🧠 Why CoreDNS is Needed

| Feature           | Description                          |
| ----------------- | ------------------------------------ |
| Service Discovery | Pods communicate using service names |
| Scalability       | Handles large volumes of DNS queries |
| Flexibility       | Plugin-based architecture            |
| Caching           | Reduces load on Kubernetes API       |

---

# 🏗 CoreDNS Architecture in Kubernetes

![Image](https://coredns.io/images/query-processing.png)

![Image](https://miro.medium.com/1%2AqOeIh0U8exjjYyjkZorEBA.png)


### DNS Flow Inside Cluster

1. Pod sends DNS query
2. Query goes to CoreDNS Service (`kube-dns`)
3. CoreDNS uses **kubernetes plugin**
4. It queries Kubernetes API
5. Returns Service IP
6. Result cached

---

# 🚀 Practical Example

## Step 1: Create Pods

```bash
kubectl run nginx1 --image nginx
kubectl run banpod --image nginx
```

Check IPs:

```bash
kubectl get pod -o wide
```

Example:

```
banpod   → 10.40.0.1
nginx1   → 10.38.0.1
```

---

## Step 2: Try Pod-to-Pod by Name

```bash
kubectl exec -it banpod -- curl nginx1
```

❌ Output:

```
Could not resolve host: nginx1
```

### Why?

👉 Pods are NOT automatically discoverable by name
👉 Only **Services** get DNS records

---

# ✅ Solution: Expose Pod as Service

```bash
kubectl expose pod banpod \
  --port=80 \
  --target-port=8080 \
  --name=banpod-service \
  --type=ClusterIP
```

Now Kubernetes creates DNS entry:

```
banpod-service.default.svc.cluster.local
```

Now other Pods can access:

```bash
curl banpod-service
```

---

# 🔍 Checking CoreDNS Installation

CoreDNS runs in `kube-system` namespace.

```bash
kubectl get pods -n kube-system
```

You should see:

```
coredns-xxxx   Running
```

---

## CoreDNS Service

```bash
kubectl get svc -n kube-system
```

Output:

```
kube-dns   ClusterIP   10.96.0.10   53/UDP,53/TCP
```

👉 `10.96.0.10` is the cluster DNS server IP.

---

# 📄 Pod DNS Configuration

Inside any pod:

```bash
kubectl exec -it banpod -- bash
cat /etc/resolv.conf
```

Output:

```
search default.svc.cluster.local svc.cluster.local cluster.local
nameserver 10.96.0.10
options ndots:5
```

### Meaning:

| Field      | Description                |
| ---------- | -------------------------- |
| search     | Default cluster DNS suffix |
| nameserver | CoreDNS IP                 |
| ndots      | DNS resolution rule        |

---

# 📄 /etc/hosts Inside Pod

```bash
cat /etc/hosts
```

Example:

```
127.0.0.1 localhost
10.32.0.1 banpod
```

* Contains only local pod info
* Kubernetes manages this file
* NOT used for service discovery

CoreDNS handles service resolution via `/etc/resolv.conf`.

---

# 🛠 CoreDNS Configuration (Corefile)

Check ConfigMap:

```bash
kubectl describe cm coredns -n kube-system
```

Important section:

```
.:53 {
    errors
    health
    ready
    kubernetes cluster.local in-addr.arpa ip6.arpa {
       pods insecure
       ttl 30
    }
    prometheus :9153
    forward . /etc/resolv.conf
    cache 30
    loop
    reload
    loadbalance
}
```

---

## 🔎 Important Plugins

| Plugin     | Purpose                       |
| ---------- | ----------------------------- |
| kubernetes | Resolves cluster services     |
| forward    | Forwards external DNS queries |
| cache      | Caches DNS results            |
| health     | Liveness check                |
| ready      | Readiness probe               |
| prometheus | Metrics endpoint              |

---

# 🔧 CoreDNS Pod Details

```bash
kubectl describe pod coredns-xxxx -n kube-system
```

Important fields:

* Image: `registry.k8s.io/coredns/coredns:v1.11.1`
* Ports: 53/UDP, 53/TCP
* Config from ConfigMap
* Health check on port 8080
* Readiness check on port 8181

If you see:

```
Readiness probe failed
```

It may indicate:

* API server connectivity issue
* Network plugin problem
* Resource exhaustion

---

# 🧩 Default Installation

When Kubernetes cluster is installed (kubeadm):

✅ CoreDNS is installed automatically
✅ Service `kube-dns` is created
✅ ConfigMap `coredns` is created

---

# 🔥 Troubleshooting CoreDNS

## Check CoreDNS Pods

```bash
kubectl get pods -n kube-system
kubectl logs <coredns-pod> -n kube-system
```

## Check Service

```bash
kubectl get svc -n kube-system
```

## Check ConfigMap

```bash
kubectl describe cm coredns -n kube-system
```

---

## If Network Plugin Issue (Example: Weave)

```bash
kubectl delete daemonset weave-net -n kube-system
kubectl delete deployment weave-net -n kube-system
kubectl delete clusterrole weave-net
kubectl delete clusterrolebinding weave-net
kubectl delete serviceaccount weave-net -n kube-system
```

---

# 🧠 Key Concepts to Remember

* Pods communicate via Services
* CoreDNS provides internal DNS
* `/etc/resolv.conf` points to CoreDNS
* DNS format in cluster:

```
<service>.<namespace>.svc.cluster.local
```

* CoreDNS uses plugins (kubernetes, forward, cache)

---

# 🎯 Final Summary

CoreDNS is:

* 📌 Default DNS server in Kubernetes
* 🔍 Responsible for service discovery
* ⚙️ Plugin-based and scalable
* 🚀 Critical for inter-pod communication

Without CoreDNS:

* Pods must communicate using IPs
* Service discovery breaks
* Cluster communication becomes difficult

---

## 📌 One-Line Definition 

> CoreDNS is the internal DNS server in Kubernetes that enables service discovery and name resolution for Pods and Services using DNS-based architecture.



