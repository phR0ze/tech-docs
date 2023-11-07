# Distro comparison
Reviewing the various Linux distros that have caught my eye and the components they are composed
from.

All of the Distros below are being reviewed in a VirtualBox VM with 4GB of RAM and (2)
`Intel E5-2637 Cores`. Obviously this will be highly subjective and serves more as a reference for
myself.

### Quick links
* [Arch Linux Distros](#arch-linux-distros)
  * [Arco Linux - Openbox](#arco-linux-openbox)
  * [Hefftor Linux](#hefftor-linux)
* [Desktop Components](#desktop-components)
  * [Window managers](#window-managers)
    * [XFWM vs Openbox](#xfwm-vs-openbox)
  * [Logout screens](#logout-screens)
    * [arcolinux-logout](#arcolinux-logout)
    * [oblogout](#oblogout)
  * [Lock screens](#lock-screens)
    * [Betterlockscreen](#betterlockscreen)
    * [i3lock-color](#i3lock-color)
    * [i3lock](#i3lock)

# Arch Linux Distros

## Arco Linux - Openbox
**Review**
* The ArcoLinuxB version as is a live environment you can install from
* Booting from it gives a minimal Arch Linux type menu then it boots to live and launches Calamares
* Terminal - URXT with splash of scrot
* Arco specific software
  * arcolinux-logout
    * integrated with BetterLockScreen UI 
    * /usr/local/bin/arcolinux-logout
    * python3 /usr/share/arcologout/arcologout.py
  * ArcoLinux BetterLockScreen GUI
    * Brad Heffernan
    * /usr/local/bin/arcolinux-betterlockscreen
    * /usr/share/arcolinux-betterlockscreen/arcolinux-betterlockscreen.py
  * ArcoLInux Conky manager
    * conkyzen
  * Arco Tweak Tool
    * Desktop Installer
* App Finder - xfc4-appfinder
* xfce4-power-manager
* xfce-clipman
* exo-open --launch TerminalEmulator
  * will launch the preferred terminal
* Sound controls
  * volumeicon
  * pavucontrol
* Install software
  * pamac is a UI for pacman
* Change resolution
  * lxrandr
  * xfce4-display-settings
* xcape
* tint2
* xfc4-taskmanager
* xfce4-appearance
* lxappearance
* Variety
* Terminals
  * lxterminal
  * xfce4-terminal
  * termite
  * Urxvt
  * Alacritty
* Ristretto image viewer from XFC4
* USB image writer - mintstick
* Theme
  * GTK2/3: Arc-Dark
  * Icons: Sardi-Arc 
* Kavantum theme engine for KDE
* Archive Manager - file roller
* xfce4-terminal --drop-down
  * Strange behavior it disappears when you click out only
* NetworkManager
* sddm
* thunar
* screenshot
  * nicer looking than xfc4-screenshooter
  * identical features except xfc4-screenshooter can open in target app
* Openbox
  * Same issue resizing borders, super hard to catch on
* picom - standalone compositor fork of compton
* Has a delayed bootloader page
* Display manager login is set to tiny resolution by default

## Hefftor Linux
[Hefftor Linux](https://hefftorlinux.net/) popped up on my radar when I was searching for a better
lock screen. Apparently Brad Hefferman made his own GUI tool for the `Betterlockscreen` scripts which
got me curious about what hes was working on. His screen shots look really neat.

**Review**
* On first boot your presented with a similar looking menu to the original Arch Linux ISO.
* It boots into a full featured Live environment and runs the Calamares installer
  * The Calamares installer is clean and slick looking
* There is a nice compositing translucent affect that windows get when you move them
* The default theme and look and feel are subdued and pleasant
* Conky date and time in a neat font on desktop
* Terminal - URXVT with no scrollbar and higher than usual tranparencey
* xfce4-panel top right for tray icons
* App launcher - Plank at bottom
* XFW4 Window Manager 
* pavucontrol - launched from 
* risretto image viewer
* catfish file search
* grsync ui for rsync
* yad icon browser
* Fonts via XFC Appearance
  * Roboto Regular 10 for both Default and Monospace font
  * Enabled anti-aliasing
  * Hinting: Slight
  * Sub pixel order: RGB
  * Custom DPI: 96
  * Terminal font `Hack`

# Desktop Components

## Window managers

### XFWM vs Openbox
* Both wrote in C
* Openbox is standalone
* XFWM is part of a DE
* XFWM has a compositor built in for true transparency

## Logout screens
Most linux desktop environments have their own integrated logout functionality which is much the same
providing a small window with a list of buttons with small text on them for the various logout
functions. Some may offer configurability as to the commands but many do not.

I much prefer large icons and large text in a splash overlay that completely covers the screen. To
date I've only found a few but `oblogout` seems to have been the first.

### arcolinux-logout
[ArcoLinux-logout](https://aur.archlinux.org/packages/arcolinux-logout/) appears to be a clone of
`oblogout` and provides much the same options.

### oblogout
[Oblogout](https://aur.archlinux.org/packages/oblogout/) has been my go to in this area providing:
* Simple set of widgets overlayed in all white with icons for common controls
  * Logout, Shutdown, Reboot, Hibernate etc...
  * Each action could be clicked to activate or use a hot key
* The actual commands executed were configurable

It has since been moved out of main line into the AUR as its not being actively maintained and is
using python2 dependencies `python2-pillow` and `python2-dbus`

## Lock screens
After reviewing a few lock screen applications I've realized that the features I'd really want
ideally in a lock screen app are:

* Configureable thumbnail image gallery
* Choose image frrom from thumbnail image gallery
* Choose affects to apply to wallpaper and/or lockscreen
* Apply image to both wallpaper and lockscreen saving affected images in cache
* Allow for customized greeter overlay
* Ability to randomly choose wallpaper/lockscreen combination

This is really a combination of the traditional lockscreen and wallpaper applications

### Betterlockscreen
[Betterlockscreen](https://github.com/betterlockscreen/betterlockscreen) wraps `i3lock-color` using a
caching mechanism to apply all affects to a chosen image then cache it and have `i3lock-color`
display it so there isn't any delay while applying affects dynamically.

Betterlockscreen uses `imagemagick`, `xdpyinfo`, `xrandr`, `xrdb` and `xset` in order to
automatically detrmine the correct size and resolution an image should be then to apply affects to
the images and cache them to be then set as an i3lock-color background and optionally keep the
wallpaper in sync.

After research I found [ArcoLinux's project work](https://github.com/arcolinux/arcolinux-betterlockscreen)
on this to provide a GUI to make this easier.

### i3lock-fancy
[i3lock-fancy](https://github.com/meskarune/i3lock-fancy#static-image) is a wrapper script around the
venderable `i3lock-color` project. The script dynamically takes a screen shot and applies affects
than calls i3lock-color.

Adding a lock icon and text to the center of an image can be done statically with:
```bash
$ convert /path/to/background.png -font Audiowide -pointsize 50 -fill white -gravity center \
    -annotate +0+160 "Password required to unlock" lock.png -gravity center -composite newimage.png
$ i3lock -i newimage.png
```

### i3lock-color
[i3lock-color](https://github.com/Raymo111/i3lock-color) is the most popular i3lock fork providing a
number of enhancments:

* Multiple configurable colors for the unlock ring
* Blurring the current screen and using that for a background
* Showing a clock in the indicator
* Positioning the various UI elements
* Changing the ring radius and thickness as well as text size

### i3lock
[i3lock](https://github.com/i3/i3lock) is a simple screen locker that is independent of any desktop
environment and can be used standalone. Its a great idea but many have found it too spartan and has
spawned a number of forks with various feature enhancements.

Features:
* able to display image in background
* uses PAM and therefor has all PAM integrations
* shows simple circle that offers a little animation when password is entered

### Cinnamon-screensaver
Cinnamon-screensaver was my favorite for some time providing a simple image with a bluring layer and
the current time and date and would offer password entry when activated. However due to its numerous
cinnamon desktop dependencies I've abandoned it.

## Window Manager
***Window Managers*** used by Desktop Environments usually provide:
* Window placment on the screen
* Window decorations
* Workspace or virtual desktops

## Desktop Manager
***Desktop Managers*** used by Desktop Enviroments usually provide:
* Desktop Wallpaper
* Desktop root window menu
* Desktop application and file manager icons

## Panel
***Panels*** used by Desktop Environments usually provide:
* Switching between open windows
* Launch applications
* Switch workspaces
* Menu plugins to browse applications or directories

## Session Manager
***Session Managers*** used by Desktop Environments usually provide:
* Controls for login
* Power managment
* Multiple login sessions

## Application Finder
***Application Finders*** used by Desktop Environments usually provide:
* View of the applications installed in categories

## File Manager
***File Managers*** usec by Desktop Environments usually provide:
* File management
* Unique utilities like bulk renamer

## Settings Manager
***Settings Managers*** used by Desktop Environments usually provide:
* Tools to control various settings
* Keyboard shortcuts
* Appearance
* Display settings

* Appearance
  * Themes
* Desktop features
* Desktop settings
  * Desktop notifications

* Screenshooter
* PDF Viewer
* Multi-monitor management

## Desktop features
One of the core paradigms in the Desktop Environment is the visible desktop surface and your
interactions with it. Sometimes these features are abscent from a particular desktop environment,
handled by a standalone appliation or simply a feature of a larger application.

* Desktop wallpaper
* Desktop application launcher icons
* Desktop file management folder icons

## Desktop settings
Another typical desktop environment feature is 

<!-- 
vim: ts=2:sw=2:sts=2
-->
