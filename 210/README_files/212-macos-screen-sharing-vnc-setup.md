# 212 — Enabling Screen Sharing (VNC Server) on Mac Mini

## Overview

macOS has a built-in VNC server called **Screen Sharing**. Enabling it turns your Mac Mini into a remote desktop host that any VNC client — including Screens 5 on your iPad — can connect to.

The VNC traffic runs over port `5900`. Because the connection travels through the Tailscale WireGuard tunnel, this port is **never exposed to the public internet** — it is only accessible from your tailnet devices.

## Step 1: Enable Screen Sharing

### macOS Ventura / Sonoma (13 / 14+)

```
System Settings → General → Sharing → Screen Sharing → toggle ON
```

### macOS Monterey (12) and earlier

```
System Preferences → Sharing → check ✅ Screen Sharing
```

You will see a notice: *"Other users can access your computer's screen at vnc://100.64.1.10/ or by looking for 'Mac Mini' in the Finder sidebar."*

Note the `vnc://` address shown — this is exactly what you will enter in your VNC client on the iPad, using the Tailscale IP.

## Step 2: Configure Who Can Connect

In the Screen Sharing settings panel:

```
Allow access for: ● All users   (simplest for personal use)
                  ○ Only these users:  [add specific accounts]
```

For a personal Mac Mini with a single user account, **All users** is fine.

## Step 3: (Optional) Enable Remote Management

**Remote Management** is a superset of Screen Sharing — it includes Apple Remote Desktop (ARD) capabilities for additional administrative tasks. If you only need to see and control the screen, plain **Screen Sharing** is sufficient.

```
System Settings → General → Sharing → Remote Management
```

> Note: Remote Management and Screen Sharing cannot both be active simultaneously on the same Mac. Choose one.

## Step 4: Set a VNC Password (Recommended)

For an additional layer of security on top of Tailscale's encryption, set a VNC-specific password:

### macOS Ventura / Sonoma:
```
System Settings → General → Sharing → Screen Sharing → (i) button → 
  "VNC viewers may control screen with password" → Set password
```

### macOS Monterey and earlier:
```
System Preferences → Sharing → Screen Sharing → Computer Settings →
  ✅ VNC viewers may control screen with password: [set password]
```

> **Note on security**: The VNC password is a secondary defence. Tailscale's WireGuard encryption is the primary protection. Without the Tailscale tunnel, the VNC port (`5900`) is not reachable from the internet at all. You do not need to open any firewall ports on your router.

## Step 5: Verify VNC is Listening

```bash
# In Terminal on the Mac Mini
sudo lsof -iTCP:5900 -sTCP:LISTEN
# Should show:
# screenshar  ... TCP *:rfb (LISTEN)
```

Or verify with:

```bash
netstat -an | grep 5900
# tcp4   0   0  *.5900   *.*   LISTEN
```

## macOS Firewall Considerations

If you have the macOS Application Firewall enabled:

```
System Settings → Privacy & Security → Firewall
```

You need to ensure incoming VNC connections are allowed:

```
Firewall Options → ✅ "screensharingd" → Allow incoming connections
```

Since Tailscale connections arrive via the `utun` (WireGuard) interface rather than the physical network interface, macOS generally handles this correctly without additional configuration. But if connections fail, checking the firewall is the first thing to try.

## macOS Display Behaviour When No Monitor Is Connected

Mac Minis are often used headlessly (no physical display). By default, macOS drops to a very low default resolution when no monitor is detected.

To get a usable resolution when connecting from your iPad:

### Option 1: Use a "Headless Display Adapter" (hardware dongle, ~€5)
- Plugs into the HDMI/Thunderbolt port
- Fools macOS into thinking a monitor is connected
- Forces a proper resolution (e.g., 1920×1080 or 4K)

### Option 2: macOS Sonoma+ virtual display
- macOS Sonoma (14+) supports **virtual display resolution** — you can set a resolution even without a physical display
- System Settings → Displays → set desired resolution

### Option 3: Resolution via VNC client
Some VNC clients (like Jump Desktop) can request a specific resolution at connection time.

## Summary Checklist

```
✅ System Settings → General → Sharing → Screen Sharing = ON
✅ VNC password set (optional but recommended)
✅ Tailscale running on Mac Mini with auto-start at login
✅ Mac Mini configured to not sleep
✅ Verified port 5900 is listening (netstat or lsof)
✅ Headless display adapter or virtual display configured (if no monitor)
```

Once these are in place, your Mac Mini is ready to accept VNC connections from your iPad over the Tailscale tunnel.
