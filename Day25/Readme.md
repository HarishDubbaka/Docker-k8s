# Kubernetes ConfigMaps and Secrets

Kubernetes, the leading container orchestration platform, is designed to help developers and operators manage containerized applications at scale. It provides a suite of powerful primitives to facilitate application deployment, configuration, scaling, and management. Two of its core and often underestimated features are **ConfigMaps** and **Secrets**.

These Kubernetes-native objects are critical to building secure, scalable, and maintainable applications. They allow you to separate configuration from code and protect sensitive data such as credentials or API tokens. In modern DevOps workflows, where infrastructure and application lifecycle are tightly coupled, understanding and leveraging ConfigMaps and Secrets effectively is a must.

---

## 🔧 Understanding ConfigMaps

### What Are ConfigMaps?
A ConfigMap is a Kubernetes object used to store **non-sensitive configuration data** in key-value pairs. It enables you to inject external configuration into your applications without rebuilding container images. With this approach, you can easily adjust configurations between different environments like dev, staging, and production.

**Key points:**
- **Key-Value Store:** Each ConfigMap is composed of key-value pairs.
- **Multiple Consumption Methods:** ConfigMap data can be exposed as environment variables, command-line arguments, or mounted files.
- **Scoped by Namespace:** ConfigMaps are namespace-bound.
- **No Automatic Reload:** Applications must be restarted or coded to detect changes.

---

### Creating a ConfigMap

**From literals:**
```bash
kubectl create configmap my-config \
  --from-literal=firstname=Harish \
  --from-literal=secondname=Dubbaka
```

**From YAML:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-config
data:
  firstname: Harish
  secondname: Dubbaka
```

**From a file:**
```bash
kubectl create configmap my-config --from-file=config.properties
```

---

### Using ConfigMaps in Pods
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-config
spec:
  containers:
    - name: my-container
      image: nginx:latest
      env:
        - name: FIRSTNAME
          valueFrom:
            configMapKeyRef:
              name: my-config
              key: firstname
        - name: SECONDNAME
          valueFrom:
            configMapKeyRef:
              name: my-config
              key: secondname
  restartPolicy: Never
```

**Verify inside container:**
```bash
kubectl exec -it my-config -- /bin/sh
echo $FIRSTNAME   # Harish
echo $SECONDNAME  # Dubbaka
```

---

## 🔐 Understanding Secrets

### What Are Secrets?
Kubernetes Secrets securely hold **sensitive data** such as credentials, tokens, private keys, and connection strings. They are similar to ConfigMaps but intended for confidential information.

**Key points:**
- **Secure Key-Value Store:** For sensitive data.
- **Access Control:** Supports fine-grained RBAC.
- **Volume and Env Mounts:** Can be injected as env vars or mounted files.
- **Types:** Opaque, dockerconfigjson, TLS, etc.
- **Encoding:** Base64-encoded (not encrypted by default).

---

### Creating a Secret

**From CLI:**
```bash
kubectl create secret generic my-secret \
  --from-literal=username=admin \
  --from-literal=password=bantu123
```

**Verify:**
```bash
kubectl get secrets
kubectl describe secret my-secret
```

**Using in Pod:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-demo
spec:
  containers:
  - name: demo-container
    image: nginx
    env:
    - name: USERNAME
      valueFrom:
        secretKeyRef:
          name: my-secret
          key: username
    - name: PASSWORD
      valueFrom:
        secretKeyRef:
          name: my-secret
          key: password
```

**Verify inside container:**
```bash
kubectl exec -it secret-demo -- /bin/sh
echo $USERNAME   # admin
echo $PASSWORD   # bantu123
```

---

## 🅾️ ConfigMaps vs. Secrets

| Feature          | ConfigMap            | Secret                  |
|------------------|----------------------|-------------------------|
| Purpose          | Non-sensitive data   | Sensitive data          |
| Encoding         | Plaintext            | Base64 encoded          |
| Access Controls  | Basic RBAC           | Strict RBAC + encryption|
| Use Cases        | Flags, configs       | Credentials, tokens     |
| Security Level   | Low                  | High                    |

---

## ✅ Best Practices
- **Enable Encryption at Rest:** Encrypt Secret data in etcd.
- **Limit Exposure:** Only expose to Pods that need them.
- **Use Namespaces Wisely:** Isolate workloads and apply RBAC.
- **Audit Access:** Monitor and log access to Secrets.
- **Rotate Regularly:** Rotate secrets and credentials.
- **CI/CD Automation:** Integrate into pipelines.
- **Avoid Mixing:** Never store secrets in ConfigMaps.

---

## 🏁 Conclusion
Kubernetes ConfigMaps and Secrets play a vital role in application configuration and data security. They help streamline deployments, enforce separation of concerns, and protect your infrastructure against misconfiguration and data leaks.

By mastering their creation, usage, and management, you can enforce best practices across environments and ensure a consistent and secure deployment workflow.
