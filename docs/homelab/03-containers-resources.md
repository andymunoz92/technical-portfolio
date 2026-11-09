# 03 — LXC containers and resource management

[← Index](README.md) · [← Networking](02-networking.md) · [Next: Tailscale →](04-tailscale.md)

## Why containers rather than a VM for every service?

The host has only 8 GB physical RAM and a mechanical HDD. Each additional full VM would have a guest kernel and a minimum memory/OS footprint. **LXC containers share the Proxmox host's Linux kernel** and isolate processes and resources using kernel namespaces and cgroups; this lets multiple small Linux services coexist at lower memory overhead.

That does **not** mean containers are always more secure or the same as VMs. Since they share a kernel, a host-kernel vulnerability can weaken isolation. For untrusted workloads or different operating systems, a VM may be more suitable.

## Actual services

| Guest | Role | OS | Status |
|---|---|---|---|
| CT 100 | Tailscale subnet-router gateway | Debian 13 LXC | Running and remotely tested |
| Separate LXC | Pi-hole DNS filtering | Debian 13 LXC | Running and DNS-tested |
| Separate LXC | Nginx static webserver | Debian 13 LXC | Running and HTTP/SSH-tested |

The Tailscale guest was configured with **1 vCPU, 512 MiB RAM, 8 GiB disk**, with startup at boot. It was created as an **unprivileged** LXC and allowed access to the TUN device required by Tailscale. Other CT resource limits should be checked on the live server before publishing numbers.

## Key concepts

- **Proxmox node:** the physical hypervisor host.
- **Container template:** an OS root filesystem image used to create an LXC.
- **Container ID (`CTID`):** unique identifier used by Proxmox tools.
- **Root filesystem (rootfs):** container's storage location.
- **cgroups:** limit/control CPU, memory and related resources.
- **Namespaces:** isolate resources such as processes, network interfaces, and mount points.
- **Unprivileged LXC:** container root is mapped to a non-root identity on the host, reducing the consequences of some compromises.
- **TUN device:** virtual network interface for userspace tunnels; this requires careful device permissions when used inside LXC.

## Read-only Proxmox commands

```bash
pct list                    # Enumerate containers
pct status 100              # Check a specific CT
pct config 100              # Review allocation and features
pct exec 100 -- hostnamectl # Run a read-only command in CT100
pvesm status                # Storage pools and usage
free -h                     # Host memory & swap
```

**Important:** avoid making changes with `pct set`, reboot, or storage commands on a working system without capturing and reviewing the current configuration.

## Memory and swap trade-offs

- Allocated guest limits do not guarantee equal real-time use; measure observed memory consumption under load.
- Swap can prevent an immediate out-of-memory crash but is much slower on HDD and can cause severe latency.
- A single HDD provides limited random I/O and is a potential single point of failure; backups are not the same as having a second copy of the host disk.
- Avoid scheduling multiple memory-heavy guests before checking the host and existing container utilization.

## Answer to: “Why not Docker?”

> “Docker is great for packaging and deploying applications. I used Proxmox-managed LXC containers because I wanted isolated lightweight Debian environments I could administer like small Linux hosts, each with its own network address and services. I can still evaluate Docker inside an appropriately configured VM later, but I chose LXC first to stay within 8 GB of RAM.”

## Practical interview exercise

Explain how you would add a fourth monitoring service without destabilizing DNS and remote access: measure current memory, choose LXC vs. VM, define a resource ceiling, avoid port/IP conflicts, plan rollback, validate after deployment, and update documentation.
