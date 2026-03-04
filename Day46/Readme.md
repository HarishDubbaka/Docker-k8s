## What Are Custom Resource Definitions (CRDs)?

CRDs allow you to extend Kubernetes by defining your own resource types. These custom resources work just like Kubernetes native resources, such as Pods or Services, but are specifically designed to meet your unique needs.

---

## Why Are CRDs Useful?

* Extend Kubernetes to handle things it normally can’t, like managing external services or custom workflows.
* Behave like regular Kubernetes resources — you can create, update, or delete them using the Kubernetes API.
* Essential for building **Kubernetes Operators**, which automate complex application lifecycle management.

---

## What Is a Custom Resource Definition (CRD)?

A CRD is a mechanism to extend the Kubernetes API by defining custom resource types.

* Developers can create objects that behave like built-in Kubernetes resources.
* Example: A `Database` CRD could automate provisioning and scaling database instances.
* **Important:** CRDs alone do not perform actions — a **controller** is required to act on CRs.

---

## Difference Between an API and CRDs

| Feature     | Kubernetes API                                           | CRDs                                             |
| ----------- | -------------------------------------------------------- | ------------------------------------------------ |
| Purpose     | Built-in way for components to interact programmatically | Extend Kubernetes API with custom resource types |
| Managed By  | Kubernetes itself                                        | User-defined + requires controller for actions   |
| Examples    | Pods, Services, Deployments                              | MySQLCluster, AppConfig, S3Bucket                |
| Flexibility | Fixed resources                                          | Custom abstractions for specific applications    |

**API extensions** (Aggregated API Servers, Admission Webhooks) allow deeper customization, but CRDs are easier and faster to implement for most use cases.

---

## How to Create a Kubernetes CRD

### 1️⃣ Define the CRD YAML

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: foos.example.com
spec:
  group: example.com
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                size:
                  type: integer
                color:
                  type: string
  scope: Namespaced
  names:
    plural: foos
    singular: foo
    kind: Foo
    shortNames:
    - f
```

---

### 2️⃣ Apply the CRD

```bash
kubectl apply -f crd.yml
kubectl get crd | grep foo
```

> Output confirms the CRD is registered:
> `foos.example.com  2026-03-04T04:37:12Z`

---

### 3️⃣ Create a Custom Resource (CR)

```yaml
apiVersion: example.com/v1
kind: Foo
metadata:
  name: my-foo
spec:
  size: 3
  color: "black-and-white"
```

Apply it:

```bash
kubectl apply -f cr.yml
kubectl get Foo
```

> Output:
> `NAME     AGE`
> `my-foo   108s`

---

### 4️⃣ Build and Run the Sample Controller

The controller watches `Foo` resources and creates Deployments automatically.

```bash
# Ensure Go is installed
sudo apt-get update -y
sudo apt install golang-go -y

# Fix go.mod if needed
sed -i 's/go 1\.24\.0/go 1.18/' go.mod
sed -i '/^godebug/d' go.mod
go mod tidy

# Build the sample-controller
go build -o sample-controller .

# Run the controller (keep running in a separate terminal)
./sample-controller -kubeconfig=$HOME/.kube/config
```

> **Note:** Keep this process running in the background or a separate terminal. The controller continuously monitors for `Foo` resources and reconciles them into Deployments.

---

### 5️⃣ CRDs vs Deployments

* **CRD + CR alone** does **not** create a Deployment.
* **Controller running** = CRs are reconciled into actual Kubernetes objects (Deployments, Services, etc).

```bash
kubectl get deployments
# You should now see a deployment created by the controller
```

---

### ✅ Key Takeaways

* **CRD = API definition** (Kubernetes recognizes a new type)
* **CR = Instance** (declares desired state)
* **Controller = Automation** (creates/manages resources based on CR)

**Rule:**

```text
CRD + CR ≠ Deployment automatically
CRD + CR + Controller = Deployment (or other resources)
```

---

### 6️⃣ Clean Up (Optional)

```bash
kubectl delete cr my-foo
kubectl delete crd foos.example.com
```

---

This now **includes building and running the sample controller** so that your CRs will actually create Deployments.

