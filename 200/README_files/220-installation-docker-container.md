# 220 — Installation: Docker & Containers

## Tailscale as a Docker Sidecar

The most common pattern is running Tailscale as a **sidecar container** that provides network connectivity to your application container.

### docker-compose Example

```yaml
version: "3.9"

services:
  tailscale:
    image: tailscale/tailscale:latest
    container_name: ts-sidecar
    hostname: my-app-node          # becomes MagicDNS hostname
    environment:
      - TS_AUTHKEY=${TS_AUTHKEY}   # reusable or ephemeral auth key
      - TS_STATE_DIR=/var/lib/tailscale
      - TS_USERSPACE=false         # use kernel WireGuard if available
      - TS_EXTRA_ARGS=--advertise-tags=tag:container
    volumes:
      - ts-state:/var/lib/tailscale
      - /dev/net/tun:/dev/net/tun  # required for kernel WireGuard
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    restart: unless-stopped

  my-app:
    image: my-app:latest
    network_mode: service:tailscale   # share Tailscale's network namespace
    depends_on:
      - tailscale

volumes:
  ts-state:
```

### Key Environment Variables

| Variable | Description |
|----------|-------------|
| `TS_AUTHKEY` | Auth key for automatic authentication |
| `TS_STATE_DIR` | Where Tailscale stores its state |
| `TS_USERSPACE` | `true` = userspace WireGuard (no privileges needed), `false` = kernel |
| `TS_EXTRA_ARGS` | Additional `tailscale up` arguments |
| `TS_HOSTNAME` | Override the node hostname |
| `TS_ACCEPT_DNS` | `true` to use MagicDNS inside the container |

## Ephemeral Containers (CI/CD)

For CI runners that should be automatically cleaned up:

```bash
# Create an ephemeral auth key in the Tailscale admin panel
# Settings → Keys → Generate auth key → check "Ephemeral"

docker run -d \
  --name ts-ci-runner \
  -e TS_AUTHKEY=tskey-auth-xxxxx \
  -e TS_EXTRA_ARGS="--ephemeral" \
  tailscale/tailscale:latest
```

The node disappears from the tailnet admin panel as soon as the container stops.

## Userspace Mode (No Privileges)

If you cannot grant `NET_ADMIN` capabilities (e.g., in restricted Kubernetes environments):

```yaml
environment:
  - TS_USERSPACE=true    # Uses Go userspace WireGuard implementation
  - TS_TUN=userspace-networking
```

Userspace mode works everywhere but has slightly lower throughput than kernel mode.

## Networking Patterns

### Pattern 1: Sidecar (application shares Tailscale's network)
```
Container pod:
  [tailscale container] ←──── network namespace ────► [app container]
                                                            ↑
                                              Accessible via Tailscale IP
```

### Pattern 2: Reverse proxy (Tailscale exposes a local service)
```
Internet device ──► Tailscale IP:8080 ──► tailscale container ──► app:8080
```

### Pattern 3: Subnet router (container exposes a LAN)
```yaml
environment:
  - TS_ROUTES=192.168.1.0/24   # advertise local subnet into tailnet
```

## Verify Container is Connected

```bash
docker exec ts-sidecar tailscale status
docker exec ts-sidecar tailscale ip -4
docker exec ts-sidecar tailscale ping <other-node>
```
