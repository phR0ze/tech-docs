# Network Bridge
A network bridge is a virtual network device that behaves like a switch allowing virtual devices to 
be connected to your network and behave as if they were real devices. 

**References**
* [Arch Linux Wiki](https://wiki.archlinux.org/title/Network_bridge)

There are a number of ways to create bridged networks and many processes are deprecated or only used 
by niche segments of the community.

### Quick links
* [Overview](#overview)
* [Prerequisites](#prerequisites)
  * [Disable netfiltering for bridges](#disable-netfiltering-for-bridges)
* [Creating a bridge](#creating-a-bridge)
  * [via nm-applet](#via-nm-applet)
  * [via nmcli](#via-nmcli)
  * [via-iproute2](#via-iproute2)

## Overview

### Terms
* Controller - refers to the bridge itself
* Ports - refer to the devices the bridge is connecting
* Bind - process of connecting devices to the bridge
* STP (Spanning Tree Protocol) - is only useful for large complicated networks
* Autoconnect

### IP address of bridge
A virtual bridge interface will essentially be replacing your existing NIC configuration, using the 
same IP address, DNS etc.. and adds on the functionality of allowing virtual devices 

Note: `VirtualBox` gets around this concept of the network bridge replacing the host's typical 
networking (i.e. having to have an IP address assigned to the bridge and disable the original NIC's 
configuration) by having their own networking module that exchanges network packets directly, 
circumventing the host operating system's network stack. [see VirtualBox manual](https://www.virtualbox.org/manual/ch06.html)

**References**
* [Redhad docs](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/configuring_and_managing_networking/configuring-a-network-bridge_configuring-and-managing-networking)

## Prerequisites

### Enable forwarding of traffic to VMs
1. Edit the `/etc/sysctl.d/10-vms.conf`
2. Add the following:
   ```
   net.ipv4.ip_forward = 1
   ```

### Disable netfiltering for bridges
[netfilter is currently enabled on bridges by default](https://bugzilla.redhat.com/show_bug.cgi?id=512206#c0).
This is unneeded additional overhead that can be confusing when trouble shooting. The libvirt team 
recommends disabling it for all bridge devices.

Add a sysctl entry to disable this e.g:
1. Edit the `/etc/sysctl.d/10-bridges.conf`
2. Add the following:
   ```
   net.bridge.bridge-nf-call-ip6tables = 0
   net.bridge.bridge-nf-call-iptables = 0
   net.bridge.bridge-nf-call-arptables = 0
   ```
3. Create a udev rule `/etc/udev/rules/99-bridges.rules` to apply the sysctl settings when the bridge module is loaded
   ```
   ACTION=="add", SUBSYSTEM=="module", KERNEL=="bridge", RUN+="/sbin/sysctl -p /etc/sysctl.d/10-bridges.conf"
   ```
4. You can test that the sysctl config took affect after starting up the bridge with
   ```bash
   $ sudo sysctl -a | grep bridge
   net.bridge.bridge-nf-call-arptables = 0
   net.bridge.bridge-nf-call-ip6tables = 0
   net.bridge.bridge-nf-call-iptables = 0
   ```

## Creating a bridge

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
