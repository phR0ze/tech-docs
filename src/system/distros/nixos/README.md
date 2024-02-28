# NixOS
`NixOS` is a Linux distribution built on top of the `Nix` package manager which leverages the 
`Nix Expression Language` to make it `trivial to share development and build environments`. At the 
system level this means being able to build new systems in a declarative way that are essentially 
identical to the original declared state. Additionally NixOS claims to make packages Reproducible and 
installing or upgrading Reliable. The claim is that you can make changes, install new software and 
upgrade fearlessly because changes are done in isolation and can be rolled back.

### My perspective
In practice `NixOS` doesn't deliver on its goals out of the box. There is considerable complicated 
configuration required to actually be able to rebuild systems that are identical to the orginal 
declared state. Additionally `NixOS` predominantly focuses on the system level and almost entirely 
lacks support for declarative system wide user configuration. `Home Manager`, although commonly sited
as filling this gap, unfortunately focuses primarily on a multi-user approach that doesn't seem to 
mesh well with your typical single-user consumer systems.

Additionally those new to NixOS find it challenging due to overlapping similar terminology. The 
primary confusion I've found comes from the fact that the ecosystem is quite broad and `Nix`, the 
package manager and heart of NixOS, is cross-platform and cross-distro. This means documentation and 
tutorials are fractured and sometimes not applicable at all or difficult to follow as the target OS 
is different. Although the benefits of Nix and NixOS out weigh these concerns it does make the 
ecosystem more unapproachble than prior ecosystems like Arch Linux that I'm coming from.

