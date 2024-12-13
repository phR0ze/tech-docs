# Virtual Machines <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />

### Quick links
- [.. up dir](../README.md)
- [Overview](#overview)
  - [Virt Manager vs Libvirt vs QEMU vs KVM](#virt-manager-vs-libvirt-vs-qemu-vs-kvm)
- [Virtual Box](#virtual-box)
  - [USB Access in VM](#usb-access-in-vm)

### Linked pages
- [libvirt](libvirt/README.md)
- [QEMU](qemu/README.md)
- [VirtManager](virt_manager/README.md)

### Linked pages
- [QEMU](qemu/README.md)

## Overview

### Virt Manager vs libvirt vs QEMU vs KVM
Libvirt is a collection of software that provides a convenient way to manage virtual machines and 
other virtualization functionality. `libvirtd` provides a daemon and API library and `virsh` provides 
a command line utility. The project's primary goal is to provide a single way to manage multiple 
different virtualization providers/hypervisors regardless of the implementation e.g. `KVM`, `Xen`, 
`LXC`, `VirtualBox` etc... `Virt Manager` was created to provide a GUI interface for libvirt.

KVM isn't fully functional on its own. It only provice a kernel API to userspace. The most popular 
way to make KVM fully functional is to use QEMU which provides the rest of the pieces and a command 
line interface. libvirt in turn uses QEMU to then manage KVM. `qemu-kvm` was a fork of QEMU with KVM 
acceleration built in that was merged back in as `qemu -enable-kvm`. So your tools stack will look 
something like.

`Virt Manager` => `libvirtd` => `QEMU` => `KVM` 

## Virtual Box
There are a number of solutions in the Linux world for Virtual Machine creation and managment, but 
I've found for a number of years that Virtual Box is the simplest cross platform way to deal with 
Virtual Machines.

### USB Access in VM
[Accessing host USB devices in guest](https://wiki.archlinux.org/index.php/VirtualBox#Accessing_host_USB_devices_in_guest)
requires that your user be part of the vboxusers group.

```bash
# Check which groups your user is in
$ groups

# Add your use to the vboxusers group
$ sudo usermod -a -G vboxusers <USER>
```
