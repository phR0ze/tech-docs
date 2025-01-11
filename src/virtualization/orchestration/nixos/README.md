# NixOS <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

NixOS has the ability to configure and manage a number of different containerization technologies 
directly through Nix using the `containers` directive.

Although there are a number of other solutions for managing Docker containers, LXC containers and 
various other solutions e.g. Incus, LXD, Proxmox, Portainer, Dockge, CasaOS etc... they all require 
imperative interaction, to some degree, to configure and deploy the base orchestration system as well 
as any services that you desire. With NixOS you can do this all declaratively with the Nix expression 
language. Giving you the ability to quickly reproduce your deployments.

### Quick links
* [.. up dir](../README.md)
* [Containers](#containers)
  * [nixos-container](#nixos-container)
  * [Persisting state](#persisting-state)

## Containers
One key benefit of using nix for building containers is that Nix makes your containers fully 
reproducible based on git SHA's from the original source, thus guaranteeing correctness.

**References**
* [systemd-nspawn](https://wiki.archlinux.org/title/Systemd-nspawn)
* [NixOS Containers - wiki](https://nixos.wiki/wiki/NixOS_Containers)
* [NixOS Containers - manual](https://nixos.org/manual/nixos/stable/#ch-containers)
* [Docker explained](https://github.com/paulboot/docker-explained/blob/master/Systemd%20Containers%20and%20systemd-nspawn%20Explained.md)
* [Beard hat code](https://blog.beardhatcode.be/2020/12/Declarative-Nixos-Containers.html)

### nixos-container
NixOS provides the ability, out of the box, to be able to spin up `systemd-nspawn` containers quite 
easily with `sudo nixos-container create foo`. Systemd's `machinectl` will be able to see this 
container as well.

`nspawn` containers are much like LXC containers. They are stored in `/var/lib/nixos-containers`
and have a deep integration with NixOS giving a fully declarative way to create containers based on 
NixOS. Thus you can simply build out a new `services` directory in your nixos-config and start 
creating service definitions, one for each homelab service you'd like to create.

* managable by `machinectl`
* stores files at `/var/lib/nixos-containers/<container>`
* supports persistent disk or not using `containers.<container>.ephemeral = true;`
* supports mounting files from host into container

### Persisting state
State is persisted by default. However you can disable this with the 
`containers."my-container".ephemeral = true;` flag which is a best practice as you want to manage and 
backup your state separately. As such take advantage of mounts to mount a host directory into the 
guest container where you can persist data.

In this example the `/mnt/wasabiData` path is mounted at the container's `/var/log/httpd` location. 
The host folder needs to already exist and have user permissions setup for the guest.

```nix
containers.wasabi.bindMounts = {
  "/var/log/httpd" = {
    hostPath = "/mnt/wasabiData/";
    isReadOnly = false;
  };
};
```

### Networking for reverse proxies
By default nix containers use the host network. In order to configure a container to be used for 
reverse proxies you need to configure it using the `privateNetwork = true;`. NixOS will then 
automatically create the necessary virtual devices named starting with `ve-` and followed by the 
container e.g. `ve-nextcloud`.

```nix
containers.nextcloud = {
  privateNetwork = true;
  hostAddress = "192.168.100.10";
  localAddress = "192.168.100.11";

  config = { config, pkgs, lib, ... }: {
    networking.firewall.allowedTCPPorts = [80];
    system.stateVersion = "24.05";
  };
};
```

In order to allow the container to communicate on the network or externally 
we need to configure that in the host configuration using the `networking.nat` settings.
```nix
networking.nat.enable = true;
networking.nat.internalInterfaces = [ "ve-+" ];
networking.nat.externalInterface = "enp1s0";
```

### Unprivileged containers
Drop the privileges of a container to a non-privileged user with nspawn's `-U` option using 
`containers."my-container".extraFlag = [ "-U" ];`. This option ensures that the root user inside the 
container does not have UID 0 outside the container but rather a large rando number.

### Port forwarding
In the container's configuration

```nix
containers.nextcloud = {
  forwardPorts = [
    { hostPort = 80; containerPort = 80; }
  ];
};
```

### Mount host files in container
Any container can have read only or writable access to host files via the `bindMount` command.

```nix
containers.nextcloud = {
  config = { config, pkgs, lib, ... }: {
    bindMounts = {
      "/root/data" = {
        hostPath = "/home/user1/data/";
        isReadOnly = false;
      };
    };
  };
};
```

### Nextcloud container
NixOS provides an example right out of the box on how to set up a NixOS based container for Nextcloud 
that will automatically start at boot and has its own private network subnet.

```nix
networking.nat = {
  enable = true;
  internalInterfaces = ["ve-+"];
  externalInterface = "ens3";
  # Lazy IPv6 connectivity for the container
  enableIPv6 = true;
};

containers.nextcloud = {
  autoStart = true;
  privateNetwork = true;
  hostAddress = "192.168.100.10";
  localAddress = "192.168.100.11";
  hostAddress6 = "fc00::1";
  localAddress6 = "fc00::2";
  config = { config, pkgs, lib, ... }: {

    services.nextcloud = {
      enable = true;
      package = pkgs.nextcloud28;
      hostName = "localhost";
      config.adminpassFile = "${pkgs.writeText "adminpass" "test123"}"; # DON'T DO THIS IN PRODUCTION - the password file will be world-readable in the Nix Store!
    };

    system.stateVersion = "23.11";

    networking = {
      firewall = {
        enable = true;
        allowedTCPPorts = [ 80 ];
      };
      # Use systemd-resolved inside the container
      # Workaround for bug https://github.com/NixOS/nixpkgs/issues/162686
      useHostResolvConf = lib.mkForce false;
    };
    
    services.resolved.enable = true;

  };
};
```

## Networking
* Layer 1 = Physical layer
* Layer 2 = MAC address
* Layer 3 = IP address

**References**
* [Miro's notes](https://mirosval.sk/blog/2023/nix-macvlan-networking/)

### macvlan
Creating a macvlan with;
```nix
networking = {
  useDHCP = false;
  macvlans.${app.name} = {
    interface = "${app.nic}";
    mode = "bridge";
  };
  interfaces.${app.name}.ipv4.addresses = [
    { address = "${app.ip}"; prefixLength = 32; }
  ];
  firewall.interfaces.${app.name}.allowedTCPPorts = [ 80 ];
};
```

* Creates units: macvlan0-netdev.service, network-addresses-macvlan0.service, network-setup.service, systemd-sysctl.service

**/etc/systemd/system/macvlan0-netdev.service**
```
[Unit]
After=network-pre.target sys-subsystem-net-devices-.device
Before=network-setup.service
BindsTo=sys-subsystem-net-devices-.device
Description=Vlan Interface macvlan1
PartOf=network-setup.service

[Service]
Environment="LOCALE_ARCHIVE=/nix/store/xbyg4x37l32d5py04cfhh74m23d11jfs-glibc-locales-2.38-44/lib/locale/locale-archive"
Environment="PATH=/nix/store/3nrnh8sj8yxd5wj6s5l7llxpzpi5w6p5-iproute2-6.7.0/bin:/nix/store/x1xcjw5628crkk1pwr12y7nwbzkc3969-coreutils-9.4/bin:/nix/store/i6y16f2jzcv1g1k12qdkislh3yqk2rl7-findutils-4.9.0/bin:/nix/store/11b3chszacfr9liy829xqknzp3q88iji-gnugrep-3.11/bin:/nix/store/y1y3rml47qnh0giqd32mj07qxxqy13qg-gnused-4.9/bin:/nix/store/s5dx34869kl4fd5p913j5mis32c7f98k-systemd-255.2/bin:/nix/store/3nrnh8sj8yxd5wj6s5l7llxpzpi5w6p5-iproute2-6.7.0/sbin:/nix/store/x1xcjw5628crkk1pwr12y7nwbzkc3969-coreutils-9.4/sbin:/nix/store/i6y16f2jzcv1g1k12qdkislh3yqk2rl7-findutils-4.9.0/sbin:/nix/store/11b3chszacfr9liy829xqknzp3q88iji-gnugrep-3.11/sbin:/nix/store/y1y3rml47qnh0giqd32mj07qxxqy13qg-gnused-4.9/sbin:/nix/store/s5dx34869kl4fd5p913j5mis32c7f98k-systemd-255.2/sbin"
Environment="TZDIR=/nix/store/lb1z8nckdwx4kmwy19znh1bl1p63wxhv-tzdata-2024a/share/zoneinfo"
ExecStart=/nix/store/5xzfmwpfpap0yi7db0hvbw59gri7cx4i-unit-script-macvlan1-netdev-start/bin/macvlan1-netdev-start
ExecStopPost=/nix/store/sjrvr2zkbcjkpgxmm5yq9zq2brym0v44-unit-script-macvlan1-netdev-post-stop/bin/macvlan1-netdev-post-stop
RemainAfterExit=true
Type=oneshot

[Install]
WantedBy=network-setup.service sys-subsystem-net-devices-macvlan1.device
```
Following the `ExecStart` script we find
```bash
ip link show dev "macvlan1" >/dev/null 2>&1 && ip link delete dev "macvlan1"
ip link add link "" name "macvlan1" type macvlan \
  mode bridge
ip link set dev "macvlan1" up
```

### MacVLAN/MacVTAP
MacVLAN creates virtual layer 2 devices, (i.e. have their own MAC addresses), which share the 
physical ethernet port. MacVALAN won't get DHCP addresses from the network router.

Use MacVLAN in `bridge` mode if you want your containers to act as though they are physical devices 
on the same network as your router.

Goal: each container appears on local network with its own static IP address and all containers are 
able to communicate on the network freely.
```nix
{ config, pkgs, lib, ... }:
{
  networking.defaultGateway = "192.168.0.1";
  networking.nameservers = [ ... ];
  networking.interfaces.enp3s0.ipv4.addresses = [
    { address = "192.168.0.151"; prefixLength = 24; }
  ];

  containers.test1 = {
    autoStart = true;
    ephemeral = true;
    macvlans = [ "enp3s0" ];

    config =
      { config, lib, pkgs, ... }:
      {
        networking.defaultGateway = "192.168.0.1";
        networking.nameservers = [ ... ];
        networking.interfaces.mv-enp3s0.ipv4.addresses = [
          { address = "192.168.0.152"; prefixLength = 24; }
        ];
      };
  };
  # repeat with containers.test2 and ip address 192.168.0.153, etc...
}
```

### Tun/Tap
A TAP interface is a new virtual layer 2 device like macvlan but no layer 1 attached to it. This 
virtual device is exposed in user space for any application to write data to it. Works well for VPNs. 
Or virtual point to point network connections. TUN devices are just like TAP but they operate on 
layer 3 instead of layer 2.

### MacVLAN vs Bridge
Seems they are very similar but not understanding the difference
