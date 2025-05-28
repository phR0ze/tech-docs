# Recovery <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />

### Quick links
* [.. up dir](..)
* [Disable drive](#disable-drive)
* [Disable display manager](#disable-display-manager)
* [chroot into target disk](#chroot-into-target-disk)
* [Troubleshooting Upgrades](#troubleshooting-upgrades)

### Linked pages

## Disble drive
A failed drive can be disabled during the boot process by:

1. Waiting until the system boots to recovery mode
2. Entering the root user's password to login
3. Disable the systemctl unit for the drive at runtime
   ```bash
   $ systemctl mask --runtime mnt-storage1.mount
   ```
4. Boot into the full system command line
   ```bash
   $ systemctl isolate multi-user.target
   ```
5. Login then remove the drive disabling
   ```bash
   $ systemctl unmask --runtime mnt-storage1.mount
   ```
6. Make your NixOS changes as your fully online at this point
7. Reboot the system
   ```bash
   $ reboot
   ```

## Disble display manager
Often it is X11 that gets messed up on an upgrade and you can get more information by disabling the 
display manger via ssh or chrooting in.

```bash
$ sudo systemctl disable lxdm
$ sudo reboot
```

## chroot into target disk
Some times your system won't boot. You can boot from a live usb and chroot into the target disk to 
troubleshoot the issue further

1. Plugin your live USB and boot from it
2. Determine your target disk with `lsblk`
3. Mount your target disk e.g. `sudo mount /dev/sda3 /mnt`
4. Chroot into the disk: `sudo arch-chroot /mnt`

### Read the system logs
```bash
$ journalctl --list-boots
```

# Troubleshooting Upgrades
All my issue happen on upgrades with Arch Linux. It isn't every time but it happens enough that I 
loath upgrades, which most likely makes it worse since I put off upgrading until I need some new 
software that needs a core lib upgraded to work correctly.

## 2024.01.31

### Failed to connect to system bus
Upon attempting to login in I was confronted by the popup message `Unable to contact settings server` 
and was unable to login. Similar to the error message https://github.com/neutrinolabs/xrdp/issues/2646

This began occuring after the latest upgrade of the system. The upgrade had failed with:
```
(16/30) Reloading system bus configuration...
Failed to reload-or-try-restart dbus.service: Unit dbus.service not found.
```

1. Boot into a recovery USB
2. Chroot into the target disk
3. Read the system logs `journalctl`
4. Notice failure messages
   ```
   systemd-logind.service: Failed with result 'exit-code'
   Failed to start User Login Management
   dbus.socket: Socket service dbus.service not loaded, refusing
   Failed to listen D-Bus System Message Bus Socket
   Failed to connect to system bus: No such file or directory
   ```
5. Disable the display manager `systemctl disable lxdm`
6. Reboot `sudo reboot`
7. Login then run `sudo systemctl status systemd-logind`
8. Check logs `journalctl -xeu systemd-logind.service` and got same messages
   ```
   Failed to connect to system bus: No such file or directory
   ```
9. Checking dbus `systemctl status dbus` got the same message
   ```
   Failed to connect to bus: No such file or directory
   ```

After doing some additional research I ran across https://bbs.archlinux.org/viewtopic.php?id=292174 
which aligns perfectly with everything I'm seeing. Apparently the `dbus-broker` style package isn't 
correctly switching over if you had the older daemon style dbus before. When switching over you also 
need the `dbus-broker-units` to be installed to get it working again. 

1. Reinstall `dbus-broker` to ensure its correct and `dbus-daemon` wasn't there
   ```bash
   $ sudo pacman -S dbus-broker
   ```
2. Install the units to allow systemd to set things up
   ```bash
   $ sudo pacman -S dbus-broker-units
   $ sudo systemctl status dbus
   ```
3. Reboot
   ```bash
   $ sudo reboot
   ```

### Thunar failed to start
The next thing I noticed when I checked the logs `journalctl -b` Thunar had failures on start 
due to missing `libjbig.so.2.1`.
```
/usr/bin/Thunar: error while loading shared libraries: libjbig.so.2.1: cannot open shared object file: No such file or directory
thunar.service: Main process exited, code=exited, status=127/n/a
thunar.service: Failed with result 'exit-code'.
Failed to start Thunar file manager.
```

Solution:
1. Search for which package owns that file
   ```bash
   $ sudo pkgfile -u
   $ sudo pkgfile libjbig.so.2.1
   extra/jbigkit
   ```
2. Install new missing dependency
   ```
   $ sudo pacman -S jbigkit
   ```

<!-- 
vim: ts=2:sw=2:sts=2
-->
