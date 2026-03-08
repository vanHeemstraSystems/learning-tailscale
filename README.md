# Learning Tailscale

> *"Your own private internet — peer-to-peer, encrypted, zero-config. Like a VPN, but one that actually works the way you think."*

## What is This Repository?

This repository is a structured learning journey through **Tailscale** — the WireGuard-based mesh VPN that turns a patchwork of devices, clouds, and networks into a single, secure, private network (called a **tailnet**) with almost no configuration overhead.

Whether you are a Cloud Engineer integrating Tailscale into a Kubernetes platform (like Atlas IDP), a developer wanting secure remote access to lab machines, or a homelab enthusiast who just wants all their devices to talk to each other — this repo walks you from zero to production-grade Tailscale knowledge.

---

## Mental Model: The Private Subway System

Traditional VPNs work like taxi cabs: every passenger (packet) must go via a central dispatcher (the VPN gateway), even if the destination is right next door. It is slow, there is a single point of failure, and the dispatcher becomes a bottleneck.

Tailscale works like your own **private underground metro system**:

- Every station (device) knows where every other station is.
- Trains (packets) travel **directly** between stations — no dispatcher needed.
- The metro map (coordination server) is maintained centrally, but carries **zero passenger traffic**.
- New stations join automatically the moment they authenticate.
- If two stations cannot connect directly (e.g., heavy firewalls), they use a **relay (DERP)** — like a transfer station — but still encrypted end-to-end.

You own the metro. Nobody else rides it. And new lines open in minutes.

---

## Repository Structure

```
learning-tailscale/
│
├── README.md                          ← You are here
│
├── 100/                               ← Foundations & Core Concepts
│   ├── README.md
│   └── README_files/
│       ├── 100-tailscale-architecture.md
│       ├── 110-wireguard-primer.md
│       ├── 120-tailnet-concepts.md
│       └── 130-coordination-server-and-derp.md
│
├── 200/                               ← Installation & First Tailnet
│   ├── README.md
│   └── README_files/
│       ├── 200-installation-linux.md
│       ├── 210-installation-macos-windows.md
│       ├── 220-installation-docker-container.md
│       └── 230-first-tailnet-walkthrough.md
│
├── 210/                               ← Apple Ecosystem: iPad → Mac Mini Remote Control
│   ├── README.md
│   └── README_files/
│       ├── 210-tailscale-on-macos.md
│       ├── 211-tailscale-on-ios-ipad.md
│       ├── 212-macos-screen-sharing-vnc-setup.md
│       ├── 213-ipad-to-mac-mini-remote-control.md
│       ├── 214-vnc-client-apps-for-ipad.md
│       └── 215-troubleshooting-apple-remote-access.md
│
├── 300/                               ← Access Control & Security
│   ├── README.md
│   └── README_files/
│       ├── 300-acl-policy-language.md
│       ├── 310-identity-providers-sso.md
│       ├── 320-device-authorization.md
│       └── 330-tailscale-ssh.md
│
├── 400/                               ← Networking Features
│   ├── README.md
│   └── README_files/
│       ├── 400-subnet-routers.md
│       ├── 410-exit-nodes.md
│       ├── 420-magic-dns.md
│       └── 430-split-tunneling-and-app-connectors.md
│
├── 500/                               ← Kubernetes & Cloud Integration
│   ├── README.md
│   └── README_files/
│       ├── 500-tailscale-on-kubernetes.md
│       ├── 510-tailscale-operator.md
│       ├── 520-tailscale-aks-azure.md
│       └── 530-tailscale-ci-cd-github-actions.md
│
├── 600/                               ← Advanced Scenarios
│   ├── README.md
│   └── README_files/
│       ├── 600-high-availability-subnet-routers.md
│       ├── 610-tailscale-funnel.md
│       ├── 620-taildrop-file-sharing.md
│       └── 630-mullvad-exit-nodes.md
│
├── 700/                               ← Observability & Troubleshooting
│   ├── README.md
│   └── README_files/
│       ├── 700-tailscale-status-and-netcheck.md
│       ├── 710-logging-and-audit-trails.md
│       ├── 720-troubleshooting-nat-traversal.md
│       └── 730-performance-tuning.md
│
├── 800/                               ← Automation & GitOps
│   ├── README.md
│   └── README_files/
│       ├── 800-tailscale-api.md
│       ├── 810-terraform-provider.md
│       ├── 820-headscale-self-hosted.md
│       └── 830-gitops-acl-management.md
│
└── 900/                               ← Reference & Cheatsheets
    ├── README.md
    └── README_files/
        ├── 900-cli-cheatsheet.md
        ├── 910-acl-examples.md
        ├── 920-glossary.md
        └── 930-further-reading.md
```

