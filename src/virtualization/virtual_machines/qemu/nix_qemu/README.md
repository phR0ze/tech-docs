# NIX QEMU <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />
NixOS has the ability to build VMs from a nix system closure for testing. 

**References**
* [Custom Images - Mayflower](https://nixos.mayflower.consulting/blog/2018/09/11/custom-images/)
* [NixOS Virtual Machines](https://nix.dev/tutorials/nixos/nixos-configuration-on-vm.html)

## Run test VM

### Build test VM
This will build a test VM from a flake where the flake has been setup with a `nixosConfigurations` 
called `vm` e.g. http://github.com/phR0ze/nixos-config. If you do this with a configuration that is 
the same as your system minus the `hardware-configuration.nix` then it should only take a min to 
build the vm and drop a `result` link in your directory.

**Using nix build**
```bash
$ nix build .#nixosConfigurations.vm.config.system.build.vm
```

**Using clu**
```bash
$ clu build vm
```

### Execute test VM
This will automatically create a disk named for your host e.g. `nixos.qcow2` in my case for the new 
VM to persist any changes on disk. It will use the VM host's nix store.

**Directly**
```bash
$ ./result/bin/run-nixos-vm
```

**Using clu**
```bash
$ clu run vm
```
