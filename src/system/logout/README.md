# Logout

Most linux desktop environments have their own integrated logout functionality which is much the same
providing a small window with a list of buttons with small text on them for the various logout
functions. Some may offer configurability as to the commands but many do not.

I much prefer large icons and large text in a splash overlay that completely covers the screen. To
date I've only found a few but `oblogout` seems to have been the first.

### Quick links
* [arcolinux-logout](#arcolinux-logout)
* [oblogout](#oblogout)

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

<!-- 
vim: ts=2:sw=2:sts=2
-->
