# Android

Researching Rust on Android. I'd really like to see a full Rust app on Android.

### Quick links
* [Tooling](#tooling)
  * [Winit](#winit)
  * [Android Activity](#android-activity)
  * [xbuild](#xbuild)
* [Retained UI](#retained-ui)

**Resources**
* [Android Rust projects](https://blog.traverseresearch.nl/building-pure-rust-apps-for-android-d1e388b431b8)
* [UniFFI](https://sal.dev/android/intro-rust-android-uniffi/)
* [Flapigen](https://github.com/Dushistov/flapigen-rs)
* [Android source with Rust](https://source.android.com/docs/setup/build/rust/building-rust-modules/overview)
* [Crossbow cross platform gameing in Rust](https://github.com/dodorare/crossbow)
* [Publish Android Bevy App](https://www.nikl.me/blog/2023/github_workflow_to_publish_android_app/)
* [Buid Rust app for Android](https://gendignoux.com/blog/2022/10/24/rust-library-android.html)

## Tooling

### Winit
Cross-platform Window handling library in pure Rust. Consumes the `android-activity` for 
crossplatform windowing.

**Notes**
* Create windows
* Handle events like window resizing, key presses, mouse movement
* Depends on platform-specific providers for showing something on the screen
* X11 and Wayland support
* Don't specify dependencies allow winit to do it for you
  * Instead consume the API re-exported by winit at `winit::platform::android::activity::*`

**Android**
* Android backend builds and exposes types from the ndk crate

### android-activity
`android-activity` provides a way to load your crate as a `cdylib` library via the `onCreate` method 
of your Android Activity class; run an `android_main()` function in a separate thread from the Java 
main thread and marshal events (such as lifecycle events and input events) between Java and your 
native thread.

This is a refactored continuation of the [ndk](#ndk) work from the rust-mobile community. This 
encapsulates the former work done in the `ndk`, `ndk-glue`, `ndk-macro` projects.

Android apps are not like your average commandline executable, instead they are always associated 
with a physical window, a so called ***Activity*** in Android terminology. The `NativeActivity` loads 
your Rust code and is where it resides, allowing apps to be created without any Java code/classes at 
all. It serves as a bridge between Android's app lifecycle and your Rust code forwarding events 
through callbacks. This integration defines how an app starts up and moves to back and foreground 
while users navigate their phone.

**Refrences**
* [Android activity github](https://github.com/rust-mobile/android-activity)
* [Android game activity](https://developer.android.com/games/agdk/game-activity)

**Features**
* `GameActivity` - new `Jetpack library` and recommended path
  * GameActivity inherits from the NativeActivity also from the JetPack `AppCompatActivity`
  * Allows for seamless use of other Jepack components like `ActionBar` and `Fragment`
  * Renders to the `SurfaceView` making it easier for games to interact with other UI components
  * GameActivity apps are expected to build all three parts of of the app into one library
* `NativeActivity` - deprecated older integration

### xbuild
Rust cross compile automation assistance for mobile. Supports Rust native and Flutter. I was able to 
create a new project with `x new <project>` and build and install it to an Android emulator and it 
worked!

**References**
* [rust xbuild github](https://github.com/rust-mobile/xbuild)
* [Cargo Apk vs xbuild](https://blog.traverseresearch.nl/building-pure-rust-apps-for-android-d1e388b431b8)

Note: if you get a `error: failed to run custom build command for glib-sys v0.14.0` you need to 
install the `gtk3-dev` package.

1. First ensure you [install the Android SDK and Emulator](../../../android/emulator/README.md)
   
1. Install `cargo install xbuild`. Release `0.2.0` had errors. Worked around it by compiling master `268939a`
   ```bash
   $ git clone https://github.com/rust-mobile/xbuild
   $ cd xbuild
   $ cargo build --release
   $ mv target/release/x ~/.cargo/bin
   ```
2. Rust doctor will examine your system and check for missing tools
   ```bash
   $ x doctor
   --------------------clang/llvm toolchain--------------------
   clang                14.0.6              /usr/bin/clang
   clang++              14.0.6              /usr/bin/clang++
   llvm-ar              unknown             /usr/bin/llvm-ar
   llvm-lib             unknown             /usr/bin/llvm-lib
   lld                  14.0.6              /usr/bin/lld
   lld-link             14.0.6              /usr/bin/lld-link
   lldb                 14.0.6              /usr/bin/lldb
   lldb-server          unknown             /usr/bin/lldb-server
   
   ----------------------------rust----------------------------
   rustup               1.25.1              /usr/bin/rustup
   cargo                1.64.0              /usr/bin/cargo
   
   --------------------------android---------------------------
   adb                  1.0.41              /usr/bin/adb
   javac                11.0.17             /usr/bin/javac
   java                 11.0.17             /usr/bin/java
   kotlin               1.7.20-release-201  /usr/bin/kotlin
   gradle               7.5.1               /usr/bin/gradle
   
   ----------------------------ios-----------------------------
   idevice_id           1.3.0-167-gb314f04  /usr/bin/idevice_id
   ideviceinfo          1.3.0-167-gb314f04  /usr/bin/ideviceinfo
   ideviceinstaller     1.1.1               /usr/bin/ideviceinstaller
   ideviceimagemounter  1.3.0-167-gb314f04  /usr/bin/ideviceimagemounter
   idevicedebug         1.3.0-167-gb314f04  /usr/bin/idevicedebug
   
   ---------------------------linux----------------------------
   mksquashfs           4.5.1               /usr/bin/mksquashfs
   ```
3. Create a new test project and build it
   ```bash
   $ x new foobar
   $ cd foobar
   $ x build
   ```
4. List connected devices
   ```bash
   $ x devices
   host                                              Linux               linux x64           cyberlinux 6.6.2-arch1-1
   adb:emulator-5554                                 generic_x86_64_arm64android x64         Android 11 (API 30)
   ```
5. Build or run on device
   ```bash
   $ x build --device adb:emulator-5554
   [1/3] Fetch precompiled artifacts
   info: component 'rust-std' for target 'aarch64-linux-android' is up to date
   Android.ndk.tar.zst [54s] ███████████████████████████████ 66.73 MiB/66.73 MiB 📥 downloadedDownloading `platforms;android-33`
   Extracting `platforms;android-33`
   [1/3] Fetch precompiled artifacts [103419ms]
   [2/3] Build rust `template`
       Downloaded phf_macros v0.8.0
       ...
       Finished dev [unoptimized + debuginfo] target(s) in 0.11s

   [2/3] Build rust [143ms]
   [3/3] Create apk [958ms]
   ```
   * xbuild automatically installed `~/.android/platforms/android-33`  
   * xbuild created `foobar/target/x/debug/android/template.apk`  
6. Install apk
   ```bash
   $ adb install ~/Projects/temp/foobar/target/x/debug/android/template.apk
   ```

## Retained UI

### Dioxus
Mobile is currently the least-supported renderer target for Dioxus. Mobile apps are rendered with 
either the platform's WebView or experimentally with Blitz WGPU renderer.

**References**
* [iOS mobile example](https://github.com/DioxusLabs/example-projects/tree/master/ios_demo)
* [xbuild Dioxus](https://altimetrikpoland.medium.com/writing-multi-platform-gui-app-in-rust-f54aba3e97b8)

* Looking for contributers
* Currently only iOS is officially supported
* Blitz WGPU renderer via Vello
  * CSS is handled via lightningcss
  * Layout is handled with Taffy

### Slint
Has some initial work and toy apps but not ready for prime tine

* Uses the `skia` renderer for GPU acceleration and native looking text rendering
* 

### Flutter RS
