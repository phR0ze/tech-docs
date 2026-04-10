# LineageOS <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

LineageOS provides and open source alternative to cellphone vendor distributed Android versions. 

LineageOS:
* removes cellphone vendor garbage
* provides support for older devices including security patches
* includes community driven customization on top of Android

### Quick links
* [.. up dir](..)
* [Overview](#overview)
  * [Versions](#versions)
* [Unlock bootloader](#unlock-bootloader)
  * [First test SIM card](#first-test-sim-card)
  * [Enable OEM unlocking](#enable-oem-unlocking)
  * [Bootloader unlock process](#bootloader-unlock-process)
  * [Fastboot mode](#fastboot-mode)
* [Config LineageOS](#config-lineageos)
  * [Root LineageOS](#root-lineageos)
  * [Use Google Apps](#use-google-apps)

### Linked pages
* [LG G6 H872](lg_g6_h872/README.md)
* [Pixel 4 XL](pixel_4_xl/README.md)
* [Pixel 8 Pro](pixel_8_pro/README.md)

## Overview

### Versions
The Pixel 8 Pro ships with Android 14 and supports the latest which is Android 16 as of Dec 2025.

| LineangeOS  | Android 
| ----------- | -----------------
| `21`        | 14
| `22.1`      | 15
| `22.2`      | 15
| `23`        | 16

## Unlock bootloader
The primary reason to unlock your bootloader is to be able to flash custom ROMs such as LineageOS

### Prerequisites
1. On NixOS install `android-tools` to get `adb` and `fastboot`
2. Upgrade or downgrade your vendor provided Android version to match the LineageOS version

### First test SIM card
Before attempting to unlock the bootloader first ensure your device works with your SIM card. To do 
so safely you'll want to reset the device then work through the first run wizard then install your 
SIM card and test that it works.

1. Work through first run wizard
  * Android 14
    1. Select `Get started`
    2. Select `Skip` to avoid setting up with another phone
    3. Select `Set up offline` 
    4. Select `Skip`, `Continue` for rest
  * Android 16
    1. Select the big forward arrow
    2. Select `English`
    3. Select `Skip` to avoid set up using another device
    4. Select `Skip` to avoid Wi-Fi setup
    5. Select `Set up offline` to skip and `Continue`
    6. Set the date, pin and ignore everything else

2. Factory reset to ensure everything is kosher
   1. Navigate to `Settings >System >Reset options`
   2. Select `Erase all data (factory reset)`

3. Load SIM card
   1. Power down the phone
   2. Plug in the SIM card
   3. Power the phone back up

### Enable OEM unlocking
1. Enable developer mode
   1. Navigate to `Settings >About phone`
   2. Scroll down to `Build number` and tap until unlocked and says `you are now a developer`

2. Allow OEM unlocking and USB debugging
   1. Navigate to `Settings >System >Developer options`
   2. Toggle on `OEM unlocking`
   3. Toggle on `USB debugging`

### Bootloader unlock process
1. Prerequisites
   1. [Install Prerequisites](#prerequisites)
   2. [First test SIM card](#first-test-sim-card)
   3. [Enable OEM unlocking](#enable-oem-unlocking)
2. Activate debugging on your phone
   1. Plug your device into your computer using a USB cable
   2. On the phone check the box for `Allow USB debugging?`
   3. Then hit the `Allow`
3. Open a shell and run `adb devices` and ensure your device is listed
   * Ensure that it doesn't say `unauthorized` otherwise see step #1 again
4. Now reboot the phone into the bootloader
   1. With the device powered on run `adb reboot bootloader`
   * you should see the bootloader screen and in red `Fastboot Mode`
5. Now unlock the bootloader
   1. Run `fastboot flashing unlock`
   2. On the phone use the volume keys to switch to `Unlock the bootloader`
   3. Press the power button to activate that option
   4. Validate it worked, on pc run: `fastboot getvar unlocked`

### Fastboot mode
In order to apply images to your devices using the fastboot commands you first neeed to:

1. Ensure you android device is detected via adb
   1. Boot adb
      ```bash
      adb devices
      * daemon not running; starting now at tcp:5037
      * daemon started successfully
      List of devices attached
      99041FFBA005FN  unauthorized
      ```
   2. Tap `Authorize`` on the debug screen on the device
   3. Check adb again
      ```bash
      adb devices
      List of devices attached
      99041FFBA005FN  device
      ```
2. Boot into the boot loader (i.e. fastboot)
   ```bash
   adb reboot bootloader
   ```

## Config LineageOS

### Root LineageOS
TBD: doesn't seem to be a problem with Lineage 23.2

LineageOS won't pass integrity checks that app developers might use such as:
* `Play Integrity`
* `SafetyNet (deprecated)`

**References**
* [Play Integrity](https://wiki.lineageos.org/quirks/snet/)

The solution is to root LineageOS and supply mods to fix this issue.

#### Magisk
Billed as the [Magic Mask for Android](https://github.com/topjohnwu/Magisk), Magisk provides a suite 
of open source software for customizing Android.

Magisk features include:
* ***MagiskSU*** which provides root access for applications
* ***Magisk Modules*** modify read-only partitions by installing modules
* ***MagiskBoot*** provides tooling for unpacking and repacking Android boot images
* ***Zygisk*** can run code in every Android application's processes

#### SafetyNet Fix
[kdrag0n's safetynet-fix](https://github.com/kdrag0n/safetynet-fix) leverages Magisk's modules to 
work around Google's SafetyNet and Play Integrity attestation. This is useful for allowing apps to 
work that normally would fail on rooted devices.

1. Download and install the Magisk Manager
2. Navigate to `Settings`
   1. Choose `Hide Magisk app`
   2. Enable `Zygisk`
   3. Enable `Enforce DenyList`
   4. Select `Configure DenyList` and add all apps that would normally fail on rooted devices
3. Navigate to Modules
   1. Install `Systemless Hosts`
   2. Install `Universal SafetyFix` v2.4.0 or newer by `kdrag0n`

### Use Google Apps
As much as I'm annoyed and frustrated by Google's spying and the security issues around Google Apps 
they still offer the best experience and I'm not ready to move off of them yet. As such I now need to
switch over to use them instead of the default LineageOS apps.

* `Google Calendar`
  * Includes Google Maps and Meet integration
* `Google Gmail`
  * Better integration with Google Gmail

### Google Family Link
see [Google Family Link > Linking a child device](../../../networking/google_family_link/README.md#linking-a-child-device)

