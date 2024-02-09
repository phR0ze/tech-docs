# NixOS
`NixOS` is a Linux distribution built on top of the `Nix` package manager which leverages the 
`Nix Expression Language` to make building new systems that are reliable and can be easily reproduced 
in a declarative way. The claim is that you can fearlessly make changes and install new software 
because they are in isolation and can always be rolled back if something breaks.

### Quick links
* [Overview](#overview)
  * [Flakes are best practice](#flakes-are-best-practice)
* [Getting started](#getting-started)
  * [Install NixOS VM](#install-nixos-vm)
  * [Install NixOS bare metal](#install-nixos-bare-metal)
  * [Configure installation](#configure-installation)
    * [Boot loader](#boot-loader)
    * [Add new user](#add-new-user)
    * [Add sudo passwordless access](#add-sudo-passwordless-access)
    * [Enable Xfce as desktop](#enable-xfce-as-desktop)
  * [Upgrade NixOS](#upgrade-nixos)
* [Flakes](#flakes)
* [Learning NixOS](#learning-nixos)
* [Build Live ISO](#build-live-iso)
* [Upgrades](#upgrades)
* [Flakes](#flakes)

## Overview
Those new to NixOS can find it challenging due to overlapping similar terminology. The primary 
confusion comes from the fact that `Nix`, the package manager and heart of NixOS, can be used 
directly on other linux distros and MacOS as well. This means `Nixpkgs`, the Nix package repository, 
and `Flakes`, the modern Nix packaging system, are both used in a wide variety of OSs and contexts 
that aren't in some cases directly applicable to `NixOS`.

**References**
* [Nix Manual](https://nixos.org/manual/nix/stable/)
* [Nixpkgs Manual](https://nixos.org/manual/nixpkgs/stable)
  * [Nixpkgs github](https://github.com/NixOS/nixpkgs)
* [NixOS Manual](https://nixos.org/manual/nixos/stable)
* [Determinate Systems](https://github.com/DeterminateSystems)
  * [Determinate Nix Installer](https://github.com/DeterminateSystems/nix-installer)
  * [FlakesHub](https://flakehub.com/)
    * Provides search and semantic versioning
* [NixOS and Flakes book](https://nixos-and-flakes.thiscute.world/best-practices/run-downloaded-binaries-on-nixos)

### Flakes are best practice
***Flakes*** are installable units of configuration that may include files, configs, packages, 
dependencies or other flakes. You can think about them like a package, but the Nix ecosystem already 
has a package type so they had to use another term. They simplify usability and improve 
reproducibility through better dependency managment than legacy methods. Despite having yet to shed 
the experimental label, `Flakes` are considered the best practice in the Nix community. Other methods 
are considered legacy. 

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

### Boot loader
NixOS recommends using `systemd` for the boot loader. I'll have to circle back on that as I recall 
having some issue with passing arguments to the kernel via the systemd boot loader.

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

### Add sudo passwordless access
https://nixos.wiki/wiki/Sudo

Add sudo passwordless access to those in the `wheel` group
```
security.sudo = {
  enable = true;
  extraRules = [{
    commands = [{ command = "ALL"; options = [ "NOPASSWD" ];}]; groups = [ "wheel" ]; }
  ];
};
```

### Enable Xfce as desktop
https://nixos.wiki/wiki/Xfce

```
services.xserver = {
  enable = true;
  desktopManager = {
    xfce.enable = true;
    xterm.enable = false;
  };
  displayManager.defaultSession = "xfce";
};
```

### Packages
NixOS requires a minimal set of packages to function.

* pkgs.nix
* pkgs.bashInteractive
* pkgs.coreutils-full
* pkgs.gnutar
* pkgs.gzip
* pkgs.gnugrep
* pkgs.which
* pkgs.curl
* pkgs.less
* pkgs.wget
* pkgs.man
* pkgs.cacert.out
* pkgs.findutils

### Kodi
https://nixos.wiki/wiki/Kodi
```
```

### Docker image
NixOS builds its own docker images rather than being build with Docker directly

You can build your latest NixOS docker image with
```bash
$ nix build ./\#hydraJobs.dockerImage.x86_64-linux
$ docker load -i ./result/image.tar.gz
$ docker run -ti nix:2.5pre20211105
```

## Upgrade NixOS
NixOS uses a rolling release with different channels based on stability from 

## Flakes
Use Flakes not channels and no `nix-env`

### Snowfall
The Snowfall project has been working on GUI tools for Nix using gtk4 and Rust

* [Effort to make more GUI centric](https://discourse.nixos.org/t/snowflakeos-creating-a-gui-focused-nixos-based-distro/21856)
* [Snowfall rust](https://github.com/snowfallorg)

* Calamares installer
* nixos-conf-editor
* nix-software-center

## Learning NixOS
* [NixOS Wiki](https://nixos.wiki/)
* [NixOS Manual](https://nixpkgs-manual-sphinx-markedown-example.netlify.app/)
* [User management](https://nixos.org/manual/nixos/stable/#sec-user-management)

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

## Flakes
[Flakes is a feature](https://nixos.wiki/wiki/Flakes) of managing Nix packages to simplify usability, 
composability and improve reproducibility of Nix installations. Flakes were created to solve some 
problems in the original Nix reproducible built paradigm see [what problems do flakes solve](https://www.tweag.io/blog/2020-05-25-flakes/).

<!-- 
vim: ts=2:sw=2:sts=2
-->
