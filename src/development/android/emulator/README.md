# Emulator

## Quick links
- [Overview](#overview)

## Overview
**Resources**
* [Android Emulator](https://android.googlesource.com/platform/external/qemu/+/2db80f7c1921a6f5d48b998378e3792e16c968a4/README.md)

The Android Emulator is actually a downstream project from QEMU adding the ability to boot Android 
devices.

### Install Emulator
* [Guid with android and qemu](https://help.clouding.io/hc/en-us/articles/4405454393756-How-to-virtualize-Android-with-QEMU-KVM)

```bash
$ sudo pacman -S android-tools android-udev qemu-system-x86
$ yay -Ga android-emulator
$ cd android-emulator
$ makepkg -s
$ sudo pacman -U ?
```

<!-- 
vim: ts=2:sw=2:sts=2
-->
