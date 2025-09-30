# x11vnc <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

X11VNC is a VNC variant that allows for sharing the default display such that you can either remote 
in or simply work at the machiner locally and it doesn't matter. Nice feature.

**Summary**
* Pros
  * Just works most of the time
  * Allows for sharing the default display
* Cons
  * Slow like all VNC solutions
  * Requires you to autologin unless you use LightDM
* Result
  * Only use this if you can't get RustDesk to work

### Quick links
* [../](../README.md)
* [Overview](#overview)

## Overview
Some VNC servers like `x11vnc` and TigerVNC's `x0vncserver` make the primary X session available for 
remote connections. This is nice as some display managers like Lightdm provide the ability to use
VNC for incoming remote logins thus you don't have to leave your system logged in to work. Otherwise 
you need to rely on the autologin on boot to get VNC to work properly.

**References**
* [Arch linux docs - lightdm](https://wiki.archlinux.org/title/LightDM)
* [Arch linux docs - x11vnc](https://wiki.archlinux.org/title/X11vnc)
* [Super user post](https://superuser.com/questions/1718646/xvnc-and-lightdm-connect-to-primary-display-0-and-login-remotely)
* [Super user x11vnc](https://superuser.com/questions/1375625/vncviewer-connect-to-primary-display-of-lightdm-on-debian-jessie-share-with-phy)

### Pro tips
* LightDM allows you to login via x11vnc
  * x11vnc doesn't require any special configuration for lightdm.
* Display emulator dongle will keep the GPU running and give you better performance
* Turning off compositing will help: `xfconf-query -c xfwm4 -p /general/vblank_mode -s off`

### NixOS install
See [x11vnc - nixos-config](https://github.com/phR0ze/nixos-config/blob/main/options/services/raw/x11vnc.nix)
