# 510 — Tailscale Kubernetes Operator

## Overview

The **Tailscale Kubernetes Operator** allows you to manage Tailscale nodes and expose Kubernetes Services into your tailnet — all using standard Kubernetes manifests and annotations.

Instead of manually deploying Tailscale sidecars, the Operator handles:
- Creating and managing Tailscale nodes for each Service
- Rotating auth keys automatically
- Cleaning up nodes when Services are deleted
- HA proxy groups for high-availability ingress

## Installation

### Prerequisites

- Kubernetes cluster (1.25+)
- Helm 3+
- A Tailscale OAuth client (Settings → OAuth clients in admin panel)

### Create OAuth Client

In the Tailscale admin panel:
1. Go to **Settings → OAuth clients**
2. Create a new client with scopes: `devices:write`, `auth_keys`
3. Note the client ID and secret

### Install via Helm

```bash
helm repo add tailscale https://pkgs.tailscale.com/helmcharts
helm repo update

helm upgrade --install \
  tailscale-operator \
  tailscale/tailscale-operator \
  --namespace=tailscale \
  --create-namespace \
  --set-string oauth.clientId=<CLIENT_ID> \
  --set-string oauth.clientSecret=<CLIENT_SECRET> \
  --wait
```

## Exposing a Service into Your Tailnet

### Via Annotation (existing LoadBalancer/ClusterIP Service)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-api
  namespace: default
  annotations:
    tailscale.com/expose: "true"            # expose this service
    tailscale.com/hostname: "my-api"        # MagicDNS name: my-api.<tailnet>.ts.net
    tailscale.com/tags: "tag:k8s-service"   # optional tag
spec:
  selector:
    app: my-api
  ports:
    - port: 8080
      targetPort: 8080
```

After applying, the Operator:
1. Creates a Tailscale Pod in the `tailscale` namespace
2. Registers it as a node named `my-api` in your tailnet
3. All tailnet members with ACL access can reach `my-api` on port 8080

### Via Ingress Resource

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-ingress
  namespace: default
  annotations:
    kubernetes.io/ingress.class: tailscale
spec:
  rules:
    - host: my-app           # → my-app.<tailnet>.ts.net
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-app
                port:
                  number: 80
```

## ProxyGroup (High Availability)

For production services, use a `ProxyGroup` to run multiple Tailscale ingress replicas:

```yaml
apiVersion: tailscale.com/v1alpha1
kind: ProxyGroup
metadata:
  name: prod-ingress
  namespace: tailscale
spec:
  type: ingress
  replicas: 2
  tags:
    - "tag:k8s-ingress"
```

## Subnet Router on Kubernetes

Advertise your Kubernetes Pod and Service CIDR into the tailnet:

```yaml
apiVersion: tailscale.com/v1alpha1
kind: Connector
metadata:
  name: k8s-subnet-router
  namespace: tailscale
spec:
  subnetRouter:
    advertiseRoutes:
      - "10.0.0.0/16"    # Pod CIDR
      - "10.96.0.0/12"   # Service CIDR
  tags:
    - "tag:k8s-subnet-router"
```

This allows any tailnet device to reach Kubernetes Pods and Services directly by IP.

## Connecting to the Kubernetes API Server

With the Tailscale Operator, you can expose `kube-apiserver` as a tailnet node, enabling `kubectl` access without a bastion or VPN:

```bash
# After exposing the API server via the Operator
kubectl config set-cluster my-cluster \
  --server=https://k8s-api.my-tailnet.ts.net:6443
```

## Verify Operator Status

```bash
# Check Operator pods
kubectl get pods -n tailscale

# Check created Tailscale nodes
kubectl get proxies -n tailscale

# View Operator logs
kubectl logs -n tailscale deployment/operator
```
