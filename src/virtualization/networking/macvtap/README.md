# macvtap <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

`macvtap` is a newer networking driver that simplifies virtualized networking by replacing the 
venerable `tun/tap` and/or virtual bridge macvlan combination to provide the same functionality 
without requiring the bridge.

Use `macvtap` if you want a MAC address and presence on the phyical lan. This is a great fit for an 
application that you want to interact with the other networked devices in a way that simulates a full 
physical device e.g. unique MAC and address.

### Quick links
* [.. up dir](../README.md)
* [Overview](#overview)
  * [Create via Nix](#create-via-nix)
  * [Create via iproute2](#create-via-iproute2)
  * [Create via docker](#create-via-docker)

## Overview

**References**
* [MacVTAP guide](https://gist.github.com/lukasnellen/d597f52441d6ca65ea0f0c79c9c170e7)
* [Red Hat virtual interfaces](https://developers.redhat.com/blog/2018/10/22/introduction-to-linux-interfaces-for-virtual-networking)
* [Sirchia nixos-config](https://github.com/sirchia/NixOS/blob/main/etc/nixos/container-services/traefik.nix)
* [Macvlan vs Macvtap](https://suhu0426.github.io/Web/Presentation/20150120/index.html)

Before macvtap you could accomplish the same thing by creating a virtual bridge then using macvlans 
to connect to the virtual bridge. This had the undesirable effect though of changing your primary 
interface from your NIC to the virtual bridge which was in effect the same thing but clutters up the 
interface list and wasn't intutitive for those not in the know.

![bridge macvlan](../../../data/images/bridge_macvlan.png)
![macvlan](../../../data/images/macvlan1.png)

**macvtap modes**
* In `bridge` mode it can fully communicate with the parent interface. Usually what you want

### Create via Nix
Nix doesn't have support for configuring virtualized networking on its own

```nix
systemd.services."podman-network-${app.name}" = {
  path = [ pkgs.podman pkgs.iproute2 ];
  serviceConfig = {
    Type = "oneshot";
    RemainAfterExit = true;
    ExecStop = [
      "podman network rm -f ${app.name}"
      "ip route del ${app.nic.ip2} dev ${app.name} &>/dev/null || true"
    ];
  };
  script = ''
    if ! podman network exists ${app.name}; then
      podman network create -d macvlan --subnet=${app.nic.subnet} --gateway=${app.nic.gateway} -o parent=${app.nic.name} ${app.name}
    fi

    # Setup host to container access by adding an explicit route
    ip route add ${app.nic.ip2} dev ${app.name} &>/dev/null || true
  '';
};
```

### Create via bash
Note in order to get access from the host to the guests directly you need to additionally add a 
macvlan interface for the host to then see the macvtap interfaces. This is only necessary for the 
host and guest to communicate. The guest and the rest of the LAN can already communicate.

![macvlan](../../../data/images/macvlan1.png)

1. Create `macvlan0@ens18` with the steps below
   ```bash
   sudo ip link add macvlan0 link ens18 type macvlan mode bridge
   sudo ip addr add 192.168.1.60/32 dev macvlan0
   sudo ip link set macvlan0 up
   sudo ip route add 192.168.1.61/32 dev macvlan0
   sudo ip route add 192.168.1.62/32 dev macvlan0
   ```
2. Remove when done 
   ```bash
   $ ip route del 192.168.1.60
   $ ip route del 192.168.1.61
   $ ip link set macvlan0 down
   $ ip link delete macvlan0
   ```

### Create via docker
* `subnet` should be your network subnet
* `gateway` should be your network gateway
* `-o parent=ens18` should be your device's NIC found with `ip a`
* `my-macvlan` is the name of your new macvlan network

1. Create macvlan
   ```bash
   docker network create -d macvlan \
     --subnet=192.168.1.0/24 \
     --gateway=192.168.1.1 \
     -o parent=ens18 \
     my-macvlan
   ```

2. Inspect docker network
   ```bash
   docker network inspect my-macvlan
   ```

3. Attach a container to the macvlan network
   1. Deploy a test container, needs priviledged for ping
      ```bash
      docker run --privileged --rm -dit \
        --name alpine0
        --network my-macvlan \
        --ip=192.168.1.60 \
        alpine:latest \
        /bin/sh
      ```
   2. View how the container sees its networking
      ```bash
      docker exec -it alpine0 ip a
      docker exec -it alpine0 ip route
      ```
4. Create a local macvlan for the host to see the containers
   ```bash
   sudo ip link add macvlan0 link ens18 type macvlan mode bridge
   sudo ip addr add 192.168.1.60/32 dev macvlan0
   sudo ip link set macvlan0 up
   sudo ip route add 192.168.1.61/32 dev macvlan0
   sudo ip route add 192.168.1.62/32 dev macvlan0
   ```
