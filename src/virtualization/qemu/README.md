# QEMU <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />
Documenting my experience with QEMU

Fundamentally QEMU is awesome. CLI based, fast and reliable; but every time I try to do anything 
complicated it takes forever to figure it out. I'm coming from a VirtualBox background though which 
made everything way simpler than this.

### Quick links
* [Overview](#overview)
  * [Keyboard Shortcuts](#keyboard-shortcuts)
  * [Install QEMU](#install-qemu)
  * [QEMU Monitor](#qemu-monitor)
  * [Quickemu](#quickemu)
* [Linked Pages]()
  * [NIX QEMU](nix_qemu/README.md)
* [Parameters](#parameters)
  * [General params](#general-params)
  * [CPU/Mem params](#cpu-mem-params)
  * [Video params](#video-params)
  * [Networking params](#networking-params)
  * [Drive params](#drive-params)
  * [Window display params](#window-display-params)
* [VM Templates](#vm-templates)
  * [Create Windows 7 VM](#create-windows-7-vm)
  * [Create Windows 8 VM](#create-windows-8-vm)
  * [Create Windows 10 VM](#create-windows-10-vm)
* [VM Snapshots](#vm-snapshots)
  * [Run in Immutable mode](#run-in-immutable-mode)

## Overview
Quick EMUlator is a command line virtual machine system. It is fast, portable and has excellent guest 
support. It is the swiss army knife of virtualization. QEMU supports everything and more than other 
hypervisors. Its one downside is that its rather complicated to figure out how to do what you want to 
do and the available GUIs don't help much.

**References**
* [Archlinux QEMU](https://wiki.archlinux.org/title/QEMU)
* [Gentoo linux docs](https://wiki.gentoo.org/wiki/QEMU/Options)
* [QEMU Essential commands](https://blog.usro.net/2024/10/essential-qemu-commands-a-quick-guide-for-beginners-and-power-users/)

### Keyboard Shortcuts

* **Keyboard Capture** - press `Ctrl+Alt+G` to release the keyboard control grab. Shows this in the 
  window title bar as well as a reminder.

### Install QEMU
Install the command line version only

* **Arch Linux**
  ```bash
  $ sudo pacman -S qemu-base
  ```

* **NixOS** - see 
[nixos-confi/options/virtualization/virt-manager](https://github.com/phR0ze/nixos-config/blob/main/options/virtualization/virt-manager.nix)

### QEMU Monitor
You can access the monitor console by pressing `Ctrl+Alt+2` and return to the normal window with 
`Ctrl+Alt+1`.

You can expose the monitor over TCP with the runtime argument `-monitor 
tcp:127.0.0.1:<PORT>,server,nowait` then use netcat `nc 127.0.0.1 <PORT>` and send it commands.

### Quickemu
[Quickemu](https://github.com/quickemu-project/quickemu), available in nixpkgs, is a set of wrapper 
shell scripts that take the wildly configurable QEMU and try to do the ***right thing*** when 
creating virtual machines.

* automatically download upsteam OS isos
* automatically choose optimal configuration best suited for your computer when running the VM
* store vm and configuration anywhere
* VirGL acceleration
* USB device pass-through
* Network port forwarding

Quickemu works off the assumption that there is a lot going on in QEMU and its hard to remember. 
Quickemu will figure out a lot of it for you and store it in a config along side your `qcow2` image 
so you can refer to that.

## Parameters

### General params

| Parameters                                      | Description
| ----------------------------------------------- | ---------------------------------------------------
| `-name "Arch Linux"`                            | Set the window title of your VM
| `-enable-kvm`                                   | Enables the KVM subsystem for hardware acceleration

### CPU/MEM params

| Parameters                                      | Description
| ----------------------------------------------- | ---------------------------------------------------
| `-m 4G`                                         | Specifies 4G of RAM 
| `-cpu host`                                     | CPU model recommended for best performance
| `-smp 4`                                        | Specifies the number of cores the guest is permitted to use

**CPU Details backing selections above**
https://www.qemu.org/docs/master/system/qemu-cpu-models.html

***TL;DR*** use `-cpu host` then limit cores with `-smp NUMBER`

QEMU provides two options here: either `Host passthrough` or `Named model`.
1. `Host passthrough` is the recommended CPU config to use if live migration is not required as 
   it most accurately takes advantage of the existing hardware for maximum performance. Despite being 
   the recommendation though it is not the default.

2. `Named model` allows for specifying a particular CPU type or generic type for more isolation 
   from the host and provide live migration support.

**Default x86 CPU models** the default CPU type is the `qemu32` or `qemu64` which are guaranteed to 
work with maximum compatibility. Despite being the defaults though the community is discouraging 
their use in production for security reasons as the compatibility also means security vulnerabilities 
which make them unsuitable for production exposed systems. That said for internal testing this isn't 
a concern.

*libvirt* will choose a CPU model for you based that most closely matches your CPU's feature set.

In any case it is possible to add or remove CPU features if needed but is only needed in advanced 
cases.

Run the following to see the list if support CPUs
```bash
$ qemu-system-x86_64 -cpu ?
```

### Video params

| Parameters                                      | Description
| ----------------------------------------------- | ---------------------------------------------------
| `-vga virtio`                                   | Use improved graphics performance driver
| `-device VGA,vgamem_mb=32`                      | Increase the default 16M of video memory

### Networking params

| Parameters                                      | Description
| ----------------------------------------------- | ---------------------------------------------------
| `-nic user,model=virtio`                        | Simple pass through networking with highspeed `virttio` driver
| `-nic user`                                     | Simple pass through networking with generic driver
| `-nic bridge,br=virbr0,helper=$(type -p qemu-bridge-helper)` | Bridged networking

### Drive params

| Parameters                                      | Description
| ----------------------------------------------- | ---------------------------------------------------
| `-drive file=arch1.qcow2,media=disk,if=virtio`  | Attach your virtual disk to the guest as `/dev/vda`
| `-cdrom alpine-standard-3.8.0-x86_64.iso`       | Attach ISO as cdrom

### Window display params

| Parameters                                      | Description
| ----------------------------------------------- | ---------------------------------------------------
| `-display sdl,gl=on`                            | Use standard window for VM display with OpenGL support

## VM Templates
Documenting a few setups I can experiement with

### Create Windows 7 VM
see [Parameters](#paramters) for explanations

Windows doesn't have drivers to support Virtio 
* `-nic user` non virtio simple pass-through networking for install
* `-hda` for simply generic hdd support without advanced virtio drivers

Adding in bridge networking see [QEMU Networking >Create Network Bridge](qemu_networking/README.md#create-network-bridge)
* `-nic bridge,br=virbr0,helper=$(type -p qemu-bridge-helper)`

1. Create a new sparse qcow2 drive to install to
   ```bash
   $ cd ~/Projects/vms
   $ qemu-img create -f qcow2 win7.qcow2 40G
   ```

3. Create a new VM using the new drive and ISO
   ```bash
   $ qemu-system-x86_64 \
       -name Win7 \
       -enable-kvm \
       -m 4G -cpu host -smp 4 \
       -nic user \
       -vga virtio -display sdl,gl=on \
       -hda win7.qcow2 \
       -cdrom win7.iso
   ```

4. Walk through the Windows installation wizard
   * Configure as you like it and shutdown

5. Ensure the network bridge is started
   ```bash
   $ virsh net-start default
   ```

6. Run that same VM later
   ```bash
   $ qemu-system-x86_64 -name Win7 -enable-kvm -m 4G -cpu host -smp 4 -vga virtio \
      -nic bridge,br=virbr0,helper=$(type -p qemu-bridge-helper) \
      -display sdl,gl=on -hda win7.qcow2
   ```

### Create Windows 8 VM
see [Parameters](#paramters) for explanations

Windows doesn't have drivers to support Virtio 
* `-net nic -net user` non virtio simple pass-through networking for install
* `-hda` for simply generic hdd support without advanced virtio drivers

Adding in bridge networking see [QEMU Networking >Create Network Bridge](qemu_networking/README.md#create-network-bridge)
* `-nic bridge,br=virbr0,helper=$(type -p qemu-bridge-helper)`

1. Create a new sparse qcow2 drive to install to
   ```bash
   $ cd ~/Projects/vms
   $ qemu-img create -f qcow2 win8.qcow2 40G
   ```

3. Create a new VM using the new drive and ISO
   ```bash
   $ qemu-system-x86_64 \
       -name Win8 \
       -enable-kvm \
       -m 4G -cpu host -smp 4 \
       -net nic -net user \
       -vga virtio -display sdl,gl=on \
       -hda win8.qcow2 \
       -cdrom win8.iso
   ```

4. Walk through the Windows installation wizard
   * Configure as you like it and shutdown

5. Ensure the network bridge is started
   ```bash
   $ virsh net-start default
   ```

6. Run that same VM later
   ```bash
   $ qemu-system-x86_64 -name Win8 -enable-kvm -m 4G -cpu host -smp 4 -vga virtio \
      -nic bridge,br=virbr0,helper=$(type -p qemu-bridge-helper) \
      -display sdl,gl=on -hda win8.qcow2
   ```

### Create Windows 10 VM
see [Parameters](#paramters) for explanations

Quickemu will automate downloading all necessary drivers for a good experience as well as creating a 
configuration file for you to then launch the VM with these drivers.

1. Create windows 10 configuration and download needed drivers to `windows-10`
   ```bash
   $ quickget windows 10
   ```

2. 
**Config**
```bash
guest_os="windows"
iso="win10.iso"
fixed_iso="virtio-win.iso"
disk_img="win10.qcow2"
tpm="on"
secureboot="off"
```

## VM Snapshots

### Run in Immutable mode
Running a VM in immutable mode will discard all changes when the VM is powered off just by running 
with the `-snapshot` parameter. However you can still save the state as a snapshot if you run the 
`commit all` from the monitor console.

<!-- 
vim: ts=2:sw=2:sts=2
-->
