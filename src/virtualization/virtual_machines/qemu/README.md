# QEMU <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Quick EMUlator is a command line virtual machine system. It is fast, portable and has excellent guest 
support. It is the swiss army knife of virtualization. QEMU supports everything and more than other 
hypervisors. Its one downside is that its rather complicated.

### Quick links
- [.. up dir](../README.md)
- [Overview](#overview)
  - [Keyboard Shortcuts](#keyboard-shortcuts)
  - [Install QEMU](#install-qemu)
  - [QEMU Monitor](#qemu-monitor)
  - [Quickemu](#quickemu)
- [Getting Started](#getting-started)
  - [Boot from ISO](#boot-from-iso)
  - [Windows Virtio Support](#windows-virtio-support)
  - [Create Windows 7 VM](#create-windows-7-vm)
  - [Create Windows 8 VM](#create-windows-8-vm)
  - [Create Windows 10 VM](#create-windows-10-vm)
- [Video params](#video-params)
  - [QXL VGA](#qxl-vga)
- [Parameters](#parameters)
  - [General params](#general-params)
  - [CPU/Mem params](#cpu-mem-params)
  - [Networking params](#networking-params)
  - [Drive params](#drive-params)
  - [Window display params](#window-display-params)
- [Networking](#networking)
  - [Create Network Bridge](#create-network-bridge)
  - [Access denied by acl file](#access-deniced-by-acl-file)
  - [Port forwarding](#port-forwarding)
- [VM Snapshots](#vm-snapshots)
  - [Run in Immutable mode](#run-in-immutable-mode)

### Linked pages
  - [Quickemu](quickemu/README.md)
  - [NIX QEMU](nix_qemu/README.md)
  - [SPICE](spice/README.md)

## Overview

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

## Getting Started
There is so much to QEMU and you might just need a simple place to start. This section is for that.

### Boot from ISO
I often want to test out bootable ISOs

1. Create a new sparse qcow2 drive to install to
   ```bash
   $ cd ~/Projects/nixos-config
   $ qemu-img create -f qcow2 nixos.qcow2 20G
   ```

3. Create a new VM using the new drive and ISO
   ```bash
   $ qemu-system-x86_64 \
       -name NixOS \
       -enable-kvm \
       -m 4G -cpu host -smp 4 \
       -nic user \
       -vga virtio -display sdl,gl=on \
       -hda nixos.qcow2 \
       -cdrom result/iso/cyberlinux-25.11.20250809.85dbfc7-x86_64-linux.iso
   ```

### Windows Virtio Support
The [Virtio Win Guest Tools](https://github.com/virtio-win/virtio-win-guest-tools-installer) project 
provides a pre-built installer that you can use inside the guest to provide support for the Virtio 
drivers. Once installed you can reboot using the `virtio` backend driver.

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


## Video params
There are a lot of emulated display devices in QEMU. Typical recommendations are use:
1. `virtio vga` or `virtio gpu` with guest drivers
2. `qxl vga` with guest drivers

First of all all display options have a short `-vga <NAME>` and a long `-device <NAME,vgamem_mb=32` 
way of being specified for additional options.

**References**
* [Display devices in QEMU](https://www.kraxel.org/blog/2019/09/display-devices-in-qemu/)

| Parameters              | VGA | BIOS | UEFI | Purpose
| ----------------------- | --- | ---- | ---- | ---------------------------------------------------
| `-vga vga`              | yes | yes  | yes  | Compability
| `-vga virtio`           | yes | yes  | yes  | Modern and built for VMs
| `-device virtio-gpu-pci`| no  | no   | yes  | Same as virtio but no VGA support saves 8mb memory
| `-vga qxl`              | yes | yes  | yes  | Slighly dated

### Standard VGA
`-vga std` or `-device VGA,vgamem_mb=32` is the default display device on x86 hardware. It provides a 
full VGA compatibility and support for simple linear framebuffer. It is the best choice for 
compatibiltiy. Its default is `16mb` of video ram.

The UEFI setup allows to choose the display resolution which OVMF will use to initialize the display 
at boot. Press ESC at the tianocore splash screen to enter setup, then go to `Device Manager` > 
`OVMG` > `Platform Configuration`

### Virtio VGA
`-vga virtio` or `-device virtio-vga,virgl=on,gl=on` is a modern virtio-based display device designed 
for Virtual Machines. It comes with VGA compatibility mode. You need a guest OS driver to make full 
use of the device. The device provides optional hardware-assisted opengl accelerations support using 
the `virgl=on` option. This device has no dedicated video memory and doesn't support the option.

There is a `virtio gpu` option which is the same thing without the VGA support and saves 8mb of 
memory. Probably the better option if you have guest drivers that is. Additionally you can run either 
one as a separate process with:
```
qemu \
 -chardev socket,id=vgpu,path=vgpu.sock \
 -device vhost-user-vga,chardev=vgpu \
```

```
-vga none -device virtio-vga-gl -display gtk,gl=on,show-cursor=on  
```

### QXL VGA
`-vga qxl` or `-device qxl-vga` is a slightly dated device desigend for virtual machines. You need a 
guest driver to make full use of this device. This device has support for 2D acceleration. Modern 
display devices though use 3D engines for everything and so this doesn't help. The same thing happens 
on the software side where they all use 3D not 2D. Spice and qxl offload 2D acceleration to the spice 
client, but this is complex and increasingly useless. `-device qxl` is the same but lacks vga support.

Show options: `qemu-system-x86_64 -device qxl-vga,?`
* `vgamem_mb` default is 16, change it to 32 for better performance
```
-vga none -device qxl-vga,id=video0,vgamem_mb=64,bus=pci.0,addr=0x2

-vga qxl -global qxl-vga.vgamem_mb=32 

-vga qxl -global qxl-vga.ram_size=134217728 -global qxl-vga.vram_size=134217728 -global qxl-vga.vgamem_mb=32 

-vga qxl -global qxl-vga.vram_size=9437184 -device
qxl,id=video1,vram_size=67108864,bus=pci.0,addr=0x8
```

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

## Networking

### Create Network Bridge
A network bridge is the virtual equivalent of a physical network switch you plug your ethernet cables 
into. This allows your virtual network devices and your host machine to be part of the same network 
when connected through the network bridge virtual switch.

It's common for Docker and VirtualBox and other virtualization technologies to create network bridges 
to allow for virtual devices to communicate. Libvirt, which many use to manage QEMU, will also create 
a similar bridge network which QEMU is already setup in `/etc/qemu/bridge.conf` for users to access.

Regardless of the virtualization manager used essentially what happens is that the virtualization 
manager creates the bridge network then assigns it an IP so that the host OS can participate in the 
network and talk to the VMs then assigns the VMs tap devices that are part of the bridge network.

1. Start the `virbr0` network bridge using `virsh`
   ```bash
   $ virsh net-start default
   ```

2. List bridge networks out to verify it exists
   ```bash
   $ virsh net-list
   $ ip link show type bridge
   ```

3. Use `qemu-bridge-helper` which automatically runs with root privileges to setup the `tap0` and 
   attach to the brige and pass it back to QEMU and allows QEMU to not have to be run as root. 
   ```
   -nic bridge,br=virbr0,helper=$(type -p qemu-bridge-helper)
   ```
4. Notice that we now have a new interface attached to `virbr0`. Inside the VM it should have 
   automatically been assigned an IP e.g. `192.168.122.76`
   ```bash
   $ ip link show master virbr0
   11: tap0 ...
   ```

5. Add a physical interface to the bridge which is consumed by the bridge so use a spare e.g. `eno1`
   ```bash
   $ sudo ip link set eno1 up
   $ sudo ip link set eno1 master virbr0
   ```

### Access denied by acl file
If you run your vm with the `-nic bridge` and immediately get a `access denied by acl file` error 
then you'll need to allow QEMU access to the bridge. This is controlled by the setting `allow virbr0` 
in `/etc/qemu/bridge.conf` which I do in my `nixos-config/options/virtualization/virt-manager.nix` 
configuration.

Note: this solved the issue on Arch Linux but on NixOS I additionally had to use `qemu-bridge-helper` 
to solve the issue.

### Port forwarding
This mechanism it good for connecting to a single service from the VM. However if you'd like to allow 
the VM to interact fully on your network then you'd want to configure a Bridge network for it.

Port forward VM's port 22 to host port 2222 such that any requests over host:2222 would get 
redirected to the VM on port 22.
```
-netdev user,id=net0,hostfwd=tcp::2222-:22
```

## VM Snapshots

### Run in Immutable mode
Running a VM in immutable mode will discard all changes when the VM is powered off just by running 
with the `-snapshot` parameter. However you can still save the state as a snapshot if you run the 
`commit all` from the monitor console.
