# NixOS
Making reproducible, declarative and reliable systems is NixOS's mission.

### Quick links
* [Overview](#overview)
  * [Reproducible](#reproducible)
  * [Declarative](#declarative)
  * [Reliable](#reliable)
* [Get Started](#get-started)
* [Build Live ISO](#build-live-iso)
* [Upgrades](#upgrades)
* [Flakes](#flakes)

## Overview
Nix appears to have the same goal in mind as I've always wanted i.e. the ability to automate the 
deployment of a full system in a reliable way with all custom configuration intact. They approach 
desktop systems much like enterprise systems in that their apply system is akin to Terraform's 
approach to cloud infrastructure.

**References**
* [NixOS main site](https://nixos.org/)
* [NixOS Wiki](https://nixos.wiki)
* [Effort to make more GUI centric](https://discourse.nixos.org/t/snowflakeos-creating-a-gui-focused-nixos-based-distro/21856)

**Features**
* Seems to have better package support than Arch Linux
  * Stable packages: `zoom`, `vscode`, `rustdesk`, `vopono`, `inxi`, `ccextractor`, `epson-escpr2`, 
  `golangci-lint`, `makemkv`, `hardinfo`, `i3lock`, `losslesscut`, `pnmixer`, `mycli`
  * Missing: `xnviewmp`, `winff`, `arcologout`
* Most up to date distribution
* Abilty to try new tools risk free in shell environments that are cleaned up
* Create shell env configs that can be reused to setup an environment
* Build docker images using nix language then import them into docker

### Reproducible
Nix builds packages in isolation from each other. This ensures that they are reproducible and don't 
have undeclared dependencies, so if a package works on one machine it will work on another.

### Declarative
Nix makes it trivial to share development and build environments for your projects, regardless of 
what programming languages and tools you're using.

### Reliable
Nix ensures that installing or upgrading one package cannot break other packages. It allows you to 
roll back to previous versions, and ensures that no packages is in an inconsistent state during an 
upgrade.

## Get Started
1. Navigate to https://nixos.org/download#nixos-iso and download the latest GNOME based image
   ```bash
   $ wget https://channels.nixos.org/nixos-23.05/latest-nixos-gnome-x86_64-linux.iso
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
4. Following runs just need the cdrom dropped.
   ```bash
   $ qemu-system-x86_64 -enable-kvm -m 4G -nic user,model=virtio -drive file=nix1.qcow2,media=disk,if=virtio
   ```

### Notes on Live ISO installer
* ISO has few options
  * Installer, nomodeset, copytoram, debug, tty console Memtest86+
* Took so long to load I almost thought it froze
* Live ISO comes with
  * NixOS Installer, Firefox, Nix Manual, Shell, GParted, and the basic Gnome apps
* NixOS Installer (Calamares)
  * Looks great and seems smooth and has all modern options for an installer
  * Desktop option is neat, it gives apparently an option to choose which desktop to install
    * GNOME, Plasma, Xfce, Pantheon, Cinnamon, MATE, Enlightenment, LXQt, Budgi, Deepin, No desktop
  * Unfree software enablement option
  * Trying Budgie
  * Check the restart option and click next to reboot
* Using Grub for boot loader

### First run experience
Much of this will based on my Budgie impressions but capturing the NixOS items here
* Install only took 9.5 GB by default
* Pull down vim temporaily: `nix-shell -p vim`

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

## Flakes
[Flakes is a feature](https://nixos.wiki/wiki/Flakes) of managing Nix packages to simplify usability, 
composability and improve reproducibility of Nix installations. Flakes were created to solve some 
problems in the original Nix reproducible built paradigm see [what problems do flakes solve](https://www.tweag.io/blog/2020-05-25-flakes/).

<!-- 
vim: ts=2:sw=2:sts=2
-->
