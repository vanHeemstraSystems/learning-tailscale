# 810 — Terraform Provider for Tailscale

## Overview

The **Tailscale Terraform provider** (`tailscale/tailscale`) allows you to manage your tailnet configuration as Infrastructure-as-Code: ACLs, DNS settings, auth keys, and device tags.

This integrates naturally with existing Azure/cloud Terraform modules — your network security posture and your infrastructure are declared in the same codebase.

## Provider Configuration

```hcl
# versions.tf
terraform {
  required_providers {
    tailscale = {
      source  = "tailscale/tailscale"
      version = "~> 0.17"
    }
  }
}

# providers.tf
provider "tailscale" {
  api_key = var.tailscale_api_key  # or use TS_API_KEY env var
  tailnet = var.tailnet_name       # e.g., "acme-corp.github"
}
```

## Managing ACL Policy

```hcl
# acl.tf
resource "tailscale_acl" "main" {
  acl = jsonencode({
    groups = {
      "group:developers" = ["alice@acme.com", "bob@acme.com"]
      "group:ops"        = ["ops@acme.com"]
    }

    tagOwners = {
      "tag:server"     = ["group:ops"]
      "tag:k8s-node"   = ["group:ops"]
      "tag:ci-runner"  = ["autogroup:admin"]
    }

    acls = [
      {
        action = "accept"
        src    = ["group:developers"]
        dst    = ["tag:server:22"]
      },
      {
        action = "accept"
        src    = ["tag:k8s-node"]
        dst    = ["tag:k8s-node:*"]
      }
    ]
  })
}
```

## Managing DNS

```hcl
# dns.tf
resource "tailscale_dns_preferences" "main" {
  magic_dns = true
}

resource "tailscale_dns_nameservers" "main" {
  nameservers = ["8.8.8.8", "8.8.4.4"]
}

# Split DNS: resolve internal domain via internal DNS server
resource "tailscale_dns_split_nameservers" "internal" {
  domain      = "internal.acme.com"
  nameservers = ["10.0.0.53"]
}
```

## Generating Auth Keys

```hcl
# keys.tf
resource "tailscale_tailnet_key" "ci_runner" {
  reusable      = true
  ephemeral     = true
  preauthorized = true
  expiry        = 7776000  # 90 days in seconds

  tags = ["tag:ci-runner"]

  description = "GitHub Actions CI runner key"
}

output "ci_runner_auth_key" {
  value     = tailscale_tailnet_key.ci_runner.key
  sensitive = true
}
```

## Device Tagging

```hcl
# Fetch an existing device and apply tags
data "tailscale_device" "k8s_node_01" {
  name     = "k8s-node-01"
  wait_for = "60s"
}

resource "tailscale_device_tags" "k8s_node_01" {
  device_id = data.tailscale_device.k8s_node_01.id
  tags      = ["tag:k8s-node", "tag:prod"]
}
```

## Subnet Route Approval

```hcl
resource "tailscale_device_subnet_routes" "subnet_router" {
  device_id = data.tailscale_device.subnet_router.id
  routes    = ["10.0.0.0/16", "192.168.1.0/24"]
}
```

## Integration with Azure Terraform

A typical pattern: provision AKS with Terraform and simultaneously configure Tailscale access to the new cluster:

```hcl
# After creating AKS cluster, generate a Tailscale key for the Operator
resource "tailscale_tailnet_key" "aks_operator" {
  reusable      = true
  ephemeral     = false
  preauthorized = true
  tags          = ["tag:k8s-node"]
  description   = "AKS ${azurerm_kubernetes_cluster.main.name} Operator"
}

# Pass the key to the Helm release
resource "helm_release" "tailscale_operator" {
  name       = "tailscale-operator"
  repository = "https://pkgs.tailscale.com/helmcharts"
  chart      = "tailscale-operator"
  namespace  = "tailscale"

  set_sensitive {
    name  = "oauth.clientSecret"
    value = tailscale_tailnet_key.aks_operator.key
  }

  depends_on = [azurerm_kubernetes_cluster.main]
}
```

## State and Secrets Management

- Store Terraform state in Azure Blob Storage (backend `azurerm`)
- Store `tailscale_api_key` in Azure Key Vault, retrieve via `azurerm_key_vault_secret`
- Never commit API keys to Git — use environment variables or secret managers

```hcl
# Retrieve API key from Azure Key Vault
data "azurerm_key_vault_secret" "tailscale_key" {
  name         = "tailscale-api-key"
  key_vault_id = azurerm_key_vault.main.id
}

provider "tailscale" {
  api_key = data.azurerm_key_vault_secret.tailscale_key.value
}
```
