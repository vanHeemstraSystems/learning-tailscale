# 200 — Installation: Linux

## Quick Install (Ubuntu/Debian)

```bash
# Add Tailscale's package signing key and repository
curl -fsSL https://pkgs.tailscale.com/stable/ubuntu/noble.noarmor.gpg \
  | sudo tee /usr/share/keyrings/tailscale-archive-keyring.gpg >/dev/null

curl -fsSL https://pkgs.tailscale.com/stable/ubuntu/noble.tailscale-keyring.list \
  | sudo tee /etc/apt/sources.list.d/tailscale.list

# Install
sudo apt-get update
sudo apt-get install tailscale

# Start the daemon
sudo systemctl enable --now tailscaled

# Authenticate (opens browser or prints auth URL)
sudo tailscale up
```

## RHEL / CentOS / Fedora

```bash
sudo dnf config-manager --add-repo https://pkgs.tailscale.com/stable/rhel/9/tailscale.repo
sudo dnf install tailscale
sudo systemctl enable --now tailscaled
sudo tailscale up
```

## Alpine Linux

```bash
apk add tailscale
rc-update add tailscale
rc-service tailscale start
tailscale up
```

## Headless / Server Installation

For servers with no browser, authenticate via URL:

```bash
sudo tailscale up --authkey=<your-auth-key>
```

Auth keys can be created in the Tailscale admin panel under **Settings → Keys**. They are especially useful for automated provisioning and CI/CD.

### Auth Key Types

| Type | Reusable | Expires | Use Case |
|------|----------|---------|----------|
| One-off | No | Configurable | Single device provisioning |
| Reusable | Yes | Configurable | Automation / scripts |
| Ephemeral | Yes | Device removed on disconnect | CI runners, containers |

## Verify Installation

```bash
# Check daemon status
sudo systemctl status tailscaled

# Check tailscale status
tailscale status

# View your tailnet IP
tailscale ip -4

# View all connected peers
tailscale status --peers
```

## Running as Non-Root

The Tailscale daemon (`tailscaled`) runs as root. The CLI (`tailscale`) can be run as any user that is in the `tailscale` group:

```bash
sudo usermod -aG tailscale $USER
# Re-login for group change to take effect
```

## Key Files

| Path | Description |
|------|-------------|
| `/etc/tailscale/` | Configuration directory |
| `/var/lib/tailscale/` | State directory (keys, peer cache) |
| `/var/run/tailscale/tailscaled.sock` | Unix socket for CLI communication |
| `/usr/bin/tailscale` | CLI binary |
| `/usr/sbin/tailscaled` | Daemon binary |

## Uninstall

```bash
sudo tailscale down
sudo systemctl stop tailscaled
sudo apt-get remove tailscale   # or dnf remove, apk del
```
