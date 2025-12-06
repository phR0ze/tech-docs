# LineageOS <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

LineageOS provides and open source alternative to cellphone vendor distributed Android versions. 

LineageOS:
* removes cellphone vendor garbage
* provides support for older devices including security patches
* includes community drive customization on top of Android

### Quick links
* [.. up dir](..)
* [Overview](#overview)
  * [Versions](#versions)
* [Installation](#installation)

## Overview

## Installation
I'll be using the Pixel 8 Pro in my installation steps and [LineageOS Wiki install guide]()

**References**
* [Google Pixel 8 Pro - LineageOS Wiki](https://wiki.lineageos.org/devices/husky/)

### Versions
The Pixel 8 Pro ships with Android 14

| LineangeOS  | Android 
| ----------- | -----------------
| `21`        | 14
| `22.1`      | 15
| `22.2`      | 15
| `23`        | 16

### Prerequisites
1. On NixOS install `android-tools` to get `adb` and `fastboot`
2. Upgrade or downgrade your vendor provided Android version to match the LineageOS version

### Enable OEM unlocking
1. Enable developer mode
   1. Navigate to `Settings >About phone`
   2. Scroll down to `Build number` and tap until unlocked and says `you are now a developer`

2. Allow OEM unlocking and USB debugging
   1. Navigate to `Settings >System >Developer options`
   2. Toggle on `OEM unlocking`
   3. Toggle on `USB debugging`

### Unlock bootloader
1. Plug your device into your compute using a USB cable
2. Open a shell and run `adb devices` and ensure your device is listed
3. With the device powered on run `adb reboot bootloader`
4. Now unlock the bootloader with `fastboot fashing unlock`

### Upgrade Android
The first thing you'll want to do is upgrade to the latest Android version that corresponds with the 
LineageOS version that your using. For Pixel devices it is not necessary to upgrade to intermediate 
versions first. For example the Pixel 8 Pro shipped with Android 14 and I'll be jumping straight to 
Android 16.

***WARNING:*** the Pixel 8 Pro released with Android 14 which has a vulnerable bootloader. Thus any 
new versions of Android contain a bootloader update that increments the anti-rollback version and 
block downgrades. Thus once you go forward you can't go back to Android 14. Additionally you need to 
flash both the A and B install slots to avoid bricking your phone. This means after flashing and 
booting into Android 16 you need to flash again to get the other slot.

1. [Install Prerequisites](#prerequisites)
2. [Enable OEM unlocking](#enable-oem-unlocking)
3. [Unlock the bootloader](#unlock-bootloader)

4. Download the Android image
   1. Navigate to [Developers Android Images](https://developers.google.com/android/images)
   2. Search for `"husky" for Pixel 8 Pro`
   2. Download Android 16 latest e.g. `BP4A.251205.006, Dec 2025`
   3. Unzip the OTA image

5. Flash the new Android image
   1. Switch the the OTA image directory
   2. Run the `flash-all` script

### Flash LineageOS to Device
This is the process to use for flashing LineageOS to your device.

