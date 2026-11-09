# 05 — Pi-hole: DNS resolution and filtering

[← Index](README.md) · [← Tailscale](04-tailscale.md) · [Next: Webserver →](06-nginx-ssh.md)

## Problem solved

I wanted DNS-based blocking for selected devices on my LAN and potentially while away through Tailscale, **without changing DNS for every device on my home router**. I installed Pi-hole in a dedicated Debian LXC with a stable LAN IP.

## What actually happens in a DNS query?

A client asks a DNS resolver which IP address corresponds to a hostname. For example, a browser might resolve a website name before starting an HTTP/TLS connection. Pi-hole receives queries as a DNS resolver and consults filtering lists:

```text
Client ── DNS A/AAAA query ──> Pi-hole
                                   ├── Blocked name? Return blocked response
                                   └── Allowed? Send/resolve using configured upstream resolver
```

- **DNS** maps domain names to resource records such as A (IPv4) and AAAA (IPv6).
- **Upstream resolver** handles queries Pi-hole does not already answer or filter.
- **Blocklist / gravity database** contains domains that Pi-hole should block according to your configured rules.
- **NXDOMAIN** can indicate that a domain name does not exist; a blocking answer may also be `0.0.0.0` depending on settings and record type.
- **DNS-based blocking limits:** DNS cannot reliably remove every advertisement (e.g. first-party ads delivered from the same domain as content).

## Actual deployment and verification

1. Created a dedicated Debian 13 LXC and reserved an address for it.
2. Installed Pi-hole natively in the LXC (**not as a Docker container**).
3. Configured an upstream resolver and additional lists; refreshed the gravity database.
4. Opened the Pi-hole admin interface and confirmed the service was running.
5. Queried Pi-hole directly from a client. A normal hostname returned a normal IP; an ad domain returned `0.0.0.0`.
6. Tested DNS through the remote-access path with Tailscale in a subsequent check.

## Commands and exactly what they verify

```bash
# Inside Pi-hole LXC
systemctl is-active pihole-FTL     # Does the DNS service run?
pihole status                      # Pi-hole's service status
ss -lntup | grep ':53'            # Is a process listening on port 53?

# On a device allowed to reach Pi-hole
dig @<pihole-ip> example.org A
dig @<pihole-ip> doubleclick.net A
```

**Do not treat a direct `dig` as proof that all device DNS traffic is being filtered.** To prove system-level usage, also inspect the client's configured DNS and Pi-hole's query log, and consider encrypted DNS bypasses.

## Key comparisons

| Technology | What it controls | What it does not necessarily control |
|---|---|---|
| Pi-hole | DNS responses for clients that send it queries | All browser content and all client traffic |
| Firewall | Packet flows based on interface, address, protocol/port and rules | Domain-level ad policy by itself |
| Tailscale | Authenticated encrypted transport and access to routed destinations | DNS content filtering by itself |
| HTTPS | Encrypts the HTTP session to a web destination | Whether a domain is blocked by Pi-hole |

## Common problems

- **No DNS response:** verify IP reachability, UDP/TCP 53, `pihole-FTL` and firewall rules.
- **Pi-hole dashboard works but clients are not blocked:** client DNS may point elsewhere, or the query/domain is not in a blocklist.
- **DNS loops:** do not configure upstream DNS so it recursively points back at Pi-hole.
- **Pi-hole down:** clients that only use Pi-hole may lose DNS even if their network connection itself remains up.
- **Remote DNS not working:** verify route approval, access policy and whether the remote client is explicitly querying Pi-hole.

## Interview statement

> “I deployed Pi-hole in a separate container to understand and control DNS resolution. I tested it rather than just assuming the web dashboard meant blocking worked: a known ad domain returned a blocking address, while a normal domain resolved correctly. I intentionally didn't replace the entire household's DNS, which kept the experiment isolated.”

**Reference:** [Official Pi-hole installation guidance](https://docs.pi-hole.net/main/basic-install/).
