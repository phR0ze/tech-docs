# Desktop Environments

### Quick links
* [Performance](#performance)
  * [XFCE vs LXDE](#xfce-vs-lxde)
  * [XFCE vs KDE Plasma](#xfce-vs-kde-plasma)
* [Budgie Desktop](#budgie-desktop)
* [Cinnamon Desktop](#cinnamon-desktop)
* [Deepin Desktop](#deepin-desktop)
* [KDE Plasma Desktop](#plasma-desktop)
* [Lumina Desktop](#lumina-desktop)
* [LXDE Desktop](#lxde-desktop)
* [LXQt Desktop](#lxqt-desktop)
  * [LXQt splash](#lxqt-splash)
* [MATE Desktop](#mate-desktop)
* [System76 Desktop](#system76-desktop)
* [XFCE Desktop](#xfce-desktop)

To compare the different ***Desktop Environments*** I'll first attempt to break down their features
into common groups I can compare and contrast.

|                | Display Mngr | Window Mngr | Desktop Mngr | Panel | Session Mngr | App Finder | File Mngr | Settings |
| -------------- | ------------ | ----------- | ------------ | ----- | ------------ | ---------- | --------- | -------- |
| Budgie         |              |             |              |       |              |            |           |          |
| Cinnamon       |              |             |              |       |              |            |           |          |
| Deepin         |              |             |              |       |              |            |           |          |
| Englightenment |              |             |              |       |              |            |           |          |
| GNOME          |              |             |              |       |              |            |           |          |
| Pantheon       |              |             |              |       |              |            |           |          |
| Plasma         |              |             |              |       |              |            |           |          |
| LXDE           |              |             |              |       |              |            |           |          |
| LXQt           |              |             |              |       |              |            |           |          |
| MATE           |              |             |              |       |              |            |           |          |
| XFCE           |              |             |              |       |              |            |           |          |

## Performance
Performance comes down to the amount of resources consumed by the desktop environment that is then
not available for actual user applications. 

### XFCE vs LXDE

### XFCE vs KDE Plasma
According to [Jason Evangelho](https://www.forbes.com/sites/jasonevangelho/2019/10/23/bold-prediction-kde-will-steal-the-lightweight-linux-desktop-crown-in-2020/#5562672f26d2) the latest version of `KDE Plasma 5.17` are
approaching a similar usage as XFCE, which has long been heralded for its low resource usage, with
`KDE Plasma 5.17` using `~503MB` and `XFCE` using `525MB`. Another test using a more feature rich
build puts `XFCE => 949MB` and `KDE => 957MB`

# Budgie Desktop
The [Budgie Desktop](https://docs.buddiesofbudgie.org) is developed independently from a specific 
Linux distribution but can be found easily on NixOS, Ubuntu, Arch Linux, Fedora and Solus. I'm 
evaluating it using the NixOS distro.

You can also install it manually with:
```
services.xserver.enable = true;
services.xserver.desktopManager.budgie.enable = true;
services.xserver.displayManager.lightdm.enable = true;
```

## Apps
* Document viewer: Atril
* Image viewer: Eye of MATE

## Desktop Icons
* Decent looking desktop icons with clean right click `Open in Terminal` option

## File Manager
* Nemo
* Looks very clean and intuitive
* Sidebar similar to windows with `My Computer`, `Devices` and `Network` options, but nice

## Login Manager
Seems to be using lightdm I think

* Simple but elegant login screen

## Panels
* Budgie Menu shows you all your installed applications, neatly organized into categories to improve 
discoverability, and with lightning fast application searching.

## Screen Shot
* Budgie screen shot
* Shows preview of captured image before saving
* Pretty simple but elegant and modern looking
  * Same features as XFCE

## Theme
* Nice looking high contrast dark and white theme


# Cinnamon Desktop
The [Cinnamon Desktop](https://en.wikipedia.org/wiki/Cinnamon_(desktop_environment) is a fork of
`GNOME 2`. I'm evaluating it via the [Linux Mint](https://linuxmint.com/) distro.

# Deepin Desktop

# KDE Plasma Desktop
Evaluating via [Kubuntu](https://kubuntu.org/) distro.


# Lumina Desktop

# LXDE Desktop
LXQt is the continuation of LXDE

# LXQt Desktop
[LXQt](https://github.com/lxqt/lxqt/wiki) is a lightweight Qt desktop environment that still aims to
have a modern look and feel. I'm evaluating it via the [Lubuntu](https://lubuntu.me/) distro.

Resources:
* [LXQt - Arch Wiki](https://wiki.archlinux.org/title/LXQt)

## LXQt splash
<img src="../docs/images/lxqt/splash.jpg">

## LXQt display manager
<img src="../docs/images/lxqt/display-manager.jpg">

## LXQt power management
<img src="../docs/images/lxqt/power-management.jpg">

## LXQt lock screen
The lock screen turned into a screen saver once it set for awhile
<img src="../docs/images/lxqt/lock-screen.jpg">
<img src="../docs/images/lxqt/lock-screen2.jpg">

## LXQt accessories
**FeatherPad - simple notepad clone**
<img src="../docs/images/lxqt/accessories-featherpad.jpg">

**KCalc - simple calculator**
<img src="../docs/images/lxqt/accessories-kcalc.jpg">

**LXQt Archiver - derived from File Roller of Gnome**
<img src="../docs/images/lxqt/accessories-lxqt-file-archiver.jpg">

**QtPass - basic UI frontend to pass unix password manager**
<img src="../docs/images/lxqt/accessories-qtpass.jpg">

  * PCManFM-Qt File Manager
  * TeXInfo
  * Vim
  * compton
  * nobleNote
  * picom

**LXQt Image Viewer - simple imager viewer slightly more advanced than gpicview**
<img src="../docs/images/lxqt/graphics-image-viewer.jpg">

* Graphics
  * LXImage
  * LibreOffice Draw
  * ScreenGrab
  * Skanlite
* Internet
  * BlueDevil Send File
  * BlueDevil Wizard
  * Firefox
  * Quassel IRC
  * Transmission (Qt)
  * Trojita
* Office
  * LibreOffice
  * LibreOffice Calc
  * LibreOffice Draw
  * LibreOffice Impress
  * LibreOffice Match
  * LibreOffice Writer
  * Trojita
  * qpdfview
* Sound & Video
  * K3b
  * PulseAudio Volume Control
  * VLC
* System Tools
  * BlueDevil Send File
  * BlueDevil Wizard
  * Discover
  * Fcitx
  * Htop
  * KDE Partition Manager 
  * Muon Package Manager 
  * QTerminal
  * QTerminal drop down
  * Startup Disk Creator
  * qps
* Preferences
  * LXQt settings
  * Additional Drivers
  * Advanced Network Configuration
  * Alternatives Configurator
  * Apply Full Upgrade
  * Fcitx Configuration
  * Input Method
  * Printers
  * Screensaver
* About LXQt
* Leave
  * Hibernate
  * Leave
  * Logout
  * Reboot
  * Shutdown
  * Suspend
* Lock Screen

## PCManFM-Qt

* File Manager
* Desktop wallpaper
* Desktop folder browsing icons
* Desktop Application Launcher Icons
* Right click for desktop preferences and 

## Power Control

## Volume Control

## Removable Media

# MATE Desktop
The [MATE Desktop](https://mate-desktop.com/) is a fork of `GNOME 2`. I'm evaluating it via the
[Ubuntu MATE](https://ubuntu-mate.org/) distro.

# XFCE Desktop
[XFCE](https://www.xfce.org/about) is a lightweight GTK desktop environment. I'm evaluating it via
the [XUbuntu](https://xubuntu.org/download) distro.

* GTK+ toolkit

<!-- 
vim: ts=2:sw=2:sts=2
-->
