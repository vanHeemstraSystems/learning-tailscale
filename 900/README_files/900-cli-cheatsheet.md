# 900 — Tailscale CLI Cheatsheet

## Authentication

```bash
tailscale up                                  # authenticate (opens browser)
tailscale up --authkey=tskey-auth-xxxx        # headless auth
tailscale up --authkey=tskey-auth-xxxx --hostname=my-node
tailscale logout                              # disconnect and de-authenticate
tailscale down                                # disconnect (keep authenticated)
```

## Status & Diagnostics

```bash
tailscale status                              # show all peers
tailscale status --peers                      # show only peer devices
tailscale status --active                     # show only active peers
tailscale status --json                       # JSON output
tailscale ip -4                               # your Tailscale IPv4
tailscale ip -6                               # your Tailscale IPv6
tailscale ip <hostname>                       # IP of a specific peer
tailscale ping <hostname-or-ip>               # test connectivity to peer
tailscale ping --verbose <hostname>           # verbose connection details
tailscale netcheck                            # network capability check
tailscale whois <ip>                          # identify owner of a Tailscale IP
tailscale bugreport                           # generate debug bundle
```

## Networking

```bash
# Subnet routing
tailscale up --advertise-routes=192.168.1.0/24
tailscale up --advertise-routes=10.0.0.0/8,172.16.0.0/12
tailscale up --accept-routes                  # accept peers' advertised routes

# Exit nodes
tailscale up --advertise-exit-node            # become an exit node
tailscale up --exit-node=<hostname>           # use a specific exit node
tailscale up --exit-node=<hostname> --exit-node-allow-lan-access
tailscale up --exit-node=""                   # stop using exit node

# Tags
tailscale up --advertise-tags=tag:server,tag:prod

# Accept DNS from tailnet
tailscale up --accept-dns=true
tailscale up --accept-dns=false               # disable MagicDNS
```

## SSH

```bash
tailscale ssh <hostname>                      # SSH via Tailscale (no keys needed)
tailscale ssh user@<hostname>                 # SSH as specific user
```

## File Transfer (Taildrop)

```bash
tailscale file cp ./myfile.txt <hostname>:    # send a file
tailscale file get ~/Downloads/               # receive pending files
```

## Serve & Funnel

```bash
# Expose a local service on the tailnet
tailscale serve 3000                          # expose localhost:3000
tailscale serve https / http://localhost:3000
tailscale serve status                        # show current serve config
tailscale serve reset                         # clear all serve configs

# Expose a service to the public internet via Funnel
tailscale funnel 3000                         # public HTTPS endpoint
tailscale funnel status
tailscale funnel reset
```

## Certificates

```bash
tailscale cert <hostname>.tailnet.ts.net      # get a TLS cert for a node
```

## Lock (Network Lock)

```bash
tailscale lock init <key1> <key2> ...         # initialise network lock
tailscale lock status                          # show lock status
tailscale lock add <key>                       # add a trusted signing key
tailscale lock remove <key>                    # remove a signing key
tailscale lock sign <node-key>                 # sign a new node key
```

## Administration

```bash
tailscale set --auto-update                   # enable auto-updates
tailscale update                              # update Tailscale to latest
tailscale version                             # show installed version
```

## Common Flag Combinations

```bash
# Server with subnet routing and tagging
sudo tailscale up \
  --advertise-routes=10.0.0.0/16 \
  --advertise-tags=tag:server \
  --accept-routes \
  --ssh

# CI runner (ephemeral, tagged)
tailscale up \
  --authkey=$TS_AUTHKEY \
  --hostname=ci-runner-$(hostname) \
  --advertise-tags=tag:ci-runner

# Kubernetes node
tailscale up \
  --authkey=$TS_AUTHKEY \
  --advertise-tags=tag:k8s-node \
  --accept-routes \
  --accept-dns
```
