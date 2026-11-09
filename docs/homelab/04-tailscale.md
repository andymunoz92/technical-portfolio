# 04 — Private remote access with Tailscale

[← Index](README.md) · [← Containers](03-containers-resources.md) · [Next: Pi-hole →](05-pihole.md)

## Problem solved

Proxmox management is on the home LAN, but I wanted to administer it while traveling or on a public Wi-Fi connection. Instead of directly publishing port 8006 on my router, I deployed a **Tailscale subnet-router LXC**. It acts as a gateway between devices authenticated to my tailnet and explicitly advertised private destinations.

## Terms that matter

- **Overlay network:** a virtual network built over an underlying IP network.
- **WireGuard tunnel:** encrypted tunnel technology used by Tailscale for device-to-device connections.
- **Tailnet:** the Tailscale network of authorized devices and users.
- **Subnet router:** a Tailscale node that forwards packets to private IPs of services that do not run Tailscale themselves.
- **Exit node:** routes a client's general internet traffic through a remote node; **not** the same function as a subnet router and **not** the function used here.
- **IP forwarding:** kernel capability that allows a Linux node to relay traffic not addressed to itself.
- **Route advertisement/approval:** advertising a route on the gateway and approving it through tailnet administration are both necessary for intended access; access-control policy also applies.
- **TUN device:** tunnel interface used by Tailscale; LXC must be configured to make it available.

## Actual sequence

1. Created a small **unprivileged Debian 13 LXC**, CT 100, with 1 vCPU, 512 MiB RAM and an 8 GiB disk.
2. Enabled the appropriate container features and `/dev/net/tun` access for Tailscale; exact commands depend on the Proxmox/LXC version.
3. Installed/authenticated the Tailscale client inside the CT.
4. Enabled **IPv4 forwarding** so it could function as a subnet router.
5. Advertised selected private `/32` routes (single private hosts, rather than the entire /24 LAN), approved the routes in Tailscale admin settings, and connected the remote MacBook client.
6. Verified Proxmox access from an external Wi-Fi network. Later tested private DNS access to Pi-hole.

## Useful read-only diagnostics

```bash
# In Tailscale LXC:
tailscale status
tailscale ip -4
sysctl net.ipv4.ip_forward
ip route
ls -l /dev/net/tun

# On Proxmox host:
pct status 100
pct config 100
```

## Example: route advertisement (configuration change; illustrative only)

```bash
# Replace values with approved destination IPs before running.
# Run INSIDE the Tailscale subnet-router LXC as an authorized administrator.
sudo tailscale set --advertise-routes=<management-ip>/32,<dns-ip>/32
```

The above is a teaching illustration. The live set of advertised/approved routes should be inspected before modification. A `/32` route authorizes reachability to one address at the routing layer; application permissions and tailnet ACLs are separate controls.

## Validate the actual paths

- **Overlay:** does `tailscale status` show the gateway connected?
- **Routing:** is IPv4 forwarding active and is the target `/32` approved?
- **Transport:** can the remote device reach the required port (e.g. TCP 8006)?
- **Application:** does the Proxmox HTTPS login load and authenticate? (Never capture credentials in documentation.)
- **DNS:** does a remote `dig @<dns-ip> example.org` return a response? This is a direct DNS test, not proof the whole client device is using Pi-hole by default.

## Security note

Tailscale authentication, routes and ACLs reduce unwanted exposure, but **do not automatically harden a weak admin password or an unpatched service**. Keep Proxmox patched, use least-privilege access, and protect tailnet accounts with strong authentication. Do not claim a full security audit was performed.

## Interview statement

> “I deployed Tailscale inside its own LXC as a subnet router, enabled IP forwarding, and approved specific host routes rather than routing my entire LAN. I validated remote access from a separate public Wi-Fi network. I also understood that subnet routing is different from an exit node: it grants access to selected private destinations rather than routing all internet traffic through home.”

**Reference:** [Tailscale subnet-router documentation](https://tailscale.com/docs/features/subnet-routers).
