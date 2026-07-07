# Online Boutique Helm Chart

This repository contains the Helm chart version of the Online Boutique Kubernetes project.

The application was originally deployed using normal Kubernetes YAML files, then migrated to Helm to make the deployment cleaner, reusable, and easier to maintain.

---

## Final Result

The application is successfully deployed using Helm and is accessible from the browser through the frontend NodePort.

Application URL in the local lab:

```text
http://192.168.1.14:30080
```

**Application running in browser:**

<img width="1917" height="976" alt="image" src="https://github.com/user-attachments/assets/d2fd5da2-1524-45d8-af2e-54ef9a123cc2" />


---

## Pods Status

All application Pods are running successfully in the `online-boutique` namespace.

**All Pods Running:**

<img width="982" height="237" alt="image" src="https://github.com/user-attachments/assets/c9272ffd-5bb1-4aaa-9496-5ecbd6755f3a" />

---

## Project Goal

The goal of this work was to migrate the existing Online Boutique Kubernetes deployment from plain YAML manifests to a Helm chart.

Instead of editing many Kubernetes files manually, repeated configuration values were moved into `values.yaml`.

This makes the deployment easier to update, cleaner to manage, and more reusable.

---

## What Is Helm?

Helm is a package manager and templating tool for Kubernetes.

It helps package Kubernetes resources such as Deployments, Services, PersistentVolumes, and PersistentVolumeClaims into one reusable chart.

With Helm, Kubernetes YAML files are stored as templates, and configurable values are stored in `values.yaml`.

Example template value:

```yaml
namespace: {{ .Values.namespace | quote }}
```

The real value comes from `values.yaml`:

```yaml
namespace: online-boutique
```

After rendering, Helm produces normal Kubernetes YAML.

---

## Project Structure

```text
.
├── Chart.yaml
├── values.yaml
└── templates/
    ├── adservice.yaml
    ├── cartservice.yaml
    ├── checkoutservice.yaml
    ├── currencyservice.yaml
    ├── emailservice.yaml
    ├── frontendservice.yaml
    ├── paymentservice.yaml
    ├── productcatalogservice.yaml
    ├── recommendationservice.yaml
    ├── redis-cart.yaml
    └── shippingservice.yaml
```

---

## Main Helm Files

### Chart.yaml

`Chart.yaml` contains metadata about the Helm chart.

It includes:

- Chart name
- Chart version
- Application version
- Chart description
- Chart type

Current chart information:

```yaml
apiVersion: v2
name: online-boutique
description: Online Boutique microservices Helm chart
type: application
version: 0.1.0
appVersion: "1.0.0"
```

### values.yaml

`values.yaml` contains the configurable values used by the Helm templates.

In this project, it manages:

- Namespace
- Replica count
- Image registry
- Image tag
- Image pull policy
- Backend service type
- Frontend service type
- Frontend NodePort
- Service ports
- Redis configuration
- Shopping assistant address
- Environment variable values
- Profiler setting

### templates/

The `templates/` directory contains the Kubernetes manifests as Helm templates.

Each microservice has its own template file.

The templates still define normal Kubernetes resources such as:

- Deployments
- Services
- PersistentVolume
- PersistentVolumeClaim

But repeated values are now dynamic and come from `values.yaml`.

---

## Why Helm Was Used

Before Helm, the project used normal Kubernetes YAML files.

That worked, but many values were repeated across many files, such as:

- Namespace
- Image registry
- Image tag
- Image pull policy
- Replica count
- Service type
- Ports
- Environment variable addresses

If one value needed to change, many YAML files had to be edited manually.

After migrating to Helm, these values are managed from one place: `values.yaml`.

This makes the project:

- Cleaner
- Easier to update
- Easier to maintain
- More reusable
- More professional as a Kubernetes deployment

---

## What Was Done

The existing Kubernetes YAML files were moved into the Helm `templates/` directory.

Then the hardcoded repeated values were replaced with Helm variables.

The following values were moved into `values.yaml`:

- Namespace
- Replica count
- Image registry
- Image tag
- Image pull policy
- Backend service type
- Frontend service type
- Frontend NodePort
- Service ports
- Redis name and port
- Environment variable service addresses
- Profiler disable setting

---

## Namespace

The namespace is managed from `values.yaml`.

```yaml
namespace: online-boutique
```

Used in templates as:

```yaml
namespace: {{ .Values.namespace | quote }}
```

Rendered result:

```yaml
namespace: "online-boutique"
```

---

## Replicas

Most microservices use the same replica count:

```yaml
replicaCount: 1
```

Redis has its own replica value:

```yaml
redis:
  replicaCount: 1
```

Redis is separated because it uses persistent storage and should not be scaled accidentally with the stateless microservices.

---

## Image Configuration

All custom microservice images use the same registry, tag, and pull policy:

```yaml
image:
  registry: docker.io/online-boutique
  tag: v1
  pullPolicy: Never
```

Example rendered image:

```text
docker.io/online-boutique/frontendservice:v1
```

The image pull policy is set to `Never` because the images were already built and imported locally into the Kubernetes node runtime.

---

## Service Types

Backend services use `ClusterIP` because they only need internal communication inside the cluster.

```yaml
backend:
  service:
    type: ClusterIP
```

The frontend service uses `NodePort` because it needs to be accessible from outside the cluster.

