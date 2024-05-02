# Android

### Quick links
* [General procedures](#general-procedures)
  * [Load SIM card](#load-sim-card)
  * [OEM unlocking is greyed out](#oem-unlocking-is-greyed-out)
  * [Factory reset](#factory-reset)
  * [Enable Developer mode](#enable-developer-mode)
  * [Allow OEM unlocking and USB debugging](#allow-oem-unlocking-and-usb-debugging)
  * [Opting out of the Beta program](#opting-out-of-the-beta-program)
* [Rooting](rooting/README.md)
* [New phone configuration](#new-phone-configuration)

## New phone choice
My must have features for a new phone in my opinion:
* Unlocked
* Wireless charging
* Supported by LineageOS

Usually this leads me to the Google Pixel line as the Pro versions all have wireles charging are 
easily unlocked and are well supported by LineageOS.

## New phone setup
This walk through is being done with the Google Pixel 6 Pro a.k.a `raven`

* OS support ***LineageOS 21/Android 14***
* [LineageOS Device Wiki page](https://wiki.lineageos.org/devices/raven/)

### 1. Ensure device is functional as is
1. [Load your SIM card into your new phone and boot it](#load-sim-card)  
2. Ensure everything is working with the Android that came with it  
   1. Send and receive SMS  
   2. Make and receive phone calls  
   3. Use Wifi and LTE both  
   4. Use Camera and Video  

### 2. Flash Stock Android 14
WIP not tested

1. [Allow OEM unlocking and USB debugging on your device](#allow-oem-unlocking-and-usb-debugging)
2. Download the factory Android 14 image for your device
   1. Navigate to https://developers.google.com/android/images
   2. Scroll down to the `Acknowledge` button and click it to proceed
   3. Scroll down to the last build for the `"raven" for Pixel 6 Pro` section
   4. Download [14.0.0 (AP1A.240405.002, Apr 2024)](https://dl.google.com/dl/android/aosp/raven-ap1a.240405.002-factory-877e050e.zip)
   5. Unzip: `unzip raven-ap1a.240405.002-factory-877e050e.zip`
   6. Change directory: `cd raven-ap1a.240405.002`
3. Flash the factory image to the device
   2. Start the device in `fastboot` mode
      ```bash
      $ adb reboot bootloader
      ```
   3. Flash the bootloader to the inactive slot named `other`. this is not a placeholder
      ```bash
      $ fastboot --slot=other flash bootloader bootloader-raven-slider-1.2-8739948.img
      ```
      If the flash was successful this command will print `OKAY [ ... ]` ***Do not proceed*** unless 
      this is the case.
   4. After flashing the inactive slot, reboot to that slot to ensure that the bootloader will be 
      marked as bootable. Run the following exactly.
      ```bash
      $ fastboot set_active other
      $ fastboot reboot bootloader
      $ fastboot set_active other
      $ fastboot reboot bootloader
      ```
   5. Finally reboot to the current OS
      ```bash
      $ fastboot reboot
      ```


      $ fastboot flashing unlock


## General procedures

### Load SIM card
1. Power down the phone
2. Plug in the SIM card
3. Power the phone back up

### OEM unlocking is greyed out
1. Power down and remove SIM
2. Factory reset your phone
3. Initiate setup enabling Wi-Fi
2. Re-enable developer mode
   * if OEM unlocking is still greyed out there is no workaround

### Factory reset
1. Navigate to ***Settings >System >Reset options***
2. Select ***Erase all data (factory reset)***

### Enable Developer mode
1. Navigate to ***Settings >About phone***
2. Swipe to the bottom and tap the ***Build number*** 7-8 times until it prompts your for your 
   password and says ***you are now a developer***

### Allow OEM unlocking and USB debugging
1. First [enable developer mode](#enable-developer-mode)
2. Navigate to ***Settings >System >Developer options***
3. Toggle on ***OEM unlocking***
   * [OEM unlocking is greyed out](#oem-unlocking-is-greyed-out)
3. Toggle on ***USB debugging***

### Opting out of the Beta program
1. Navigatge to https://www.google.com/android/beta
2. Loging with your google account associated with your device
3. Choose the Opt out option
   * ***WARNING*** you will then be required to downgrade to the latest stable version which will 
   erase all your data on your phone.

