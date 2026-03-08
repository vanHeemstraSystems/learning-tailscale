# 210 — Apple Ecosystem: iPad → Mac Mini Remote Control

> *"Your iPad becomes a window into your Mac Mini — from any café, airport, or couch in the world. Tailscale is the invisible bridge that makes it feel like you never left home."*

This module focuses entirely on the **Apple ecosystem use case**: installing Tailscale on a Mac Mini (the host) and an iPad (the remote controller), then layering a VNC client on top so you can see and control your Mac Mini's full desktop from your iPad — securely, from anywhere.

## Contents

| File | Description |
|------|-------------|
| [210-tailscale-on-macos.md](README_files/210-tailscale-on-macos.md) | Installing and configuring Tailscale on macOS (Mac Mini) |
| [211-tailscale-on-ios-ipad.md](README_files/211-tailscale-on-ios-ipad.md) | Installing and configuring Tailscale on iPadOS |
| [212-macos-screen-sharing-vnc-setup.md](README_files/212-macos-screen-sharing-vnc-setup.md) | Enabling Screen Sharing / VNC on the Mac Mini |
| [213-ipad-to-mac-mini-remote-control.md](README_files/213-ipad-to-mac-mini-remote-control.md) | End-to-end walkthrough: controlling Mac Mini from iPad |
| [214-vnc-client-apps-for-ipad.md](README_files/214-vnc-client-apps-for-ipad.md) | Comparing VNC client apps: Screens 5, RealVNC, RustDesk, Jump Desktop |
| [215-troubleshooting-apple-remote-access.md](README_files/215-troubleshooting-apple-remote-access.md) | Common issues and fixes for the iPad → Mac Mini setup |

## The Full Picture

```
┌────────────────────────────────────────────────────────────────────┐
│                        Your Tailnet                                │
│                                                                    │
│   ┌───────────────────┐            ┌──────────────────────────┐    │
│   │   iPad            │            │   Mac Mini               │    │
│   │   (iPadOS 17+)    │            │   (macOS 14+)            │    │
│   │                   │            │                          │    │
│   │  ┌─────────────┐  │            │  ┌──────────────────┐    │    │
│   │  │ Tailscale   │  │◄──────────►│  │ Tailscale        │    │    │
│   │  │ (VPN layer) │  │  WireGuard │  │ (VPN daemon)     │    │    │
│   │  └──────┬──────┘  │  mesh      │  └──────────────────┘    │    │
│   │         │          │  tunnel    │                          │    │
│   │  ┌──────▼──────┐  │            │  ┌──────────────────┐    │    │
│   │  │ Screens 5   │  │            │  │ macOS Screen     │    │    │
│   │  │ (VNC client)│  │◄──────────►│  │ Sharing (VNC     │    │    │
│   │  └─────────────┘  │  VNC over  │  │ server, port     │    │    │
│   │                   │  Tailscale │  │ 5900)            │    │    │
│   └───────────────────┘            └──────────────────────────┘    │
│                                                                    │
│   iPad can be anywhere in the world. Mac Mini stays home.          │
└────────────────────────────────────────────────────────────────────┘
```

## Why Two Layers?

| Layer | Tool | Role |
|-------|------|------|
| **Network** | Tailscale | Creates the secure, encrypted tunnel between iPad and Mac Mini. No ports opened on your router. Works through any NAT, firewall, or mobile network. |
| **Remote Desktop** | VNC (Screens 5 / built-in) | Transmits the Mac Mini's display to the iPad and sends your touch/keyboard input back. Runs *inside* the encrypted Tailscale tunnel. |

Tailscale handles the "how do I reach my Mac Mini from anywhere" problem. VNC handles the "how do I see and control its screen" problem. Together, they replace expensive commercial products like TeamViewer or AnyDesk — for free or near-free.

## Learning Objectives

After completing this module you will be able to:

- Install and configure Tailscale on both macOS (Mac Mini) and iPadOS (iPad)
- Enable macOS Screen Sharing (VNC server) on the Mac Mini
- Install and configure a VNC client on the iPad
- Connect to the Mac Mini from the iPad using the Tailscale IP or MagicDNS hostname
- Keep the Mac Mini reachable even after restarts (auto-start Tailscale)
- Choose the right VNC client app for your workflow
- Troubleshoot the most common connection failures
