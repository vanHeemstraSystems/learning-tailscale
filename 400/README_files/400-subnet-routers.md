# 400 — Subnet Routers

## What is a Subnet Router?

A **subnet router** is a Tailscale node that advertises one or more IP subnets into your tailnet. Devices on those subnets don't need to have Tailscale installed — they become reachable through the router node.

## Use Cases

- Access legacy systems (printers, NAS, PLCs, IoT devices) that can't run Tailscale
- Expose an entire Azure VNet or AWS VPC to your tailnet
- Connect to on-premises networks from remote locations
- Bridge multiple cloud environments together

## Setup

### Step 1: Enable IP Forwarding on the Router Node

```bash
# Linux — enable kernel IP forwarding
echo 'net.ipv4.ip_forward = 1' | sudo tee -a /etc/sysctl.d/tailscale.conf
echo 'net.ipv6.conf.all.forwarding = 1' | sudo tee -a /etc/sysctl.d/tailscale.conf
sudo sysctl -p /etc/sysctl.d/tailscale.conf
```

### Step 2: Advertise the Subnet

```bash
# Advertise a single subnet
sudo tailscale up --advertise-routes=192.168.1.0/24

# Advertise multiple subnets
sudo tailscale up --advertise-routes=192.168.1.0/24,10.0.0.0/16

# Combine with other flags
sudo tailscale up \
  --advertise-routes=192.168.1.0/24 \
  --advertise-tags=tag:subnet-router
```

### Step 3: Approve in Admin Panel

Advertised routes require admin approval before they are distributed to peers:

```
Tailscale Admin → Machines → <your router node> → Edit route settings → Enable routes
```

Or via API:
```bash
curl -X POST "https://api.tailscale.com/api/v2/device/<device-id>/routes" \
  -H "Authorization: Bearer $TS_API_KEY" \
  -d '{"routes": ["192.168.1.0/24"]}'
```

### Step 4: Accept Routes on Client Devices

```bash
# Accept advertised routes on a Linux client
sudo tailscale up --accept-routes

# Verify routes are visible
ip route | grep 192.168.1
```

## Network Diagram

```
Physical LAN (192.168.1.0/24)
  ├── 192.168.1.1  Router/Gateway
  ├── 192.168.1.10 NAS (no Tailscale)
  ├── 192.168.1.20 Printer (no Tailscale)
  └── 192.168.1.50 Subnet Router Node ← has Tailscale installed
                         │
                   Tailscale tailnet
                         │
                ┌────────┴────────┐
                │                 │
         My Laptop           Remote Server
       (any location)      (any cloud/location)
         ↓                       ↓
  ssh 192.168.1.10         ping 192.168.1.20
  (reaches NAS via          (reaches printer via
   subnet router)            subnet router)
```

## High Availability Subnet Routers

For production, run two subnet routers advertising the same routes. Tailscale automatically fails over if one goes down:

```bash
# Router node 1
sudo tailscale up --advertise-routes=10.0.0.0/16

# Router node 2 (same subnet)
sudo tailscale up --advertise-routes=10.0.0.0/16
```

Both nodes appear in the admin panel. Failover is automatic and typically completes within seconds.

## Troubleshooting

```bash
# Check what routes are being advertised
tailscale status --peers | grep -A2 <router-hostname>

# Verify routing on the subnet router
sudo tailscale routes show

# Test connectivity through the subnet router
tailscale ping 192.168.1.10   # if this device is behind the router
```
