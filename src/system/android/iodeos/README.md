# iodeOS <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Note: I gave up on this for now because as iodeOS is only on Android 15 and will require major 
changes to my workflow that I'm not ready for yet i.e. switching off Google services.

iodeOS is an Android operating system freed from Google trackers. iodeOS is powered by LineageOS, an 
open source OS that expands the functionality and lifespan of mobile devices.

iodeOS:
* makes all internet connections visible and allows you to control them
* permanently runs a built-in adblocker that automatically blocks based on blacklists
* provides built-in parental control 
* uses F-Droid and Aurora Store for apps
* replaces Google's DNS with Quad9's in all parts of the system
* A-GPS patches to avoid leaking IMSI and SUPL
* captive portal switch from `connectivitycheck.gstatic.com` to `captiveportal.kuketz.de`
* NTP server change to `pool.ntp.org`

### Quick links
* [.. up dir](..)
* [Install iodeOS](#install-iodeos)

## Overview


* [Degoogle how to guides](https://blog.iode.tech/category/how-to-guides/)
* [Manual install steps](https://gitlab.iode.tech/ota/ota#how-to-flash-google-pixel-3-4-5-6-6a-6-pro-7-7a-7-pro-8-8-pro-9-9-pro-9-pro-xl)
* [App compatibility list](https://community.iode.tech/t/editable-list-apps-compatible-with-iodeos/808)
  * Fail
    * ChatGPT
    * Google Chrome

## Install iodeOS
I'll be using the Pixel 8 Pro through out this documentation as an example.
iodeOS has an [automated installer](https://iode.tech/installation/) that works on Windows and Linux 
but I prefer to do it the manual way.
 
### Versions
The Pixel 8 Pro ships with Android 14.

| iodeOS  | LineangeOS  | Android 
| ------- | ----------- | ----------------
| `5`     | `21`        | 14
| `6`     | `22`        | 15
| `7`     | `23`        | 16

**References**
* [Google Pixel 8 Pro - LineageOS Wiki](https://wiki.lineageos.org/devices/husky/)

### Upgrade Android to match target iodeOS
Similar to LineageOS you need to upgrade your phone to the version of Android that matches the target 
iodeOS version

1. Download Android 15
   1. lkj
2. Install Android 15
   1. 

### Prerequisites
1. On NixOS install `android-tools` to get `adb` and `fastboot`
2. Upgrade or downgrade your vendor provided Android version to match the LineageOS version



### Install

1. [Download the OTA zip](https://gitlab.iode.tech/ota/release/-/tree/master/husky)




## Google alternatives

### Open Camera
Freezes

### Google Email
[Gmail alternatives](https://blog.iode.tech/degoogle-your-life-mail/)
* Protonmail
  * Free plan comes with 500MB of storage 

### Google maps
Works without being logged in on iodeOS. Every time its used it will pop up a notification that 
Google Play Services needs to be updated, which of course iodeOS doesn't have.
Fix - simply turn off that type of notification

### Google Meet

### Google Drive

### Googlg Docs

### Google Sheets

### Google YouTube
[Watch YoutTube without being tracked](https://blog.iode.tech/how-to-watch-youtube-without-being-tracked/)

Android
* NewPipe
  * Parses the YouTube website
  * Doesn't need Google Play Services to run
Linux
* FreeTube 
