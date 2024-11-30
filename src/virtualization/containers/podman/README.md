# Podman
<img align="left" width="48" height="48" src="../data/images/logo_256x256.png">

### Quick links
* [Overview](#overview)
  * [Podman vs Docker](#podman-vs-docker)
  * [Install and configure in NixOS](#install-and-configure-in-nixos)

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

### Install and configure in NixOS
Provides `podman` and `podman-compose`

```nix
{pkgs, ...}:
{
  virtualisation = {
    podman = {
      enable = true;
      dockerCompat = true; # provide docker alias

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
  };
  environment.systemPackages = with pkgs; [
    podman-compose
  ];
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

## Agenix


## Networking in Podman
NixOS allows for advanced declarative networking to allow containers to communicate with each other.

```nix
{lib, ...}:
{
  virtualisation.oci-containers.containers."echo-http-service" = {
    image = "hashicorp/http-echo:latest";
    extraOptions = ["-text='Hello, World!'" "--network=web"];
    ports = ["5678:5678"];
  };

  # Create anynetworks on boot
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
