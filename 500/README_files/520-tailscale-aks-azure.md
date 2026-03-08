# 520 — Tailscale on AKS (Azure Kubernetes Service)

## Why Tailscale + AKS?

Azure Kubernetes Service clusters are often deployed in private VNets with no public endpoint. Traditional access methods (Azure Bastion, P2S VPN, ExpressRoute) are either expensive or complex to configure.

Tailscale provides an alternative: connect your AKS workloads directly into your tailnet with minimal configuration, enabling:

- `kubectl` access to private AKS clusters from any tailnet device
- Service-to-service communication across Azure, on-prem, and other clouds
- Secure CI/CD access (GitHub Actions → AKS) without exposing the API server publicly

## Architecture

```
┌─────────────────────────────────────────────┐
│  Azure (AKS Private Cluster)                │
│                                             │
│  ┌─────────────────────────────────────┐    │
│  │  Tailscale Operator (Deployment)    │    │
│  │  ┌────────────┐  ┌──────────────┐   │    │
│  │  │ Proxy Pod  │  │ Subnet Router│   │    │
│  │  │ (Service)  │  │ (VNet CIDR)  │   │    │
│  │  └─────┬──────┘  └──────┬───────┘   │    │
│  └────────┼────────────────┼───────────┘    │
└───────────┼────────────────┼───────────────-┘
            │                │
      Tailscale mesh (WireGuard)
            │                │
   ┌────────▼────────────────▼─────────┐
   │  Your tailnet                     │
   │  - Developer laptops              │
   │  - GitHub Actions runners         │
   │  - Other cloud workloads          │
   └───────────────────────────────────┘
```

## Setup: Private AKS + Tailscale Operator

### Step 1: Create AKS Cluster (private)

```bash
az aks create \
  --resource-group myRG \
  --name myAKS \
  --enable-private-cluster \
  --network-plugin azure \
  --vnet-subnet-id /subscriptions/.../subnets/aks-subnet \
  --node-count 3
```

### Step 2: Get Kubeconfig (from Azure VM or Jump Host)

```bash
az aks get-credentials --resource-group myRG --name myAKS
```

### Step 3: Install Tailscale Operator

```bash
# Create Tailscale OAuth secret
kubectl create namespace tailscale
kubectl create secret generic operator-oauth \
  --from-literal=client_id=<CLIENT_ID> \
  --from-literal=client_secret=<CLIENT_SECRET> \
  -n tailscale

# Install Operator
helm upgrade --install \
  tailscale-operator tailscale/tailscale-operator \
  --namespace tailscale \
  --set oauth.clientId=<CLIENT_ID> \
  --set oauth.clientSecret=<CLIENT_SECRET>
```

### Step 4: Expose the API Server

```yaml
# tailscale-apiserver-proxy.yaml
apiVersion: tailscale.com/v1alpha1
kind: ProxyClass
metadata:
  name: apiserver-proxy
spec:
  statefulSet:
    pod:
      tailscaleContainer:
        extraArgs:
          - --kube-apiserver-proxy=true
```

### Step 5: Access from Any Tailnet Device

```bash
# Update kubeconfig to use Tailscale endpoint
kubectl config set-cluster myAKS \
  --server=https://myaks.<tailnet>.ts.net

# Verify access
kubectl get nodes
```

## Subnet Router: Expose Azure VNet to Tailnet

Expose your entire AKS VNet (including non-Kubernetes Azure resources) into the tailnet:

```yaml
apiVersion: tailscale.com/v1alpha1
kind: Connector
metadata:
  name: azure-vnet-router
  namespace: tailscale
spec:
  subnetRouter:
    advertiseRoutes:
      - "10.224.0.0/12"    # AKS node CIDR
      - "10.0.0.0/8"       # Azure VNet range
  tags:
    - "tag:azure-subnet-router"
```

This means any Azure resource (VMs, databases, storage accounts with private endpoints) becomes reachable from any tailnet device by private IP.

## GitHub Actions → AKS

See module 530 for complete GitHub Actions workflow patterns for deploying to private AKS clusters via Tailscale.

## Security Notes for Azure

- Use **Managed Identity** for AKS nodes (avoid storing credentials)
- Use **Azure Key Vault** for Tailscale auth keys in production
- Tag AKS Tailscale nodes with `tag:aks-node` and write ACLs accordingly
- Enable **device expiry** for ephemeral nodes (CI runners, etc.)
- Consider **Tailscale Funnel** for exposing webhook endpoints without full tailnet access
