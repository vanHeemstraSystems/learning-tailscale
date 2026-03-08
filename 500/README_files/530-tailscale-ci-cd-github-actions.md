# 530 — Tailscale in GitHub Actions

## Overview

GitHub Actions runners have no network access to your private infrastructure by default. Tailscale solves this by joining each runner to your tailnet as an ephemeral node for the duration of the workflow.

## The `tailscale/github-action` Action

Tailscale provides an official GitHub Action that:
1. Installs Tailscale on the runner
2. Authenticates using an ephemeral auth key
3. Connects the runner to your tailnet
4. On workflow completion, the ephemeral node is automatically removed

## Basic Usage

```yaml
# .github/workflows/deploy.yml
name: Deploy to Private AKS

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # Connect runner to tailnet
      - name: Connect to Tailscale
        uses: tailscale/github-action@v2
        with:
          authkey: ${{ secrets.TS_AUTHKEY }}
          tags: tag:ci-runner
          # Optional: hostname for this runner
          hostname: github-runner-${{ github.run_id }}

      # Now kubectl works against your private AKS cluster
      - name: Get AKS credentials
        run: |
          az login --service-principal \
            -u ${{ secrets.AZURE_CLIENT_ID }} \
            -p ${{ secrets.AZURE_CLIENT_SECRET }} \
            --tenant ${{ secrets.AZURE_TENANT_ID }}
          az aks get-credentials \
            --resource-group myRG \
            --name myAKS

      - name: Deploy to AKS
        run: |
          kubectl apply -f k8s/
          kubectl rollout status deployment/my-app
```

## Secrets Setup

In your GitHub repository:

```
Settings → Secrets and variables → Actions → New repository secret

TS_AUTHKEY   = tskey-auth-xxxxxx (ephemeral, reusable key from Tailscale admin)
```

Create the auth key:
1. Tailscale Admin → **Settings → Keys**
2. Click **Generate auth key**
3. Enable **Reusable** and **Ephemeral**
4. Add tag: `tag:ci-runner`
5. Copy the key

## ACL Policy for CI Runners

```json
{
  "tagOwners": {
    "tag:ci-runner": ["autogroup:admin"]
  },
  "acls": [
    // CI runners can access the Kubernetes API server
    {
      "action": "accept",
      "src":    ["tag:ci-runner"],
      "dst":    ["tag:k8s-node:6443"]
    },
    // CI runners can push to internal registries
    {
      "action": "accept",
      "src":    ["tag:ci-runner"],
      "dst":    ["tag:registry:5000"]
    }
  ]
}
```

## Advanced: Connecting to Multiple Environments

```yaml
jobs:
  deploy-staging:
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - uses: tailscale/github-action@v2
        with:
          authkey: ${{ secrets.TS_AUTHKEY_STAGING }}
          tags: tag:ci-runner,tag:staging-deployer

  deploy-prod:
    runs-on: ubuntu-latest
    environment: production
    needs: [deploy-staging]
    steps:
      - uses: tailscale/github-action@v2
        with:
          authkey: ${{ secrets.TS_AUTHKEY_PROD }}
          tags: tag:ci-runner,tag:prod-deployer
```

Different tags enable different ACL policies per environment.

## Self-Hosted Runners

For self-hosted runners that are permanently on your tailnet, you don't need the GitHub Action — they already have network access. But for **GitHub-hosted runners** (the default `ubuntu-latest`), the Tailscale GitHub Action is the recommended approach.

## Troubleshooting

```yaml
- name: Debug Tailscale connection
  run: |
    tailscale status
    tailscale ping my-aks-node
    tailscale netcheck
```
