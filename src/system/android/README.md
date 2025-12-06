# Android <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />

### Quick links
* [.. up dir](..)
* [New phone choice](#new-phone-choice)
* [New phone setup](#new-phone-setup)
* [New phone configuration](#new-phone-configuration)

### Linked pages
* [Rooting](rooting/README.md)

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

### Ensure device is functional as is
1. [Load your SIM card into your new phone and boot it](#load-sim-card)  
2. Ensure everything is working with the Android that came with it  
   1. Send and receive SMS  
   2. Make and receive phone calls  
   3. Use Wifi and LTE both  
   4. Use Camera and Video  

### Flash Stock Android 14
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


