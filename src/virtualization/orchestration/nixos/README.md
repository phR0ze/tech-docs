# NixOS <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

NixOS has the ability to configure and manage a number of different containerization technologies 
directly through Nix using the `containers` directive.

Although there are a number of other solutions for managing Docker containers, LXC containers and 
various other solutions e.g. Incus, LXD, Proxmox, Portainer, Dockge, CasaOS etc... they all require 
to some degree imperative interaction to manually configure and deploy the base orchestration system 
as well as any services that you desire. With NixOS you can do this all declaratively with the Nix 
expression language giving you the ability to quickly reproduce your deployments.

### Quick links
* [.. up dir](../README.md)
* [Containers](#containers)

## Containers
The benefits of Nix are that you can build a fully reproducible, Docker compatible, image using 
git SHA's from the original source that will guarantee correctness and incorporate it into your 
system deployment in such a way that it becomes an automated service rather than a manually deployed 
and managed entity separate from your main configuration.

**References**
* [systemd-nspawn](https://wiki.archlinux.org/title/Systemd-nspawn)
* [NixOS Containers - wiki](https://nixos.wiki/wiki/NixOS_Containers)
* [NixOS Containers - manual](https://nixos.org/manual/nixos/stable/#ch-containers)
* [Docker explained](https://github.com/paulboot/docker-explained/blob/master/Systemd%20Containers%20and%20systemd-nspawn%20Explained.md)

### nixos-container
NixOS provides the ability, out of the box, to be able to spin up a native container quite easily 
with `sudo nixos-container create foo`. Systemd's `machinectl` will be able to see this container as 
well. So what is this actually doing? NixOS is using `systemd-nspawn` to create these containers.

These `nspawn` containers are much like LXC containers. They are stored in `/var/lib/nixos-containers`
and have a deep integration with NixOS giving a fully declarative way to create containers based on 
NixOS. Thus you can simply build out a new `services` directory in your nixos-config and start 
creating service definitions, one for each homelab service you'd like to create.

* managable by `machinectl`
* stores files at `/var/lib/nixos-containers/<container>`
* supports persistent disk or not using `containers.<container>.ephemeral = true;`
* supports mounting files from host into container

### Private networking
NixOS will automatically create the virtual devices necessary to make this work using the service 
name e.g. `ve-nextcloud`. 

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
networking.nat.externalInterfaces = "enp1s0";
```

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
