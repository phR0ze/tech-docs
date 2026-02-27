# Android <img align="left" width="48" height="48" src="../../data/images/logo_256x256.png">

Android is finally morphing into something that might be worthwhile. The advent of NixOS, Flutter and 
Rust's move into the mobile space have finally built out a platform that is removed from the taint of 
Java far enough that its worth investing in.

### Quick links
- [.. up dir](../README.md)
* [Getting started](#getting-started)
  * [Install Android SDK](#install-android-sdk)
  * [ADB](#adb)
    * [Push files to device](#push-files-to-device)
    * [Install app on device](#install-app-on-device)
  * [Bootup process](#bootup-process)
    * [Application](#application)
    * [MainActivity](#mainactivity)
  * [Layout](#layout)
    * [Empty activity layout basics](#empty-activity-layout-basics)
    * [Legacy layouts and views](#legacy-layouts-and-views)
    * [Constraint Layout](#constraint-layout)
* [Additional research](#additional-research)
  * [AGDK](#agdk)

### Linked pages
- [Android Studio](android_studio/README.md)
- [Emulator](emulator/README.md)
- [Images](images/README.md)
- [Kotlin](kotlin/README.md)
- [Permissions](permissions/README.md)

## Getting started

References:
* [Android Env - NixOS Manual](https://nixos.org/manual/nixpkgs/unstable/#android)
* [Android developers guides](https://developer.android.com/guide)
* [ImageView Class](https://developer.android.com/reference/android/widget/ImageView)

### Install Android SDK
The Android SDK includes all the tooling to build and work with Android apps. Luckily it can be 
installed and used with VScode and is not dependent on Android Studio which is slow and clunky.

- [see my nixos-config](https://github.com/phR0ze/nixos-config/blob/main/options/development/android.nix)
- Check the installed versions with `sdkmanager --list_installed` gives me
  - build-tools `35.0.0`
  - cmdline-tools `13.0`
  - emulater `35.1.4`
  - extras;google;gcm `3`
  - ndk `27.0.12077973`
  - platform-tools `35.0.2`
  - platform `android-29`

## ADB
Open a shell and list out your connected devices. If you only have one you can run commands without 
calling out the target and they will default to that device.

```shell
$ adb devices
List of devices attached
emulator-5554	device
```

## Push files to device
Push files to your device with `adb push <local file> <remote location>`

Example:
```shell
$ adb push *.jpg /sdcard/Download/
```

## Install app on device
```shell
$ adb install APP.apk
```

## Bootup proces
Application is instantiate first

### Application
Application is the base class for maintaining global application state. You can provide your own 
implementation by creating a subclass and specifying the fully-qualified name of this subclass as the 
`android:name` attribute in your `AndroidManifest.xml`'s `<application>` tag. The Application class 
is instantiated before any other class when the process for your application is created.
* [Android dev - Application](https://developer.android.com/reference/android/app/Application.html)

### MainActivity

## Layout
Every project will have the `app >res >layout >activity_main.xml` that describes the main layout for 
your application.

### Empty activity layout basics
1. Open your `activity_main.xml`
2. Switch to split view by clicking the tiny `Split` icon above the design view to the right
3. Remove the default `TextView` control
4. Expand the design `Palette` in the top left of the design split window

### Legacy layouts and views
The following layouts have been put into Android's legacy layout category and their replacement views 
are called out. The primary reason is that the new layouts have been fine tuned for better 
performance.

| Legacy layout  | Modern Replacment
| -------------- | -----------------
| GridLayout     | RecyclerView + GridLayoutManager
| ListView       | RecyclerView
| TabHost        | TabLayout 
| RelativeLayout | ConstraintLayout
| GridView       | RecyclerView + GridLayoutManager

### Constraint Layout
Constraint Layout is the default that you'll receive for any Android apps you create. It is newer and 
combines some of the more powerful aspects of the older layouts i.e. `relative positioning` like 
RelativeLayout and `weighted positioning` like LinearLayout for better dynamically placed views.

### Card View
Card View is Android's default implementation for `Elevation Shadowing` and `Rounded Corners`. 
Essentially it is the `FrameLayout` with some background drawing happening for you. The overhead of 
rendering the shadowing and rounded corners is minimal. It is extremely common to use the `CardView` 
as the top level layout of a single item and then have a `Constraint Layout` inside it for multiple 
sub-components. This Card View item is then added to other layouts like the Recycler View.

### Recycler View
The Recycler View is one of Android's latest, most popular and optimized containers. The Recycler 
View contains a list of View Holder objects. Each View Holder is responsible for displaying a single 
item in the recycler view and only as many view holder items as can be viewed plus a few extra will 
be created. As the user scrolls the view holder items will be recycled and new data loaded into them 
and displayed rather than displaying all data at once. This behaves kind of like a stream reducing 
resources needed to load the content.

The adapter class overrie is responsible for creating view holder objects as needed. The adapter 
binds data to the view holder as needed. The recycler view then works in conjunction with a layour 
manager that you choose to display the view holder objects.

**Resources:**
* [Build a photo viewer](https://www.geeksforgeeks.org/how-to-build-a-photo-viewing-application-in-android/)

**Using the Recycler View:**
* Add the recycler view to your layout
  * From the design palette containers section drop in the `RecyclerView`
  * Set the recycler layout width and height to `match_parent`
  * Delete the layout editor tools entries
  * Give the RecyclerView an id `android:id="@+id/rvMain"`
* Design the XML layout for a single item in the recycler view
  * From the file explorer right click on `res/layout` and choose `New >Layout Resource file`
  * Name the new file `single_item`
* Create a `RecyclerView.ViewHolder` subclass
* Create a `RecyclerView.Adapter` subclass
* Use a `RecyclerView.LayoutManager` like the built in `linear layout manager` or `grid layout manager`
