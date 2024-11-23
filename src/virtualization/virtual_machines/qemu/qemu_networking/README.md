# QEMU Networking <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

### Quick links
* [Network Bridge](#network-bridge)
  * [Network Bridge Overview](#network-bridge-overview)
  * [Create Network Bridge](#create-network-bridge)
  * [Allow Denied issue](#allowed-denied-issue)
* [Port forwarding](#port-forwarding)

## Windows Virtio Support
The [Virtio Win Guest Tools](https://github.com/virtio-win/virtio-win-guest-tools-installer) project 
provides a pre-built installer that you can use inside the guest to provide support for the Virtio 
drivers. Once installed you can reboot using the `virtio` backend driver.

## Network Bridge
**References**
* [Arch linux bridge](https://wiki.archlinux.org/title/Network_bridge)
* [How to use bridged networking](https://linuxconfig.org/how-to-use-bridged-networking-with-libvirt-and-kvm)

### Network Bridge Overview
A network bridge is the virtual equivalent of a physical network switch you plug your ethernet cables 
into. This allows your virtual network devices and your host machine to be part of the same network 
when connected through the network bridge virtual switch.

It's common for Docker and VirtualBox and other virtualization technologies to create network bridges 
to allow for virtual devices to communicate. You can see the devices:
```bash
$ ip link show type bridge
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

### Create Network Bridge
Bridging the Guest network with the host allows for connecting back and forth between the two as per 
usual and a much better networking experience.

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

### Allow Denied issue
If you run your vm with the `-nic bridge` and immediately get a `access denied by acl file` error 
then you'll need to allow access to the bridge. This is controlled by the setting `allow virbr0` in 
`/etc/qemu/bridge.conf` which I do in my `nixos-config/options/virtualization/virt-manager.nix` 
configuration.

Note: this solved the isseu on Arch Linux but on NixOS I additionally had to use `qemu-bridge-helper` 
to solve the issue.

## Port forwarding
This mechanism it good for connecting to a single service from the VM. However if you'd like to allow 
the VM to interact fully on your network then you'd want to configure a Bridge network for it.

Port forward VM's port 22 to host port 2222 such that any requests over host:2222 would get 
redirected to the VM on port 22.
```
-netdev user,id=net0,hostfwd=tcp::2222-:22
```

