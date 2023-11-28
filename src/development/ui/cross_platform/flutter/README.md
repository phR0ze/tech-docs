# Flutter

Flutter is Google’s free, open-source software development kit (SDK) for cross-platform mobile 
application development. Using a single platform-agnostic codebase, Flutter helps developers build 
high-performance, scalable applications with attractive and functional user interfaces for Android or 
IOS. Flutter relies on a library of pre-made widgets that make it simple for even people with limited 
programming or development experience to launch their own mobile applications quickly.

Created by Google in 2015 and officially launched in 2018, Flutter has quickly become the toolkit of 
choice for developers. According to Statista, Flutter has recently surpassed React Native to become 
the number one mobile app development framework.

source [content URL](https://stackoverflow.blog/2022/02/21/why-flutter-is-the-most-popular-cross-platform-mobile-sdk/)

### Quick links
* [Overview](#overview)
* [Setup Flutter on Arch Linux](#setup-flutter-on-arch-linux)
* [Flutter with Android Studio](#flutter-with-android-studio)
  * [Configure Flutter project in Android Studio](#configure-flutter-project-in-android-studio)
* [Flutter without Android Studio](#flutter-without-android-studio)
  * [Create new project](#create-new-project)
  * [Build for Desktop](#build-for-desktop)
* [Flutter Patterns](#flutter-patterns)
  * [Splash Screen](#splash-screen)

## Overview
Flutter is a layered system comprising the framework, the engine, and platform-specific embedders. 
Flutter applications are built using Google’s Dart object-oriented programming language. The Flutter 
engine itself is written primarily in C/C++. And the Skia library is the backbone of Flutter’s 
graphics capabilities on desktop platforms while Impeller handles graphics for iOS and Android.

Flutter’s suitability for cross-platform development goes beyond code portability. Unlike other 
cross-platform frameworks such as React Native and Xamarin, Flutter-built user interfaces (UI) are 
also platform-agnostic because Flutter’s Skia rendering engine does not require any platform-specific 
UI components. All desired UI components have been rebuilt in Flutter and are readily available thus 
reducing development time.

![Flutter Architecture](../../../../data/images/flutter-arch.webp)

**References**
* [Flutter Awesome](https://flutterawesome.com/)
* [Flutter multi-platform](https://flutter.dev/multi-platform)
* [Flutter Folio Example App site](https://flutter.gskinner.com/folio/)
* [Flutter Folio Example App Github](https://github.com/gskinnerTeam/flutter-folio)
* [Flutter 3.16 update](https://dev.to/svprdga/flutter-316-released-android-impeller-preview-game-toolkit-updates-ios-extensions-and-more-1c7n)
* [Flame - Flutter Based Game Engine](https://pub.dev/packages/flame)
* [Awesome Flame](https://github.com/flame-engine/awesome-flame)
* [Flutter docs](https://flutter.dev/docs)
* [Flutter cookbook](https://docs.flutter.dev/cookbook)

**Benefits**
* Cross-platform Android, iOS, WASM, Windows, MacOS, Linux
* Dart language compiles to JavaScript or machine code
* Native-compiled performance with no embedded browser engine
* Hardware-accelerated graphics for all platforms
* Widget library adapts for any screen size
  * Material 3 and Cupertino visual styles
* Rendering engine allows for control of every pixel
* During development Flutter apps run in a VM that offers stateful hot reload of changes
  * For release, Flutter apps are compiled directly to machine code
* Supported by Google
* Flutter Casual Games Toolkit
* Impeller in 3.16 for improved rendering engine
  * Only good for Vulkan i.e newer devices
  * OpenGL not yet supported i.e. older devices

**Negatives**
* Flutter apps include all necessary code for widgets and don't rely on platform making binaries bigger
* Still relatively new platform
* Have to learn Dart

## Setup Flutter on Arch Linux

**References**
* [Arch Linux getting started with flutter](https://dev.to/nabbisen/flutter-3-on-arch-linux-getting-started-fc0)

1. Install [android tooling](../../../android/emulator)

2. Install [android studio](../../../android)
   1. Install the `Flutter` plugin from the `File >Settings... > Plugins` menu
   2. Install the `Dart` plugin from the `File >Settings... > Plugins` menu

3. Install pre-requisites
   ```bash
   $ sudo pacman -S dart clang cmake ninja base-devel
   ```

4. Install Flutter 3.0
   ```bash
   $ yay -GA flutter
   $ cd flutter
   $ makepkgs -s
   $ sudo pacman -U flutter-3.16.0-1-x86_64.pkg.tar.zst
   ```

5. Run flutter doctor after setting exceptions
   ```bash
   $ git config --global --add safe.directory /opt/flutter
   $ flutter config --android-sdk $ANDROID_HOME
   $ flutter doctor --android-licenses
   ```

## Flutter with Android Studio

### Configure Flutter project in Android Studio
1. Click the `New Flutter Project`
2. Select the `Flutter` entry on the left then set the `Fluter_SDK path` to `/opt/flutter`
3. Click the `Next` on the flutter view
4. Name the project e.g. `cross_platform_example` then click `Create`
5. Wait a min for the project template to be downloaded and populated

6. Create a new emulator in the IDE and click the `Run` button for an android build

## Flutter without Android Studio

### Create new project
```bash
$ flutter create <project-name>
```

### Run with Flutter commandline
Running an application directly with Flutter in this way will provide an interactive terminal for 
updating and managing your application including hot-reloads, hot-restarts and links to pull up debug 
tools in the browser for interacting with the Dart VM that is used during debug builds.

1. Launch app on Desktop with the Dart VM via Flutter
   ```bash
   $ flutter run -d linux
   ```

2. Launch app on Emulator with the Dart VM via Flutter
   1. Start the emulator
      ```bash
      $ emulator -avd android30
      ```
   2. List out the available devices
      ```bash
      $ adb devices
      List of devices attached
      emulator-5554	device
      ```
   3. Run on the emulator
      ```bash
      $ flutter run -d emulator-5554
      ```

2. Press `r` to hot-reload your app after changes

### Build for Desktop
Flutter uses GTK for Linux

1. Build for linux Desktop
   ```bash
   $ flutter build linux --release
   ```
2. Binary will end up in `build/linux/x64/release/bundle`
   * `data` contains the application's data assets, such as fonts or images 
   * `lib` contains the required .so library files

## Flutter Patterns

### Splash Screen
Splash screens, a.k.a launch screens, provide a simple initial experience while your app loads. They 
set the stage for your application, while allowing time for the app engine to load and your app to 
initialize.

## Rust Integration

**References**
* [Cargo flutter](https://github.com/flutter-rs/cargo-flutter)
* [Flutter app demo desktop](https://github.com/flutter-rs/flutter-app-demo)

<!-- 
vim: ts=2:sw=2:sts=2
-->
