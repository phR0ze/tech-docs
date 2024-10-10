# Build VM
NixOS has a mechanism built into it for generating a virtual machine based off from a given 
configuration e.g. `nix build .#nixosConfigurations.vm.config.system.build.vm`.

### Quick links

## Overview

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
