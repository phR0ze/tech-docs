# Rooting <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

The primary reason to root an Android device is to be able to flash custom ROMS onto the device and 
remove unwanted bloat apps. Although root can be obtained in some cases without an unlocked 
bootloader I'm more interested in the typical path:

1. Unlock the bootloader
2. Flash a custom ROM that is also rooted

### Quick links
* [.. up dir](..)
* [Unlock bootloader](#unlock-bootloader)
  * [First test SIM card](#first-test-sim-card)
  * [Enable OEM unlocking](#enable-oem-unlocking)
  * [Unlock via USB](#unlock-via-usb)
* [Magisk](#magisk)
  * [SafetyNet Fix](#safetynet-fix)

## Unlock bootloader
Before attempting to unlock the bootloader first ensure your device works with your SIM card. To do 
so safely you'll want to reset the device then work through the first run wizard then install your 
SIM card and test that it works.

### First test SIM card
1. Work through first run wizard
   1. Select `Get started`
   2. Select `Skip` to avoid setting up with another phone
   3. Select `Set up offline` 
   4. Select `Skip`, `Continue` for rest

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

### Unlock via USB
1. 

## Magisk
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
