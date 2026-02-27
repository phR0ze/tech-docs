# Lock screens

### Quick links
* [Betterlockscreen](#betterlockscreen)
* [i3lock-fancy](#i3lock-fancy)
* [i3lock-color](#i3lock-color)
* [i3lock](#i3lock)
* [Cinnamon-screensaver](#cinnamon-screensaver)

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
