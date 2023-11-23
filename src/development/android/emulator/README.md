# Emulator

## Quick links
- [Overview](#overview)

## Overview
**Resources**
* [Android Emulator](https://android.googlesource.com/platform/external/qemu/+/2db80f7c1921a6f5d48b998378e3792e16c968a4/README.md)
* [Android SDK folder structure](https://medium.com/michael-wallace/how-to-install-android-sdk-and-setup-avd-emulator-without-android-studio-aeb55c014264)
* [Android Emulator without Android Studio](https://medium.com/@yohan.ardiansyah90/how-to-run-android-emulator-for-development-without-android-studio-f0e73682af3a)

The Android Emulator is actually a downstream project from QEMU adding the ability to boot Android 
devices.

### Install Emulator
1. Install android dependencies
   ```bash
   $ sudo pacman -S android-tools android-udev jre11-openjdk-headless jdk11-openjdk
   ```
2. Download the commandline tools
   1. Navigate to https://developer.android.com/studio
   2. Scroll down to the botto section and click the linux zip for the commandline tools
   3. Accept the terms and agreements
   4. Extract the contents of `commandlinetools-linux-10406996_latest.zip` to `~/android/sdk`
   5. Move extracted file `mv cmdline-tools ~/android/sdk/tools`
3. Set the android home location
   ```bash
   $ export ANDROID_HOME=~/android/sdk
   $ export ANDROID_SDK_ROOT=$ANDROID_HOME
   ```



2. Install android emulator
   ```bash
   $ yay -Ga android-emulator
   $ cd android-emulator
   $ makepkg -s
   $ sudo pacman -U android-emulator-32.1.15-1-x86_64.pkg.tar.zst
   ```
3. Install android sdk to get the `avdmanager` and `sdkmanager`
   ```bash
   $ yay -Ga android-sdk
   $ cd android-sdk
   $ makepkg -s
   $ sudo pacman -U android-sdk-26.1.1-2-x86_64.pkg.tar.zst
   $ PATH=$PATH:/opt/android-sdk/tools/bin
   ```
4. Install android platform tools
   ```bash
   $ yay -Ga android-sdk-platform-tools
   $ cd android-sdk-platform-tools
   $ makepkg -s
   $ sudo pacman -U android-sdk-platform-tools-34.0.5-2-x86_64.pkg.tar.zst
   ```
5. Install android 11 SDK 30 platform
   ```bash
   $ yay -Ga android-platform-30
   $ cd android-platform-30
   $ makepkg -s
   $ sudo pacman -U android-platform-30-30_r03-1-any.pkg.tar.zst
   ```
6. Create an AVD emulator device
   ```bash
   $ avdmanager create avd --name android30 --package "system-images;android-30;google_apis;x86_64"
   ```

### Start Emulator
```bash
$ emulator -avd Pixel_4a_API_30
```

<!-- 
vim: ts=2:sw=2:sts=2
-->
