# Emulator <img align="left" width="48" height="48" src="../../../data/images/logo_256x256.png">

The Android Emulator is actually a downstream project from QEMU and adds the ability to boot Android 
devices. This means we can run it directly without the need for Android Studio which is slow and 
clunky.

**Resources**
* [Android Emulator](https://android.googlesource.com/platform/external/qemu/+/2db80f7c1921a6f5d48b998378e3792e16c968a4/README.md)
* [Android SDK folder structure](https://medium.com/michael-wallace/how-to-install-android-sdk-and-setup-avd-emulator-without-android-studio-aeb55c014264)
* [Android Emulator without Android Studio](https://medium.com/@yohan.ardiansyah90/how-to-run-android-emulator-for-development-without-android-studio-f0e73682af3a)

### Quick links
- [.. up dir](../README.md)
* [Install Emulator](#install-emulator)
  * [Install Android 7 API 24](#install-android-7-api-24)
  * [Install Android 11 API 30](#install-android-11-api-30)
* [Start Emulator](#start-emulator)
* [Vulkan graphics](#vulkan-graphics)

## Install Emulator
Android tooling uses JAVA 17 as of 2023.11.19

1. Install android dependencies
   ```bash
   $ sudo pacman -S protobuf android-tools android-udev jre17-openjdk-headless jdk17-openjdk kotlin gradle
   $ export JAVA_HOME=/usr/lib/jvm/java-17-openjdk
   ```
2. Download the commandline tools
   1. Navigate to [Android Studio downlaods](https://developer.android.com/studio)
   2. Scroll down to the bottom section and click the linux zip for the commandline tools
   3. Accept the terms and agreements
   4. Download the zip to `~/.android/cmdline-tools`
   4. Extract the contents of `commandlinetools-linux-10406996_latest.zip`
   5. Rename extracted dir `mv cmdline-tools latest`
   6. Set the android SDK home location; Note: [ANDROID_SDK* are only used by older apps](https://developer.android.com/tools/variables)
      ```bash
      $ PATH=$PATH:$HOME/.android/cmdline-tools/latest/bin
      $ export ANDROID_HOME=~/.android
      $ export ANDROID_SDK_HOME=$HOME
      $ export ANDROID_SDK_ROOT=$ANDROID_HOME
      ```
3. Install the emulator; ends up in `~/.android/emulator`
   ```bash
   $ sdkmanager emulator
   $ PATH=$PATH:$HOME/.android/emulator
   ```

### Install Android 7 API 24
1. Install tooling for specific [Android API](https://apilevels.com/)
   ```bash
   $ sdkmanager "platforms;android-24"
   $ sdkmanager "build-tools;24.0.3"
   $ sdkmanager "system-images;android-24;google_apis;x86_64"
   ```

2. Create an AVD emulator device
   ```bash
   $ avdmanager list device | grep pixel_4
   $ avdmanager create avd --name android24 --package "system-images;android-24;google_apis;x86_64" --device pixel_4_xl
   ```

### Install Android API 30
1. Install tooling for specific [Android API](https://apilevels.com/)
   ```bash
   $ sdkmanager "platforms;android-30"
   $ sdkmanager "build-tools;30.0.3"
   $ sdkmanager "system-images;android-30;google_apis;x86_64"
   ```

2. Create an AVD emulator device
   ```bash
   $ avdmanager list device | grep pixel_4
   $ avdmanager create avd --name android30 --package "system-images;android-30;google_apis;x86_64" --device pixel_4_xl
   ```

## Start Emulator
1. List out available virtual devices from both tools
   ```bash
   $ avdmanager list avd
   $ emulator -list-avds
   ```

2. Start your virtual device
   ```bash
   $ emulator -avd android30
   ```

## Delete Emulator
```bash
$ avdmanager delete avd -n android30
```

## Vulkan graphics
Vulkan is a modern cross-platform 3D graphics API designed to minimize abstraction between device 
graphics hardware and your game. Vulkan is the primary low-level graphics API on Android replacing 
OpenGL ES. All Android 10 (API level 29) and higher devices support Vulkan 1.1 which is 85% of the 
market.

**References**
* [Vulkan Android docs](https://developer.android.com/games/develop/use-vulkan)

## Other Emulators
As of 2024 there are more options for emulating Android

### WayDroid
A fork of Anbox it runs Android in a container using LXC. Waydroid offers:

* Super easy to install and get running
* [Installation instructions](https://docs.waydro.id/usage/install-on-desktops)

### Android-x86 Emulator
  * Install from ISO in Virtual Box
  * Wi-Fi and Bluetooth

### Bliss OS
  * ISO based install

### Anbox
Deprecated and lacks Google Play Store support

<!-- 
vim: ts=2:sw=2:sts=2
-->
