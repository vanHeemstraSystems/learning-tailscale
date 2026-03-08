# 100 — Tailscale Architecture

## Overview

Tailscale creates a **mesh overlay network** on top of your existing internet connections. Each device in the network (called a *node*) connects directly to every other node it needs to reach, rather than funnelling traffic through a central server.

## The Three Layers

### 1. Identity Layer

Every device authenticates using an external identity provider (Google, Microsoft, GitHub, Okta, etc.). Tailscale never stores passwords — it delegates entirely to your SSO provider.

Each authenticated device receives:
- A stable **Tailscale IP** in the `100.64.0.0/10` CGNAT range
- A **public/private WireGuard keypair** generated locally (private key never leaves the device)
- A **MagicDNS hostname** (e.g., `my-laptop.tailnet-name.ts.net`)

### 2. Control Plane (Coordination Server)

Hosted by Tailscale at `controlplane.tailscale.com`. Responsibilities:

- Receives and stores each node's **public key** and current IP/port
- Distributes **peer lists** so nodes know about each other
- Distributes **ACL policies** (who can connect to what)
- Issues **DERP relay server assignments** when direct connections are blocked

**What it does NOT do:**
- It never sees your actual data traffic
- It cannot decrypt your connections
- It does not act as a VPN gateway

### 3. Data Plane (WireGuard mesh)

Actual traffic flows directly between nodes using WireGuard tunnels:

```
Node A ──────────────────────────► Node B
       WireGuard encrypted tunnel
       (direct UDP, or via DERP)
```

## NAT Traversal

Most devices sit behind NAT (home routers, cloud NAT gateways). Tailscale uses multiple techniques to punch through:

1. **Direct UDP**: Attempts a direct connection first
2. **STUN** (Session Traversal Utilities for NAT): Discovers the public IP/port of each node
3. **UPnP / NAT-PMP / PCP**: Requests port forwarding from the router if supported
4. **DERP Relay**: Fallback when all direct methods fail — traffic is still end-to-end encrypted

## IP Addressing

All Tailscale nodes receive an IP in `100.64.0.0/10` (Carrier-Grade NAT space, RFC 6598). This range was chosen specifically to avoid conflicts with common private networks (`10.x`, `172.16.x`, `192.168.x`).

Example tailnet:
```
my-laptop      → 100.64.0.1
dev-server     → 100.64.0.2
k8s-node-01    → 100.64.0.3
home-nas       → 100.64.0.4
```

## Comparison: Traditional VPN vs Tailscale

| Aspect | Traditional VPN | Tailscale |
|--------|----------------|-----------|
| Topology | Hub-and-spoke | Full mesh |
| Data path | All traffic via gateway | Direct peer-to-peer |
| Latency | Gateway adds latency | Minimal — shortest path |
| Single point of failure | Yes (VPN gateway) | No |
| Setup time | Hours to days | Minutes |
| Firewall rules | Complex | None required |
| Scales to | Hundreds of users (with effort) | Thousands of devices |

## Key Takeaway

The genius of Tailscale is separating the **control plane** (who can connect to what) from the **data plane** (the actual traffic). The control plane is centralised for management convenience; the data plane is fully distributed for performance and resilience.
