# 920 — Glossary

| Term | Definition |
|------|-----------|
| **ACL** | Access Control List — HuJSON policy file defining which tailnet members can connect to which resources |
| **Auth Key** | A pre-shared secret used to add devices to a tailnet without interactive browser authentication |
| **CGNAT** | Carrier-Grade NAT — the `100.64.0.0/10` IP range (RFC 6598) used for Tailscale node addresses |
| **Coordination Server** | Tailscale's cloud control plane that distributes public keys and policies; handles no user data traffic |
| **DERP** | Designated Encrypted Relay for Packets — Tailscale's relay protocol for when direct connections are blocked |
| **Ephemeral Node** | A tailnet node that is automatically removed when it goes offline (common for CI runners and containers) |
| **Exit Node** | A tailnet device that routes a peer's internet traffic, acting like a traditional VPN gateway |
| **Funnel** | A Tailscale feature that exposes a tailnet service to the public internet via a managed HTTPS endpoint |
| **Headscale** | Open-source, self-hosted implementation of the Tailscale coordination server |
| **HuJSON** | Human JSON — JSON with comments (`//` and `/* */`) and trailing commas, used for Tailscale ACL files |
| **Magic DNS** | Automatic DNS resolution where every tailnet node gets a hostname like `machine.tailnet.ts.net` |
| **MagicDNS** | See Magic DNS |
| **NAT Traversal** | Techniques (STUN, hole-punching) used to establish direct connections between devices behind NAT |
| **Node** | Any device running the Tailscale client software and authenticated into a tailnet |
| **Node Key** | The WireGuard public key of a tailnet node; used by peers to encrypt traffic destined for that node |
| **Operator** | The Tailscale Kubernetes Operator — a controller that manages Tailscale nodes as Kubernetes resources |
| **ProxyGroup** | A Kubernetes CRD that creates a highly-available set of Tailscale ingress proxies |
| **Serve** | A Tailscale feature that exposes a local service to other tailnet members via HTTPS |
| **Split DNS** | Configuration where certain DNS domains are resolved by a private resolver, others by a public resolver |
| **STUN** | Session Traversal Utilities for NAT — used to discover a device's public IP and port for NAT traversal |
| **Subnet Router** | A tailnet node that advertises a physical IP subnet, making non-Tailscale devices reachable via the tailnet |
| **Tag** | A label applied to a device that provides machine-level identity for ACL policies, independent of the user |
| **Taildrop** | Tailscale's built-in file transfer feature between tailnet devices |
| **Tailnet** | Your private Tailscale network — all devices authenticated to the same account/organisation |
| **Tailscale SSH** | Using Tailscale identity for SSH authentication, replacing traditional key-based SSH |
| **WireGuard** | The open-source VPN protocol (built into Linux kernel since 5.6) that powers all Tailscale data-plane connections |
| **`100.x.y.z`** | Shorthand for a Tailscale node's IP address in the CGNAT range |
| **`tailscaled`** | The Tailscale daemon process that manages the WireGuard interface and handles routing |
