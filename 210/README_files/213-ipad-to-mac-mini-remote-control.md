# 213 — iPad → Mac Mini: End-to-End Walkthrough

## Prerequisites Checklist

Before starting, confirm all of these are in place:

```
Mac Mini:
  ✅ Tailscale installed and running (auto-start at login)
  ✅ Screen Sharing enabled (System Settings → Sharing)
  ✅ VNC password set (optional but recommended)
  ✅ Not sleeping (Energy settings configured)
  ✅ Headless adapter if no monitor is connected

iPad:
  ✅ Tailscale installed and connected to the same tailnet
  ✅ VNC client installed (Screens 5 or alternative — see module 214)
```

## Step 1: Find the Mac Mini's Tailscale Address

Two ways to get the address you will need:

### Option A: From the Tailscale app on iPad
```
Tailscale app → tap "mac-mini" → note the IP: 100.64.x.y
```

### Option B: Via MagicDNS hostname (if MagicDNS is enabled)
```
mac-mini                          ← short name
mac-mini.your-tailnet.ts.net      ← fully qualified hostname
```

The MagicDNS hostname is more reliable long-term because it does not change even if the Tailscale IP is reassigned.

## Step 2: Connect with Screens 5

*This walkthrough uses **Screens 5** by Edovia — the most polished native VNC client for iPad. See module 214 for alternatives.*

### First-time: Create a new connection

1. Open **Screens 5** on your iPad
2. Tap the **+** button → **New Screen**
3. Fill in the connection details:

   ```
   Name:      Mac Mini (or any label you like)
   Address:   100.64.x.y   (your Mac Mini's Tailscale IP)
                    — OR —
              mac-mini.your-tailnet.ts.net   (MagicDNS hostname)
   Port:      5900  (default, leave unchanged)
   Display:   Screen 1 (default)
   ```

4. Under **Authentication**:
   ```
   Type:     VNC Password
   Password: [the VNC password you set in System Settings → Sharing]
   ```

5. Tap **Save**

### Connecting

Tap the **Mac Mini** connection card → Screens 5 establishes the VNC session over the Tailscale tunnel. You will see the Mac Mini's full desktop on your iPad.

### Screens 5 + Tailscale Integration (Native)

Screens 5 has **built-in Tailscale integration** — it can list your tailnet devices directly:

```
Screens 5 → + New Screen → Connect via Tailscale
  → Your tailnet devices appear automatically
  → Tap "mac-mini" → it fills in the address for you
```

This is the smoothest connection method: no IP to memorise, no hostname to type.

## Step 3: Navigating the Mac Mini from Your iPad

Once connected, your iPad shows the full Mac Mini desktop. Here is how the touch controls map:

| iPad gesture | Mac Mini action |
|-------------|----------------|
| Single tap | Left mouse click |
| Two-finger tap | Right mouse click |
| Two-finger scroll | Scroll wheel |
| Two-finger pinch/zoom | Zoom in/out on the remote display |
| Three-finger drag | Pan the remote screen |
| Hardware keyboard | Types on the Mac Mini |
| Apple Pencil tap | Precise pointer click |
| Apple Pencil drag | Click and drag |

## Step 4: Using the Screens 5 Toolbar

Screens 5 shows a collapsible toolbar at the top/bottom of the screen (iPad). Key controls:

```
Toolbar icons:
  ⌘  → Command key modifier (for ⌘+C, ⌘+V, etc.)
  ⌥  → Option/Alt key
  ⌃  → Control key
  ⇧  → Shift key
  📋 → Clipboard (paste content from your iPad to the Mac Mini)
  ⌨️  → Software keyboard toggle
  Fn → Function keys (F1–F12)
  ⊞  → Window / Mission Control shortcuts
  ⚙️  → Connection settings
```

## Step 5: Key Workflow Tips

### Clipboard sharing
Text and URLs copy seamlessly between your iPad and Mac Mini:
- Copy text on iPad → tap 📋 in Screens toolbar → paste on Mac Mini
- Copy text on Mac Mini → it appears in iPad clipboard

### Using a hardware keyboard with iPad
Connect a Magic Keyboard or Bluetooth keyboard to your iPad. Keystrokes are forwarded directly to the Mac Mini — you get a near-native typing experience.

### Magic Trackpad with iPad
If you pair a Magic Trackpad with your iPad (via Bluetooth), Screens maps trackpad gestures to mouse movements on the Mac Mini. This gives you a laptop-quality remote control experience from an iPad.

### Multiple displays on Mac Mini
If your Mac Mini has multiple monitors connected (or virtual displays configured):
```
Screens 5 toolbar → Window icon → select display 1, 2, or "All"
```

### Curtain mode
When you connect from Screens 5, you can optionally enable **Curtain Mode**:
```
Screens 5 → Connection settings → Enable Curtain Mode
```
This blanks the Mac Mini's physical display while you are connected, useful for privacy in shared spaces.

## Workflow Example: Working from a Café

```
1. iPad: Swipe down Control Center → tap Tailscale VPN tile → connected
2. iPad: Open Screens 5 → tap "Mac Mini" connection
3. [Screens 5 connects via Tailscale tunnel to Mac Mini at home]
4. Mac Mini desktop appears on iPad
5. Use Magic Keyboard for typing, Screens 5 toolbar for Mac shortcuts
6. When done: ⌘+Q to quit apps on Mac Mini, then close Screens 5
7. iPad: Tailscale remains connected for other tailnet services
```

## Measuring Connection Quality

```bash
# In Terminal on the iPad (via SSH or Tailscale SSH):
# Or on any device, to check round-trip time to Mac Mini:

# From another Mac in your tailnet:
tailscale ping mac-mini

# From the iPad's Screens 5:
Connection settings → Connection Quality indicator (bottom of screen)
```

A direct Tailscale connection typically gives **2–20ms latency** on a home broadband connection — excellent for remote desktop use. A relay (DERP) connection adds **20–80ms** depending on the relay region, which is still usable for most tasks.
