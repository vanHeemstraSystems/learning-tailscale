# 300 — Access Control & Security

> *"By default, all devices in your tailnet can reach all other devices. That is fine for a homelab. For anything else, you want ACLs."*

## Contents

| File | Description |
|------|-------------|
| [300-acl-policy-language.md](README_files/300-acl-policy-language.md) | HuJSON policy syntax, groups, tags, and rules |
| [310-identity-providers-sso.md](README_files/310-identity-providers-sso.md) | Connecting Google, Azure AD, GitHub, Okta to your tailnet |
| [320-device-authorization.md](README_files/320-device-authorization.md) | Device approval workflows, key expiry, and posture checks |
| [330-tailscale-ssh.md](README_files/330-tailscale-ssh.md) | Passwordless SSH using Tailscale identity, session recording |

## Learning Objectives

- Write ACL policies using HuJSON syntax
- Implement least-privilege access between nodes
- Configure tag-based machine identity for service-to-service access
- Set up Tailscale SSH as a replacement for key-based SSH
- Understand device approval and key expiry flows
