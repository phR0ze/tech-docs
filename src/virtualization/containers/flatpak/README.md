# Flatpak <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Containers aim to alleviate the ***runs on my machine*** problem that developers face. ***Flatpak***, 
***AppImage*** and ***Snaps*** are ***universal packages*** and aim to solve this problem as well. 
Although they are both solving the same problem they don't necessarily use the same technologies 
despite behaving in a similar way. Universal apps are not tied to the underlying linux distro's 
package manager and can run on any distro that supports universal packages.

Snaps are slower than AppImage and Flatpak while Flatpak and AppImage are about the same with 
AppImage being slightly faster. Generally Snaps have a negative acceptance in the community due to 
the Snap store being carefully controlled by Canonical and automatically upgrading your packages i.e. 
reduced user control.

### Quick links
* [.. up dir](../README.md)
* [Flatpak Overview](#flatpak-overview)
  * [Install flatpak](#install-flatpak)
* [Config flatpak](#config-flatpak)
  * [Update flatpak database](#update-flatpak-database)
  * [Search for flatpak package](#search-for-flatpak-package)
  * [List flatpak packages](#list-flatpak-packages)
  * [Install/Update flatpak package](#install-update-flatpak-package)
  * [Run flatpak package](#run-flatpak-package)
* [Build flatpak packages](#build-flatpak-packages)

# Flatpak Overview
[Flatpak](https://flatpak.org/) provides the ability to have Linux apps that run anywhere by
including the applications dependencies with the application. It also runs them in a sandboxed 
isolated environment that makes them more secure.

**Resources**
* [Flatpak docs](https://docs.flatpak.org/en/latest/)
* [Arch Linux docs](https://wiki.archlinux.org/index.php/Flatpak)
* [Comparision of Flatpak](https://github.com/AppImage/AppImageKit/wiki/Similar-projects#comparison)

### Install flatpak

**NixOS**
```nix
environment.systemPackages = with pkgs; [
  pkgs.flatpak
];
services.flatpak.enable = true;
xdg.portal.enable = true;
```

**Arch linux**
```bash
$ sudo pacman -S flatpak flatpak-builder elfutils patch xdg-desktop-portal-gtk fakeroot fakechroot
```
## Config flatpak
Binaries are installed to `/var/lib/flatpak/exports/bin` which gets added to the path by
`/etc/profile.d/flatpak-bindir.sh`. Configuration is stored at `~/.var/app/APP_NAME`

### Granting permissions
Install `Flatseal` to set permissions for other flatpak apps

### Update flatpak database
By default flatpak comes preconfigured to use its upstream `flathub` repo but we need to update the
local database to reflect what is there.

```bash
$ flapak update
```

### Search for flatpak package
```bash
$ flatpak search libreoffice
```

### List flatpak packages

**List installed**
```bash
$ flatpak list
```

**List Remote**
```bash
$ flatpak remote-ls
```

### Install/Update flatpak package

**Install package**
```bash
$ flatpak install <name>
```

**Update package**
```bash
$ flatpak update <name>
```

### Run flatpak package
```bash
$ flatpak run <name>
```

### Uninstall flatpak package

**Standard uninstall**
```bash
$ flatpak uninstall <name>
```

**Remove orphans**
```bash
$ flatpak uninstall --unused
```

### Viewing sandbox permissions
Flatpak applications come with predefined sandbox rules which defines the resources and file system
paths the application is allowed to access.

**View permissions**
```bash
$ flatpak info --show-permissions <package>
```

## Build flatpak packages
* warcraft 2
* hedgewars
