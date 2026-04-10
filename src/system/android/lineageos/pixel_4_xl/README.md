# Pixel 4 XL <img style="margin: 6px 13px 0px 0px" align="left" src="../../../../data/images/logo_36x36.png" />

Instructions for installing LineageOS 23.2 i.e. Android 16 on my Pixel 4 XL

### Quick links
* [.. up dir](..)
* [Overview](#overview)
  * [Versions](#versions)
  * [Prerequisites](#prerequisites)
* [Installation](#installation)
  * [Upgrade Android](#upgrade-android)
  * [Flash LineageOS](#flash-lineageos)

## Overview

### Versions
The Pixel 4 XL shipped with Android 10 and Google officially supported it up to Android 13. LineageOS
has extended that support all the way to Android 16 with LineageOS 23.2.

| LineageOS   | Android
| ----------- | -----------------
| `17.1`      | 10
| `18.1`      | 11
| `19.1`      | 12L
| `20`        | 13
| `21`        | 14
| `22.1`      | 15
| `22.2`      | 15
| `23.0`      | 16
| `23.2`      | 16 (current)

### Prerequisites
* On NixOS you'll need to install `android-tools` to get `adb` and `fastboot`
* see [../README.md#unlock-bootloader](../README.md#unlock-bootloader)

## Installation
[Google Pixel 4 XL - LineageOS Wiki](https://wiki.lineageos.org/devices/coral/)

### Upgrade Android
The LineageOS 23.2 instructions require that we upgrade the Pixel 4 XL to the newest version of
Android 13 that is available for this device.

Note: it doesn't matter if you have an older Android version or and older LineageOS on your phone
already. The upgrade process below will wipe your phone and get Android 13 correctly installed.

1. Ensure you've already completed:
   * [Bootloader unlock process](../README.md#bootloader-unlock-process)

2. Download the Android image
   1. Navigate to [Developers Android Images](https://developers.google.com/android/images)
   2. Search for `"coral" for Pixel 4 XL`
   3. Download the last Android 13 release `13.0.0 (TP1A.221005.002.B2, Feb 2023)`
      * File is named `coral-tp1a.221005.002.b2-factory-<hash>.zip`
   4. Unzip the factory image

3. Flash the new Android image
   1. Enter [fastboot mode](../README.md#fastboot-mode) on the device
   2. Switch to the factory image directory on the PC
      ```bash
      cd coral-tp1a.221005.002.b2
      ```
   2. Execute the flash
      ```bash
      ./flash-all.sh
      ```
   3. The script will completely automate the process no intervention needed even if it looks like
      it. Once finished the device will boot into Android 13

4. Repeat [Bootloader unlock process](../README.md#bootloader-unlock-process) as needed

### Flash LineageOS
Once the prerequisite [upgrade of Android](#upgrade-android) has been done then we can move on to 
switching over to LineageOS.

1. Download the images artifacts
   1. Navigate to [LineangeOS coral images](https://download.lineageos.org/devices/coral/builds)
   2. Download:
     * `lineage-23.2-20260403-nightly-coral-signed.zip`
     * `boot.img`
     * `dtbo.img`
     * `super_empty.img`
     * `vbmeta.img`

2. Flash the Lineage hardware configuration i.e. `dtbo.img` from earlier
   1. Enter [fastboot mode](#fastboot-mode) on the device
   2. Now flash `fastboot flash dtbo dtbo.img`
   3. Reboot into bootloader again `fastboot reboot bootloader`

3. Install the Lineage Recovery i.e. `boot.img` from earlier
   1. You should already be in fastboot mode
   2. Flash recovery `fastboot flash boot boot.img`
   3. Use the `Vol up` button to switch to `Recovery Mode` then press `Pwr`
   4. LineageOS 23.2 recovery will load with purple theme

4. Factory reset your device from the Lineage Recovery
   1. Tap `Factory Reset` then `Format data / factor reset`
   2. Finally tap `Format data`
   3. Tap the back arrow at the top to go back to the main menu

5. Install LineageOS
   1. Tap `Apply update` then `Apply from ADB`
   2. On your PC run `adb -d sideload lineage-23.2-20260403-nightly-coral-signed.zip`
      * Note: watch the device not the PC as the PC won't show the correct output
   3. Once the device completes it will prompt you to reboot into recovery mode select `Yes`

6. Install Google add-ons
   1. Navigate to [the google apps](https://wiki.lineageos.org/gapps/) page
   2. Download the `arm64` version for `LineageOS 23 (Android 16)`
   3. On the device tap `Apply Update` then `Apply from ADB`
   4. On PC run: `adb -d sideload MindTheGapps-16.0.0-arm64-20260409_073023.zip`
   5. On the device it will say signature validation failed, tap `Yes` to continue
   6. Navigate back to the main menu and tap `Reboot system now`
   7. Your done. It will boot into LineageOS
