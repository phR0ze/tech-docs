# Wine <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />

Documentation for various wine related configurations

### Quick links
* [.. up dir](..)
* [Overview](#overview)
* [Install Wine](#install-wine)
  * [Install Wine for NixOS](#install-wine-for-nixos)
  * [Install Wine for Arch Linux](#install-wine-for-arch-linux)
* [Wine Configuration](#wine-configuration)
  * [Create new 32bit Wine Prefix](#create-new-32bit-wine-prefix)
  * [Delete a Wine prefix](#delete-a-wine-prefix)
  * [Reset dll overrides](#reset-dll-overrides)
* [Troubleshooting](#troubleshooting)
  * [Audio problems](#audio-problems)
  * [Video problems](#video-problems)
    * [Wine GLXBadFBConfig](#wine-glxbadfbconfig)
    * [Ensure you have 32-bit drivers](#ensure-you-have-32bit-drivers)

## Overview
Wine uses the `WINEPREFIX` as separate C-drive/registries. Typically you'll want one per app unless
the apps are sharing configuration or depend on each other.

## Install Wine

### Install Wine for NixOS
The `x86_64-linux` package supports 32bit and 64bit applications by default. On other platforms only 
the 32 bit version is supported.

**References**
* [Wine - NixOS docs](https://nixos.wiki/wiki/Wine)

Installing the latest 32bit and 64bit versions is the best path
```nix
{
  environment.systemPackages = with pkgs; [
    wineWowPackages.staging   # support both 32bit and 64bit applications
    winetricks                # support all versions
  ];
}
```

### Install Wine for Arch Linux
https://wiki.archlinux.org/index.php/wine

```bash
# Install Wine
$ sudo pacman -S winetricks wine-gecko wine-mono zenity

# Ensure 32-bit dependencies are installed
$ sudo pacman -S lib32-freetype2 lib32-libpulse lib32-mpg123 lib32-libusb
```

## Wine Configuration
In most cases you'll want to create a `wine prefix` per game so that you can have a custom wine 
configuration for each game. Wine prefixes are simply a folder where wine will store all the 
configuration files e.g. windows registry settings, windows version, target application files etc... 

### Create new 32bit Wine Prefix
Wine by default creates a single `~/.wine` prefix as 64bit but often its old Windows apps were trying
to run so we'll need a new 32bit prefix created.

```bash
$ WINEARCH=win32 WINEPREFIX=~/.wine/prefixes/<prefix> wincfg
```

### Delete a Wine prefix
Since a wine prefix is just a directory, all you need to do is delete the directory associated with
it. Typically they will be stored in `~/.wine` or `~/.wine/prefixes`.

```bash
$ rm -rf ~/.wine/prefixes/steam
```

### Reset dll overrides
```
$ WINEARCH=win32 WINEPREFIX=~/.wine/prefixes/warcraft2 winetricks alldlls=default
```

## Troubleshooting

### Audio problems
If you don't see your audio devices show up in a 32bit prefix under the `winecfg > Audio` tab its
probably because you need to install the 32bit drivers for your devices.

Install the pulse audio 32bit driver:
```bash
$ sudo pacman -S lib32-libpulse
```

### Video problems

#### Wine GLXBadFBConfig
[Wine GLXBadFBConfig bug](https://bugs.winehq.org/show_bug.cgi?id=50859#c11)

**Workaround:**
```bash
# Prefix your wine commands with the mesa environment variable override
$ MESA_GL_VERSION_OVERRIDE=4.5 WINEARCH=win32 ....
```

#### Ensure you have 32-bit drivers
If you get video issues in a 32 prefix ensure you have the 32bit drivers

Example:
```bash
$ sudo pacman -S lib32-nvidia-340xx-utils lib32-nvidia-340xx
```

<!-- 
vim: ts=2:sw=2:sts=2
-->
