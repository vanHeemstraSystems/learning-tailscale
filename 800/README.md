# 800 — Automation & GitOps

> *"If you are clicking in the admin panel to manage your tailnet, you are not at scale yet."*

## Contents

| File | Description |
|------|-------------|
| [800-tailscale-api.md](README_files/800-tailscale-api.md) | REST API — managing devices, keys, ACLs programmatically |
| [810-terraform-provider.md](README_files/810-terraform-provider.md) | Infrastructure-as-Code with the Tailscale Terraform provider |
| [820-headscale-self-hosted.md](README_files/820-headscale-self-hosted.md) | Running your own coordination server with Headscale |
| [830-gitops-acl-management.md](README_files/830-gitops-acl-management.md) | Managing ACL policies via Git, PR reviews, and CI pipelines |

## Learning Objectives

- Query and manage tailnet state via the Tailscale REST API
- Define tailnet resources (ACLs, DNS, keys) with Terraform
- Deploy and operate Headscale for self-hosted control plane
- Implement a GitOps workflow for ACL policy management
