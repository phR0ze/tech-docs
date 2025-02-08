# Build VM <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />

NixOS has a mechanism built into it for generating a virtual machine based on any desired 
configuration e.g. `nix build .#nixosConfigurations.vm.config.system.build.vm`.

### Quick links
* [.. up dir](..)
* [Overview](#overview)

## Overview
The NixOS `config.system.build.toplevel` option is the topmoust level of the NixOS evalution and it's 
what is eventually evaluated to build your system. The NixOS module system, which lives in 
`nixos/lib` is responsible for evaluating the module system.

**References**
* [NixOS build vm docs](https://nixos.wiki/wiki/NixOS:nixos-rebuild_build-vm)

## Configuring VM resources
By default the VM is configured to have 1 CPU and 1024MiB of memory. You can changes these options 
through `virtualisation.vmVariant.virtualisation` e.g.

```nix
virtualisation.vmVariant = {
  # following configuration is added only when building VM with build-vm
  virtualisation = {
    cores = 4;         
    diskSize = 10 * 1024;
    memorySize = 4 * 1024;
  };
}
```
