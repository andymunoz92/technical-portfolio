# 07 — Verification and troubleshooting log

[← Index](README.md) · [← Webserver](06-nginx-ssh.md) · [Next: Security →](08-security-roadmap.md)

> A strong technical portfolio distinguishes **“I installed it”** from **“I demonstrated it works.”** This page records testable evidence and lessons rather than claiming every component is production-hardened.

## Evidence log

| Component | Observation | Conclusion | Remaining caveat |
|---|---|---|---|
| Proxmox | Web dashboard reached after installation; host services active | Hypervisor management working | Not evidence of HA, backups or production readiness |
| Tailscale | Outside-network access to Proxmox from public Wi-Fi | Private remote route and management access worked | Periodically recheck access policies and approved routes |
| Pi-hole | Direct query to resolver: allowed domain resolved, known ad domain returned `0.0.0.0` | DNS resolver and filtering worked for those queries | Does not prove universal device-level DNS enforcement |
| Webserver | Default/custom page served via Nginx | Static content works on LAN | Public HTTPS/reverse proxy not yet deployed |
| SSH | Mac terminal SSH to webserver worked | Server authentication and basic connectivity worked | VS Code Remote-SSH failure still needs controlled retest |

## A layered diagnostic runbook

1. **Define exact symptom.** Example: “Web page fails from my Mac while away from home,” not simply “network broken.”
2. **Identify the path.** Client → Tailscale → advertised route → private host → port → web service.
3. **Check scope.** Does the failure occur on LAN, via Tailscale, or both?
4. **Inspect addressing/routes.** `ip -br a`, `ip route`, Tailscale status and route approvals.
5. **Inspect service.** `systemctl status <service>`, `ss -lntup`, `journalctl -u <service>`.
6. **Make a protocol-specific request.** `curl -I` for HTTP, `dig` for DNS, SSH client verbose logs for SSH.
7. **Change one variable at a time.** Capture the before/after command output and revert unsuccessful changes.
8. **Document cause, remediation and validation.** Do not confuse symptoms with confirmed root cause.

## Example: distinguish DNS failure from connectivity failure

```bash
ping -c 2 <gateway-ip>               # Link/L3 control test
getent hosts debian.org               # Resolver configured on this machine
dig @<dns-ip> debian.org A            # Specific DNS server
dig @<dns-ip> doubleclick.net A      # Known filter test
curl -I http://<webserver-ip>         # HTTP response, not just ping
```

## Example: VS Code Remote-SSH discrepancy

**Observed:** direct SSH from the Mac terminal succeeded; VS Code Remote-SSH reported `No route to host` in a separate attempt.

**Hypotheses to test, not diagnosed facts:** different SSH binary/config, an out-of-date VS Code host alias, Tailscale route difference, stale extension state, or a different local network context.

**Discriminating tests:** compare `ssh -G webserver` and `ssh -vvv webserver` in the VS Code and terminal contexts; verify the same destination IP, identity and network route. Investigate logs before changing server configuration.

## Retest checklist after reboot

- [ ] Proxmox web UI and SSH respond
- [ ] `pct list` shows expected containers started
- [ ] Tailscale agent connected; approved routes still active
- [ ] Pi-hole resolves allowed domains and filters a test domain
- [ ] Nginx serves the landing page
- [ ] Terminal SSH works using the dedicated key
- [ ] VS Code Remote-SSH works, or failure is documented
- [ ] Backups and configuration snapshots are verified (planned until tested)

## Lessons learned

- Use **independent paths** to test network access (outside Wi-Fi is stronger than only LAN testing).
- For routing issues, distinguish an absent route from a blocked port and an inactive service.
- For DNS issues, distinguish a resolver that is down from a configured client that never queries it.
- Record unresolved discrepancies rather than claiming everything is fixed.
