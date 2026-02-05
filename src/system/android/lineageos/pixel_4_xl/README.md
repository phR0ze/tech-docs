# Pixel 4 XL <img style="margin: 6px 13px 0px 0px" align="left" src="../../../../data/images/logo_36x36.png" />

Instructions for installing LineageOS 23 i.e. Android 16 on my Pixel 4 xl

WIP: starting from my Pixel 8 Pro instructions

### Quick links
* [.. up dir](..)
* [Overview](#overview)
  * [Versions](#versions)
* [Unlock bootloader](#unlock-bootloader)
  * [First test SIM card](#first-test-sim-card)
  * [Enable OEM unlocking](#enable-oem-unlocking)
  * [Bootloader unlock process](#bootloader-unlock-process)
* [Installation](#installation)
  * [Upgrade Android](#upgrade-android)
* [Flash LineageOS to Device](#flash-lineageos-to-device)
* [Root LineageOS](#root-lineageos)
* [Config LineageOS](#config-lineageos)
  * [Use Google Apps](#use-google-apps)

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

## Installation
I'll be using the Pixel 8 Pro in my installation steps and [LineageOS Wiki install guide]()

**References**
* [Google Pixel 8 Pro - LineageOS Wiki](https://wiki.lineageos.org/devices/husky/)

### Upgrade Android
The first thing you'll want to do is upgrade to the latest Android version that corresponds with the 
LineageOS version that your using. For Pixel devices it is not necessary to upgrade to intermediate 
versions first. For example the Pixel 8 Pro shipped with Android 14 and I'll be jumping straight to 
Android 16.

1. First [unlock the bootloader](bootloader-unlock-process)

4. Download the Android image
   1. Navigate to [Developers Android Images](https://developers.google.com/android/images)
   2. Search for `"husky" for Pixel 8 Pro`
   2. Download Android 16 latest `BP4A.251205.006, Dec 2025`
      * File is named `husky-bp4a.251205.006-factory-1ffce50c.zip`
   3. Unzip the OTA image

5. Flash the new Android image
   1. Switch the the OTA image directory
   2. Run `./flash-all.sh`
   3. Once finished the device will boot into the new Android

6. ***WARNING*** upgrading any Pixel Android 14 phone requires that you also do a Full OTA upgrade as 
   well to avoid potentially bricking the device. This is because the Android 14 bootloader had a 
   vulnerability that is being patched and the anit-rollback is being enforced. Thus we need to flash 
   both the A and B upgrade slots have a bootable image.
   1. First [test the SIM card in the new Android](first-test-sim-card)
   2. Navigate to [OTA images](https://developers.google.com/android/ota) which is different than 
      just images
   3. Download the one corresponding to `BP4A.251205.006, Dec 2025`
      * File is named `husky-ota-bp4a.251205.006-c5c57836.zip`
   4. Repeat the [enable OEM unlocking](../rooting/README.md#enable-oem-unlocking) instructions
   5. Plug your device into your computer using a USB cable
   6. On the phone check the box for `Allow USB debugging?` then hit `Allow`
   7. Open a shell and run `adb devices` and ensure your device is listed
   8. Reboot the phone into the bootloader, run: `adb reboot recovery`
   9. Hold the `Power` button and press `Up Volume` to get options
   10. Scroll down to `Apply update from ADB` using the `Down Voume` button then `Power` to select it
   11. On your PC navigate to where the OTA image was downloaded
   11. Run: `adb sideload husky-ota-bp4a.251205.006-c5c57836.zip`
   12. Once complete boot the system into Android OS ensuring it boots successfully

### Flash LineageOS to Device
Once the prerequisite [upgrade of Android](#upgrade-android) has been done then we can move on to 
switching over to LineageOS.

1. Download the images artifacts
   1. Navigate to [LineangeOS husy images](https://download.lineageos.org/devices/husky)
   2. Download:
     * `boot.img`
     * `dtbo.img`
     * `vendor_boot.img`
     * `vendor_kernel_boot.img`
     * `lineage-23.0-20251205-nightly-husky-signed.zip`
3. Reboot into bootloader `adb reboot bootloader` then run
   ```bash
   $ fastboot flash boot boot.img
   $ fastboot flash dtbo dtbo.img
   $ fastboot flash vendor_kernel_boot vendor_kernel_boot.img
   $ fastboot flash vendor_boot vendor_boot.img
   ```
4. Reboot into new Lineage Recovery `adb reboot recovery`
5. Pepare for the Lineage OS install
   1. Tap `Factory Reset`
   2. `Format data/factory reset` and confirm
   3. Return the the main menu
6. Sideload the LineageOS `.zip` but do not reboot
   1. On the device tap `Apply Update` then `Apply from ADB`
   2. On PC run: `adb -d sideload lineage-23.0-20251205-nightly-husky-signed.zip`
   3. Reboot to recovery to start installing add-ons
7. Install Google add-ons
   1. Navigate to [the google apps](https://wiki.lineageos.org/gapps/) page
   2. Download the `arm64` version for `LineageOS 23 (Android 16)`
   3. On the device tap `Apply Update` then `Apply from ADB`
   4. On PC run: `adb -d sideload MindTheGapps-16.0.0-arm64-20250812_214353.zip`
   5. On the device it will say signature validation failed, tap `Yes` to continue
   6. Navigate back to the main menu and tap `Reboot system now`

## Root LineageOS
LineageOS won't pass integrity checks that app developers might use such as:
* `Play Integrity`
* `SafetyNet (deprecated)`

**References**
* [Play Integrity](https://wiki.lineageos.org/quirks/snet/)

The solution is to root LineageOS and supply mods to fix this issue.

### Magisk
Billed as the [Magic Mask for Android](https://github.com/topjohnwu/Magisk), Magisk provides a suite 
of open source software for customizing Android.

Magisk features include:
* ***MagiskSU*** which provides root access for applications
* ***Magisk Modules*** modify read-only partitions by installing modules
* ***MagiskBoot*** provides tooling for unpacking and repacking Android boot images
* ***Zygisk*** can run code in every Android application's processes

### SafetyNet Fix
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

## Config LineageOS

### Use Google Apps
As much as I'm annoyed and frustrated by Google's spying and the security issues around Google Apps 
they still offer the best experience and I'm not ready to move off of them yet.

* `Google Calendar`
  * Includes Google Maps and Meet integration
* `Google Gmail`
  * Better integration with Google Gmail
* 
