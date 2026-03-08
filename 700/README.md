# 700 — Observability & Troubleshooting

> *"A network you can't observe is a network you can't trust."*

## Contents

| File | Description |
|------|-------------|
| [700-tailscale-status-and-netcheck.md](README_files/700-tailscale-status-and-netcheck.md) | Key CLI commands for inspecting tailnet health |
| [710-logging-and-audit-trails.md](README_files/710-logging-and-audit-trails.md) | Access logs, configuration audit logs, and streaming to SIEM |
| [720-troubleshooting-nat-traversal.md](README_files/720-troubleshooting-nat-traversal.md) | Diagnosing and fixing connection issues |
| [730-performance-tuning.md](README_files/730-performance-tuning.md) | Throughput, MTU, kernel vs userspace WireGuard |

## Learning Objectives

- Use `tailscale status`, `tailscale ping`, `tailscale netcheck` fluently
- Diagnose relay vs direct connections
- Stream Tailscale logs to an external SIEM
- Understand MTU and performance factors
- Troubleshoot common NAT traversal failures
