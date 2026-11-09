# 02 — Linux networking: Ethernet, bridge, static IP, gateway

[← Index](README.md) · [← Host](01-host-installation.md) · [Next: Containers →](03-containers-resources.md)

## Design goal

Maintain predictable connectivity for the Proxmox host and separately addressed LXC services. Keep the host on **wired Ethernet** rather than rely on Wi-Fi association or roaming. An internal Linux bridge lets the host and containers use the same physical Ethernet connection.

## Anatomy of the network

```text
Internet
  │
Home router / default gateway
  │ physical Ethernet cable
  ▼
NIC (eno1) ── Linux bridge (vmbr0) ── host management interface
                         ├── veth link to Tailscale LXC
                         ├── veth link to Pi-hole LXC
                         └── veth link to Nginx LXC
```

- **`eno1`**: physical Ethernet interface. The NIC talks to the switch/router at the data-link layer.
- **`vmbr0`**: software bridge, behaving like a small Ethernet switch. It forwards frames based on MAC addresses among the host, containers, and the attached physical NIC.
- **`veth`**: virtual Ethernet pair that links a container's virtual NIC to the host-side bridge.
- **IP address / subnet mask:** identifies a host in a local IP network. `/24` represents a 255.255.255.0 network mask; `/32` matches exactly one IPv4 destination.
- **Default gateway:** the next hop for traffic outside directly connected networks.
- **DHCP reservation vs. static configuration:** a reservation tells a DHCP server to lease a particular address to a device; a static configuration is set directly on the OS. Coordinate them to avoid conflicts.

## What was implemented

The host's bridge had a stable private IPv4 address and default gateway. The host's wired `eno1` was connected to `vmbr0`. Service containers received stable distinct addresses. The host's Wi-Fi was disabled/removed from the intended topology.

> Public examples use placeholders; the private interview notes retain the original address map.

## Read-only verification commands

```bash
ip -br link                       # Interface names and up/down state
ip -br addr                       # Assigned IPv4/IPv6 addresses
ip route                          # Routes including default gateway
bridge link                       # Ports connected to Linux bridges
cat /etc/network/interfaces       # Debian/Proxmox interface config
ping -c 2 <gateway-ip>            # Gateway reachability (if ICMP allowed)
getent hosts debian.org           # DNS name resolution test
```

## Separate L2, L3, L4, and L7 in troubleshooting

| Layer | Question | Example evidence |
|---|---|---|
| L2: link | Is the NIC up and the bridge connected? | `ip -br link`, `bridge link` |
| L3: network | Does the host have an IP and route to the destination? | `ip route`, `ping` |
| L4: transport | Is the TCP/UDP service reachable? | `ss -lntup`, `nc -vz <ip> 22` |
| L7: application | Does the expected protocol behave correctly? | `curl -I`, `dig`, SSH login |

A successful `ping` proves neither DNS nor web-server functionality. A successful DNS lookup proves neither service availability nor that the route is protected. Test the actual protocol.

## Why I chose stable addresses

A stable host management address prevents browser bookmarks and SSH aliases from breaking after a reboot. Stable DNS and web-server addresses make services easier to locate and troubleshoot. For a production deployment, I'd also maintain an address inventory and monitor for IP collisions.

## Interview explanation

> “I connected the physical Ethernet interface to a Proxmox Linux bridge. That bridge gives my containers normal LAN connectivity without buying a separate network card for each guest. I also configured stable addresses so SSH, DNS, and management URLs didn't move around. When something failed, I checked link state, addressing, routing, listening ports, and then the application itself.”

## Practice prompts

- Why is `vmbr0` a bridge rather than a router?
- What breaks if you configure the gateway on the wrong interface?
- What is the difference between `/24` and `/32`?
- Why can an IP be reachable while a hostname fails?
- When should a host use DHCP, reservation, or static IP configuration?
