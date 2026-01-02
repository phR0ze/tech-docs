# LG G6 H872 <img style="margin: 6px 13px 0px 0px" align="left" src="../../../../data/images/logo_36x36.png" />

[See the the original work here](https://github.com/phR0ze/lglaf-h872) I'm documenting where I'm 
picking up to reset and install the newest LineageOS version that the LG G6 can support which is 
LineageOS 21 or Android 14. Since this device currently has LineangeOS 17 this will be a bit of a 
jump.

### Quick links
* [.. up dir](..)
* [Overview](#overview)
 
## Overview
Released in 2017 the `LG G6 H872` is just an inexpensive phone i.e. $150 in 2020 that is compatible 
with MINT mobile to get an actual phone number and not a VOIP number like republic wireless or other 
cheap providers.

* 5.7" 1440x2880 display
* 13MP Camera
* 3300mAh battery
* Snapdragon 821/4 GB RAM
* Android 9.0

**Special boot modes**
* `Recovery` - With device powered off `Vol Down` + `Power` until LG logo appears then release `Power` 
  for a second and hold it again until recovery comes up.
* `Fastboot` - With device powered off hold `Vol Up` + `Power`

## Step by step process
This device has long since been phased out but there is an old archived solution with support for 
LineageOS 21. I originally had installed Sky Hawk recovery so that's what I'll be using for the 
flashing.

### Install newest supported recovery
1. Download the latest bits from [Oddsolutions LG G6 h872](https://updater.oddsolutions.us/devices/h872/builds)
   1. Download `lineage-21.0-20250220-UNOFFICIAL-h872.zip`, `boot.img` and `recovery.img`
2. [Enable OEM unlocking](../README.md#enable-oem-unlocking)
3. Push image over to device
   1. Plug device into linux machine
   2. Check your device is listed `adb devices`
   3. Push flash file over: `adb push lineage-21.0-20250220-UNOFFICIAL-h872.zip /sdcard/`
3. Reboot into the current recovery
   1. Execute reboot: `adb -d reboot recovery`
4. Flash the newest supported LineageOS i.e. 21
   1. Hit the `Flash` button
   2. Select the `lineage-21.0-20250220-UNOFFICIAL-h872.zip`
   3. Confirm and wait for it to finish
   4. Reboot
