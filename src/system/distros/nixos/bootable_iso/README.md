# Bootable ISO
The NixOS minimal ISO image is a great example of a minimal system to start from. I'll be documenting 
my journey to rebuild the minimal ISO with changes.

### References
* [Building a bootable ISO image](https://github.com/NixOS/nix.dev/blob/master/source/tutorials/nixos/building-bootable-iso-image.md)
* [NixOS generators](https://github.com/nix-community/nixos-generators)
* [NixOS Installer Configs](https://github.com/NixOS/nixpkgs/tree/master/nixos/modules/installer/cd-dvd)
* [Create a custom NixOS ISO](https://haseebmajid.dev/posts/2024-02-04-how-to-create-a-custom-nixos-iso/)
  * [Haseeb's dotfiles](https://gitlab.com/hmajid2301/dotfiles)
* [Building a NixOS live ISO](https://nixos.org/manual/nixos/stable/index.html#sec-building-image)

## Modify the minimal ISO

**References**
* [Creating a NixOS live CD](https://nixos.wiki/wiki/Creating_a_NixOS_live_CD)

### How to change the ISO name

### Example adding extra packages
Build the Minimal NixOS ISO using the upstream configuration and add in the `git` and `jq` packages.

1. Create a dir for your flake
   ```bash
   $ mkdir nixos-iso
   ```
2. Create the flake root `flake.nix`
   ```
   {
     description = "Minimal NixOS installation media";
   
     inputs.nixos.url = "nixpkgs/23.11";
   
     outputs = { self, nixos }: {
       nixosConfigurations = {
   
         # ISO configuration
         iso = nixos.lib.nixosSystem {
           system = "x86_64-linux";
           modules = [
             "${nixos}/nixos/modules/installer/cd-dvd/installation-cd-minimal.nix"
             ({ pkgs, ... }: {
               environment.systemPackages = [
                 pkgs.git
                 pkgs.jq
               ];
             })
           ];
         };
       };
     };
   }
   ```
3. Turn your flake into a git repo to make it recognized as a flake
   ```bash
   $ git init
   ```
4. Commit or at the least stage your files so the flake system will see them
   ```bash
   $ git add .
   ```
5. Now build the iso
   ```bash
   $ nix build .#nixosConfigurations.iso.config.system.build.isoImage
   ```
6. The ISO will end up in `result/iso/`

## Minimal ISO composition
The minimal NixOS ISO starts with the [installation-cd-minimal.nix](https://github.com/NixOS/nixpkgs/blob/master/nixos/modules/installer/cd-dvd/installation-cd-minimal.nix) profile.

* pulls in [../../profiles/minimal.nix](https://github.com/NixOS/nixpkgs/blob/master/nixos/modules/profiles/minimal.nix)
  to start from scratch
  * overrides some of the minimal exclusions to avoid uncached builds
* pulls in [./installation-cd-base.nix](https://github.com/NixOS/nixpkgs/blob/master/nixos/modules/installer/cd-dvd/installation-cd-base.nix)
  * sets up the ISO configuration
    * iso name
* pulls in [all-hardware.nix](https://github.com/NixOS/nixpkgs/blob/master/nixos/modules/profiles/all-hardware.nix)
  * adds a bunch of kernel modules to support hardware
* pulls in [base.nix](https://github.com/NixOS/nixpkgs/blob/master/nixos/modules/profiles/base.nix)
  * includes disk partitioning packages
  * includes hardware tool packages
  * calls out supported filesystems
* pulls in [installation-device.nix](https://github.com/NixOS/nixpkgs/blob/master/nixos/modules/profiles/installation-device.nix)
  * sets up `nixos` user
  * sets up blank password logins for `nixos` and `root`
  * sets up passwordless sudo for `wheel` group
  * sets up getty autologin for `nixos`
  * sets up the boot text describing accounts
  * configures sshd with root access
  * enable wireless but don't start it
  * `includes packages in the ISO's NIX store to speed up the install`
  * pulls in `../installer/scan/detected.nix`
    * enables free hardware
  * pulls in `../installer/scan/not-detected.nix`
    * enables non free hardware
  * pulls in [./clone-config.nix](https://github.com/NixOS/nixpkgs/blob/master/nixos/modules/profiles/clone-config.nix)
    * 
  * pulls in [../installer/cd-dvd/channel.nix](https://github.com/NixOS/nixpkgs/blob/master/nixos/modules/installer/cd-dvd/channel.nix)
    * copies in the channel data
* pulls in [iso-image.nix](https://github.com/NixOS/nixpkgs/blob/master/nixos/modules/installer/cd-dvd/iso-image.nix)
  * creates grub menu entries 
  * creates EFI refind menu entires
  * creates EFI boot image
  * configures syslinux theme
  * configures eltorito etc...

<!-- 
vim: ts=2:sw=2:sts=2
-->
