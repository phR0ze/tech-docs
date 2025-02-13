# Network Bridge <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

A network bridge is a virtual network device that behaves like a switch. Use a bridge when you want 
to establish communication channels between VMs, containers, your host and the rest of the LAN.

There are a number of ways to create a bridge and many processes are deprecated or only used 
by niche segments of the community.

### Quick links
* [.. up dir](../README.md)
* [Overview](#overview)
  * [IP address of bridge](#ip-address-of-bridge)
* [Creating a bridge](#creating-a-bridge)
  * [on NixOS](#on-nixos)
  * [on Ubuntu](#on-ubuntu)
  * [via-iproute2](#via-iproute2)

## Overview

**References**
* [Bridged networking](https://code.lardcave.net/2019/07/20/1/)
* [Arch Linux Wiki](https://wiki.archlinux.org/title/Network_bridge)
* [Red Hat Virtual Interfaces](https://developers.redhat.com/blog/2018/10/22/introduction-to-linux-interfaces-for-virtual-networking#bridge)

### IP address of bridge
A virtual bridge interface will essentially be replacing your existing NIC configuration, using the 
same IP address, DNS etc.. and adds on the functionality of allowing virtual devices.

Normally this means your new `br0` virtual switch gets your IP address and your physical NIC 
`eno1` won't have an address assigned to it anymore eventhough it's the one being used for the 
actual physical transport.

Note: `VirtualBox` gets around the bridge replacing the host's primary NIC's networking by having
their own networking kernel module that circumventing the host operating system's network stack. [see 
VirtualBox manual](https://www.virtualbox.org/manual/ch06.html). So their virtual bridge doesn't have 
the same direct impact that the standard Linux virtual bridge does. So if you not using Virtual Box 
and are using the standard tools then you need to replace your primary nic's functionality with the 
bridge.

**References**
* [Redhad docs](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/configuring_and_managing_networking/configuring-a-network-bridge_configuring-and-managing-networking)

## Creating a bridge
![Bridge](../../../data/images/bridge1.png)

### on NixOS
There are 3 parts to the configuration. First create a network bridge device and bind it to your 
primary NIC. Second create tap devices attached to the bridge for qemu. This can be done as part of 
the QEMU setup. Finally configure the host's firewall to allow packets to move across the bridge.

**References**
* [NixOS containers bridge](https://nixos.wiki/wiki/NixOS_Containers)

```nix
# Create the bridge attached to the physical interface ens18
networking.useDHCP = false;
networking.bridges."br0".interfaces = [ "ens18" ];

# Configure Static IP bridge
networking.interfaces."br0".ipv4.addresses = [ 
  { address = "192.168.1.40"; prefixLength = 24; }  
];
networking.defaultGateway = "192.168.1.1";
```

2. Create a macvlan called `host` for the host to communicate on the bridge with virtual devices.
   ```nix
   networking.macvlans."host" = {
     interface = "br0";
     mode = "bridge";
   };
   ```

3. Optionally configure the `host` macvlan with a static IP
   ```nix
   networking.interfaces."host".ipv4.addresses = [
     { address = "192.168.1.50"; prefixLength = 32; }
   ];
   ```

4. Optionally configure the `host` macvlan with DHCP
   ```nix
   networking.interfaces."host".useDHCP = true;
   ```

### via iproute2
If not using NixOS the modern way to manually create a bridge is with the `iproute2` utilities the 
provide control and monitoring for various aspects of networking.

The following creates a virtual bridge named `br0` using the physical link `eth0`. By adding the 
`eth0` to the bridge your allowing the virtualized traffice to run over that phyiscal link.

1. Create network bridge
   ```bash
   ip link add br0 type bridge
   ip link set eth0 master br0
   ip link set tap1 master br0
   ip link set tap2 master br0
   ip link set veth1 master br0
   ```

2. Create a macvlan called `host` for the host to communicate on the bridge with virtual devices.
   ```bash
   sudo ip link add link "br0" name "host" type macvlan mode bridge
   sudo ip addr add 192.168.1.60/32 dev "host"
   sudo ip link set dev "host" up
   sudo ip route add 192.168.1.61/32 dev "host"
   ```
2. Remove when done 
   ```bash
   $ ip route del 192.168.1.60
   $ ip route del 192.168.1.61
   $ ip link set macvlan0 down
   $ ip link delete macvlan0
   ```


2. Remove network bridge
   ```bash
   sudo ip link br0 down
   sudo ip link delete br0 type bridge
   ```

<!-- 
vim: ts=2:sw=2:sts=2
-->
