# Android Studio <img align="left" width="48" height="48" src="../../data/images/logo_256x256.png">

Android Studio is slow and clunky. I'd recommend avoiding it if at all possible. I'm including my 
outdated instructions for setup I used with Arch Linux.

### Quick links
- [.. up dir](../README.md)
* [Install Android Studio](#install-android-studio)
* [Update Android Studio](#update-android-studio)
* [Configure Android Studio](#configure-android-studio)
  * [Map keyboard shortcuts](#map-keyboard-shortcuts)
  * [Turn off paramater hints](#turn-off-paramater-hints)
* [Create a new project](#create-a-new-project)
* [Create a new virtual device](#create-a-new-virtua-device)
* [Logging](#logging)
  * [System out](#system-out)

### Install Android Studio
Based on IntelliJ IDEA, Android studio can be installed from the AUR

References:
* [Archlinux wiki - Android Studio](https://wiki.archlinux.org/title/android#Android_Studio)
* [Install kvm on arch linux](https://transang.me/install-kvm-on-arch-linux/)
* Android studio's configuration is stored in `~/.android`. Delete this to reset the app

1. Install OpenJDK and python virtual environment
   ```shell
   $ sudo pacman -S android-tools android-udev libmtp qemu virt-manager dnsmasq dmidecode vde2
   ```

2. Add yourself to the `adbusers` and `kvm` groups
   ```shell
   $ sudo usermod -aG adbusers $USER
   $ sudo usermod -aG kvm $USER
   ```

3. Reboot your machine and start the `libvirtd` service
   ```shell
   $ sudo reboot
   $ sudo systemctl enable libvirtd
   $ sudo systemctl start libvirtd
   ```

4. Install Android Studio from the AUR
   ```shell
   $ yay -Ga android-studio
   $ cd android-studio; makepkg -s
   $ sudo pacman -U android-studio-2022.3.1.22-1-x86_64.pkg.tar.zst
   ```

5. First run configuration  
   a. Launch `android-studio`  
   b. Keep `Do not import settings` checked and click `OK`  
   c. Disable `Data Sharing` i.e. Google collecting your data, click `Don't Send`  
   d. Click `Next` to start the wizard and choose `Custom` install type  
   e. `Select the default JDK Location` the path option as `/opt/android-studio/jre`  
   f. `Select UI Theme` as `Darcula`  
   g. Select `SDK Components Setup` options installed to `/home/<USER>/Android/Sdk`  
   h. Agree to the Terms and Conditions  

6. Install `IdeaVim`  
   a. Pull up `Settings` by clicking the gear in the top right  
   b. Select `Plugins...`  
   c. Search for and select `IdeaVim` and click `Install`  
   d. Click `Restart IDE`  

7. Unset `_JAVA_OPTIONS` from `~/.bashrc`

## Update Android Studio
1. Build the latest package from the AUR
   ```shell
   $ yay -Ga android-studio
   $ cd android-studio
   $ makepkg -s
   ```
2. Install the package you built
   ```bash
   $ sudo pacman -U android-studio-2021.2.1.15-1-x86_64.pkg.tar.zst
   ```

## Configure Android Studio

### Map keyboard shortcuts
1. Navigate to `File >Settings`
2. Choose the `Keymap` on the left
3. Search for your target e.g. `rename` on the right
4. Select `Main Menu >Refactor >Rename`
5. Double click to change options

### Turn off parameter hints
1. Navigate to `File >Settings...`
2. Navigate to `Editor >Inlay hints`
3. Turn them off

## Create a new project
Activities are simply screen views

1. Create a new project directory
   ```shell
   $ mkdir ~/Projects/mrsa
   ```

2. Create a new project in Android Studio
   a. Select `Create a New Project` from the IDE  
   b. Choose `Empty Activity`  
   c. Set `Package name` to `com.example.mrsa`  
   d. Set `Save location` to our projecdt `~/Projects/mrsa`  
   e. Choose `Language` as `Kotlin`  
   f. Choose minimum SDK as `API 24: Android 7.0 (Nougat)`  

## Create a new virtual device
Before you can test anything you'll need to create at least one virtual device

1. Open up the `Device Manager` from the `No Devices` drop down in the top right
2. Click `Create virtual device`
3. Choose `Phone > Pixel 4 XL`
4. Click the `Download` link next to the recommended image

## Logging
Having decent logging is essential to writing an Android app.

1. Open the `Logcat` tool window at the bottom
2. Filter on your app name e.g. `Kotlin` as their is so much logging unrelated to what your doing
3. Add logging lines to your app `Log.i("Kotlin", "Hello World")`
4. Run your app and see the output show up

### System out
Anything printed out using `println` and the like will show up in the `Logcat` log viewer under the 
`System.out` filter name. In this way you can easily test quick code examples

println interpolation
```kotlin
var exists = false
println("hello world: $exists")
println("hello world: ${exists.toString()}")
```


