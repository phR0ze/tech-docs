# Virtualization
<img align="left" width="48" height="48" src="../data/images/logo_256x256.png">
Documenting various virtualization technologies

### Quick links
* [Overview](#overview)
* [ProxMox](#prox-mox)
* [Virtual Box](#virtual-box)
  * [USB Access in VM](#usb-access-in-vm)

## Overview
### Libvirt
Libvirt is a collection of software that provides a convenient way to manage virtual machines and 
other virtualization functionality. `libvirtd` provides a daemon and API library and `virsh` provides 
a command line utility. The projects primary goal is to provide a single way to manage multiple 
different virtualization providers/hypervisors.

* QEMU
* KVM








## ProxMox
ProxMox Virtual Environment is an open source server virtualization management solution based on 
QEMU/KVM and LXC. Users can manage virtual machines, containers, highly available clusters, storage 
and networks via a web interface or CLI.

**References**
* [ProxMox VE linux course](https://www.youtube.com/watch?v=LCjuiIswXGs)

**Features**
* Web UI
* Virtualization platform
* Super low resource overhead
* Manage VMs as well as Containers
* Self contained full solution

Need to research this. Seems like a nice way to manage VMs and containers

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

## Virt Manager

### VDA hard disk
Virt Manager seems to create `vda` devices by default rather than `sda` devices.


### Release keyboard capture
Press `Ctrl+Alt`

<!-- 
vim: ts=2:sw=2:sts=2
-->
