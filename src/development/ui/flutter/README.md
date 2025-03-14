# Flutter <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Flutter is Google’s free, open-source software development kit (SDK) for cross-platform mobile 
application development. Using a single platform-agnostic codebase, Flutter helps developers build 
high-performance, scalable applications with attractive and functional user interfaces for Android or 
IOS. Flutter relies on a library of pre-made widgets that make it simple for even people with limited 
programming or development experience to launch their own mobile applications quickly. Flutter’s 
suitability for cross-platform development goes beyond code portability. Unlike other cross-platform 
frameworks such as React Native and Xamarin, Flutter-built user interfaces (UI) are also 
platform-agnostic because Flutter’s Skia rendering engine does not require any platform-specific 
UI components. All desired UI components have been rebuilt in Flutter and are readily available thus 
reducing development time. Most notably though is Flutter's use of modern declarative UI design 
bringing mobile app development out of the Java dark ages into something fresh and clean.

Created by Google in 2015 and officially launched in 2018, Flutter has quickly become the toolkit of 
choice for developers. According to Statista, Flutter has recently surpassed React Native to become 
the number one mobile app development framework.

source [content URL](https://stackoverflow.blog/2022/02/21/why-flutter-is-the-most-popular-cross-platform-mobile-sdk/)

Flutter employs a layered system comprising the framework, the engine, and platform-specific embedders. 
While Flutter applications are built using Google’s Dart object-oriented programming language. The 
Flutter engine itself and the Skia library, which is the backbone of Flutter’s graphics capabilities, 
are written primarily in C/C++.

![Flutter Architecture](../../../../data/images/flutter-arch.webp)

**References**
* [Flutter samples](https://github.com/flutter/samples/tree/main)
* [Flutter Awesome](https://flutterawesome.com/)
* [Flutter multi-platform](https://flutter.dev/multi-platform)
* [Flutter Folio Example App site](https://flutter.gskinner.com/folio/)
* [Flutter Folio Example App Github](https://github.com/gskinnerTeam/flutter-folio)
* [Flutter plugins for desktop](https://github.com/google/flutter-desktop-embedding)
* [Flutter 3.16 update](https://dev.to/svprdga/flutter-316-released-android-impeller-preview-game-toolkit-updates-ios-extensions-and-more-1c7n)
* [Flame - Flutter Based Game Engine](https://pub.dev/packages/flame)
* [Awesome Flame](https://github.com/flame-engine/awesome-flame)
* [Flutter docs](https://flutter.dev/docs)
* [Flutter cookbook](https://docs.flutter.dev/cookbook)
* [Dart pad with Flutter](https://dartpad.dev/?id=e7076b40fb17a0fa899f9f7a154a02e8)
* [Pub dev - Dart and Flutter package repo](https://pub.dev/)
* [Flutter MDI ideas](https://itnext.io/desktop-gui-implementation-using-flutter-web-part-3-draggable-resizable-windows-46ea26049605)

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
* Uses Dart which is slow and single threaded

### Quick links
- [.. up dir](../README.md)
* [Getting Started](#getting-started)
  * [Setup Flutter on Arch Linux](#setup-flutter-on-arch-linux)
  * [Create a new Flutter project](#create-a-new-flutter-project)
  * [Run on Android Emulator](#run-on-android-emulator)
* [Publish app](#publish-app)
  * [Build release](#build-release)
* [Images](#images)
  * [Display an Image](#display-an-image)
* [Packages](#packages)
* [Flutter app examples](#flutter-app-examples)
  * [AuthPass](#auth-pass)
* [Flutter Patterns](#flutter-patterns)
  * [Android permissions](#android-permissions)
  * [Launch Screen](#launch-screen)
  * [Responsive](#responsive)
  * [Build methods](#build-methods)
  * [Custom Widgets](#custom-widgets)
  * [Launch Screen](#launch-screen)
* [Project](#project)
  * [Project ID](#project-id)
  * [pubspec.yaml](#pubspec-yaml)
    * [Dependencies](#dependencies)
* [Flutter platform](#flutter-platform)
  * [Basic widgets](#basic-widgets)
  * [Future Builder](#future-builder)
  * [Layout](#layout)
  * [Keyboard Input](#keyboard-input)
  * [MediaQuery](#mediaquery)
  * [Navigation](#navigation)
  * [Persist configuration](#persist-configuration)
  * [State](#state)
  * [Slivers](#slivers)
  * [Themes](#themes)
  * [Window decorations](#window-decorations)
* [Scrolling](#scrolling)
  * [Efficient scrolling](#efficient-scrolling)
  * [Infinite scrolling easy](#infinite-scrolling-easy)
  * [Infinite scrolling with scroll controller](#infinite-scrolling-with-scroll-controller)
  * [Fancy scrolling](#fancy-scrolling)
  * [Resume position](#resume-position)
* [Dart](#dart)
  * [Parameters](#parameters)
  * [static vs final vs const](#static-vs-final-vs-const)

# Getting Started
In my opinion the best approach to Flutter application development uses the flutter cli for creating 
and interacting with your project and a desktop environment to run and test your application and 
finally the Android emulator as a standalone component to run and test your Android version of the 
application.

Note: the Android Studio path is slow, clunky and overly complicated IMO.

## Setup Flutter on Arch Linux
Reference: [getting started with Flutter on Arch Linux](https://dev.to/nabbisen/flutter-3-on-arch-linux-getting-started-fc0)

1. Install [android tooling](../../../android/emulator)

2. Install [android studio](../../../android/index.html#android-studio)
   1. Install the `Flutter` plugin from the `File >Settings... > Plugins` menu
   2. Install the `Dart` plugin from the `File >Settings... > Plugins` menu

3. Install pre-requisites
   ```bash
   $ sudo pacman -S dart clang cmake ninja base-devel
   ```

4. Install Flutter latest
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

## Create a new Flutter project
1. [Install and configure VS Code](../../../editors/vscode)

2. Create your project using the Flutter CLI
   ```bash
   $ flutter create --platforms=linux,android <project-name>
   ```

3. Load the project in VSCode then

4. Install the Flutter extensions:
   | Name                     | Identifier                            |
   | ------------------------ | ------------------------------------- |
   | Remove Comments          | `rioj7.vscode-remove-comments`        |
   | Dart                     | `dart-code.dart-code`                 |
   | Flutter                  | `dart-code.flutter`                   |
   | Flutter riverpod Helpers | `evils.vscode-flutter-riverpod-helper`|
   | The Riverpod Extension   | `houssembousmaha.ri`                  |

5. Add the riverpod packages using the Flutter CLI
   ```bash
   $ flutter pub add flutter_riverpod
   $ flutter pub add riverpod_annotation
   $ flutter pub add dev:riverpod_generator
   $ flutter pub add dev:build_runner
   $ flutter pub add dev:custom_lint
   $ flutter pub add dev:riverpod_lint
   ```

6. Add supporting packages using the Flutter CLI
   ```bash
   $ flutter pub add json_annotation
   $ flutter pub add freezed_annotation
   $ flutter pub add dev:freezed
   $ flutter pub add dev:json_serializable
   ```

7. Add the river pod import and wrap the app with a `ProvideScope` in your `main.dart`
   1. Remove comments from your `main.dart` by selecting everythin then hitting `Ctrl+Shift+P` and 
      entering `Comments: Remove All Comments` and pressing enter.
   ```dart
   import 'package:flutter_riverpod/flutter_riverpod.dart';
   
   void main() {
     runApp(const ProviderScope(child: MyApp()));
   }
   ```

8. Run the new app locally on your Linux box
   1. Open the `main.dart`
   2. Hit `F5`

## Run on Android Emulator
Althought the Android Emulator works as a Standalone service I found it slow and often froze up when 
starting it back up after using it once before; something with the saved state maybe. At any rate I 
had the most success developing locally with Linux and then creating a new Android emulator each time 
I wanted to try it out on Android.

1. Create an Android virtual device
   ```bash
   $ avdmanager create avd --name android30 --package "system-images;android-30;google_apis;x86_64" --device pixel_4_xl
   ```

2. Start your Android virtual device
   ```bash
   $ emulator -avd android30
   ```

3. In the lower right of VSCode choose the target device option and choose `android30`
   * Every time you save your changes it will be hot-reloaded on the target device

4. Delete your Android virtual device when done
   ```bash
   $ avdmanager delete avd -n android30
   ```

# Publish app

## Build release
Build a release apk for testing with:
```bash
$ flutter build apk --release
```

# Images

## Animated Images
By default Flutter supports `Animated GIF` and  `Animated WebP`

## Disk Cache

* [`flutter_cache_manager`](https://pub.dev/packages/flutter_cache_manager)
  * [Medium article](https://nureddineraslan.medium.com/caching-in-flutter-an-effective-approach-to-speed-up-your-app-77e688d3fbe5)
* [`get_storage`](https://pub.dev/packages/get_storage)

## Global imageCache
Flutter implements a least-recently-used global singleton image cache of up to 1000 images, and up 
to 100 MB. It is used internally by ImageProvider. The image cache is created during startup by the 
PaintingBinding.createImageCache method and is managed by the system. The cache holds a list of 
`live` references. An image is considered live if its listener count never drops to zero. The 
`putIfAbsent` method is the main entry-point to the cache API. It returns the previously cached image 
for the given key, if available; if not, it calls the given callback to obtain it first. In either 
case, the key is moved to the most recently used position.

* [ImageCache class docs](https://api.flutter.dev/flutter/painting/ImageCache-class.html)

Flutter's global image cache is not persisted on disk however. Thus we get packages like the 
`flutter_cache_manager` package that do this for you.

## Image vs ImageProvider
The `ImageProvider` is what provides the actual image to the `Image` widget. It is an abstract class 
that represents a way to obtain an image. If you need a custom image provider then you'll want to use 
the `Image` base constructor directly. Otherwies you can simply allow the Image widget to use the 
default image provider using one of the factory constructors. The Image widget is responsible for 
displaying the image on the screen.

**Getting an Image:**
* `Image.asset()`
* `Image.network()`
* `Image.file()`
* `Image.memory()`

**Getting an ImageProvider:**
* `AssetImage()`
* `NetworkImage()`
* `FileImage()`
* `MemoryImage()`

**Convert an ImageProvider to and Image**
```dart
Image(image: myImageProvider)
```

**Convert an Image to and ImageProvider**
```dart
myImageProvider.image
```

## Caching thumbnails

















## Display an Image
The ***Image*** class renders an image to the screen. Flutter supports: `JPEG`, `PNG`, `Animated 
GIF`, `Animated WebP`, `BMP`, and `WBMP`. In order to render the images to the screen Flutter needs 
to cache them as raw images so a lot of memory may be required. You can work around this by setting 
the custom decode size to limit how much memory is consumed by the cache.

* `Image.asset()`
* `Image.network()`
* `Image.file()`
* `Image.memory()`

# Packages

## Used and found useful
A list of packages I've found to be helpful and have used in applications I've written.

* [`device_info_plus`](https://pub.dev/packages/device_info_plus)
  * License: BSD-3
  * Provides the ability to check the Android SDK version
  * Flutter favorite

* [`quiver`](https://pub.dev/packages/quiver)
  * License: Apache 2
  * Utilities for making core Dart libaries easier and more convenient
  * Owned by `google.dev` team

* [`path_provider`](https://pub.dev/packages/path_provider)
  * License: BSD-3
  * Provides easy access to paths for storing state and caching
  * Flutter favorite

## Didn't end up using
* [`easy_image_viewer`](https://pub.dev/packages/easy_image_viewer)
  * License: MIT
  * Provides the ability to display an image with typical image features
    * Swipe left/right for page changes
    * Swipe down for dismissing
    * Pinch to zoom, pan
  * Unable to use it as the page swiping and dimissing interfer with pinch to zoom

## Interesting packages

* [`vm_service`](https://pub.dev/packages/vm_service)
  * The `extended_image` package example uses this to show a really slick sub-window that shows 
  memory consumption

* [`extended_image`](https://pub.dev/packages/extended_image)
  * License: MIT
  * memory caching
  * disk caching option built off `path_provider` and direct file manipulation
  * using modern version of flutter
  * zoom and pan with reset button `Simple > zoom/pan image`
  * google photo like dismiss `Simple > SlidePage`
  * crop tool `Simple > ImageEditor`
  * image viewer example `Complex demos > WaterfallFlow` 
    * nice layout and demostrates network caching
  * complex editor example offers selection between
    * Image Dart - stable
    * ImageEditor android/ios native - faster
  * news browser like example with nice layout
    * pull down to refresh
    * multiple images to scroll through
    * clickable images that can be: zoomed, dismissed, saved and pulled up for details


* [`photo_view`](https://pub.dev/packages/photo_view)
  * License: MIT
  * double tap zoom with two allowed before resetting
  * pan once in zoom mode
  * rotate and zoom with slider controls
  * pinch to rotate and zoom
  * able to show SVG as an image
  * Show any widget not just image e.g. container, text or svg
  * programatic rotation
  * Seems to have found a way to use dismissible and zoom together

* [`flutter_cache_manager`](https://pub.dev/packages/flutter_cache_manager)
  * Uses `provider_path` to provide the cache directory and stores there

* [`flutter_launcher_icons`](https://pub.dev/packages/flutter_launcher_icons)
  * CLI to change your launcher icon for you

* [`flutter_native_splash`](https://pub.dev/packages/flutter_native_splash)
  * Custom image as splash screen instead of default white image

* [`sliding_up_panel`](https://pub.dev/packages/sliding_up_panel)
  * Nice way to show a images details

* [`media_store_plus`](https://pub.dev/packages/media_store_plus)
  * Provides MediaStore plugin access

* [`smooth_page_indicator`](https://pub.dev/packages/smooth_page_indicator)
  * Assome animated page indicator for scrolling images

**Flutter favorites**
* [`battery_plus`](https://pub.dev/packages/battery_plus)
* [`flame`](https://pub.dev/packages/flame)
* [`fluent_ui`](https://pub.dev/packages/fluent_ui)
* [`flutter_local_notifications`](https://pub.dev/packages/flutter_local_notifications)
* [`flutter_rust_bridge`](https://pub.dev/packages/flutter_rust_bridge)
* [`go_router`](https://pub.dev/packages/go_router)
* [`json_serializable`](https://pub.dev/packages/json_serializable)
* [`url_launcher`](https://pub.dev/packages/url_launcher)

[Most liked packages](https://pub.dev/packages?q=platform%3Alinux&sort=like)
* [`dio`](https://pub.dev/packages/dio) 

[material.io](https://pub.dev/publishers/material.io/packages)
* [`adaptive_breakpoints`](https://pub.dev/packages/adaptive_breakpoints)
* [`adaptive_components`](https://pub.dev/packages/adaptive_components)
* [`adaptive_navigation`](https://pub.dev/packages/adaptive_navigation)
* [`dynamic_color`](https://pub.dev/packages/dynamic_color)
* [`google_fonts`](https://pub.dev/packages/google_fonts)

[google.dev packages](https://pub.dev/publishers/google.dev/packages)
* [`googleapis_auth`](https://pub.dev/packages/googleapis_auth)
* [`file`](https://pub.dev/packages/file)
  * A pluggable, mockable file system abstraction for Dart with local and in-memory files
* [`retry`](https://pub.dev/packages/retry)

[gskinner](https://pub.dev/publishers/gskinner.com/packages)
* [`flutter_animate`](https://pub.dev/packages/flutter_animate)
* [`universal_platform`](https://pub.dev/packages/universal_platform)

## Deprecated packages
* [`provider`](https://pub.dev/packages/provider)
  * Use Riverpod instead i.e. the 2.0 version from the same author

# Flutter app examples

## AuthPass
[AuthPass](https://authpass.app/) is a Free and Open Source password manager for Android, iOS, macOS, 
Linux and Windows that is compatible with KeePass.

**References**
* [authpass - Github](https://github.com/authpass/authpass)

# Flutter Patterns
* ***Custom Widgets*** are a best practice for reusability
* ***Pass Widgets*** widgets as parameters to other widgets is a standard practice
* Keep state as close to the area of concern as possible
  * Keep it in the widget if that is all that needs it
  * Keep it at the lowest level in the widget tree that ensures it lives long enough to be useful

## Android permissions
see [android/permissions](../../../android/permissions/index.html) for Android permission specifics. 
I'll just cover the Flutter side here.

To qualify to use the Manage External Storage permission you need to:
* File managers
* Backup and restore apps
* Anti-virsus apps
* Document management apps
* On-device files search
* Disk and file encryption
* Device to device data migration

`File Browser???`

1. Add the permission handler to your app
   ```bash
   $ flutter pub add permission_handler
   ```

2. Ensure you app is setup for Androidx which mine was by default
   1. Edit `android/gradle.properties`
   2. Check for `android.useAndroidX=true`
   3. Check for `android.enableJetifier=true`

3. Ensure the `compileSdkVersion` is set to `33`
   1. Edit `android/app/build.gradel`
   2. Add
      ```
      android {
        compileSdkVersion 33
      }
      ```

4. Add any needed permissions to your `AndroidManifest.xml` file
   * See [All possible permissions listed for Flutter](https://github.com/Baseflow/flutter-permission-handler/blob/main/permission_handler/example/android/app/src/main/AndroidManifest.xml)
   1. Edit `android/app/src/main/AndroidManifest.xml`
   2. At the top before the `<application` section add the desired permissions
      ```xml
      <!--  -->
      <uses-feature android:name="android.hardware.camera" android:required="false" />
      <uses-permission android:name="android.permission.CAMERA"/>

      <!-- Read and write external storage for Android 12 and lower -->
      <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"
           android:maxSdkVersion="32" />
      <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" 
             android:maxSdkVersion="29"  />

      <!-- Broad access to external storage -->
      <uses-permission android:name="android.permission.MANAGE_EXTERNAL_STORAGE" />

      <!-- *************************************************** -->
      <!-- Granular media permissions for Android 13 and newer -->
      <!-- *************************************************** -->

      <!-- Permission to read image files from external storage -->
      <uses-permission android:name="android.permission.READ_MEDIA_IMAGES" />

      <!-- Permission to read video files from external storage -->
      <uses-permission android:name="android.permission.READ_MEDIA_VIDEO" />

      <!-- Permission to read audio files from external storage -->
      <uses-permission android:name="android.permission.READ_MEDIA_AUDIO" />

      <!-- Permissions options for the `manage external storage` group -->
      <uses-permission android:name="android.permission.MANAGE_EXTERNAL_STORAGE" />
      ```

5. Use the `permission_handler` and `device_info_plus` packages and
   ```dart
   // Android 13 or higher
   await Permission.photos.request();
   await Permission.manageExternalStorage.request();
   ```

## Build methods
In Flutter when a parent widget receives a callback and updates its internal state it will then 
trigger a rebuild which will then rebuild all the widgets below it if they have changed. The 
framework does this by comparing the newly built widgets with the previously built widgets and only 
applying the differences to the underlying RenderObject.

## Custom Widgets
Reduce your widgets to only accept what they use as parameters not all app state.

## Launch Screen
Launch screens, a.k.a splash screens, provide a simple initial experience while your app loads. They 
set the stage for your application, while allowing time for the app engine to load and your app to 
initialize.

**References**
* [Flutter docs - Splash screen](https://docs.flutter.dev/platform-integration/android/splash-screen)

## Custom Scroll Controler

## Responsive
Flutter apps might appear on screens of many different sizes. Ideally you want your app to be both 
***adaptive*** and ***responsive***. Flutter's primary means of handling responsiveness is to use the 
`LayoutBuilder` which will redraw the UI whever the size constraints change providing the ability for 
things like:

```dart
child: Layoutbuilder(
  builder: (context, constraints) {
    if (constraints.maxWidth > 1200) {
      return UltraWideLayout();
    } else if (constraints.maxWidth > 600) {
      return WideLayout();
    } else {
      return NarrowLayout();
    }
```

```dart
if (Theme.of(ctx).platform == TargetPlatform.iOS) {
  // Alternatively you can use dart:io.Platform
}
```

The constraint sizes are device neutral meaning they are supposed to be the same thing on every 
screen and it doesn't matter resolution, density etc...

***Responsive***  
Typically, a responsive app has had its layout tuned for the available screen size. Often this means 
re-laying out the UI if the user resizes the window, or changes the device's orientation.

***Adaptive***  
Adaptive apps can adapt to the device's platform capabilities such as different kinds of input 
devices e.g. keyboard and mouse vs touch.

**References**
* [Flutter docs - responsive app](https://docs.flutter.dev/ui/layout/responsive/adaptive-responsive)
* [Building adaptive apps](https://docs.flutter.dev/ui/layout/responsive/building-adaptive-apps)


# Project
Every app has a `main()` which runs your root widget. Every widget has a `build` which must return a 
widget or tree of widgets so that Flutter nows what to draw. Widgets can be nested as desired.

**References**
* [Flutter UI docs](https://docs.flutter.dev/ui)
* [Material 3 demo](https://flutter.github.io/samples/web/material_3_demo/)
* [Material 3 docs](https://docs.flutter.dev/ui/widgets/material)
* [Material 3 Icons](https://fonts.google.com/icons)

## Project ID
When using the package `path_provider` and other tools for resolving paths on disk etc... that use 
your project name as an identifier it will default to something like `com.example.riverpod_moview`. 
You can change this for the different platforms with:

***For Linux***
  * Edit `linux/CMakeLists.txt`
  * Change `set(APPLICATION_ID "com.example.riverpod_movies")`
  * to `set(APPLICATION_ID "riverpod_movies")`
  * This makes `getApplicationSupportDirectory` resolve to `/home/<user>/.local/share/riverpod_movies`

## pubspec
The `pubspec.yaml` in the root of your Flutter project is the main project file tracking dependencies 
and configuration for your project

### version
The app version called out in the `pubspec.yaml` uses the typical semver format `major.minor.patch` 
but then adds on an optional build number separated by the plus sign `+`

Example:
```
1.0.0+1
```

### Dependencies
Flutter/dart dependencies can be specified by git repo as well
```yaml
dependencies:
  window_manager:
    git:
      url: https://github.com/leanflutter/window_manager.git
      ref: main
```

# Flutter platform

## Basic widgets
* ***Text*** styled text

## Future Builder
The `FutureBuilder` widget will take a future and handle the different states of the future i.e. 
completed, failed and waiting.

## Layout

* ***BoxDecoration*** lets you decorate the `Container` widget
* ***Column*** takes any number of children and puts them in a column from top to bottom
* ***Container*** lets you create a rectangular visual element that can be decorated
* ***Expanded*** wrapper to be used inside other widgets to greedily expand to fill available space
* ***LayoutBuilder*** common parent to trigger rebuilds if screen size orientation or size changes
* ***Row*** takes any number of children and puts them in a row from left to right
* ***Scaffold*** per screen/page it provides basic material styling and layout components
* ***SafeArea*** defines space that should be visible beyond the camera and options menus etc...
* ***Stack*** lets you place widgets on top of each other in paint order
* ***Placeholder()*** fantastic idea for building out layouts without any content yet
* ***Positioned*** can be used on children of a `Stack` to position them relative to it

## Keyboard input

## MediaQuery
[MediaQuery](https://api.flutter.dev/flutter/widgets/MediaQuery-class.html) provides a way to 
interact with your device to make it more responsive.

`MediaQuery.of`

* Get device size information for layout
  * Screen size
* Check device orientation
* Check areas that are obscured by cameras


## Navigation
Flutter's Material 3 implementation provides several navigation options:
* ***AppBar*** - advanced navigation and actions bar at top
* ***BottomAppBar*** -  advanced navigation and actions bar at bottom
* ***BottomNavigationBar*** - advanced navigation and actions bar at bottom
* ***NavigationBar*** - simple switching between primary destinations
* ***NavigationDrawer*** - lkj
* ***NavigationRail*** - bar on the side of the screen that can optionally expand to include labels
* ***TabBar*** - lkj

**References**
* [Material 3 Navigation](https://docs.flutter.dev/ui/widgets/material#Navigation)
* [Navigation basics - Flutter docs](https://docs.flutter.dev/cookbook/navigation/navigation-basics)

The `NavigationRail` provides a left side navigation menu that can be:
* `extended` to show icon and label in a larger screen setting
* `selectedIndex` provides option to choose which button is the default selection
* `onDestinationSelected` callback when buttons are pressed

**Routes** are a natural part of the navigation conversation. Each route is just a widget that we 
load as a new screen using the `Navigator.push` method.

**Push a new route**
```dart
onPressed: () {
  Navigator.push(
    context,
    MaterialPageRoute(builder: (context) => const SecondRoute()),
  );
}
```

**Return to the prior route**
```dart
onPressed: () {
  Navigator.pop(context);
}
```

## Persist configuration
Persisting data can be done in a variety of ways in Flutter. I'm looking for:
* Standard Linux convention `~/.config/<app>.yaml` convention
* Automatically convert into a singleton class in the app

**Potential Solution**
1. Use `xdg_directories` and/or `path_provider` to handle pathing
2. Use `global_configuation` singleton pattern
3. Use `cli_config` yaml support concept

**Package possibilities**
* [`xdg_directories`](https://pub.dev/packages/xdg_directories)
  * Provide access to Linux home, data and cache dirs
* [`path_provider`](https://pub.dev/packages/path_provider)
  * Basically just rolling your own `shared_preferences`
  * Cross platform access to temp dir, app dir, docs dir, cache dir, downloads
  * `getApplicationDocumentsDirectory()` is a directory in Android that the OS created for your app
  * `getTemporaryDirectory()` for data that doesn't need to outlive the current session
  * Use `dart:io` to read/write files directly with these paths
  * [Reading/Writing files](https://docs.flutter.dev/cookbook/persistence/reading-writing-files)
  * [Global Configuration](https://pub.dev/packages/global_configuration)
    * Seems to have done this but combined it with a singleton inside the app when loaded
    * Uses a JSON file as its storage mechanism. I'd prefer YAML or TOML
    * Supports network configuration
* [`cli_config`](https://github.com/dart-lang/tools/tree/main/pkgs/cli_config)
  * Dart team created and supported for all platforms except Web
  * Command line args => Environment vars => JSON or YAML config file support
  * Odd command line argument definition `-Dsome_key=some_value`
  * Does allow for an optional path though so really mostly fits my needs
* [`shared_preferences`](https://pub.dev/packages/shared_preferences)
  * [Shared Preferences - LogRocket](https://blog.logrocket.com/using-sharedpreferences-in-flutter-to-store-data-locally/)
  * Interesting idea, stores simple key values in file `shared_preferences.json` in app support path
  * not sure what that resoves to on linux but doesn't fit conventions as is for sure
  * Could be modified to fit linux conventions though without much effort, although I'd use yaml

## State
Flutter uses `StatefulWidgets` to generate `State` objects, which are then used to hold state. 
Widgets and State objects have different life cycles. Widgets are used for presentation and are 
frequently destroyed and recreated with changes while State is persisted between calls to the 
`build()` method.

**References**
* [State Management - Flutter docs](https://docs.flutter.dev/data-and-backend/state-mgmt/intro)
* [Provider vs BLoC](https://www.miquido.com/blog/flutter-architecture-provider-vs-bloc/)
* [Provider vs BLoC - Medium](https://medium.com/@dihsar/bloc-vs-provider-in-flutter-a-comprehensive-comparison-fbd0f6c41e50)

* ***ChangeNotifier*** creates state and dependencies can watch for notifications and rebuild

**Provider vs BLoC**  
Both are populare state management libraries. ***Provider*** provides the current data model to the 
place where we currently need it, it contains some data and notifies observers when a change occurs. 
In the Flutter SDK, this type is called a ***ChangeNotifier***. For this notifier to work we need the 
***ChangeNotifierProvider*** which provides observed objects for all its descendants. ***BLoC*** 
a.k.a. Business Logic Components is a Flutter architecture similar which tries more closely to mimic 
the common Android paradigm of MVP or MVVM. It provides separation of the presentation layer from the 
business logic rules.

Provider's pattern is to create a new class extending the ChangeNotifier and capture your data as 
class fields and allows for class methods to interact with the data in a custom way. BLoC's pattern 
is based on events. You define the events that your screens will produce then you define the BLoC 
objects that take the events. Because of BLoC's granular nature you can more granularly control the 
updates to your app wiget tree thus speeding up your whole app. BLoC offers more control and is more 
complex to use. Idustry concensus is to use Provider unless you need more complexity and performance 
and then use BLoC which is more verbose requiring more boiler plate code.

## Slivers
A sliver is a portion of a scrollable area inside a CustomScrollView that can be configured 
accordingly to behave in a certain way. Using slivers we can create a plethora of different scrolling 
effects. Slivers lazy build their views when the widgets come into the viewport. This makes it ideal 
for showing a great number of children without worrying about memory issues.

**References**
* [Slivers - Log Rocket](https://blog.logrocket.com/building-custom-flutter-scrollview/)

***CustomScrollView***  
A widget that uses multiple Slivers rather than just one. 

**Slivers that can go inside CustomScrollView**
* `SliverAppBar` - creates a collapsible app bar by setting both the `flexibleSpace` and `expandedHeight`
* `SliverToBoxAdapter` - sliver that allows you to wrap non-sliver widgets inside a CustomScroolView
* `SliverGrid` - sliver that displays a 2D array
* `SliverFixedExtentList` - like SliverList but guarantees all items will remain the same size
* `SliverList` - sliver that renders a list in a linear array along the scroll view's main axis.
  * `SliverChildListDelegate` - specifies a fixed list of children that are created all at once
  * `SliverChildBuilderDelegate` - specifies how to build them lazily
* `SliverOpacity` - sliver makes its sliver child partially transparent
* `SliverPadding` - sliver that creates empty space around another sliver

## Themes
You can choose any color with `Color.fromRGB0(0, 255, 0, 1.0)` or `Color(0xFF00FF00)`. Flutter uses 
an app wide Theme that all widgets respect to provide a matching suite of colors.

**Set Theme Color Scheme** with the `seedColor`
```dart
theme: ThemeData(
  useMaterial3: true,
  colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
),
```

**Set text to medium**
```dart
final style = theme.textTheme.displayMedium!.copyWith(
  color: theme.colorScheme.onPrimary,
);
```

**Get current theme**
```dart
final theme = Theme.of(context);
```

## Window decorations
the `bitsdojo_window` package provides support for creating your own window decorations. The bits 
window package has an interesting side affect. Because it is hiding the initial window and only 
showing it once it is draw you get a very smooth window creation that doesn't start at a default size 
then visibly get transformed into the desired size. Instead this all happens hidden then is smoothly 
displayed when ready.

1. Create a new project
   ```bash
   $ flutter create --platforms=linux bitsdojo_windowing
   $ cd bitsdojo_windowing
   $ flutter pub add bitsdojo_window
   ```
2. Remove system window decorations
   1. Edit `linux/my_application.cc`
   2. Add to the top of the file `#include <bitsdojo_window_linux/bitsdojo_window_plugin.h>`
   3. Find the `gtk_window_set_default_size(window, 1280, 720);` line and add the following before
      ```
      auto bdw = bitsdojo_window_from(window);            // <-- add this line
      bdw->setCustomFrame(true);                          // <-- add this line
      //gtk_window_set_default_size(window, 1280, 720);   // <-- comment this line
      ```
3. Make your main function in `main.dart` look like
   ```dart
   void main() {
     runApp(const MyApp());

     doWhenWindowReady(() {
       const initialSize = Size(600, 450);
       appWindow.minSize = initialSize;
       appWindow.size = initialSize;
       appWindow.alignment = Alignment.center;
       appWindow.title = "Custom title"
       appWindow.show();
     });
   }
   ```
4. Set custom window title
   ```dart
       appWindow.show();
   ```


# Scrolling
Flutter has many built-in widgets that automatically scroll and also offers a variety of widgets that 
you can customize to create specific scrolling behaviors. Many Flutter widgets support scrolling out 
of the box and do most of the work for you. For example the `SingleChildScrollView` automatically 
scrolls its child when necessary. 
**References**
* [Scrolling - docs](https://docs.flutter.dev/ui/layout/scrolling)
* [Sliver workshop](flutter.dev/go/sliver-workshop)

## Shrink wrap
Shrink wrap forces a nested view to evaluate its entire item list size to then show properly in the 
parent and can be a performance issue. If you have large sets of items in your inner lists your going 
to get dropped frames and stutters in your UI. In this case use the `CustomScrollView` and `Slivers` 
to solve the issue.

## Efficient scrolling
More advanced widgets like the `ListView` and `GridView` display multiple items and provide a 
constructor that requires a builder method to build the child items on demand i.e. lazy loading. This 
is important because they only create those widgets that are visible or will soon become visible. In 
this way your app appears responsive and performant when you need to display large amounts of data. 
These are both good options for a lot of data.

Switching to the `CustomScrollView` and the `SliverGrid` and `SliverChildDelegateBuilder` is the same 
thing as the `GridView.builder` efficiency wise and complexity really, but you get more options on 
how to customize it. The `ListView` and `GridView` just simplify it a little.

## Infinite scrolling easy
A simpler way to get infinite scrolling is to simply add to trigger a fetch of new data when you are 
at some number of items from the end. This will asynchronously retrieve the data and load it into the 
list of items and then populate it on the screen as you scroll to it.

1. Add a fetch page method to your riverpod provider. It is critical that you don't specify the 
   `AsyncLoading` or any other state change other than new data or you'll loose the scroll position.
   ```dart
   Future<void> fetchNextPage() async {
     // If we've reached the page limit then were done
     if (_totalPages != -1 && _currentPage >= _totalPages) {
       return;
     }
 
     // Fetch the next page and specifically not setting loading state to avoid resetting
     // the scroll position for infite scrolling.
     // state = const AsyncLoading();
     state = await AsyncValue.guard(() async {
       final dto = await locate<TMDB>().getNowPlayingMovies(page: _currentPage++);
 
       // Update the page if reset
       if (_totalPages == -1) {
         _totalPages = dto.totalPages;
       }
 
       // Convert to movies and add to the state and replace it
       final movies = state.value ?? [];
       return movies + dto.results.map((x) => model.Movie.fromJson(x.toJson())).toList();
     });
   }
   ```
2. Trigger the page fetch in your `onNextPageRequested` in your list or grid view
   ```dart
   (content, index) {
     if (index == media.length - 5) {
       widget.onNextPageRequested?.call();
     }
   }
   ```

## Infinite scrolling with scroll controller
For advanced features like infinite scrolling or resuming scrolling you'll need to use your own 
scroll controller and implemente logic to handle these use cases for your app. Best practice is to 
fetch the data in advance to the user requesting it thus by the time the user makes the fetch request 
by scrolling the data is already available to view.

1. Setup your scroll controller and current position in your stateful widget.
   ```dart
   final _scrollController = ScrollController();
   var _currentScrollOffset = 0.0;
   ```

2. Setup your scroll listener in the state init method and implement the standard dispose
   ```dart
   @override
   void initState() {
     super.initState();
     _scrollController.addListener(_onScroll);
   }

   @override
   void dispose() {
     _scrollController.dispose();
     super.dispose();
   }
   ```

3. Implement your on scroll method to trigger an action if the bottom of the scroll has been hit and 
   save your current position in your state.
   ```dart
   void _onScroll() {
     if (_scrollController.offset >= _scrollController.position.maxScrollExtent &&
         !_scrollController.position.outOfRange) {
       setState(() {
         _currentScrollOffset = _scrollController.offset;
       });
       widget.onNextPageRequested?.call();
     }
   }
   ```

2. Use the `jumpTo` method to resume your position

## Fancy scrolling
Provides options for fancy scrolling in the UI with `SliverList`, `SliverGrid`, or `SliverAppBar`. 
These provide the same functionality as their non-sliver version, but you can customize their 
scrolling behavior inside the `CustomScrollView` and combine them.

**References**
* [Fancy scrolling - docs](https://docs.flutter.dev/ui/layout/scrolling/slivers)

Slivers are the basic building blocks for scrolling in Flutter. A Sliver is a portion of a scrollable 
area that display the content based on its configuration. Slivers manage the display of their 
children when they become visible and apply the scrolling effects on them. All scrollable widgets 
like `ListView` and `GridView` are built on top of Slivers.

**SliverAppBar**  
The sliver app bar has a lot of neat options like `floating` which scrols it out of view but back as 
soon as you scroll back down at all. It also has the `snap` option which makes it snap back in a 
little crisper and faster than a regular scroll would. We can also use the expanded height to provide 
more room in the begining for a background image or something that will fold up when scrolled.


# Dart

## Parameters
* [Dart functions docs](https://dart.dev/language/functions)

The parameters of a class constructor or fucntion are required by default.
```dart
class Test {
  final String x;
  Test(this.x);
}
```

## static vs final vs const
* ***static*** means a member is available on the class itself instead of on instances of the class
* ***final*** means single-assignment i.e. it must be initialized and cannot be changed
* ***const*** means that the object can be determined completely at compile time and frozen

# Rust Integration

**References**
* [Cargo flutter](https://github.com/flutter-rs/cargo-flutter)
* [Flutter app demo desktop](https://github.com/flutter-rs/flutter-app-demo)

<!-- 
vim: ts=2:sw=2:sts=2
-->
