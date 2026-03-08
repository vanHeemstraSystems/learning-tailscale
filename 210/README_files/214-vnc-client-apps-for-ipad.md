# 214 — VNC Client Apps for iPad: Comparison

When controlling a Mac Mini from your iPad over Tailscale, you need a VNC client app on the iPad. Here are the main options, compared honestly.

## Quick Recommendation

| Use Case | Recommended App |
|----------|----------------|
| Best overall Apple experience | **Screens 5** (paid, native Tailscale integration) |
| Free, cross-platform | **RealVNC Viewer** |
| Free, open-source, self-hosted servers | **RustDesk** |
| High-performance, gaming/creative work | **Jump Desktop** |
| Built into macOS (Mac-to-Mac only) | **macOS Screen Sharing** |

---

## Screens 5 (by Edovia)

**Best choice for iPad → Mac Mini over Tailscale.**

Screens 5 features native Tailscale integration, letting you connect via Tailscale directly from the connection sheet without manually entering IP addresses. It is built specifically for Apple devices and feels at home on iPadOS.

**Highlights:**
- Directly integrates with Tailscale — tailnet devices appear automatically in the connection list.
- Apple Pencil support lets you interact with remote content naturally, including drag-and-drop and precise clicking.
- Clipboard sharing supports rich text, URLs, and images between your iPad and Mac Mini.
- Multi-display support: easily select which display to view on multi-screen Macs, or show them all.
- Curtain Mode blanks the Mac Mini's physical display while you are remotely connected
- File transfer: drag and drop files between your iPad and Mac using the Screens interface.
- Touch ID / Face ID to lock the connection

**Requirements:**
- Screens requires iPadOS 17.0 or later.
- Screens Connect (the companion Mac helper app) requires macOS 10.13 High Sierra or later.
- For a Mac Mini, you do **not** need Screens Connect if you use Tailscale — Tailscale provides the network path and macOS's built-in VNC server provides the remote desktop.

**Pricing:** Free to download with a paid subscription or one-time lifetime purchase to unlock full features.

**App Store:** [Screens 5 on App Store](https://apps.apple.com/app/screens-5-vnc-remote-desktop/id1663047912)

---

## RealVNC Viewer (by RealVNC)

**Best free option.**

The official VNC client from the creators of the VNC protocol. Free, no subscription required for basic use.

**Highlights:**
- Completely free
- Cross-platform (Windows, macOS, Linux, iOS, Android)
- Solid touch controls for iPad
- Direct connection via IP address or hostname
- No built-in Tailscale integration — enter the Tailscale IP manually

**How to connect to Mac Mini via Tailscale:**
```
RealVNC Viewer → + New Connection
Address: 100.64.x.y   (Mac Mini's Tailscale IP)
Name:    Mac Mini
Connect → enter VNC password when prompted
```

**Pricing:** Free

**App Store:** [RealVNC Viewer on App Store](https://apps.apple.com/app/vnc-viewer-remote-desktop/id352019548)

---

## RustDesk (open-source)

**Best if you want fully self-hosted remote desktop with no third-party cloud.**

RustDesk is an open-source remote access tool with clients for Windows, Mac, Linux, iOS, and Android. It uses the latest streaming codecs to feel snappy, keeps your data on your own devices, and is free for non-commercial use.

To connect via Tailscale, paste the Tailscale IP of the device you want to control into the "Control Remote Desktop" section of the RustDesk window. If you've enabled direct IP access in RustDesk settings, it connects immediately.

**Highlights:**
- Fully open-source
- Works with Tailscale via direct IP access mode
- Modern codec performance
- Requires RustDesk to be installed on **both** iPad and Mac Mini

**Setup on Mac Mini:**
```
1. Install RustDesk on Mac Mini
2. RustDesk Settings → Security → enable "Direct IP Access"
3. Note the RustDesk ID (nine-digit number)
```

**Connecting from iPad:**
```
RustDesk (iPad) → Control Remote Desktop
Enter: 100.64.x.y   (Mac Mini's Tailscale IP)
Connect → enter the access password
```

**Pricing:** Free for personal use

**App Store:** [RustDesk on App Store](https://apps.apple.com/app/rustdesk-remote-desktop/id1581225044)

---

## Jump Desktop (by Phase Five Systems)

**Best for high-performance needs (design, video, interactive work).**

Jump Desktop uses its own **Fluid remote desktop protocol** in addition to VNC/RDP. The Fluid protocol is designed for high frame rates and responsiveness, making it suitable for creative workflows where screen updates are frequent.

**Highlights:**
- Fluid protocol for high-frame-rate remote sessions (up to 60fps)
- RDP support (for Windows machines in the same tailnet)
- VNC support (for Mac Mini with Screen Sharing)
- Multi-display support
- Clipboard sync
- Mouse precision mode for design apps

**Connecting to Mac Mini via Tailscale:**
```
Jump Desktop → + Add Computer → Manual
Address: 100.64.x.y  (or MagicDNS hostname)
Protocol: VNC
Port: 5900
```

**Pricing:** One-time purchase (no subscription)

**App Store:** [Jump Desktop on App Store](https://apps.apple.com/app/jump-desktop-rdp-vnc-fluid/id364876095)

---

## macOS Screen Sharing (Mac-to-Mac only)

If you ever want to control your Mac Mini from **another Mac** (rather than from an iPad), macOS has a built-in **Screen Sharing** app:

```bash
# Method 1: Finder
# Finder → Go → Connect to Server → vnc://100.64.x.y

# Method 2: Screen Sharing app
# /System/Library/CoreServices/Applications/Screen Sharing.app
# Enter: vnc://100.64.x.y  or  mac-mini.ts.net
```

This is not available for iPad, but useful for a Mac-to-Mac scenario — for example, controlling the Mac Mini from a MacBook while both are on the tailnet.

---

## Feature Comparison Table

| Feature | Screens 5 | RealVNC | RustDesk | Jump Desktop |
|---------|-----------|---------|----------|-------------|
| Price | Paid | Free | Free | One-time |
| Native Tailscale integration | ✅ | ❌ | ❌ | ❌ |
| Apple Pencil support | ✅ | ❌ | ❌ | ❌ |
| Curtain Mode | ✅ | ❌ | ❌ | ❌ |
| File transfer (iPad↔Mac) | ✅ | ❌ | ❌ | ❌ |
| Multi-display | ✅ | ❌ | ❌ | ✅ |
| High-frame-rate protocol | ❌ (VNC) | ❌ (VNC) | ✅ (own codec) | ✅ (Fluid) |
| RDP support | ❌ | ❌ | ❌ | ✅ |
| Face/Touch ID lock | ✅ | ❌ | ❌ | ✅ |
| Open source | ❌ | ❌ | ✅ | ❌ |
| iPadOS version required | 17+ | 14+ | 16+ | 16+ |
