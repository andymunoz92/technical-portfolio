# 01 — Preparing hardware and installing Proxmox VE

[← Index](README.md) · [Next: Networking →](02-networking.md)

## Goal

Turn an older general-purpose desktop into an always-on virtualization host, without requiring new hardware.

| Component | Observed setup |
|---|---|
| Host | HP Pavilion 550-127c |
| CPU | AMD A10-7800 |
| RAM | 8 GB installed (less usable to services) |
| Disk | 1 TB HDD |
| OS foundation | Debian 13 (Trixie) |
| Hypervisor | Proxmox VE 9.2, installed as Debian packages |
| Firmware configuration | UEFI boot; virtualization enabled; Secure Boot disabled during this build |
| Network | Wired Ethernet (Wi-Fi removed/disabled for the host) |

## Actual chronological decisions

1. **Inspect the hardware and BIOS:** Confirm the CPU architecture supports 64-bit Linux and check CPU virtualization settings. An AMD platform typically calls this **AMD-V / SVM**. **Why:** hardware virtualization allows KVM-based VMs; Linux containers share the host kernel and do not require separate guest kernels.
2. **Prepare boot media:** Several direct Proxmox installer images did not appear as bootable in the host's UEFI selection. This was a **specific installation-media/firmware issue**, not a universal Proxmox defect.
3. **Choose a workaround:** Install Debian 13 from a netinst USB that boots correctly. Then add the appropriate Proxmox packages using the official Debian-install method.
4. **Configure network and packages:** Install Proxmox and configure the bridge/network explicitly. Installing Proxmox on top of Debian is supported but requires more manual configuration than the normal Proxmox ISO installer.
5. **Verify the management plane:** Reboot and confirm that `pveproxy`/`pvedaemon` are running and that the web UI is available over HTTPS on port 8006 from the permitted LAN.

**Accuracy note:** The original exact terminal command history for the installation was not preserved in the available records. The next commands are checks and examples, not a verbatim transcript.

## Inspecting a running host (read-only)

```bash
pveversion -v                  # Proxmox components and versions
uname -r                       # Running kernel
hostnamectl                    # Hostname, OS and kernel information
lscpu                          # CPU architecture and virtualization flags
free -h                        # RAM and swap
lsblk -o NAME,SIZE,FSTYPE,MOUNTPOINT
systemctl is-active pveproxy pvedaemon
```

## Understanding the stack

- **BIOS/UEFI:** firmware initializes hardware and selects the boot loader. UEFI/Legacy issues can stop an installer before Linux starts.
- **Debian:** operating-system foundation with the Linux kernel, package manager, services, networking, and storage.
- **Proxmox VE:** management platform on top of Debian for KVM virtual machines, LXC containers, storage, networking, backups, and a web UI.
- **KVM VM:** emulates a machine running its own guest kernel; stronger kernel isolation but more overhead.
- **LXC container:** isolates Linux userspace processes while sharing the host kernel; lighter for small services.

## How I explain the installation challenge in an interview

> “I was repurposing older desktop hardware, and the Proxmox installer media wasn't being recognized correctly in UEFI mode. I tested the boot path separately, installed Debian 13 from a working netinst image, and then installed Proxmox VE as packages. I validated the result by checking services, the running kernel, and the remote management interface. It taught me to separate boot-media problems from operating-system and application problems.”

## Questions to be able to answer

1. What is the difference between Debian and Proxmox VE?
2. What role does UEFI play before the OS loads?
3. Why can an ISO fail to boot even if it is correctly downloaded?
4. What changes when installing Proxmox on Debian instead of using its ISO?
5. Why are backups essential before modifying boot settings or disks?

**Reference:** [Proxmox installation documentation](https://github.com/proxmox/pve-docs/blob/master/pve-installation.adoc).
