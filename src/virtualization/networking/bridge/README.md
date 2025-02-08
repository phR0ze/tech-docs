# Network Bridge <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

A network bridge is a virtual network device that behaves like a switch allowing virtual devices to 
be connected to your host and local network via virtual ethernet devices (VETH). `macvlan` provides 
similar functionality with a slightly simpler approach by handling the creation of the virtual 
ethernet devices for you so you don't have to create the VETH devices.

**References**
* [Arch Linux Wiki](https://wiki.archlinux.org/title/Network_bridge)

There are a number of ways to create bridged networks and many processes are deprecated or only used 
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
* [Red Hat Virtual Interfaces](https://developers.redhat.com/blog/2018/10/22/introduction-to-linux-interfaces-for-virtual-networking#bridge)

### IP address of bridge
A virtual bridge interface will essentially be replacing your existing NIC configuration, using the 
same IP address, DNS etc.. and adds on the functionality of allowing virtual devices.

Normally this means your new `bridge0` virtual switch gets your IP address and your physical NIC 
`eno1` won't have an address assigned to it anymore eventhough its being used by the bridge.

Note: `VirtualBox` gets around this concept of the network bridge replacing the host's typical 
networking (i.e. having to have an IP address assigned to the bridge and disable the original NIC's 
configuration) by having their own networking kernel module that circumventing the host operating 
system's network stack. [see VirtualBox manual](https://www.virtualbox.org/manual/ch06.html). For 
everyone else you need to use a standard bridge that replaces your nic as explained above.

**References**
* [Redhad docs](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/configuring_and_managing_networking/configuring-a-network-bridge_configuring-and-managing-networking)

## Creating a bridge

### on NixOS

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

### via iproute2
![Bridge](../../../data/images/bridge1.png)

1. Create network bridge
   ```bash
   ip link add br0 type bridge
   ip link set eth0 master br0
   ip link set tap1 master br0
   ip link set tap2 master br0
   ip link set veth1 master br0
   ```
2. Remove network bridge
   ```bash
   sudo ip link br0 down
   sudo ip link delete br0 type bridge
   ```

This creates a bridge named `br0` and adds a physical device `eth0` and sets two TAP devices `tap1`, 
`tap2` and a VETH device `veth1` as its slaves as shown in the diagram above.
the bridge.

<!-- 
vim: ts=2:sw=2:sts=2
-->
