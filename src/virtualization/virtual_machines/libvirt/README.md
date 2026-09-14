# libvirt <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Libvirt is a toolkit for managing virtualization platforms. It provides a stable C API, a daemon
(`libvirtd`), and a command line client (`virsh`) that together offer a single, hypervisor-agnostic
way to create and manage virtual machines, networks and storage regardless of the underlying
virtualization technology e.g. `QEMU/KVM`, `Xen`, `LXC`, `VirtualBox` etc...

### Quick links
- [.. up dir](..)
- [Overview](#overview)
  - [Install libvirt](#install-libvirt)
  - [libvirt vs QEMU](#libvirt-vs-qemu)
- [virsh Basics](#virsh-basics)
  - [Connecting](#connecting)
  - [Domain Management](#domain-management)
  - [Editing a Domain](#editing-a-domain)
  - [Snapshots](#snapshots)
- [Networking](#networking)
  - [Default Network](#default-network)
  - [Bridged Network](#bridged-network)
- [Storage Pools](#storage-pools)
- [Permissions](#permissions)

## Overview

**References**
* [libvirt documentation](https://libvirt.org/docs.html)
* [virsh command reference](https://libvirt.org/manpages/virsh.html)
* [Archlinux libvirt wiki](https://wiki.archlinux.org/title/Libvirt)

### Install libvirt
* **Arch Linux**
  ```bash
  $ sudo pacman -S libvirt virt-install qemu-base dnsmasq
  $ sudo systemctl enable --now libvirtd
  ```

* **NixOS** - see
[nixos-config/options/virtualization/virt-manager](https://github.com/phR0ze/nixos-config/blob/main/options/virtualization/virt-manager.nix)
which enables `virtualisation.libvirtd.enable = true;`

### libvirt vs QEMU
It's easy to conflate the two since libvirt spends most of its time managing QEMU, but they solve
different problems and sit at different layers of the stack.

`Virt Manager` => `libvirtd` => `QEMU` => `KVM`

|                   | libvirt                                                                                     | QEMU                                                                                                    |
| ----------------- | ------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------|
| **What it is**    | A management layer and daemon (`libvirtd`) with a CLI (`virsh`) and API                     | The actual emulator/hypervisor that runs the guest CPU, devices and I/O                                 |
| **Scope**         | Hypervisor agnostic - drives QEMU/KVM, Xen, LXC, VirtualBox etc... through the same API     | Specific to QEMU only, no concept of other backends                                                     |
| **Configuration** | Declarative XML per domain/network/storage-pool, persisted and reloaded across host reboots | Long, imperative command line flags passed once at process launch, e.g. `qemu-system-x86_64 -m 4G ...`  |
| **Persistence**   | VM, network and storage definitions are stored by `libvirtd` and survive reboots            | No built-in persistence - you own remembering/reconstructing the launch command yourself                |
| **Networking**    | Manages bridges, NAT and DHCP for you via `virsh net-*` and `dnsmasq`                       | Requires you to build the bridge/tap devices and pass `-nic`/`-netdev` manually                         |
| **Automation**    | Good fit for orchestration tools since it exposes a stable API                              | Better fit for quick one-off, throwaway VMs where you want direct control of every flag                 |
| **GUI**           | `virt-manager`/`GNOME Boxes` talk to `libvirtd`, giving you a GUI on top                    | No native GUI - QEMU only exposes the monitor console                                                   |

In short: QEMU is the engine, libvirt is the dashboard and control system wrapped around it.
When you need fine-grained one-off control of specific QEMU flags (custom display devices,
experimental parameters), drop down to [QEMU](../qemu/README.md) directly. When you want
persistent, easily managed VMs with networking and storage handled for you, use libvirt.

## virsh Basics

### Connecting
`virsh` connects to the local system QEMU/KVM driver by default. You can also target a session
(user, unprivileged) instance or a remote host.

```bash
$ virsh -c qemu:///system list --all   # system wide, requires libvirtd group membership
$ virsh -c qemu:///session list --all  # per-user, unprivileged
```

### Domain Management
A `domain` in libvirt terms is a single virtual machine instance.

| Command | Description
| ------- | -----------
| `virsh list --all`                | List all domains including stopped ones
| `virsh start <name>`              | Start a defined domain
| `virsh shutdown <name>`           | Request a graceful ACPI shutdown
| `virsh destroy <name>`            | Force power off (equivalent to pulling the plug)
| `virsh undefine <name>`           | Remove the domain definition (does not delete disks)
| `virsh dominfo <name>`            | Show basic info about a domain
| `virsh console <name>`            | Attach to the domain's serial console

### Editing a Domain
Every domain is backed by an XML definition. Editing it directly with `virsh edit` validates the
schema on save and applies safely, which is much less error prone than hand editing the file on
disk.

```bash
$ virsh edit <name>
```

### Snapshots
```bash
$ virsh snapshot-create-as <name> snap1 "before upgrade"
$ virsh snapshot-list <name>
$ virsh snapshot-revert <name> snap1
```

## Networking

### Default Network
libvirt ships a default NAT network backed by the `virbr0` bridge and a `dnsmasq` instance for
DHCP/DNS, see [QEMU Networking](../qemu/README.md#networking) for how QEMU attaches to it.

```bash
$ virsh net-start default
$ virsh net-autostart default   # start automatically on libvirtd startup
$ virsh net-list
```

### Bridged Network
For a VM to be a full peer on your LAN (rather than NAT'd behind the host), define a bridged
network in libvirt so `virsh` manages the bridge lifecycle rather than doing it by hand.

```bash
$ virsh iface-bridge eno1 br0
```

## Storage Pools
libvirt manages disk images through storage pools rather than raw paths, which lets `virt-manager`
and other tooling discover and allocate volumes consistently.

```bash
$ virsh pool-list --all
$ virsh vol-list default
```

## Permissions
To run `virsh`/`virt-manager` without `sudo` your user needs to be part of the `libvirt` group.

```bash
$ sudo usermod -a -G libvirt <username>
```
