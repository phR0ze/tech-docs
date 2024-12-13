# Podman <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

### Quick links
* [Overview](#overview)
  * [Podman vs Docker](#podman-vs-docker)
  * [Systemd Security Hardening](#systemd-security-hardening)
  * [Compose2Nix](#compose2nix)
  * [Install and configure in NixOS](#install-and-configure-in-nixos)
* [Networking](#networking)
  * [Static IP for Container](#static-ip-for-container)

## Overview

### Podman vs docker
Podman is essentially just an implementation of docker that doesn't have the commonly complained 
about shortcomings. Podman was built to seamlessly replace Docker in development flows.

|                | Docker                         | Podman
| -------------- | ------------------------------ | ----------------------------------
| Daemon         | Uses docker daemon             | Daemonless architecture
| Root           | Runs containers as root only   | Runs containers as root or non-root 
| Images         | Can build containers images    | Uses Buildah to build container images
| Platform       | Monolithic                     | Different binaries
| Docker swarm   | yes                            | no
| Docker compose | yes                            | yes
| Cross platform | Linux, macOS, Windows          | Linux, macOS, Windows (with WSL)

### Systemd Security Hardening
Containers and systemd only provide as much security as you apply yourself. Start with systemd's 
tooling e.g `systemd-analyze security podman-stirling-pdf`.

### Compose2Nix
Compose2Nix converts Docker Compose files into nix `oci-container` configs. The tool also creates all 
networks and volumes that are part of the compose project. 

Note: provides some great insights into how to set things up but you can get a cleaner configuration 
by hand if your willing to put in a little work.

**References**
* [Compose2Nix github](https://github.com/aksiksi/compose2nix)

1. Install via shell
   ```bash
   nix shell github:aksiksi/compose2nix
   ```
2. Convert compose file
   ```bash
   compose2nix -project stirling-pdf docker-compose.yml
   ```

### Install and configure in NixOS
Provides `podman` and `podman-compose`

```nix
{pkgs, ...}:
{
  virtualisation.podman = {
    enable = true;
    dockerCompat = true; # provide docker alias
    extraPackages = [
      pkgs.podman-compose
    ];

    # Allows docker containers to refer to each other by name
    defaultNetwork.settings.dns_enabled = true;

    # Removes dangling containers and images that are not being used. It won't remove any volumes by default
    autoPrune = {
      enable = true;
      dates = "weekly";
      # Removes stuff older than 24h and doesn't have the label important
      flags = [
        "--filter=until=24h"
        "--filter=label!=important"
      ];
    };
  };
}
```

### Declarative docker image deployment

**Nginx test**
```nix
{config, ...}:
{
  virtualisation.oci-containers.containers."nginx" = {
    image = "docker.io/nginx:alpine";
    environmentFiles = [
      config.age.secrets.secret1.path
    ];
  };
}
```

**Echo service**
```nix
{
  virtualisation.oci-containers.containers."echo-http-service" = {
    image = "hashicorp/http-echo:latest";
    extraOptions = ["-text='Hello, World!'"];
    ports = ["5678:5678"];
  };
}
```

### Declarative docker image building
```nix
{
  inputs.nixpkgs.url = "github:nixos/nixpkgs/nixos-22.05";
  outputs = { self, nixpkgs };
  let
    pkgs = nixpkgs.legacyPackages.x86_64-linux;
  in
  {
    my-image = pkgs.dockerTools.buildLayeredImage {
      name = "my-container";
      tag = "latest";
      contents = {
        pkgs.hello
      };
      config.Cmd = [ "hello" ];
    };
  };
}
```

```bash
nix build .#my-image
ls -lah result
docker load < result
docker run my-container
```

## Networking

### Static IP for Container
There are a number of ways to surface your container services. We can use `macvlan` networking to 
assign our container an IP address on our LAN and allow it to participate as a first class citizen on 
the LAN; however this poses some dangers as the container is running potentially untrusted code and 
has supply chain weaknesses and a rather large attack surface. If the container were compromised it 
is then in the heart of your LAN with access to all systems. That said a static IP associated with a 
container can be very convenient. One way we can get both isolation and a static IP is to use the 
standard docker bridge networking with a dhcp internal address and then a separate host `macvlan` per 
service with the host macvlan having an assigned IP.

**Application specific static IP with isolation e.g. Stirling PDF**
1. Create a host based macvlan for your application
   1. Determine your host's primary NIC e.g. `ens18`
      ```bash
      $ ip a
      1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
          link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
          inet 127.0.0.1/8 scope host lo
             valid_lft forever preferred_lft forever
      2: ens18: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
          link/ether bc:24:11:9f:c8:58 brd ff:ff:ff:ff:ff:ff
          altname enp0s18
          inet 192.168.1.50/24 scope global ens18
             valid_lft forever preferred_lft forever
      ```
   2. Create your app specific macvlan, name can only be about 14 chars
      ```bash
      $ sudo ip link add name stirling-pdf link ens18 type macvlan mode bridge
      $ sudo ip addr add 192.168.1.60/32 dev stirling-pdf
      $ sudo ip link set stirling-pdf up
      ```
2. Now start a container and forward a port to your new macvalan IP
   ```nix
   virtualisation.oci-containers.containers.pdf = {
     image = "docker.io/frooodle/s-pdf:latest";
     ports = [ "192.168.1.60:80:8080" ];
   };
   ```

### NixOS notes
NixOS allows for advanced declarative networking to allow containers to communicate with each other.

```nix
{lib, ...}:
{
  virtualisation.oci-containers.containers."echo-http-service" = {
    image = "hashicorp/http-echo:latest";
    extraOptions = ["-text='Hello, World!'" "--network=web"];
    ports = ["5678:5678"];
  };

  # Create any networks on boot
  system.activationScripts.createPodmanNetworkWeb = lib.mkAfter ''
    if ! /run/current-system/sw/bin/podman network exists web; then
      /run/current-system/sw/bin/podman network create web
    fi
  '';
}
```


<!-- 
vim: ts=2:sw=2:sts=2
-->
