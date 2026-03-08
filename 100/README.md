# 100 — Foundations & Core Concepts

> *"Before you connect your first device, understand what you are actually building."*

This module answers the question: **how does Tailscale work?** We explore the architecture, the WireGuard protocol underneath it, and the key concepts that every subsequent module builds upon.

## Contents

| File | Description |
|------|-------------|
| [100-tailscale-architecture.md](README_files/100-tailscale-architecture.md) | The three-plane model: control plane, data plane, and the tailnet |
| [110-wireguard-primer.md](README_files/110-wireguard-primer.md) | WireGuard fundamentals — the cryptographic backbone of Tailscale |
| [120-tailnet-concepts.md](README_files/120-tailnet-concepts.md) | Nodes, tailnets, IP addressing (CGNAT 100.x.y.z), and identity |
| [130-coordination-server-and-derp.md](README_files/130-coordination-server-and-derp.md) | What the coordination server does (and doesn't do), and DERP relay fallback |

## Learning Objectives

After completing this module you will be able to:

- Explain the difference between Tailscale's control plane and data plane
- Describe how WireGuard creates encrypted peer-to-peer tunnels
- Understand why Tailscale IPs are always in the `100.64.0.0/10` range
- Explain how NAT traversal works using STUN and DERP
- Compare Tailscale's mesh topology to traditional hub-and-spoke VPNs

## The Three-Plane Mental Model

```
┌─────────────────────────────────────────────────────────┐
│  CONTROL PLANE (Tailscale coordination server)          │
│  • Stores public keys for each node                     │
│  • Distributes peer lists and policies (ACLs)           │
│  • Carries NO data traffic                              │
└─────────────────────┬───────────────────────────────────┘
                      │ key exchange only
         ┌────────────▼────────────┐
         │    NODE A               │          NODE B
         │  100.64.0.1             │◄────────►100.64.0.2
         │  (your laptop)          │  Direct WireGuard tunnel
         └─────────────────────────┘  (or via DERP relay)
```

The coordination server is like a **phone book** — it helps devices find each other, but once they have each other's numbers, they call directly.
