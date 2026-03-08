# 210 — Tailscale on macOS (Mac Mini)

## Three Ways to Install Tailscale on macOS

Apple's privacy and security model means there are three distinct installation methods, each with slightly different behaviour.

### Option 1: App Store (Recommended for most users)

```
App Store → search "Tailscale" → Install (by Tailscale Inc.)
```

- Runs as a standard macOS app inside the sandbox
- Auto-updates via the App Store
- Menu bar icon appears after first launch
- Best for personal use and home setups

### Option 2: Direct Download (tailscale.com)

```bash
# Download from https://tailscale.com/download/macos
# Open the .pkg installer → Follow prompts
```

- Installs a system extension (`tailscaled` daemon)
- Can be managed via CLI (`tailscale` command in Terminal)
- Supports headless/server operation
- Better for always-on Mac Mini use cases

### Option 3: Homebrew

```bash
brew install --cask tailscale
open -a Tailscale   # initialises the helper and CLI shim
```

## First-Time Setup

1. Launch Tailscale from the Applications folder or menu bar
2. Click **Log in**
3. A browser window opens — authenticate with Apple ID, Google, Microsoft, or GitHub
4. Grant the VPN configuration permission when prompted (this is how Tailscale creates the WireGuard tunnel)
5. Your Mac Mini is now in your tailnet

> **Tip**: Use **Sign in with Apple** to keep your Tailscale identity private and avoid sharing your email with Google or Microsoft.

## Verifying the Installation

```bash
# Open Terminal on the Mac Mini
tailscale status

# Output example:
# 100.64.1.10   mac-mini         you@    macOS  -
# 100.64.1.20   my-ipad          you@    iOS    idle

# Get your Mac Mini's Tailscale IP
tailscale ip -4
# → 100.64.1.10

# Get your MagicDNS hostname
tailscale status --json | jq '.Self.DNSName'
# → "mac-mini.your-tailnet.ts.net."
```

## Make Tailscale Start at Login (Essential for Unattended Mac Mini)

For the Mac Mini to always be reachable, Tailscale must start automatically when macOS boots, even without a user logging in.

### App Store version:
```
Tailscale menu bar icon → Preferences → ✅ Launch at Login
```

### Direct install / Homebrew version:

```bash
# The system daemon (tailscaled) starts automatically as a LaunchDaemon
# Verify it is enabled:
sudo launchctl list | grep tailscale

# If not running:
sudo launchctl load /Library/LaunchDaemons/com.tailscale.tailscaled.plist
```

> **Important for Mac Mini headless use**: If no user is logged in at the macOS login screen, Tailscale's App Store version may not start until someone logs in. The **direct install** version installs `tailscaled` as a system-level LaunchDaemon that starts at boot regardless of login status. For a Mac Mini used headlessly, prefer the direct install method.

## Keeping the Mac Mini Reachable After Restarts

For a Mac Mini acting as an always-available remote host:

1. **Set macOS to log in automatically**: System Settings → Users & Groups → Automatic Login (select your account)
2. **Enable Tailscale Launch at Login** (see above)
3. **Prevent sleep**: System Settings → Energy Saver → set "Prevent Mac from sleeping automatically when the display is off" ✅
4. **Wake for network access**: System Settings → Energy → ✅ Wake for network access

This ensures your Mac Mini is always at the login screen (or logged in) with Tailscale running — ready to accept your VNC connection from the iPad.

## macOS Tailscale Menu Bar

The Tailscale menu bar icon is your control centre on macOS:

```
Tailscale menu bar icon (▲ in menubar)
  ├── This device: mac-mini (100.64.1.10)
  ├── Connected devices
  │     ├── my-ipad      100.64.1.20  ← click to copy IP
  │     └── ...
  ├── Exit Node          ← configure exit node
  ├── Use exit node...
  ├── Admin console      ← opens browser
  ├── Preferences
  │     ├── ✅ Launch at Login
  │     ├── ✅ Accept routes
  │     └── ✅ Accept DNS
  └── Log out
```

## CLI Reference for macOS

```bash
# Check status
tailscale status

# Connect / reconnect
tailscale up

# Disconnect (stays authenticated)
tailscale down

# Check your IP
tailscale ip -4

# Ping another device
tailscale ping my-ipad

# View version
tailscale version

# Update (direct install)
tailscale update
```
