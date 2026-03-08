# 110 — WireGuard Primer

## What is WireGuard?

WireGuard is a modern, open-source VPN protocol designed for simplicity, speed, and strong cryptography. It was merged into the Linux kernel in 2020 and is the cryptographic foundation of Tailscale.

Compared to older protocols:

| Protocol | Lines of Code | Audit Surface | Performance |
|----------|--------------|---------------|-------------|
| OpenVPN  | ~70,000      | Large         | Moderate |
| IPsec    | ~400,000     | Very large    | Good |
| **WireGuard** | **~4,000** | **Tiny** | **Excellent** |

Fewer lines of code = smaller attack surface = easier to audit.

## Cryptographic Primitives

WireGuard uses a carefully selected modern cryptographic suite:

- **Curve25519** — Elliptic curve Diffie-Hellman key exchange
- **ChaCha20** — Stream cipher for data encryption
- **Poly1305** — Message authentication (MAC)
- **BLAKE2s** — Hashing
- **SipHash24** — Hash table keying
- **HKDF** — Key derivation

## How a WireGuard Tunnel Works

```
1. Each peer generates a keypair:
   Private key: stays on device, never shared
   Public key:  shared with peers

2. Peer A configures Peer B's public key + allowed IPs
   Peer B configures Peer A's public key + allowed IPs

3. When Peer A sends a packet to an IP in Peer B's AllowedIPs:
   - Packet is encrypted with Peer B's public key
   - Sent as a UDP datagram to Peer B's endpoint

4. Peer B receives the datagram:
   - Decrypts using its private key
   - Validates with Peer A's public key
   - If valid, delivers the inner packet
```

## WireGuard in Tailscale

Tailscale uses the **userspace Go implementation** of WireGuard (`wireguard-go`) rather than the kernel module, which allows it to run on platforms without kernel WireGuard support (macOS, iOS, Windows, Android).

Tailscale automates the painful manual parts of WireGuard:
- **Key distribution** — handled by the coordination server
- **Peer discovery** — handled automatically
- **Firewall configuration** — not needed
- **IP address management** — assigned automatically

Raw WireGuard requires you to manually configure every peer relationship. With 10 devices, that is 45 separate peer configurations. Tailscale does this automatically for every device you add.

## Interface

When Tailscale is running, it creates a virtual network interface:

```bash
# Linux
ip addr show tailscale0
# → inet 100.64.x.y/32

# Check WireGuard stats
sudo wg show tailscale0
```

## Key Takeaway

WireGuard provides the cryptographic tunnel; Tailscale provides the orchestration, identity, and policy layer on top. You get WireGuard's performance and security with zero of the manual configuration overhead.