### Quick links
* [Overview](#overview)
  * [Flakes as entry point](#flakes-as-entry-point)
* [Getting started](#getting-started)
  * [Install NixOS VM](#install-nixos-vm)
  * [Install NixOS bare metal](#install-nixos-bare-metal)
  * [Configure installation](#configure-installation)
    * [Boot loader](#boot-loader)
    * [Add new user](#add-new-user)
    * [Add sudo passwordless access](#add-sudo-passwordless-access)
    * [Enable Xfce as desktop](#enable-xfce-as-desktop)
  * [Upgrade NixOS](#upgrade-nixos)
* [Build bootable ISO](#build-bootalble-iso)
* [Flakes](#flakes)
* [Learning NixOS](#learning-nixos)
* [Build Live ISO](#build-live-iso)
* [Upgrades](#upgrades)
* [Flakes](#flakes)

## Overview

**References**
* [Nix Manual](https://nixos.org/manual/nix/stable/)
* [Nix Dev](https://nix.dev/tutorials/nix-language.html)
* [Nixpkgs Manual](https://nixos.org/manual/nixpkgs/stable)
  * [Nixpkgs github](https://github.com/NixOS/nixpkgs)
* [NixOS Manual](https://nixos.org/manual/nixos/stable)
* [Determinate Systems](https://github.com/DeterminateSystems)
  * [Determinate Nix Installer](https://github.com/DeterminateSystems/nix-installer)
  * [FlakesHub](https://flakehub.com/)
* [NixOS and Flakes Book](https://nixos-and-flakes.thiscute.world/preface)
* [Flakes aren't real](https://jade.fyi/blog/flakes-arent-real/)

### Flakes as entry point
There is a lot of commotion and confusion in the NixOS community around Flakes due to their 
experimental state and only partial adoption in Nix. Additionally the Flakes feature is often 
conflated with the the new Nix cli command feature because it's required for flake support. 
Regardless flakes serve a sepecific purpose are widely used and won't be going anywhere.

After digesting the Nix community blogs, docs and various sources its safe to say that Flakes solve 
the versioning issue in NixOS. Flakes track which version of packages are installed allowing NixOS to 
deliver on the rebuildable system statement. A common use case for flakes  is to use it as your entry 
point to your system configuration thus providing a reproducible way to rebuild your system even if 
the packages versions change.

### How to determine what is installed
NixOS doesn't have a ***simple*** way to determine what is installed, but it does have powerful and 
programmatic ways to evaluate and extract information about the system.

**Prettyify the json output**
* `-r` removes quotes from output
* `. |= unique` removes duplicates from the array
* `.[]` converts the array to a newline separate string

```bash
$ nix-instantiate --strict --json --eval -E 'builtins.map (p: p.name) (import <nixpkgs/nixos> {}).config.environment.systemPackages' | jq -r '. |= unique | .[]'

$ nix-store -q --references /run/current-system/sw
```

### Get file names in package
Again not much built in support for simple queries but some decent programatic APIs

***Example of looking up util-linux***
```bash
$ find $(nix-build '<nixpkgs>' -A util-linux --no-link)
```

### Delete a store item
1. Remove all links to it
2. Run the command
   ```
   $ nix-store --delete /nix/store/ghgk3fkg1fi1kz02zzkxivvcgyrpmpbb-nixos-23.11.20240211.809cca7-x86_64-linux.iso
   ```








### Prerequisites
Rust tool installable via curl bashing to assist during the install in applying configuration from 
git repos.

1. Boot from NixOS ISO
2. Partition target disk
3. Generate the hardware config
4. Configure flakes

### Remote deployments
Sometimes it nice to use a beefier machine to build and ready the target components to then remotely 
deploy to the target machine.

* [Remote deployments](https://nixos-and-flakes.thiscute.world/best-practices/remote-deployment#deploy-through-nixos-rebuild)

# Getting started

## Install NixOS VM
1. Navigate to [NixOS: the Linux distribution](https://nixos.org/download#nixos-iso)
   ```bash
   $ wget https://channels.nixos.org/nixos-23.11/latest-nixos-gnome-x86_64-linux.iso
   ```

2. Create a new sparse qcow2 drive to install to
   ```bash
   $ qemu-img create -f qcow2 nix1.qcow2 20G
   ```

3. Create a new VM using the new drive and ISO
   ```bash
   $ qemu-system-x86_64 -enable-kvm -m 4G -nic user,model=virtio -drive file=nix1.qcow2,media=disk,if=virtio \
       -cdrom latest-nixos-gnome-x86_64-linux.iso
   ```

4. Follow on runs of the VM just need the cdrom dropped.
   ```bash
   $ qemu-system-x86_64 -enable-kvm -m 4G -nic user,model=virtio -drive file=nix1.qcow2,media=disk,if=virtio
   ```
5. Once booted
   1. Select the `NixOS Installer` option at the top
6. Continue with steps in [Configure installation](#configure-installation)

## Install NixOS bare metal

1. Build NixOS bootable USB
   1. Determine the correct USB device
      ```bash
      $ lsblk
      ```
   2. Optionally wipe your USB first
      ```bash
      $ sudo wipefs --all --force /dev/sdd
      ```
   3. Copy to the dev leaving off the partition
      ```bash
      $ sudo dd bs=4M if=latest-nixos-minimal-x86_64-linux.iso of=/dev/sdd status=progress conv=fsync oflag=direct
      ```
2. Boot from USB and install
   1. Boot from the USB
   2. Select the `NixOS Installer` option at the top
3. Continue with steps in [Configure installation](#configure-installation)

## Configure installation
* [Nix Manual](https://nixos.org/manual/nix/unstable/contributing/cli-guideline)
* [Configuration collection](https://nixos.wiki/wiki/Configuration_Collection)
  * [AaronJanse configs](https://github.com/aaronjanse/dotfiles/tree/master)

1. Copy over nix configuration file
   1. Set the user password: `passwd nixos` 
   2. Determine the ip address: `ip a`
   3. Shell in via ssh from a terminal
   4. Now from another comp run: `scp configuration.nix nixos@192.168.1.206:~/`

2. Setup the target hard disk
   1. Use `clu` to partition the drive and generate a hardward config
   2. Finally run the installer, if the configuration fails to build just fix it and try again
      ```bash
      $ nixos-install --no-root-passwd
      ```
   3. Reboot
      ```bash
      $ sudo reboot
      ```

### Package vs Module
You can install packages directly with `systemPackages = [ "foo" ]` or use NixOS modules to get more 
control over the application's installation and configuration. Its a good idea to always look up in 
`https://mynixos.com` your app and see what modules a.k.a options are available.

### Add new user
In order to be able to login to your new system your user must have a temporary password using the 
`initialPassword` property. Or optionally you could use the `initialHashedPassword` but you'd need to 
hash the value first with `mkpasswd -m sha512 PASSWORD`. I find its just easier to have the user change their password 
on first run.

* [Create users](https://discourse.nixos.org/t/hashedpassword-issues-cant-sudo/13061)

```
users.users.alice = {
  isNormalUser = true;
  extraGroups = [ "wheel" "networkmanager" ];
  initialPassword = "nixos";
};
```

## Which packages are installed or will be installed

Search for a package from the command line
```
$ nix-env -qaP --description [package_name]
```

System wide software
```
/run/current-system/sw

$ nix-store -q --references /run/current-system/sw
```

### Get a list of differences in the upgrade
```bash
$ nix --extra-experimental-features "nix-command flakes" store diff-closures /run/*-system

nix show-derivation -r $(readlink -f $(which firefox)) | grep patches

nix store diff-closures /run/*-system
nixos-rebuild build 
nix store diff-closures /run/current-system /etc/nixos/result
```

### Find out what version of the nixpkgs repo was used
```bash
$ cd /etc/nixos
$ nix flake metadata
Resolved URL:  git+file:///etc/nixos
Locked URL:    git+file:///etc/nixos
Description:   System Configuration
Path:          /nix/store/j0g0jnxy4zhlx9iapp400ijjlvf583hk-source
Revision:      662e16a94336faeb2bda3464be9adaf5fe579ccf-dirty
Last modified: 2024-02-27 16:48:22
Inputs:
├───home-manager: github:nix-community/home-manager/652fda4ca6dafeb090943422c34ae9145787af37
│   └───nixpkgs follows input 'nixpkgs'
├───nixpkgs: github:nixos/nixpkgs/5bf1cadb72ab4e77cb0b700dab76bcdaf88f706b
└───nixpkgs-unstable: github:nixos/nixpkgs/73de017ef2d18a04ac4bfd0c02650007ccb31c2a
```


```
$ nix --extra-experimental-features "nix-command flakes" eval --raw github:NixOS/nixpkgs/20f65b86b6485decb43c5498780c223571dd56ef#hello
```

### Snowfall
The Snowfall project has been working on GUI tools for Nix using gtk4 and Rust

* [Effort to make more GUI centric](https://discourse.nixos.org/t/snowflakeos-creating-a-gui-focused-nixos-based-distro/21856)
* [Snowfall rust](https://github.com/snowfallorg)

* Calamares installer
* nixos-conf-editor
* nix-software-center

### nixos-install
The `nixos-install` tool installs the bootloader and NixOS to the `/mnt` path based on the 
configuration passed in. 
1. It copies Nix and its dependencies to `/mnt/nix/store`
2. It installs the current channel "nixos" unless `--no-channel-copy`

### Shell environments
You can create temporary Nix shell environments that allow you to install and use software in an 
isolated way that you can then wipe out when your done. You can also share the command that invoked 
the shell to share it with others.

**Create shell with two packages**
```bash
$ nix-shell -p cowsay lolcat
```

**Exit the shell or press Ctrl+D to clean up**
```bash
$ exit
```

**Run app directly in shell**
```bash
$ nix-shell -p cowsay --run "cowsay Nix"
```

### Search for packages
You can [search the NixOS packages](https://search.nixos.org/packages)


## Build Live ISO
Default live installer configurations are available inside `nixos/modules/installer/cd-dvd`

**References**
* [Nix generators github](https://github.com/nix-community/nixos-generators)
* [Building an Image docs](https://nixos.org/manual/nixos/stable/#sec-building-image)

You have two options:
1. Use any of those default configurations as is
2. Combine them with (any of) your host config(s) 

System images, such as the live installer ones, know how to enforce configuration settings on which 
they immediately depend in order to work correctly.

However, if you are confident, you can opt to override those enforced values with `mkForce`. 

### Build Unstable channel ISO
```bash
$ git clone https://github.com/NixOS/nixpkgs.git
$ cd nixpkgs/nixos
$ git switch nixos-unstable
$ nix-build -A config.system.build.isoImage -I nixos-config=modules/installer/cd-dvd/installation-cd-minimal.nix default.nix
```

Check the content of the image with:
```bash
$ mount -o loop -t iso9660 ./result/iso/cd.iso /mnt/iso
```

### Additional drivers or firmware
If you need additional (non-distributable) drivers or firmware in the installer, you might want to 
extend these configurations.

For example, to build the GNOME graphical installer ISO, but with support for certain WiFi adapters 
present in some MacBooks, you can create the following file at 
`modules/installer/cd-dvd/installation-cd-graphical-gnome-macbook.nix`:

```
{ config, ... }:

{
  imports = [ ./installation-cd-graphical-gnome.nix ];

  boot.initrd.kernelModules = [ "wl" ];

  boot.kernelModules = [ "kvm-intel" "wl" ];
  boot.extraModulePackages = [ config.boot.kernelPackages.broadcom_sta ];
}
```

Then build it like in the example above:

```bash
$ git clone https://github.com/NixOS/nixpkgs.git
$ cd nixpkgs/nixos
$ export NIXPKGS_ALLOW_UNFREE=1
$ nix-build -A config.system.build.isoImage -I nixos-config=modules/installer/cd-dvd/installation-cd-graphical-gnome-macbook.nix default.nix
```

## Configuration
The NixOS configuration file `/etc/nixos/configuration.nix` is actually a Nix expression, which is 
the Nix package manager's purely functional language for describing how to build packages and 
configurations. This means you have all the expressive power of that language at your disposal, 
including the ability to abstract over common patterns, which is very useful when managing complex 
systems. The syntax and semantics of the Nix language are fully described in the Nix manual

**References**
* [Nix docs on configs](https://nixos.org/manual/nixos/stable/#ch-configuration)
* [Nix config syntax](https://nixos.org/manual/nixos/stable/#sec-configuration-syntax)
* [Nix manual](https://nixos.org/nix/manual/#chap-writing-nix-expressions)

### Nix Configuration Syntax

## Upgrades

### Automatic Upgrades
You can keep NixOS up-to-date automatically by adding the following to your `configuration.nix`
```
system.autoUpgrade.enable = true;
system.autoUpgrade.allowReboot = true;
```

This enables a periodically executed systemd service named `nixos-upgrade.service`. If the `allowReboot` 
option is `false`, it runs `nixos-rebuild switch --upgrade` to upgrade NixOS to the latest version in the 
current channel. (To see when the service runs, see `systemctl list-timers`.) If `allowReboot` is `true`, 
then the system will automatically reboot if the new generation contains a different kernel, initrd 
or kernel modules. You can also specify a channel explicitly, e.g. 

```
system.autoUpgrade.channel = "https://channels.nixos.org/nixos-23.05";
```

<!-- 
vim: ts=2:sw=2:sts=2
-->
