# 500 — Kubernetes & Cloud Integration

> *"Running Tailscale on Kubernetes is the natural next step. The Tailscale Operator makes your cluster's services first-class citizens of your tailnet."*

## Contents

| File | Description |
|------|-------------|
| [500-tailscale-on-kubernetes.md](README_files/500-tailscale-on-kubernetes.md) | Patterns: sidecar, DaemonSet, subnet router on K8s |
| [510-tailscale-operator.md](README_files/510-tailscale-operator.md) | Installing and using the Tailscale Kubernetes Operator |
| [520-tailscale-aks-azure.md](README_files/520-tailscale-aks-azure.md) | Tailscale on AKS — connecting Azure workloads to your tailnet |
| [530-tailscale-ci-cd-github-actions.md](README_files/530-tailscale-ci-cd-github-actions.md) | GitHub Actions workflows using Tailscale for secure CI access |

## Learning Objectives

- Deploy the Tailscale Operator on a Kubernetes cluster
- Expose Kubernetes Services via Tailscale ingress
- Connect AKS workloads into your tailnet
- Use ephemeral auth keys in GitHub Actions for secure cluster access
- Understand the Proxy and ProxyGroup CRDs
