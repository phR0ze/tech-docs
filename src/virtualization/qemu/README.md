# QEMU
<img align="left" width="40" height="40" src="../../data/images/logo_256x256.png">
Documenting my experience with QEMU

### Quick links
* [Overview](#overview)
  * [Install QEMU](#install-qemu)
* [QEMU Monitor](#qemu-monitor)
* [VM CRUD](#vm-crud)
  * [Create VM](#create-vm)
  * [Run VM](#run-vm)
* [Networking](#networking)
  * [Network Bridge](#network-bridge)
  * [Port forwarding](#port-forwarding)

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

# Networking
`-netdev bridge,br=br0,id=net0`
* Configures the nic frontend to use bridge `br0` and ties to backend with id `net0`

`-device virtio-net-pci,netdev=net0`
* Configures the nic backend identified by `net0` to use a specific virtual driver `virtio-net-pci`

# Network Bridge
**References**
* [Arch linux bridge](https://wiki.archlinux.org/title/Network_bridge)
A network bridge is the virtual equivalent of a physical network switch you plug your ethernet cables 
into. This allows your virtual network devices and your host machine to be part of the same network 
when connected through the network bridge virtual switch.

It's common for Docker and VirtualBox and other virtualization technologies to create network bridges 
to allow for virtual devices to communicate. You can see the devices:
```bash
$ ip link
...
4: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN mode DEFAULT group default 
    link/ether 02:42:1d:c0:d6:0f brd ff:ff:ff:ff:ff:ff
5: vboxnet0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 0a:00:27:00:00:00 brd ff:ff:ff:ff:ff:ff
```

Libvirt, which many use to manage QEMU, will also create a bridge network similar to Docker or 
VirtualBox named `virbr0` which QEMU is already setup in `/etc/qemu/bridge.conf` for users to access.

Regardless of the virtualization manager used essentially what happens is that the virtualization 
manager creates the bridge network then assigns it an IP so that the host OS can participate in the 
network and talk to the VMs then assigns the VMs tap devices that are part of the bridge network.

## Creating a bridge
Although this is automatically managed by Docker, VirtualBox, Libvirt and others we can manually 
create one as well. This guide is for NetworManager users and we'll be using the cli `nmcli`.

**Note on bridge permissions**:  
By default QEMU is configured for only a specific named bridge `virbr0` to be accessible by any user. 
You can fix this by:
* Editing `/etc/qemu/bridge.conf` and changing `virbr0` to `all` OR
* Adding a new user specific file `echo "allow all" | sudo tee /etc/qemu/${USER}.conf` OR
* But the simplest solution is to just use a bridge with the name `virbr0`

1. Create a new empty bridge named `virbr0` and set to UP
   ```bash
   $ sudo nmcli conn add type bridge con-name virbr0 ifname virbr0
   $ sudo nmcli conn up virbr0
   # OR
   $ sudo ip link add virbr0 type bridge
   $ sudo ip link set dev virbr0 up
   ```

2. Assign an IP address to the bridge `virbr0`
   ```bash
   $ sudo nmcli conn add type ethernet slave-type bridge con-name bridge-br0 ifname enp1s0 master br0
   # OR
   $ sudo ip address add 192.168.1.4/24 dev virbr0
   ```
2. Setup your bridge e.g. `virbr0` to route to your default gateway for your NIC e.g. `enp1s0`
   1. First determine your NIC's ip e.g. `192.168.1.4/24` and add it to your bridge e.g. `virbr0`
      ```bash
      $ ip a show enp1s0
      ...
      inet 192.168.1.4/24...
      $ sudo ip address add 192.168.1.4/24 dev virbr0
      ```
   2. Determine your NIC's default gateway
      ```bash
      $ ip route show dev enp1s0
      default via 192.168.1.1...
      $ sudo ip route append default via 192.168.1.1 dev virbr0
      ```

3. Now add your NIC e.g. `enp1s0` to the bridge which will have 

4. Add the following flags to you QEMU VM to expose it on the bridge
   ```
   -nic bridge,br=virbr0
   ```

You can change your qemu nic settings to use a bridge with `-nic bridge,br=br0`; however you'll 
immediately hit a `access denied by acl file` error:
```bash
$ qemu-system-x86_64 -enable-kvm -m 4G -nic bridge,br=br0,model=virtio-net-pci -drive file=nix1.qcow2,media=disk,if=virtio
access denied by acl file
qemu-system-x86_64: -nic bridge,br=br0: bridge helper failed
```

This is controlled by the setting in `/etc/qemu/bridge.conf`
```
allow virbr0
```


Boot your VM and notice that it automatically creates the `tap0` network device.
```bash
$ qemu-system-x86_64 -enable-kvm -m 4G -netdev bridge,br=br0,id=net0 -device virtio-net-pci,netdev=net0 \
    -drive file=nix1.qcow2,media=disk,if=virtio
```

## Port forwarding
This mechanism it good for connecting to a single service from the VM. However if you'd like to allow 
the VM to interact fully on your network then you'd want to configure a Bridge network for it.

Port forward VM's port 22 to host port 2222 such that any requests over host:2222 would get 
redirected to the VM on port 22.
```
-netdev user,id=net0,hostfwd=tcp::2222-:22
```

<!-- 
vim: ts=2:sw=2:sts=2
-->
