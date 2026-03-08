# 211 — Tailscale on iOS / iPadOS (iPad)

## Requirements

- iPad running **iPadOS 15.0 or later** (iPadOS 17+ recommended)
- Tailscale app from the App Store (free)
- The same identity provider account (Apple ID, Google, etc.) used on the Mac Mini

## Installation

```
App Store → search "Tailscale" → Install (by Tailscale Inc.)
```

Or scan the QR code at [tailscale.com/download](https://tailscale.com/download).

## First-Time Setup

1. Open Tailscale on your iPad
2. Tap **Get Started**
3. Accept the prompt to install a VPN configuration
   - iPadOS will ask you to confirm: *"Tailscale" Would Like to Add VPN Configurations* → tap **Allow**
   - This is normal — it is how Tailscale creates the WireGuard tunnel interface on iOS
4. Tap **Log in** and authenticate with the **same identity provider** you used on your Mac Mini
5. Your iPad is now part of the tailnet

> **Key point**: Both devices must authenticate with the **same account**. If you used Google on the Mac Mini, use Google on the iPad.

## Verifying the Connection

After login, the Tailscale app shows all devices in your tailnet:

```
Tailscale app → My Devices
  mac-mini     100.64.1.10   ● Active
  my-ipad      100.64.1.20   ● This device
```

A green dot next to `mac-mini` means it is reachable.

## The VPN Toggle on iPadOS 18+

On iPads running iOS/iPadOS 18, the VPN can be toggled from Control Center. Long-press an empty space in Control Center to add the Tailscale toggle.

This means you can connect/disconnect Tailscale without opening the app — just swipe down for Control Center and tap the Tailscale tile.

## Keeping Tailscale Running on iPad

iPadOS aggressively manages background processes to preserve battery. A few things to know:

- Tailscale's VPN extension is a **system-level process** — it persists even when the app is in the background or the screen is locked
- The VPN profile you installed ensures iOS keeps the Tailscale tunnel alive
- If you turn off the VPN in iOS Settings → VPN, Tailscale disconnects — toggle it back on in the Tailscale app or Settings

### Preventing auto-disconnect

Go to **Settings → General → VPN & Device Management → VPN** and verify the Tailscale VPN profile is set to **Connect On Demand** if you want it to reconnect automatically when network changes.

## Enabling MagicDNS on iPad

In the Tailscale app:
```
Tailscale → Settings → Use Tailscale DNS → ✅ On
```

With this enabled, you can reach your Mac Mini by hostname instead of IP:
- `mac-mini` ← short name
- `mac-mini.your-tailnet.ts.net` ← fully qualified

## Finding Your Mac Mini's IP from the iPad

```
Tailscale app → My Devices → mac-mini
```

Tap the device name to see its Tailscale IP (e.g., `100.64.1.10`). Tap the IP to copy it to your clipboard — you'll paste this into your VNC client.

## Key Settings to Check

| Setting | Recommended | Why |
|---------|-------------|-----|
| Use Tailscale DNS | ✅ On | Enables MagicDNS hostnames |
| Accept routes | ✅ On | Allows reaching subnet-routed devices |
| VPN profile active | ✅ Enabled | Required for the tunnel |

## Authentication Key Expiry

By default, Tailscale sessions expire after **90 days** (personal plan). When the key expires on your iPad:
- Push notification will alert you in advance
- Open Tailscale → tap **Re-authenticate**
- Complete SSO login in the browser sheet that appears

You can configure key expiry (or disable it for specific devices) in the Tailscale admin console.
