# VirtManager

### Quick links
* [Overview](#overview)

## Overview
Virtual Machine Manager is a GUI for managing virtual machines through libvirt. It primarily targets 
KVM VMs but also works well for XEN and LXC. Embedded VNC and SPICE client viewers present full 
graphical consoles to the guest machines.

* `virt-install` is a command line tool that provides and easy way to provision operating systems into 
virtual machines.

* `virt-viewer` is a lightweight UI interface for interacting with the graphical display of 
virtualized guest OS. It can display VNC or SPICE, and uses libvirt to lookup the graphical 
connection details.

* `virt-clone` is a command line tool for cloning existing inactive guests. It copies the disk 
images, and defines a config with new name, UUID and MAC address pointing to the copied disks.

* `virt-xml` is a command line tool for easily editing libvirt domain XML using virt-install’s 
command line options.

* `virt-bootstrap` is a command line tool providing an easy way to setup the root file system for 
libvirt-based containers.

## Create a Virtual Machine
1. Click the `Create a new virtual machine` button
2. Choose ISO or CDROM
   1. Select `Local install media (ISO image or CDROM)` and click `Forward`
   2. Select your ISO image
   3. Uncheck `Automatically detect from the installation media / source`
   4. Enter `NixOS` and select `NixOS Unstable` and click `Forward`
3. Choose Memory and CPU  settings
   1. Set Memory to `4096`
   2. Set CPU to `4`









## Connect to a Virtual Machine
You can connect to a virtual machine with virt manager directly or with virt-viewer. virt-viewer is a 
slimmed down UI for the guest without all the extra VM management controls.

```bash
$ virt-viewer <guest-machine>
```
