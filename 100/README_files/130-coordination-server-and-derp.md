# 130 — Coordination Server and DERP Relays

## The Coordination Server

Tailscale's coordination server lives at `controlplane.tailscale.com`. It is the **control plane** of the tailnet.

### What it does

```
┌─────────────────────────────────────────────────────┐
│  Coordination Server                                │
│                                                     │
│  Stores:                                            │
│  • Node public keys                                 │
│  • Node endpoints (IP:port, last seen)              │
│  • ACL policy (JSON)                                │
│  • MagicDNS configuration                           │
│                                                     │
│  Sends to nodes:                                    │
│  • Updated peer list when nodes join/leave          │
│  • Updated ACL policy                               │
│  • DERP server map                                  │
└─────────────────────────────────────────────────────┘
```

### What it does NOT do

- It **never sees your data traffic** — traffic flows directly between nodes
- It **cannot decrypt your connections** — all encryption keys are node-local
- It is NOT a VPN gateway — removing it from the network doesn't drop your existing connections (nodes cache their last-known peer state)

### Self-Hosting the Control Plane

If you want to run your own coordination server, **Headscale** is the open-source alternative (see module 820). This is particularly relevant for air-gapped environments or organisations with strict data sovereignty requirements.

## DERP Relays

**DERP** stands for *Designated Encrypted Relay for Packets*.

### When DERP is Used

Direct connections fail when:
- Both peers are behind symmetric NAT (common in enterprise environments)
- Firewall rules block UDP entirely
- Mobile networks actively prevent peer-to-peer connections

In these cases, Tailscale falls back to routing traffic through a DERP relay. Crucially, **traffic is still end-to-end encrypted** — the DERP server sees only encrypted blobs and cannot read your data.

### DERP Server Locations

Tailscale operates DERP servers globally:

```
Region       Location
─────────────────────────────────
nyc          New York, USA
sfo          San Francisco, USA
ord          Chicago, USA
lax          Los Angeles, USA
ams          Amsterdam, Netherlands
fra          Frankfurt, Germany
lhr          London, UK
sin          Singapore
syd          Sydney, Australia
tok          Tokyo, Japan
... (and more)
```

Tailscale automatically selects the lowest-latency DERP server for each connection.

### Diagnosing Connection Type

```bash
# Check how you are connected to a peer
tailscale ping <peer-name>

# Output examples:
# "pong from my-server (100.64.0.2): via DERP(lhr) in 23ms"  ← relay
# "pong from my-server (100.64.0.2): via 1.2.3.4:41641 in 2ms" ← direct
```

Always aim for **direct connections** in production — DERP adds latency and depends on Tailscale's infrastructure.

### Custom DERP Servers

Enterprise plans allow you to run your own DERP servers, useful for:
- Compliance requirements (all traffic stays within your network)
- Regions not covered by Tailscale's public DERP infrastructure
- Consistently low-latency relay for specific geographic clusters

```json
// acl.json — custom DERP configuration
{
  "derpMap": {
    "Regions": {
      "900": {
        "RegionID": 900,
        "RegionCode": "custom",
        "Nodes": [{
          "Name": "1",
          "RegionID": 900,
          "HostName": "derp.my-company.internal",
          "DERPPort": 443
        }]
      }
    }
  }
}
```

## Key Takeaway

The coordination server is a **key exchange and policy distribution** service — it is not in your data path. DERP relays are a **fallback mechanism** — traffic through them is still encrypted. Knowing when you are using direct vs relay connections is critical for diagnosing performance issues.
