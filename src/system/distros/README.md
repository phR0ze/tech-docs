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
