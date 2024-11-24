# Network Bridge <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />

A network bridge is a virtual network device that behaves like a switch allowing virtual devices to 
be connected to your network and behave as if they were real devices. 

**References**
* [Arch Linux Wiki](https://wiki.archlinux.org/title/Network_bridge)

There are a number of ways to create bridged networks and many processes are deprecated or only used 
by niche segments of the community.

### Quick links
* [.. up dir](../README.md)
* [Overview](#overview)
* [Prerequisites](#prerequisites)
  * [Disable netfiltering for bridges](#disable-netfiltering-for-bridges)
* [Creating a bridge](#creating-a-bridge)
  * [on NixOS](#on-nixos)
  * [on Ubuntu](#on-ubuntu)
  * [via nm-applet](#via-nm-applet)
  * [via nmcli](#via-nmcli)
  * [via-iproute2](#via-iproute2)

## Overview

### IP address of bridge
A virtual bridge interface will essentially be replacing your existing NIC configuration, using the 
same IP address, DNS etc.. and adds on the functionality of allowing virtual devices.

Normally this means your new `bridge0` virtual switch gets your IP address and your physical NIC 
`eno1` won't have an address assigned to it anymore eventhough its being used by the bridge.

Note: `VirtualBox` gets around this concept of the network bridge replacing the host's typical 
networking (i.e. having to have an IP address assigned to the bridge and disable the original NIC's 
configuration) by having their own networking module that exchanges network packets directly, 
circumventing the host operating system's network stack. [see VirtualBox manual](https://www.virtualbox.org/manual/ch06.html)

**References**
* [Redhad docs](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/configuring_and_managing_networking/configuring-a-network-bridge_configuring-and-managing-networking)

## Prerequisites

### Enable forwarding of traffic to VMs
[Updated for NixOS](https://github.com/phR0ze/nixos-config/blob/62850b8b1772c952104c3f07d26d9515fc52b1bf/modules/boot/kernel.nix#L8)

```nix
boot.kernel.sysctl = {
  "net.ipv4.ip_forward" = 1;                  # Enable ipv4 forwarding for running containers
};
```

### Disable netfiltering for bridges
[netfilter is currently enabled on bridges by default](https://bugzilla.redhat.com/show_bug.cgi?id=512206#c0).
This is unneeded additional overhead that can be confusing when trouble shooting. The libvirt team 
recommends disabling it for all bridge devices.

```nix
boot.kernel.sysctl = {
  "net.bridge.bridge-nf-call-ip6tables" = 0;
  "net.bridge.bridge-nf-call-iptables" = 0;
  "net.bridge.bridge-nf-call-arptables" = 0;
};
```

You can verify they took affect by running
```bash
$ sudo sysctl -a | grep bridge
net.bridge.bridge-nf-call-arptables = 0
net.bridge.bridge-nf-call-ip6tables = 0
net.bridge.bridge-nf-call-iptables = 0
```

## Creating a bridge

### on NixOS

**References**
* [NixOS containers bridge](https://nixos.wiki/wiki/NixOS_Containers)

```nix
networking = {
  bridges.br0.interfaces = [ "eth0s31f6" ]; # Adjust interface accordingly
  
  # Get bridge-ip with DHCP
  useDHCP = false;
  interfaces."br0".useDHCP = true;

  # Set bridge-ip static
  interfaces."br0".ipv4.addresses = [{
    address = "192.168.100.3";
    prefixLength = 24;
  }];
  defaultGateway = "192.168.100.1";
  nameservers = [ "192.168.100.1" ];
};

containers.<name> = {
  privateNetwork = true;
  hostBridge = "br0"; # Specify the bridge name
  localAddress = "192.168.100.5/24";
  config = { };
};
```

### on Ubuntu
Physical NICs can't be shared amongst multiple systems without first virtualizing it.

1. Install `openvswitch-switch`
   ```bash
   $ sudo apt install openvswitch-switch
   ```
2. Edit `sudo vim /etc/netplan/50-cloud-init.yaml` and change it to
   ```yaml
   network:
     version: 2
     ethernets:
       ens18:
         dhcp4: false
     bridges:
       bridge0:
         interfaces: [ens18]
         addresses: [192.168.1.208/24]
         routes:
           - to: default
             via: 192.168.1.1
         nameservers:
           addresses:
             - 1.1.1.1
             - 8.8.8.8
         parameters:
           stp: true
           forward-delay: 4
         dhcp4: no
   ```
   Note:
   * Use the same IP address for your bridge as your NIC already is using
3. Apply the new netplan configuration
   1. Run: `sudo netplan apply`
   2. Note that `ip a` shows the NIC as not having an address and a new `bridge0` as having the 
      address instead.

### via nm-applet
**References**
* [Setup bridge vai network manager](https://www.happyassassin.net/posts/2014/07/23/bridged-networking-for-libvirt-with-networkmanager-2014-fedora-21/)

1. Create a new bridge connection profile
   1. Right click on the network tray icon and choose `Edit Connections...`
   2. Click the little plus icon bottom left to `Add a new connection`
   3. Choose `Virtual > Bridge` and click `Create...` 
   4. Set the `Connection name` to `Bridge`
   5. Set the `Interface name` to `virbr0` useful for QEMU which defaults to this bridge name
   6. Uncheck the `Enable STP (Spanning Tree Protocol)` checkbox to remove the 30sec network start delay
   7. Switch to the `IPv6` tab and switch `Method` to `Disabled`
2. Add bridged connections
   1. Switch to the `Bridge` and click the `Add` button under `Bridged connections`
   2. Leave the `Ethernet` selection then click `Create...`

### via nmcli
This is TBD. I never finished figuring this out as VirtualBox is just so much easier

1. Create a new empty bridge named `virbr0` and set to UP
   ```bash
   $ sudo nmcli con add type bridge con-name virbr0 ifname virbr0 stp no
   $ sudo nmcli con add type bridge con-name virbr0 ifname virbr0 stp no ipv4.method auto ipv6.method disabled connection.autoconnect yes
   ```

2. Add the physical NIC to the bridge
   ```bash
   $ nmcli con add type bridge-slave ifname enp1s0 master virbr0 con-name virbr0-enp1s0

# use as port of other device
   $ nmcli con modify virbr0 ipv4.method disabled ipv6.method disabled connection.autoconnect yes
   $ nmcli con modify bridge0 connection.autoconnect-slaves 1
   ```

3. Enable the new bridge interface
   ```bash
   $ sudo nmcli con down enp1s0
   $ sudo nmcli con up virbr0
   ```

2. Assign an IP address to the bridge
   ```bash
   $ sudo nmcli conn modify virbr0 ipv4.addresses IP_ADDRE
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

### via iproute2
* [iproute2 guide](https://superuser.com/questions/1204229/bridge-virtual-interface-into-physical-network)


<!-- 
vim: ts=2:sw=2:sts=2
-->
