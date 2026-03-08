# 700 — Tailscale Status and Netcheck

## Essential Diagnostic Commands

### `tailscale status`

```bash
# Basic status
tailscale status

# Example output:
# 100.64.0.1   my-laptop        alice@   linux   -
# 100.64.0.2   dev-server       alice@   linux   active; direct 1.2.3.4:41641, tx 1234 rx 5678
# 100.64.0.3   k8s-node-01      tagged   linux   active; relay "ams", tx 100 rx 200
# 100.64.0.4   home-nas         alice@   linux   idle

# JSON output for scripting
tailscale status --json | jq '.Peer | to_entries[] | {name: .value.HostName, ip: .value.TailscaleIPs[0], relay: .value.Relay}'

# Show only active peers
tailscale status --active
```

Connection states:
- `direct` — optimal: peer-to-peer WireGuard
- `relay "ams"` — via DERP relay in Amsterdam — investigate why direct fails
- `idle` — node is online but no recent traffic
- `-` — yourself

### `tailscale ping`

```bash
# Ping a peer by hostname or IP
tailscale ping dev-server
tailscale ping 100.64.0.2

# Example output (direct):
# pong from dev-server (100.64.0.2): via 1.2.3.4:41641 in 2ms

# Example output (relay):
# pong from dev-server (100.64.0.2): via DERP(ams) in 23ms

# Verbose: show all attempts
tailscale ping --verbose dev-server

# Ping a non-Tailscale device via subnet router
tailscale ping 192.168.1.10
```

### `tailscale netcheck`

Runs a comprehensive network diagnostic:

```bash
tailscale netcheck

# Example output:
# Report:
#   * UDP: true
#   * IPv4: yes, 1.2.3.4:4500
#   * IPv6: no
#   * MappingVariesByDestIP: false
#   * HairPinning: true
#   * PortMapping: UPnP
#   * DERP latency:
#     - ams: 15ms  ← Amsterdam (closest)
#     - fra: 20ms
#     - lhr: 25ms
#     - nyc: 85ms
```

Key fields to check:
- **UDP: false** → all connections will use DERP; check firewall UDP rules
- **MappingVariesByDestIP: true** → symmetric NAT; direct connections may fail
- **DERP latency** → identify which relay region is fastest for fallback

### `tailscale ip`

```bash
# Get your Tailscale IP
tailscale ip -4         # IPv4
tailscale ip -6         # IPv6

# Get IP of a peer
tailscale ip dev-server
```

### `tailscale whois`

```bash
# Identify who owns a Tailscale IP
tailscale whois 100.64.0.2

# Output:
# Machine: dev-server
# User:    alice@acme.com
# Tags:    tag:server
```

Useful in ACL debugging — tells you the identity behind any Tailscale IP.

### `tailscale bugreport`

```bash
# Generate a support bundle
tailscale bugreport

# Output: BUG-xxxxxxxxxxxxxx
# Share this with Tailscale support or use for self-debugging
```

### Daemon Logs

```bash
# Linux systemd
journalctl -u tailscaled -f
journalctl -u tailscaled --since "10 minutes ago"

# macOS
tail -f /var/log/tailscale/tailscaled.log

# Docker
docker logs ts-sidecar -f
```

## Interpreting `tailscale status --json`

```bash
tailscale status --json | jq '{
  self_ip: .Self.TailscaleIPs[0],
  peers: [.Peer | to_entries[] | {
    name: .value.HostName,
    ip: .value.TailscaleIPs[0],
    active: .value.Active,
    relay: .value.Relay,
    direct: (.value.Relay == "")
  }]
}'
```

## Quick Health Checklist

```bash
# 1. Is the daemon running?
systemctl is-active tailscaled

# 2. Are you authenticated?
tailscale status | head -1

# 3. Can you reach peers?
tailscale ping <peer>

# 4. Are connections direct or relayed?
tailscale status | grep relay

# 5. Is UDP blocked?
tailscale netcheck | grep "UDP:"

# 6. What does the peer see?
# (run on the peer)
tailscale ping <your-machine>
```
