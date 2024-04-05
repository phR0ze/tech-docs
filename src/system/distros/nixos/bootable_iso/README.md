# Bootable ISO
The NixOS minimal ISO image is a great example of a minimal system to start from. I'll be documenting 
my journey to rebuild the minimal ISO with changes to include a minimal installer that will make a 
NixOS that I like.

### Quick links
* [Live ISO Installer](#live-iso-installer)
  * [Build the live ISO for installation](#build-the-live-iso-for-installation)
* [Full live desktop ISO](#full-live-desktop-iso)
* [Modify the minimal ISO](#modify-the-minimal-iso)
  * [Boot into custom application](#boot-into-custom-application)
  * [Adjust the login message](#adjust-the-login-message)
  * [Example adding extra packages](#example-adding-extra-packages)
  * [Minimal ISO composition notes](#-minimal-iso-composition-notes)
  * [Minimal ISO packages](#minimal-iso-packages)
* [Offline Installer](#offline-installer)
  * [Include derivations for offline installs](#include-derivations-for-offline-installs)
  * [Pre-populate the target system nix store](#pre-populate-the-target-system-nix-store)
  * [Include system configuration for offline installs](#include-system-configuration-for-offline-installs)
  * [Include automation for offline installs](#include-automation-for-offline-installs)

## Live ISO Installer

**References**
* [Live Media with Nix flakes](https://hoverbear.org/blog/nix-flake-live-media/)
* [Create a custom NixOS ISO](https://haseebmajid.dev/posts/2024-02-04-how-to-create-a-custom-nixos-iso/)
  * [Haseeb's dotfiles](https://gitlab.com/hmajid2301/dotfiles)
* [NixOS image builders](https://github.com/nix-community/nixos-generators)
* [Building a bootable ISO image](https://github.com/NixOS/nix.dev/blob/master/source/tutorials/nixos/building-bootable-iso-image.md)
* [NixOS Installer Configs](https://github.com/NixOS/nixpkgs/tree/master/nixos/modules/installer/cd-dvd)

### Build the live ISO for installation
NixOS has a lot of reusable automation built into it. In the Arch world typically you have to start 
from scratch and build your own automation if you want control over how its being built. In the Nix 
world though this already exists.

1. Clone my nixos configuration repo
   ```bash
   $ git clone https://github.com/phR0ze/nixos-config
   ```

2. Modify the ISO profile as desired
   ```bash
   $ vim profiles/iso/default.nix
   ```

3. Commit or at the least stage your changes so Nix will see them
   ```bash
   $ git add .
   ```

4. Now build the iso
   ```bash
   $ ./clu build iso
   ```

5. The ISO will end up in `result/iso/`

## Full live desktop ISO
TBD after I solve the offline installer issue

## Modify the minimal ISO

**References**
* [Creating a NixOS live CD](https://nixos.wiki/wiki/Creating_a_NixOS_live_CD)

### Boot into custom application
Using the standard minimal installation profile we find that the `nixos` user is automatically logged 
in and a message is shown in the [installation-device.nix](https://github.com/NixOS/nixpkgs/blob/nixos-23.11/nixos/modules/profiles/installation-device.nix) file.

**Attempt #1** originally I used home-manager to create a custom `.bash_profile` in the `nixos` user's 
home directory which contained the launcher for my installer script. This worked well but requires 
home-manager which is more configuration and maintenance than I want.
```nix
{ args, pkgs, lib, ... }: {
  imports = [
    args.home-manager.nixosModules.home-manager
    "${args.nixpkgs}/nixos/modules/installer/cd-dvd/installation-cd-minimal.nix"
  ];
  home-manager = {
    extraSpecialArgs = { inherit args; };
    users.nixos = {
      home.file.".bash_profile".text = ''
        # install script launcher
      '';
      home.username = "nixos";
      home.homeDirectory = "/home/nixos";
      home.stateVersion = args.settings.stateVersion;
    };
  };
  users.users.nixos.password = "nixos";
  users.extraUsers.root.password = "nixos";
  environment.systemPackages = with pkgs; [
    git                 # Needed for clu installer automation
    jq                  # Needed for clu installer automation
  ];
}
```

**Attempt #2** a better approach `profiles/iso/default.nix` was to just drop it into the shared 
`/etc/bashrc`.
```nix
{
  programs.bash.promptPluginInit = ''
    # installer script launcher
  '';
};
```

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

## Offline Installer
Installing a linux distro offline requires install media with all the necessary components available 
without internet access. In NixOS these components are the `system configuration`, `install 
automation`, and `install payloads`.

**References**
* [ngins docker image for shared binary cache](https://github.com/pbek/nixcfg/blob/main/docker/nix-cache-nginx/nginx.conf)
* [Local shared NixOS cache](https://dataswamp.org/~solene/2022-06-02-nixos-local-cache.html)
* [Nix serve](https://github.com/edolstra/nix-serve)
* [Another offline example](https://github.com/jtraue/nixos-offline-installer/tree/master)
* [Another offline example](https://github.com/pete3n/nix-offline-iso)
* [Determinate systems](https://determinate.systems/posts/)
* [Rust installer](https://github.com/DeterminateSystems/nix-installer)
* [Nix copy](https://nixos.org/manual/nix/stable/command-ref/new-cli/nix3-copy)
  * [Nix copy to and from](https://determinate.systems/posts/moving-stuff-around-with-nix/)
* [Building a NixOS live ISO](https://nixos.org/manual/nixos/stable/index.html#sec-building-image)
* [Offline Installer](https://github.com/tfc/nixos-offline-installer/blob/master/installer-configuration.nix)
* [Get all your sources](https://linus.schreibt.jetzt/posts/include-build-dependencies.html)
* [Serving a Nix store via SSH](https://nixos.org/manual/nix/unstable/package-management/ssh-substituter.html)

### Include derivations for offline installs
In Nix parlance derivations are any configuration or packages needed to build your system and are 
stored in the nix store. NixOS has a rich ecosystem for building NixOS based media. The Nix ISO 
builder automatically populates its own nix store with all the derivations that weere used to build 
the ISO. This nix store is then stored as a squashfs file on the ISO itself and mounted during boot 
to serve up the ISO's default environment.

**References**
* [Another offline installer](https://github.com/pete3n/nix-offline-iso)
* [Offline installer](https://github.com/tfc/nixos-offline-installer/blob/master/installer-configuration.nix)
* [Include custom nix store content](https://github.com/matthewbauer/nixiosk/blob/master/boot/pxe.nix)
* [ISO Image Construction](https://github.com/NixOS/nixpkgs/blob/master/nixos/modules/installer/cd-dvd/iso-image.nix)

**Add the following to include additional packages in your ISO Nix store**
Additionally the ISO builder exposes the `isoImage` option with has several sub options for manually 
maniuplating what is included in the ISO's nix store. Once the derivations are in the ISO nix store 
they can be copied by automation to the target system using `nix copy` thus pre-populating the target 
system's nix store and bypassing any need to download these derivations from the internet.

* `isoImage.contents` includes files outside the nix store i.e. not in the resulting squashfs
* `isoImage.storeContents` includes derivations in the squashfs Nix store
* `isoImage.includeSystemBuildDependencies` includes all sources needed to build your system offline, 
  but his is overkill and swells the resulting ISO to something that won't fit on a standard usb.

**Including derivations in the ISO's nix store:**
1. We can directly add additional derivations to the ISO's nix store with
   ```nix
   isoImage.storeContents = with pkgs; [
     config.system.build.toplevel  # default ISO inclusion
     bash
     tree
     ...
   ];
   ```
2. Alternately and perhaps more conveniently any derivations needed for the ISO's own environment 
   will also be included in the ISO's nix store. This means that if you build a fully functional live 
   environment using all target derivations then the ISO's nix store will already include all 
   derivations needed for the target install

### Pre-populate the target system nix store
Everything that was included in the ISO nix store needs to be copied to the target system to be 
available during the installation. In this way we bypass the typical internet downloads and even 
derivation building to simply install the missing links and inflate files at target file paths on the 
system.

**Copy the ISO nix store to the target system's mounted /mnt/nix/store**
```bash
$ sudo nix copy --all --no-check-sigs --offline --to local?root=/mnt
```







### Include system configuration for offline installs
The flake registry seems to get downloaded and stored at `~/.cache/nix/flake-registry.json` which is 
a link back to the `/nix/store/<hash>-flake-registry.json`

### Include automation for offline installs

### Ensure the nixpkgs tarball is available offline
One of the first things that happens during an installation is that Nix will download a copy of the 
nixpkgs repo. In order to make this offline we need to do this in advance and include it in the ISO.

**References**
* [Pinning Nixpkgs](https://nixos.wiki/wiki/FAQ/Pinning_Nixpkgs)


**Note this is already being done by the ISO machinery by default** as the exact hash name was found 
in the ISO Nix store without manually adding below; but including this for reference for other 
purposes.
```nix
import (builtins.fetchTarball {
  # `nix registry list | grep nixpkgs` seems to use `source` as the name
  name = "source";

  # url is constructed from flake.lock file
  url = "https://github.com/nixos/nixpkgs/archive/1536926ef5621b09bba54035ae2bb6d806d72ac8.tar.gz";

  # Hash obtained using `nix-prefetch-url --unpack <url>`
  sha256 = "1jg7g6cfpw8qvma0y19kwyp549k1qyf11a5sg6hvn6awvmkny47v";
}) {}
```

Which ends up being `/nix/store/lwyjz70qh12nq6cb7fixl85vryzxqm3c-source` and when we use the command 
in the next section i.e. `sudo nix copy --all --no-check-sigs --offline --to local?root=/mnt` to 
populate the target nix store then we will have the store path
`/nix/store/lwyjz70qh12nq6cb7fixl85vryzxqm3c-source` available for use without a download needed


***INCLUDE CLU ON ISO as PKG***







### Local binary cache
* [Nix Local Binary Cache](https://nixos.org/manual/nix/stable/store/types/local-binary-cache-store)

1. Download and copy target store paths to your local binary cache
   ```bash
   $ nix copy --to file:///tmp/cache nixpkgs#hello
   $ nix copy --to file:///tmp/cache $(type -p firefox)
   $ nix copy --to file:///tmp/cache -f default.nix
   $ nix copy --to file:///tmp/cache?parallel-compression=true -f default.nix
   ```

2. Mount the local binary cache partition
   ```bash
   $ sudo mount /dev/local/binary/cache /mnt/cache
   ```

3. Copy all store paths from the local binary to your local store
   ```bash
   $ nix copy --all --from file:///mnt/cache
   ```

4. Unmount the local binary cache partition
   ```bash
   $ sudo umount /mnt/cache
   ```








### Other notes
Exporting and importing closures allows for sharing read-to-use applications similar to how docker 
images bundle applications for reuse.

* Nix closures can be exported as Nix ARchive (NAR) from nix store e.g. app `chord`
```
nix-store --export $(nix-store --query --requisites ./result) > chord.nar
```

And re-imported
```
nix-store --import < chord.nar
```

```
nix copy nixpkgs.ag --to file://ag.pkg 
# We now have a local directory `ag.pkg` which we can tar-up and distribute
# On the client machine, we run to install the package
nix copy --from file://ag.pkg nixpkgs.ag
```

Examples
```
nix copy --to file:///home/admin/Downloads $(type -p firefox)
nix copy --to file:///home/admin/Downloads .#deps
scp -r /nix/store/* your_user@host_name:/path/to/store/copy
```

Add `file:///the/path` to the substituters then `nix copy --no-check-sigs --to local?root=/mnt /out`

Specify an alternate store
`--store /cachedir` is an option that all nix commands can take to use a specific cache

* Don't use `NIX_STORE_DIR` as this breaks compatibility with the official binary cache and you have 
to build everything
* Flat file binary cache and copy to and from there


```
git clone -b master --depth 1 https://github.com/NixOS/nixpkgs
nix copy --from https://cache.nixos.org/ --to file://$PWD/cache ./nixpkgs#hello
nix copy --from https://cache.nixos.org/ --to file://$PWD/cache ./nixpkgs#hello --eval-store ""

# Download into local store then copy to binary cache $PWD
nix copy --to file://$PWD/cache nixpkgs#hello
```


### During install
Add the `--substituters ""` option to remove all binary cache options
```bash
nixos-install --root $root --no-bootloader --no-root-passwd \
  --system ${config.system.build.toplevel} --channel ${channel} --substituters ""
```
### Including pre-populated nix store
**On build machine**
```
$ nix copy your-config file:///path/to/usbstick/folder
```

**On install machine**
```
$ nix copy file:///path/to/usbstick/folder your-config 
```


nixos-install will run `nix build --store /mnt` which won't be able to see what is in the installer 
nix store so we need to copy everything to `/mnt` using `nix copy --no-check-sigs --to local?root=/mnt /out`
* `nix copy --to /tmp/nix /run/current-system --no-check-sigs`


<!-- 
vim: ts=2:sw=2:sts=2
-->
