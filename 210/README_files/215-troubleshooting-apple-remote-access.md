# 215 — Troubleshooting: iPad → Mac Mini Remote Access

## Diagnostic Flowchart

```
Problem: Cannot connect to Mac Mini from iPad
               │
               ▼
Is Tailscale running on Mac Mini?
   tailscale status (Terminal on Mac Mini)
         │
    ┌────┴────┐
    │         │
   YES        NO → Start Tailscale / check auto-start
    │
    ▼
Is Mac Mini visible in Tailscale app on iPad?
   Tailscale app → My Devices → see mac-mini?
         │
    ┌────┴────┐
    │         │
   YES        NO → Same tailnet? Same account? Re-authenticate?
    │
    ▼
Can iPad ping Mac Mini via Tailscale?
   Tailscale app → tap mac-mini → "Ping" (or use direct)
         │
    ┌────┴────┐
    │         │
   YES        NO → Network issue, DERP fallback? (see below)
    │
    ▼
Is Screen Sharing enabled on Mac Mini?
   System Settings → General → Sharing → Screen Sharing
         │
    ┌────┴────┐
    │         │
   YES        NO → Enable it
    │
    ▼
Is the correct VNC address / port in the VNC client?
   Address: 100.64.x.y (Tailscale IP, NOT local 192.168.x.x)
   Port:    5900
         │
    ┌────┴────┐
    │         │
   YES        NO → Fix address in VNC client
    │
    ▼
Is the VNC password correct?
```

---

## Issue: Mac Mini Not Appearing in Tailscale

**Symptoms:** iPad's Tailscale app does not list `mac-mini`, or shows it as offline.

**Causes and fixes:**

1. **Tailscale is not running on Mac Mini**
   ```bash
   # On Mac Mini Terminal:
   tailscale status
   # If shows "stopped" or no output:
   tailscale up
   ```

2. **Tailscale is not set to launch at login**
   ```
   Tailscale menu bar → Preferences → ✅ Launch at Login
   ```

3. **Mac Mini is asleep**
   ```
   System Settings → Energy Saver → uncheck "Put hard disks to sleep"
   System Settings → Energy Saver → ✅ Wake for network access
   ```

4. **Different Tailscale accounts**
   Both devices must be logged in with the same identity (e.g., same Google account). Check in Tailscale admin panel → Machines — do you see both devices?

5. **Key expired on Mac Mini**
   ```bash
   tailscale status
   # If shows "NeedsLogin":
   tailscale up
   # Complete re-authentication in browser
   ```

---

## Issue: Tailscale Connects but VNC Connection Fails

**Symptoms:** Mac Mini is visible in Tailscale, ping works, but VNC client shows "Connection refused" or times out.

**Causes and fixes:**

1. **Screen Sharing is not enabled**
   ```
   System Settings → General → Sharing → Screen Sharing → toggle ON
   ```
   Verify it is running:
   ```bash
   sudo lsof -iTCP:5900 -sTCP:LISTEN
   # Should show screensharingd or screenshar
   ```

2. **Wrong IP address in VNC client**
   - Use the **Tailscale IP** (`100.64.x.y`), not the local network IP (`192.168.x.x`)
   - The local IP is unreachable when you are outside your home network
   - Find the right IP: Tailscale app → My Devices → mac-mini → copy IP

3. **macOS firewall blocking VNC**
   ```
   System Settings → Privacy & Security → Firewall → Firewall Options
   → screensharingd → ✅ Allow incoming connections
   ```

4. **Wrong VNC password**
   - VNC password is set in System Settings → Sharing → Screen Sharing → (i) → "VNC viewers may control screen with password"
   - This is different from your macOS login password
   - Try resetting the VNC password if you are unsure

5. **VNC client using wrong port**
   - Port must be `5900`
   - Some VNC clients default to a different port or use display numbers (`:1` = port 5901, `:0` = port 5900)

---

## Issue: Connection Works but Display is Tiny / Low Resolution

**Symptoms:** Mac Mini desktop appears very small or at low resolution (e.g., 800×600 or 1024×768).

**Cause:** Mac Mini has no physical monitor connected and falls back to a low default resolution.

**Fixes:**

1. **Use a headless display adapter** (HDMI/Thunderbolt dummy plug, ~€5 on Amazon)
   - Plugs into the Mac Mini's display port
   - Forces macOS to maintain a proper resolution (1920×1080, 2560×1440, etc.)
   - The most reliable long-term solution

2. **macOS Sonoma virtual display** (software only)
   ```
   System Settings → Displays → [even without a monitor, Sonoma allows
   you to select a resolution]
   ```

3. **Use a RandR/display preference profile**
   ```bash
   # Create a display override file (advanced, use sparingly)
   # Or use the free app "SwitchResX" or "RDM" to force resolution
   ```

---

## Issue: Connection is Laggy or Slow

**Symptoms:** Screen updates are slow, keystrokes are delayed, scrolling is choppy.

**Diagnosis:**
```bash
# Check if connection is direct or via DERP relay
tailscale ping mac-mini

# Direct connection example:
# pong from mac-mini (100.64.1.10): via 1.2.3.4:41641 in 3ms ← good

# DERP relay example:
# pong from mac-mini (100.64.1.10): via DERP(ams) in 45ms ← higher latency
```

**Fixes:**

1. **If via DERP relay:** Both devices are likely behind restrictive NAT. Check:
   ```bash
   # On Mac Mini:
   tailscale netcheck
   # Look for: UDP: false → all connections forced to relay
   ```
   If UDP is blocked on your home router, enable UDP pass-through on the router or check firewall rules.

2. **VNC quality setting:** In Screens 5:
   ```
   Connection settings → Display Quality → Adaptive
   ```
   Adaptive mode reduces colour depth when bandwidth is limited, improving responsiveness.

3. **Check your internet connection:** VNC screen quality depends on upload bandwidth from the Mac Mini and download bandwidth on the iPad. Video-heavy desktops need more bandwidth.

4. **Close unnecessary apps on Mac Mini:** Reduce the number of screen redraws the VNC server needs to encode.

---

## Issue: Tailscale Disconnects on iPad When Screen Locks

**Symptoms:** Connection drops after a few minutes of iPad idle/screen lock.

**Fix:**
```
iOS/iPadOS Settings → VPN & Device Management → VPN → 
  Tailscale → Connect On Demand → ✅ Always On
```

Or, keep your iPad screen active during remote sessions using the auto-lock setting:

```
Settings → Display & Brightness → Auto-Lock → Never
```

(Remember to re-enable auto-lock when done for battery life.)

---

## Issue: Authentication Key Expired

**Symptoms:** Tailscale shows "Login required" or "NeedsLogin" on one device.

**Fix:**
- On Mac Mini: `tailscale up` in Terminal → complete SSO login in browser
- On iPad: Open Tailscale app → tap the account warning → **Re-authenticate** → complete SSO in the sheet

**Prevention:** In the Tailscale admin panel, you can **disable key expiry** for specific devices (e.g., your Mac Mini) so it never requires re-authentication:
```
Tailscale Admin → Machines → mac-mini → ... → Disable key expiry
```

---

## Quick Diagnostics Reference

```bash
# Mac Mini Terminal — full health check:
echo "=== Tailscale status ===" && tailscale status
echo "=== Tailscale IP ===" && tailscale ip -4
echo "=== VNC port listening ===" && sudo lsof -iTCP:5900 -sTCP:LISTEN
echo "=== Network check ===" && tailscale netcheck
echo "=== Ping iPad ===" && tailscale ping my-ipad
```