```yaml
frontend:
  service:
    type: NodePort
    port: 8080
    targetPort: 8080
    nodePort: 30080
```

---

## NodePort Explanation

The frontend service uses:

```yaml
port: 8080
targetPort: 8080
nodePort: 30080
```

Meaning:

- `nodePort` is the external port opened on the Kubernetes node.
- `port` is the service port inside Kubernetes.
- `targetPort` is the container port inside the frontend Pod.

Traffic flow:

```text
Browser
  ↓
NodeIP:30080
  ↓
frontend Service:8080
  ↓
frontend Pod:8080
```

---

## Services

| Service | Type | Port |
|---|---|---|
| adservice | ClusterIP | 9555 |
| cartservice | ClusterIP | 7070 |
| checkoutservice | ClusterIP | 5050 |
| currencyservice | ClusterIP | 7000 |
| emailservice | ClusterIP | 8080 |
| frontendservice | NodePort | 8080 / 30080 |
| paymentservice | ClusterIP | 50051 |
| productcatalogservice | ClusterIP | 3550 |
| recommendationservice | ClusterIP | 8080 |
| redis-cart | ClusterIP | 6379 |
| shippingservice | ClusterIP | 50051 |

---

## Redis

Redis is used by the cart service.

Redis configuration is managed in `values.yaml`:

```yaml
redis:
  name: redis-cart
  replicaCount: 1
  service:
    type: ClusterIP
    port: 6379
```

The cart service connects to Redis using:

```text
redis-cart:6379
```

Redis also uses persistent storage through:

- PersistentVolume
- PersistentVolumeClaim

This keeps Redis data outside the container lifecycle.

---

## Environment Variables

The microservices communicate using Kubernetes service names and ports.

Examples:

```text
PRODUCT_CATALOG_SERVICE_ADDR=productcatalogservice:3550
CART_SERVICE_ADDR=cartservice:7070
REDIS_ADDR=redis-cart:6379
CURRENCY_SERVICE_ADDR=currencyservice:7000
CHECKOUT_SERVICE_ADDR=checkoutservice:5050
SHIPPING_SERVICE_ADDR=shippingservice:50051
PAYMENT_SERVICE_ADDR=paymentservice:50051
EMAIL_SERVICE_ADDR=emailservice:8080
```

These values were converted from hardcoded values into Helm template values.

Example template:

```yaml
value: "productcatalogservice:{{ .Values.services.productcatalogservice.port }}"
```

Rendered output:

```yaml
value: "productcatalogservice:3550"
```


---


### frontendservice Fix

After deploying with Helm, `frontendservice` was crashing.

The logs showed:

```text
panic: environment variable "SHOPPING_ASSISTANT_SERVICE_ADDR" not set
```

Even though the shopping assistant service was not deployed in this local setup, the frontend application code required the environment variable at startup.

The issue was fixed by adding:

```text
SHOPPING_ASSISTANT_SERVICE_ADDR=shoppingassistantservice:8080
```

to the frontendservice Helm template.

---

## Helm Commands Used

Check Helm version:

```bash
helm version
```

Check Kubernetes node:

```bash
kubectl get nodes
```

Check existing Helm releases:

```bash
helm list -A
```

Create the Helm chart:

```bash
helm create online-boutique
```

Render the chart without installing:

```bash
helm template online-boutique .
```

Validate the chart:

```bash
helm lint .
```

Install the chart:

```bash
helm install online-boutique . -n online-boutique
```

Upgrade after changes:

```bash
helm upgrade online-boutique . -n online-boutique
```

Check Helm release:

```bash
helm list -n online-boutique
```

Check Helm status:

```bash
helm status online-boutique -n online-boutique
```

View the rendered manifest from the installed release:

```bash
helm get manifest online-boutique -n online-boutique
```

---

## Deployment Steps

Create the namespace if it does not exist:

```bash
kubectl create namespace online-boutique
```

Install the Helm chart:

```bash
helm install online-boutique . -n online-boutique
```

Check the Helm release:

```bash
helm list -n online-boutique
```

Check the Pods:

```bash
kubectl get pods -n online-boutique
```

Check the Services:

```bash
kubectl get svc -n online-boutique
```

---

## Final Verification

The Helm release was deployed successfully.

```text
NAME              NAMESPACE         REVISION   STATUS     CHART
online-boutique   online-boutique   2          deployed   online-boutique-0.1.0
```

All Pods are running successfully:

```text
adservice                 Running
cartservice               Running
checkoutservice           Running
currencyservice           Running
emailservice              Running
frontendservice           Running
paymentservice            Running
productcatalogservice     Running
recommendationservice     Running
redis-cart                Running
shippingservice           Running
```

The frontend service is exposed using NodePort:

```text
frontendservice   NodePort   8080:30080/TCP
```

Application access in the local lab:

```text
http://192.168.1.14:30080
```

---

## Final Status

The Online Boutique application is now deployed and managed using Helm.

The old manual Kubernetes deployment was replaced with a Helm-based deployment.

The application is running successfully, all Pods are healthy, and the frontend is accessible from the browser.

---

## Summary

This project demonstrates how to migrate a Kubernetes microservices deployment from plain YAML manifests to a Helm chart.

The main improvement is that repeated configuration values are now centralized in `values.yaml`, while Kubernetes resources are managed as reusable Helm templates.

This makes the deployment cleaner, easier to update, and easier to maintain.
