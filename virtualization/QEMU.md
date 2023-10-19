<img align="left" width="40" height="40" src="../../images/logo_256x256.png">QEMU
====================================================================================================
Documenting my experience with QEMU
<br><br>

### Quick links
* [.. up dir](..)
* [Overview](#overview)
  * [Install QEMU](#install-qemu)
* [QEMU Monitor](#qemu-monitor)
* [VM CRUD](#vm-crud)
  * [Create VM](#create-vm)
  * [Run VM](#run-vm)

# Overview
Quick EMUlator is a command line virtual machine system. It is fast, portable and has excellent guest 
support. It is the swiss army knife of virtualization. QEMU supports everything and more than other 
hypervisors.

**References**
* [Archlinux QEMU](https://wiki.archlinux.org/title/QEMU)

## Keyboard Shortcuts

### Keyboard Capture
Press `Ctrl+Alt+G` to release the keyboard control grab. Shows this in the window title bar as well 
as a reminder.

## Install QEMU
Install the command line version only

```bash
$ sudo pacman -S qemu-base
```

## QEMU Monitor
You can access the monitor console by pressing `Ctrl+Alt+2` and return to the normal window with 
`Ctrl+Alt+1`.

### Connect via TCP
You can expose the monitor over TCP with the runtime argument `-monitor 
tcp:127.0.0.1:<PORT>,server,nowait` then use netcat `nc 127.0.0.1 <PORT>` and send it commands.

# VM CRUD

## Create VM
| Parameters                                      | Description
| ----------------------------------------------- | ---------------------------------------------------
| `-enable-kvm`                                   | Enables the KVM subsystem for hardware acceleration
| `-m 2048`                                       | Specifies 2G of RAM 
| `-nic user,model=virtio`                        | Add virtual nic with high speed virtio driver
| `-drive file=alpine.qcow2,media=disk,if=virtio` | Attach your virtual disk to the guest as `/dev/vda`
| `-cdrom alpine-standard-3.8.0-x86_64.iso`       | Attach ISO as cdrom
| `-sdl`                                          | Use standard graphical output window

1. Download a test image
   1. Navigate to [arch linux mirror](http://mirrors.acm.wpi.edu/archlinux/iso/latest/)
   2. Download the latest `iso`

2. Create a new sparse qcow2 drive to install to
   ```bash
   $ qemu-img create -f qcow2 arch1.qcow2 20G
   ```

3. Create a new VM using the new drive and ISO
   ```bash
   $ qemu-system-x86_64 \
       -enable-kvm \
       -m 4G \
       -nic user,model=virtio \
       -drive file=arch1.qcow2,media=disk,if=virtio \
       -cdrom archlinux-2023.09.01-x86_64.iso
   ```

## Run VM
Pro tip to name your VMs so you can find them later

```bash
$ qemu-system-x86_64 -enable-kvm -m 4G -nic user,model=virtio -drive file=arch1.qcow2,media=disk,if=virtio
```

## Snapshots in QEMU

### Run in Immutable mode
Running a VM in immutable mode will discard all changes when the VM is powered off just by running 
with the `-snapshot` parameter. However you can still save the state as a snapshot if you run the 
`commit all` from the monitor console.

<!-- 
vim: ts=2:sw=2:sts=2
-->
