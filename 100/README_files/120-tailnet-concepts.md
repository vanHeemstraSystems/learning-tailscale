# 120 — Tailnet Concepts

## What is a Tailnet?

A **tailnet** is your private Tailscale network — the collection of all devices authenticated under a single account or organisation. Think of it as your own private internet.

Every device you add to a tailnet:
- Gets a unique `100.x.y.z` IP address (stable, never changes)
- Gets a MagicDNS hostname
- Can communicate with every other device in the tailnet (subject to ACLs)

## Nodes

A **node** is any device running the Tailscale client: laptops, servers, VMs, containers, Kubernetes pods, NAS devices, Raspberry Pis, phones.

Each node has:
- **Node ID**: a stable identifier (survives IP changes)
- **Tailscale IP**: e.g., `100.64.22.101`
- **MagicDNS name**: e.g., `dev-server.my-tailnet.ts.net`
- **Public WireGuard key**: used by other nodes to encrypt traffic to this node
- **Tags** (optional): for machine-level ACL enforcement (e.g., `tag:server`, `tag:k8s-node`)

## Identity

Tailscale uses your existing identity provider for authentication:

```
Supported providers:
  ✓ Google Workspace
  ✓ Microsoft (Azure AD / Entra ID)
  ✓ GitHub
  ✓ Okta
  ✓ OneLogin
  ✓ SAML (generic)
  ✓ Email (magic link, for personal use)
```

The user's identity from their IdP becomes their Tailscale identity, which drives ACL policy.

## IP Addressing Deep Dive

All Tailscale IPs come from `100.64.0.0/10` (RFC 6598, Carrier-Grade NAT):

```
Range:  100.64.0.0  –  100.127.255.255
Size:   ~4 million addresses
Subnet: /32 per device (point-to-point)
```

Why CGNAT space?
- Unlikely to conflict with your corporate `10.0.0.0/8` ranges
- Unlikely to conflict with home `192.168.x.x` networks
- Globally recognised as "not routable on the public internet"

## MagicDNS

Every node gets an automatic DNS hostname:

```
Format:  <machine-name>.<tailnet-name>.ts.net
Example: my-laptop.acme-corp.ts.net
Short:   my-laptop  (resolvable within tailnet)
```

MagicDNS is configured automatically — no DNS server setup required. You can also use custom domain overrides.

## Ephemeral vs Persistent Nodes

| Type | Use Case | Behaviour |
|------|----------|-----------|
| **Persistent** | Laptops, servers | Stay in tailnet until manually removed |
| **Ephemeral** | CI runners, containers | Auto-removed after going offline |

Ephemeral nodes are essential for Kubernetes and CI/CD use cases — they clean themselves up.

## Tailscale Account Types

| Plan | Devices | Users | Key Features |
|------|---------|-------|-------------|
| Personal (free) | 3 users, 100 devices | Personal only | Full feature set |
| Starter | 3 users | Teams | SSO, ACLs |
| Premium | Unlimited | Enterprises | Audit logs, SCIM, MDM |
| Enterprise | Unlimited | Large orgs | Custom DERP, SLAs |

For learning purposes, the **free personal plan** is sufficient for all exercises in this repository.
