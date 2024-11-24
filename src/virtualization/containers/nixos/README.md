# NixOS <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

NixOS has the ability to configure native systemd-nspawn containers, which are running NixOS and are 
configured and managed by NixOS using the `containers` directive.

Although there are a number of other solutions for managing Docker containers, LXC containers and 
various other solutions e.g. Incus, LXD, Proxmox, Portainer, Dockge, CasaOS etc... they all require 
to some degree imperative interaction to manually configure and deploy the base orchestration system 
as well as any services that you desire. With NixOS you can do this all declaratively with the Nix 
expression language giving you the ability to quickly reproduce your deployments.

### Quick links
* [.. up dir](../README.md)
* [NixOS Containers](#nixos-containers)

## NixOS Containers

**References**
* [NixOS Containers - wiki](https://nixos.wiki/wiki/NixOS_Containers)
* [NixOS Containers - manual](https://nixos.org/manual/nixos/stable/#ch-containers)

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
