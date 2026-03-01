# Day44 – Kubernetes Ingress vs Gateway API

One of the most widely used solutions for managing containers and orchestrating microservices is **Kubernetes**.  

Kubernetes provides two main options for managing inbound traffic to microservices:

- **Ingress**
- **Gateway API**

This document explains both approaches and why Gateway API is considered the evolution of Kubernetes networking.

---

# Kubernetes Ingress

Kubernetes **Ingress** is a resource used to manage external HTTP/HTTPS access to services inside a cluster.

It centralizes routing rules such as:
- Host-based routing
- Path-based routing
- TLS termination

Ingress relies on an **Ingress Controller** (e.g., NGINX, HAProxy) to translate rules into load balancer or proxy configurations.

### Limitations of Ingress

- Supports only **HTTP/HTTPS** (Layer 7)
- gRPC and TCP/UDP require custom extensions
- Heavy reliance on **vendor-specific annotations**
- Limited built-in traffic management
- No native advanced routing (header-based matching)
- No native L4 support

---

# Kubernetes Gateway API

The **Gateway API** is the next-generation Kubernetes networking model designed to overcome Ingress limitations.

It introduces standardized, portable resources for:

- Layer 4 (TCP/UDP)
- Layer 7 (HTTP, HTTPS, gRPC)

It is designed to eventually replace Ingress for complex traffic management.

---

## Key Gateway API Resources

| Resource | Purpose |
|-----------|----------|
| `GatewayClass` | Defines controller capabilities |
| `Gateway` | Instantiates a network gateway |
| `HTTPRoute` | Defines HTTP routing rules |
| `TCPRoute` | Defines TCP routing rules |
| `TLSRoute` | Defines TLS routing rules |
| `ReferenceGrant` | Enables secure cross-namespace routing |

---

## Advantages Over Ingress

- Multi-protocol support (HTTP, HTTPS, gRPC, TCP, UDP)
- Advanced traffic control (splitting, mirroring, direct responses)
- Header-based routing
- Weighted routing (canary, blue-green deployments)
- Cross-namespace routing
- Built-in policy support
- Reduced vendor-specific annotations
- Service mesh friendly (Istio, Linkerd, etc.)

---

# Comparison Table

| Feature | Ingress | Gateway API |
|----------|----------|-------------|
| Protocol support | HTTP/HTTPS | HTTP, HTTPS, TCP, UDP |
| Resource type | `Ingress` | `Gateway`, `HTTPRoute`, etc. |
| Routing rules | Host, Path | Host, Path, Header, Weight, Method |
| Extensibility | Limited | High |
| Controller required | Yes | Yes |
| Use case | Simple HTTP ingress | Complex traffic management |
| Namespace support | Limited | Multi-namespace |
| Status | Stable | Evolving |

---

# TL;DR

**Use Ingress if:**
- You only need simple HTTP/HTTPS routing.

**Use Gateway API if:**
- You need advanced routing
- Multi-protocol support
- Multi-team/multi-namespace architecture

---

# What is Kubernetes Gateway API?

The Gateway API is a Kubernetes networking feature that defines how traffic enters the cluster.

It was created to overcome Ingress limitations such as:

- HTTP-only support
- Limited extensibility
- Annotation fragmentation

---

# Gateway API Key Features

- Supports L4 & L7 routing
- HTTP, HTTPS, gRPC
- TCP/UDP (experimental)
- Header-based routing
- Weighted routing (canary / blue-green)
- Cross-namespace routing
- Portable configurations
- Service mesh integration

---

# Gateway API Controller

Like Ingress, Gateway API requires a **controller**.

The controller:
- Watches Gateway API resources
- Translates them into proxy/load balancer configurations
- Routes traffic to backend services

Gateway API controllers are NOT built into Kubernetes.

---

## Popular Gateway API Controllers

- NGINX Gateway Fabric
- Kong Gateway
- Envoy Gateway

In this project, we use:

**NGINX Gateway Fabric**

---

# NGINX Gateway Fabric Architecture

NGINX Gateway Fabric consists of:

## 1. Control Plane Controller

- Watches custom resources
- Creates and configures data planes
- Translates Gateway & Route resources into configuration

## 2. Data Planes

- Actual traffic routing components
- Receive traffic from external load balancer
- Route traffic to backend services

---

# Complete Gateway API Traffic Flow

1. Install Gateway API Controller.
2. Create a `GatewayClass`.
3. Create a `Gateway` resource.
4. Controller creates Data Plane pods.
5. External Load Balancer sends traffic to Data Plane.
6. `HTTPRoute` defines routing rules.
7. Data Plane routes traffic to backend services.
8. Service forwards traffic to Pods.

---

# Setup Prerequisites

- Kubernetes v1.30+
- Helm installed locally
- Kubectl installed locally
- Gateway API Controller (NGINX Gateway Fabric)

---

# Summary

The Gateway API is an improved and modern replacement for Kubernetes Ingress.

It provides:

- Standardized traffic management
- Advanced routing capabilities
- Multi-protocol support
- Better multi-team architecture
- Reduced vendor lock-in

For modern production-grade Kubernetes networking, Gateway API is the recommended approach.

---

# End of Day44
