# 300 — ACL Policy Language

## Overview

Tailscale access policies are written in **HuJSON** (Human JSON — JSON with comments and trailing commas) and managed in the **Tailscale admin panel → Access Controls**, or via the Tailscale API / Terraform provider.

The policy file is called `acl.json` internally.

## Structure

```json
{
  // Groups: named collections of users
  "groups": {
    "group:developers": ["alice@acme.com", "bob@acme.com"],
    "group:ops":        ["ops@acme.com"]
  },

  // Tag owners: which users/groups can assign these tags
  "tagOwners": {
    "tag:server":     ["group:ops"],
    "tag:k8s-node":   ["group:ops"],
    "tag:ci-runner":  ["autogroup:admin"]
  },

  // Hosts: friendly names for IPs or CIDR ranges
  "hosts": {
    "prod-db":   "100.64.1.10",
    "dev-range": "100.64.2.0/24"
  },

  // ACLs: the actual access rules
  "acls": [
    // Allow developers to SSH into servers
    {
      "action": "accept",
      "src":    ["group:developers"],
      "dst":    ["tag:server:22"]
    },
    // Allow k8s nodes to talk to each other on any port
    {
      "action": "accept",
      "src":    ["tag:k8s-node"],
      "dst":    ["tag:k8s-node:*"]
    },
    // Allow everyone to use exit nodes
    {
      "action": "accept",
      "src":    ["autogroup:member"],
      "dst":    ["autogroup:internet:*"]
    }
  ],

  // SSH rules (for Tailscale SSH feature)
  "ssh": [
    {
      "action": "accept",
      "src":    ["group:ops"],
      "dst":    ["tag:server"],
      "users":  ["root", "ubuntu"]
    }
  ]
}
```

## Special Groups

Tailscale provides built-in groups that you do not need to define:

| Group | Meaning |
|-------|---------|
| `autogroup:member` | All authenticated users in the tailnet |
| `autogroup:admin` | Tailnet administrators |
| `autogroup:self` | The user themselves (for self-service rules) |
| `autogroup:internet` | The public internet (for exit node rules) |
| `autogroup:tagged` | All tagged devices |

## Tags

Tags provide **machine-level identity** independent of the human user who owns the device. This is critical for service accounts and automated systems.

```json
// A CI runner tagged as tag:ci-runner
// can be granted access regardless of which user authenticated it

"acls": [
  {
    "action": "accept",
    "src":    ["tag:ci-runner"],
    "dst":    ["tag:k8s-node:6443"]  // Kubernetes API server
  }
]
```

Tag assignment rules (defined in `tagOwners`) control who can give a device a tag.

## Port Specifications

In `dst` rules, ports are specified as `host:port`:

```
"100.64.1.10:22"     ← specific IP, specific port
"tag:server:22"      ← all tagged servers, port 22
"tag:server:22,443"  ← multiple ports
"tag:server:*"       ← all ports
"*:*"                ← everything (use with care!)
```

## Default Policy

Without any ACL rules, Tailscale uses an implicit **allow-all** policy (all tailnet members can reach all other members). Once you add your first ACL rule, the default changes to **deny-all**, and you must explicitly allow everything you need.

## Testing ACLs

Use the **ACL preview** in the admin panel to test rules before applying:

```bash
# Via API
curl -s "https://api.tailscale.com/api/v2/tailnet/{tailnet}/acl/preview" \
  -H "Authorization: Bearer $TS_API_KEY" \
  -d '{"previewFor": "alice@acme.com"}'
```

## GitOps for ACLs

Store your `acl.json` in a Git repository and apply it via CI:

```bash
# Apply ACL via Tailscale API
curl -X POST "https://api.tailscale.com/api/v2/tailnet/-/acl" \
  -H "Authorization: Bearer $TS_API_KEY" \
  -H "Content-Type: application/json" \
  -d @acl.json
```

See module 830 for full GitOps ACL management patterns.
