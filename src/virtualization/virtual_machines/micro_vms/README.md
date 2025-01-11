# Micro VMs <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

The Micro VM project provides a path for managing production NixOS virtual machines via:
* Isolated /nix/store for the guest NixOS
* Individual VM configuration updates
* Modern Rust Hypervisor support designed for `virtio`

### Quick links
- [.. up dir](../README.md)

## Overview
Virtual Machines can be a better fit for some use cases than containers. My primary use case is for a 
full remote (i.e. via SPICE) environment in which to test system configuration or to use as a node in 
other containerization solutions such as Kubernetes, Docker swarm, Portainer or others. This provides 
an excellent decoupling of the host system from guest systems for simpler management.

**References**
- [MicroVM - Astro](https://astro.github.io/microvm.nix/)
- [Virtual Networking](https://developers.redhat.com/blog/2018/10/22/introduction-to-linux-interfaces-for-virtual-networking#macvlan)

NixOS is the ideal OS for this scenario with declarative syntax and robust rollback. It is also 
excellently integrated for one off QEMU virtual machine tests of your configuration. Unfortunately 
the project hasn't extended this management into production Virtual Machines with separate isolated 
configurations. Astro's Micro VM project was built to solve this.

`microvm.nix` creates virtual machine disk images and runner script packages for the entries of the 
`nixosConfigurations` section of a `flake.nix` file.

### Creation
VMs are only created in `/var/lib/microvms` per directory if they don't already exist. Thus you'll 
get a VM declaratively built the first time but after that you need to imperatively update them 
directly.

## Prepare for hosting MicroVMs
In order to configure your system to host MicroVMs we need to make a few changes.

### Add MicroVM host module
1. Add the `microvm.nix` flake as an input
2. Setup `microvm` to follow your nixpkgs
3. Add the `microvm.nixosModules.host` to your system modules
4. Optionally add autostart on the host for your guest MicroVMs

**Example**
```nix
{
  inputs.nixpkgs.url = "github:nixos/nixpkgs/nixos-unstable";
  inputs.microvm.url = "github:astro/microvm.nix";
  inputs.microvm.inputs.nixpkgs.follows = "nixpkgs";

  outputs = { self, nixpkgs, microvm }: {
    nixosConfigurations.server1 = nixpkgs.lib.nixosSystem {
      system = "x86_64-linux";
      modules = [
        microvm.nixosModules.host
        {
          # try to automatically start these MicroVMs on bootup
          microvm.autostart = [ "my-microvm" ];
        }
      ];
    };
  };
}
```

### Configure networking

