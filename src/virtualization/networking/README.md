# Networking <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />

Virtualized networking is critical to understand for proper container and VM orchestration.

### Quick links
* [.. up dir](../README.md)
* [Networking Overview](#networking-overview)
* [Prerequisites](#prerequisites)
  * [Enable forwarding of traffic to VMs](#enable-forwarding-of-traffic-to-vms)
  * [Disable netfiltering for bridges](#disable-netfiltering-for-bridges)

### Linked pages
* [Bridge](bridge/README.md)
* [macvlan](macvlan/README.md)
* [macvtap](macvtap/README.md)
 
## Networking Overview

**References**
* [Arch Linux Wiki](https://wiki.archlinux.org/title/Network_bridge)

### Docker networking
Docker is a great place to start as it is the most popular container technology in existance and has 
a plethora of examples and information surrounding it and most of the concepts apply to other 
container technologies as well.

**Docker Networking Drivers**
| Driver    | Description
| --------- | ----------------------------------------------------------
| bridge    | Virtual switch with custom IP range with port forwarding to host capabilities
| host      | Remove network isolation and just use the host ports directly
| none      | Completely isolate a container from the host and other containers
| overlay   | Connect multiple docker daemons together
| ipvlan    | Full control over IP addressing
| macvlan   | Use when you want a unique MAC and presence on the physical lan

**References**
* [Docker macvlan](https://blog.oddbit.com/post/2018-03-12-using-docker-macvlan-networks/)

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

## Routes

**References**
* [CIDR Calculator](https://cidr.xyz/)

### Add a single route
```bash
ip route add 192.168.1.60/32 dev macvlan0
```

### Deleting a route
```bash
ip route del 192.168.1.0/24
```

<!-- 
vim: ts=2:sw=2:sts=2
-->
