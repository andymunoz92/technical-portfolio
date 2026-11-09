# 08 — Security, operational limits, and roadmap

[← Index](README.md) · [← Troubleshooting](07-validation-troubleshooting.md)

## Existing precautions

- Management services were intended to stay **private**, reachable using Tailscale rather than an intentionally public-facing Proxmox web UI.
- A lightweight **unprivileged LXC** was used for the Tailscale gateway, with specific TUN access.
- SSH public-key authentication was configured for the developer workflow.
- Distinct containers separate the major service processes and give each service clear management boundaries.
- The host uses Ethernet only, reducing unnecessary network interface complexity.

These are design and configuration choices, not an independent security audit.

## Outstanding checks (not yet verified as completed)

- [ ] Prove SSH password authentication settings and least-privilege sudo policy
- [ ] Audit Tailscale ACLs, authentication, unused devices, and approved routes
- [ ] Review local Proxmox/LXC firewall rules and patch cadence
- [ ] Back up container configuration/data; complete a documented **restore test**
- [ ] Review log retention and add lightweight health alerting
- [ ] Store reproducible configurations in version control without secrets
- [ ] Document physical power-loss recovery and UPS decisions
- [ ] Add TLS/reverse proxy only when a public web-service security plan is approved
- [ ] Recheck VS Code Remote-SSH route and identify the root cause

## Limits and honest trade-offs

- **Single host and HDD:** no hardware high availability; host failure affects all services.
- **8 GB RAM:** small memory budget; monitoring/SIEM or IDS tasks require planning and may be more suitable for a future upgraded system.
- **Single network:** multiple LXC containers share the host/network bridge; isolation is not equivalent to segmented VLANs with firewall policy.
- **Pi-hole DNS filtering:** cannot replace firewall, IDS, or endpoint security.
- **Tailscale subnet-router:** provides private connectivity subject to ACLs and route approval; it is not automatic proof of zero trust or full host hardening.
- **Static Nginx service:** not yet production reverse proxy, public HTTPS, WAF, or load balancer.

## What I would do next—and why

| Phase | Improvement | Benefit |
|---|---|---|
| 1 | Backups with tested restores | Recovery after host, disk or container failure |
| 2 | Monitoring (RAM, disk, service availability) | Detect problems before a user reports them |
| 3 | Harden SSH, patching, and ACLs | Reduce unnecessary trust and attack surface |
| 4 | Document IaC/config management | Repeatability; limit configuration drift |
| 5 | Explore SIEM/IDS in resource-aware scope | Security telemetry and alerting practice |
| 6 | Consider public TLS site with reverse proxy | Controlled extension from private lab to public deployment |

## Sources to study (official references)

- [Proxmox installation documentation](https://github.com/proxmox/pve-docs/blob/master/pve-installation.adoc)
- [Tailscale subnet-router guide](https://tailscale.com/docs/features/subnet-routers)
- [Pi-hole native install](https://docs.pi-hole.net/main/basic-install/)
- [Nginx static content](https://docs.nginx.com/nginx/admin-guide/web-server/serving-static-content/)

## Public portfolio publication checklist

Do not upload `~/.ssh/`, private keys, tokens, user credentials, Tailscale auth material, private DNS logs, personally identifying screenshots, unredacted router admin images, or unrestricted Proxmox config dumps. Keep internal IPs and specific container MAC addresses in private notes rather than public docs. Replace account identifiers, hostnames, unique addresses and dates if necessary. Test all public links and mark unimplemented features as planned.
