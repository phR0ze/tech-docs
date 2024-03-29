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
* [Live Media with Nix flakes](https://hoverbear.org/blog/nix-flake-live-media/)
* [Offline Installer](https://github.com/tfc/nixos-offline-installer/blob/master/installer-configuration.nix)

## Offline Installer
nixos-install will run `nix build --store /mnt` which won't be able to see what is in the installer 
nix store so we need to copy everything to `/mnt` using `nix copy --no-check-sigs --to local?root=/mnt /out`

## Modify the minimal ISO

**References**
* [Creating a NixOS live CD](https://nixos.wiki/wiki/Creating_a_NixOS_live_CD)

### Boot into custom application
Using the standard minimal installation profile we find that the `nixos` user is automatically logged 
in and a message is shown in the [instllation-device.nix](https://github.com/NixOS/nixpkgs/blob/nixos-23.11/nixos/modules/profiles/installation-device.nix)

The `nixos` user account is setup to run bash as it's login program. 

1. Create the user `~/.bash_profile`
2. Add desired scripting or execute target binary

### Adjust the login message
Using the standard minimal installation profile we find that the `nixos` user is automatically logged 
in and a message is shown in the [instllation-device.nix](https://github.com/NixOS/nixpkgs/blob/nixos-23.11/nixos/modules/profiles/installation-device.nix)

NixOS has the [`services.getty`](https://mynixos.com/search?q=services.getty) options amongst others 
that could be used to modify the prompts in the shell once logged in.
* `services.getty.greetingLine` - Welcome line printed by agetty
* `services.getty.helpLine` - Help line printed by agetty below the welcome line

**Remove the help line message entirely**
```
services.getty.helpLine = mkForce ''
  If you need a wireless connection, type
  `sudo systemctl start wpa_supplicant` and configure a
  network using `wpa_cli`. See the NixOS manual for details.
'';
```

### Example adding extra packages
Build the Minimal NixOS ISO using the upstream configuration and add in the `git` and `jq` packages.

* [Creating a NixOS live CD](https://nixos.wiki/wiki/Creating_a_NixOS_live_CD)

1. Create a dir for your flake
   ```bash
   $ mkdir nixos-iso
   ```
2. Create the flake root `flake.nix`
   ```
   {
     description = "Minimal NixOS installation media";
   
     inputs.nixpkgs.url = "github:nixos/nixpkgs/nixos-23.11"; 
   
     outputs = { self, nixpkgs }: {
       nixosConfigurations = {
         iso = nixpkgs.lib.nixosSystem {
           system = "x86_64-linux";
           modules = [
             "${nixpkgs}/nixos/modules/installer/cd-dvd/installation-cd-minimal.nix"
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

### Minimal ISO composition notes
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
  * pulls in [../installer/cd-dvd/channel.nix](https://github.com/NixOS/nixpkgs/blob/master/nixos/modules/installer/cd-dvd/channel.nix)
    * copies in the channel data
* pulls in [iso-image.nix](https://github.com/NixOS/nixpkgs/blob/master/nixos/modules/installer/cd-dvd/iso-image.nix)
  * creates grub menu entries 
  * creates EFI refind menu entires
  * creates EFI boot image
  * configures syslinux theme
  * configures eltorito etc...

### Minimal ISO packages
Adding the following to your flake will generate the file `/etc/packages` which will always have the 
current list of packages for your system in it.
```nix
environment.etc."packages".text =
let
  packages = builtins.map (p: p.name) config.environment.systemPackages;
  sortedUnique = builtins.sort builtins.lessThan (lib.unique packages);
  formatted = builtins.concatStringsSep "\n" sortedUnique;
in
  formatted;
```

Alternately you can run the following on a channel based system
``` bash
$ nix-instantiate --strict --json --eval -E 'builtins.map (p: p.name) (import <nixpkgs/nixos> {}).config.environment.systemPackages' | jq -r '. |= unique | .[]'
```

<!-- 
vim: ts=2:sw=2:sts=2
-->