---

## Learning Path

Each numbered module builds on the previous one. Suggested progression:

| Module | Topic | Prerequisite |
|--------|-------|--------------|
| 100 | Foundations — how Tailscale works under the hood | None |
| 200 | Installation — getting your first tailnet running | 100 |
| 210 | **Apple Ecosystem — iPad → Mac Mini remote control** | 200 |
| 300 | Security — ACLs, SSO, device trust | 200 |
| 400 | Networking — subnets, exit nodes, Magic DNS | 200 |
| 500 | Kubernetes & Cloud — Tailscale Operator, AKS | 300, 400 |
| 600 | Advanced — Funnel, HA routers, Mullvad | 400 |
| 700 | Observability — debugging and monitoring | 500 |
| 800 | Automation — API, Terraform, Headscale | 300, 500 |
| 900 | Reference — cheatsheets and glossary | Any |

---

## Key Concepts at a Glance

| Concept | Description |
|---------|-------------|
| **Tailnet** | Your private mesh network — all your authenticated devices |
| **WireGuard** | The open-source VPN protocol powering all Tailscale connections |
| **Coordination Server** | Tailscale's cloud control plane — only exchanges keys, never traffic |
| **DERP Relay** | Fallback encrypted relay when direct P2P connection is blocked |
| **Magic DNS** | Automatic hostname resolution for every device in your tailnet |
| **ACL** | Access Control Lists — JSON/HuJSON policy controlling who can reach what |
| **Subnet Router** | A node that advertises a physical subnet into the tailnet |
| **Exit Node** | A node that routes all internet traffic for other tailnet members |
| **Tailscale SSH** | SSHing into nodes using Tailscale identity — no SSH keys needed |
| **Funnel** | Publicly expose a local service via a Tailscale-managed HTTPS endpoint |
| **Headscale** | Open-source self-hosted alternative to the Tailscale coordination server |

---

## Prerequisites

- Basic networking knowledge (IP addressing, NAT, DNS, firewalls)
- Linux command-line familiarity
- Docker basics (for container integration modules)
- Kubernetes fundamentals (for module 500+)
- A free Tailscale account: [https://tailscale.com](https://tailscale.com)

---

## Tools & Versions

| Tool | Version | Notes |
|------|---------|-------|
| Tailscale CLI | 1.x (latest stable) | Install via package manager or script |
| WireGuard | Bundled with Tailscale | No separate install needed |
| Tailscale Kubernetes Operator | 1.x | Helm-installable |
| Terraform Tailscale Provider | latest | `hashicorp/tailscale` |
| Headscale | 0.23+ | Self-hosted coordination server |
| kubectl | 1.29+ | For Kubernetes modules |

---

## Related Repositories

This repository is part of the `vanHeemstraSystems` learning series:

- [learning-crossplane-inner-workings](https://github.com/vanHeemstraSystems/learning-crossplane-inner-workings)
- [learning-crossplane-unit-testing](https://github.com/vanHeemstraSystems/learning-crossplane-unit-testing)
- [learning-crossplane-e2e-testing](https://github.com/vanHeemstraSystems/learning-crossplane-e2e-testing)
- [learning-envoyproxy](https://github.com/vanHeemstraSystems/learning-envoyproxy)
- [learning-audit-automation](https://github.com/vanHeemstraSystems/learning-audit-automation)

---

## References

- [Tailscale Documentation](https://tailscale.com/kb)
- [How Tailscale Works (blog)](https://tailscale.com/blog/how-tailscale-works)
- [WireGuard Protocol](https://www.wireguard.com)
- [Tailscale Kubernetes Operator](https://tailscale.com/kb/1236/kubernetes-operator)
- [Headscale Project](https://github.com/juanfont/headscale)
- [Tailscale GitHub](https://github.com/tailscale/tailscale)

---

*Maintained by [vanHeemstraSystems](https://github.com/vanHeemstraSystems)*
