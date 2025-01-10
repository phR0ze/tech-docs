# NixOS <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />

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
lacks support for declarative system wide ***user*** configuration. `Home Manager`, although commonly 
sited as filling this gap, unfortunately focuses primarily on a multi-user approach that doesn't seem 
to mesh well with a typical single-user consumer system.

Additionally those new to NixOS find it challenging due to overlapping similar terminology. The 
primary confusion I've found comes from the fact that the ecosystem is quite broad and `Nix`, the 
package manager and heart of NixOS, is cross-platform and cross-distro. This means documentation and 
tutorials are fractured and sometimes not applicable at all or difficult to follow as the target OS 
is different. Although ***the benefits of Nix and NixOS outweigh these concerns*** it does make the 
ecosystem more unapproachble than prior ecosystems like Arch Linux that I'm coming from.

### Quick links
* [.. up dir](..)
* [Overview](#overview)
  * [Flakes as entry point](#flakes-as-entry-point)
  * [System Versions](#system-versions)
* [Getting started](#getting-started)
  * [Install NixOS VM](#install-nixos-vm)
  * [Install NixOS bare metal](#install-nixos-bare-metal)
  * [Configure installation](#configure-installation)
    * [Add new user](#add-new-user)
  * [Upgrade NixOS](#upgrade-nixos)
* [Common operations](#common-operations)
  * [Temporarily run new software](#temporarily-run-new-software)
  * [Rollback system version](#rollback-system-version)
  * [Diff system versions](#diff-system-versions)
  * [List what changed](#list-what-changed)
  * [How to determine what is installed](#how-to-determine-what-is-installed)

### Linked pages

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
conflated with the the new Nix cli command features because it's required for flake support. 
Regardless flakes serve a sepecific critical purpose and won't be going anywhere.

After digesting the Nix community blogs, docs and various sources its safe to say that Flakes solve 
the versioning issue in NixOS. Flakes track which version of packages are installed allowing NixOS to 
deliver on the rebuildable system statement. A common use case for flakes  is to use it as your entry 
point to your system configuration thus providing a reproducible way to rebuild your system even if 
the packages versions change.

### System Versions
NixOS stores the different system versions as links in `/nix/var/nix/profiles`. The terminology of 
`profiles` and alternatly `generations` doesn't really resonate with me so I prefer to call them 
`system versions`. Given that NisOS configurations are declarative and best practice is to version 
control them with git this seems a much more natural fit in my opinion.

**References**
* [NixOS profiles](https://nixos.org/manual/nix/stable/package-management/profiles.html)

* `/run/current-system` is the currently running system version
  * It is a link directly to the running system e.g. `/nix/store/1...workstation4-24.05`
  * It might not be the same as the default boot version if you choose an alternate boot entry

* `/nix/var/nix/profiles/system` is the default system version
  * This is the version that the system will boot to by default
  * It won't be the same as `/run/current-system` if you choose an alternate boot entry
  * It links indirectly to the version via `system-VER-link`

* `/nix/var/nix/profiles/default` is a link to the latest user version
  * Used for user profiles

## Getting started

### Install NixOS VM
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

### Install NixOS bare metal

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
* [Nix docs on configs](https://nixos.org/manual/nixos/stable/#ch-configuration)
* [Nix config syntax](https://nixos.org/manual/nixos/stable/#sec-configuration-syntax)
* [Nix manual](https://nixos.org/nix/manual/#chap-writing-nix-expressions)

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

## Common operations

### Temporarily run new software
NixOS allows you to temporarily download and run new software before committing to permantenly 
installing it on your system. This allow syou to test software out and then allowing the system to 
clean it up for you automatically if you don't persist it in your configuration.

1. Download and run
   ```
   $ nix run nixpkgs#PACKAGE
   ```
2. Pass arguments to the new package with `--`
   ```
   $ nix run nixpkgs#vim -- --help
   ```

### Rollback system version
Setting the default boot entry is all that is needed to `rollback` the system. You can manually 
verify the default boot entry is changed via `/boot/grub/grub.cfg`

**Set prior system version N as default boot entry**
```bash
$ sudo /nix/var/nix/profiles/system-N-link/bin/switch-to-configuration switch
```

**Set current system as default boot entry**
```bash
$ sudo /run/current-system/bin/switch-to-configuration switch
```

**Set manually built version to default boot entry**
```bash
$ sudo result/bin/switch-to-configuration switch
```

### Diff system versions
You can diff different system versions using the `nix store diff-closures` command.

**Syntax**
* `nix store diff-closures FROM TO`

**Example:** ***FROM*** the older system-13-link version ***TO** the current version
```bash
$ nix store diff-closures /nix/var/nix/profiles/system-13-link /run/current-system 
qview: ∅ → 6.1, +1787.1 KiB
ristretto: 0.13.2 → ∅, -1179.6 KiB

```

**Diff raw changes against the current system**
1. Build the changes but neither activate it or add it to the GRUB boot menu
   ```bash
   $ nixos-rebuild build --flake "path:/etc/nixos#system"
   ```
2. Diff the resulting link against the current system version
   ```bash
   $ nix store diff-closures /run/current-system result/
   ```

### Manually add system version
If you build your system version manually or loose the links due to automatic cleanup you can add 
them back in manually as follows:

1. Get the nix store path for your target system version
   ```bash
   $ readlink -e /run/current-system
   ```
2. Create a link to your system version in `/nix/var/nix/profiles`
   ```bash
   $ cd /nix/var/nix/profiles
   $ ln -s /nix/store/...-workstation-23.05.4448.5550a85a087c system-39-link
   ```
3. Update the current system version
   ```bash
   $ rm system
   $ ln -s system-39-link system
   ```

### Force saving store paths
By adding paths to the `/nix/var/nix/gcroots` the system will ensure they are never removed.

```bash
$ ln -s /nix/store/...-workstation-23.05.4448.5550a85a087c /nix/var/nix/gcroots/save-this-version
```

Note: you can remove system versions by removing the links from `/nix/var/nix/profiles`

### List available system versions
Note: `nixos-rebuild list-generations` only seems to list out what is in `/nix/var/nix/profiles` 
which won't have an entry for your manual build.

```bash
$ nixos-rebuild list-generations --json
```

### List all packages for a system version

**Syntax**
* `nix-store -qR VERSION`

**Example**
```bash
$ nix-store -qR /run/current-system
```

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

### Invoke nix cleanup
Remove nix store items that are no longer being used.

**References**
* https://nixos.wiki/wiki/NixOS_Generations_Trimmer

```bash
$ nix-store --gc
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

### Which packages are installed or will be installed

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


<!-- 
vim: ts=2:sw=2:sts=2
-->
