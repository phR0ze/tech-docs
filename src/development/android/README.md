# Android
<img align="left" width="48" height="48" src="../../data/images/logo_256x256.png">

Documenting my learning experience with Android development

### Quick links
* [AGDK](#agdk)
* [Android Studio](#android-studio)
  * [Install Android Studio](#install-android-studio)
  * [Update Android Studio](#update-android-studio)
  * [Configure Android Studio](#configure-android-studio)
    * [Map keyboard shortcuts](#map-keyboard-shortcuts)
    * [Turn off paramater hints](#turn-off-paramater-hints)
  * [Create a new project](#create-a-new-project)
  * [Create a new virtual device](#create-a-new-virtua-device)
  * [Logging](#logging)
    * [System out](#system-out)
* [Android Devevelopment](#android-development)
  * [SDK](#sdk)
    * [Update SDK](#update-sdk)
    * [platform tools](#platform-tools)
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
* [Images and graphics](#images-and-graphics)
  * [Supported media formats](#supported-media-formats)
  * [Image loaders](#image-loaders)
  * [Glide](#glide)
    * [Add Glide library](#add-glide-library)
    * [Load Glide](#load-glide)
  * [Picasso](#picasso)
    * [Add Picasso](#add-picasso)
* [Kotlin](#kotlin)

# AGDK
Official Android Game Develeopment kit provides
* Game Activity
  * Simplifies interop between native code and Java world
* Game Text Input
  * Simple API to show and hide software keyboard input
  * Get and Set text
* Game Controller
  * Easy to use C API
  * Callbacks for detecting controller connections/disconnections
  * Device information, input data
* Frame Pacing API
* High Performance Audio
* Memory Advice API Beta

**Features**
* Get the Game libs as AAR packages from Jetpack

# Android Studio

## Install Android Studio
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

# Android Development

References:
* [Android developers guides](https://developer.android.com/guide)
* [ImageView Class](https://developer.android.com/reference/android/widget/ImageView)

## SDK
The Android SDK is all the tools to build and work with Android apps. I've installed it at:
`~/Android/Sdk`

### Update SDK
To update the Android SDK
1. Open Android Studio launch `android-studio`
2. Navigate to `Tools >SDK Manager`

### platform tools
In the SDK path there is a directory `platform-tools` where you can find the android tools

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

# Images and graphics
* ***Drawable*** is a general abstraction for something that can be drawn. Subclasses help with specific scenarios
  * Inflate an image resource (a bitmap file) saved in your project
  * Inflate an XML resource that defines the drawable properties
* ***ImageView*** displays image resources like `Bitmap` or `Drawable` and to apply tints and scale images

## Supported media formats
Apparently libraries like `Picasso` don't usually do the decoding but rather rely on [Android to 
handle that](https://developer.android.com/guide/topics/media/media-formats#core).

## Image loaders
Three main contenders exist for handling image loading, `Picasso`, `Glide` and `Fresco`. The solution 
space includes loading images asynchronously, processing errors, displaying placeholders, caching and 
transforming pictures.

Resources:
* [Image loaders](https://redwerk.com/blog/advanced-feature-in-android-image-loaders-picasso-vs-glide-vs-fresco/)
* [Image loader animations](https://redwerk.com/blog/android-image-loaders-animations-picasso-vs-glide-vs-fresco/)

| Feature       | Picasso | Glide | Fresco |
| ------------- | ------- | ----- | ------ |
| Image loading |    1    |       |        |
| URL Downloader|    1    |   1   |    1   |
| Resizing      |    1    |       |        |
| CenterCrop    |    1    |       |        |
| Rotation      |    1    |       |        |
| Error images  |    1    |       |        |
| Priority load |    1    |       |        |
| Caching       |    1    |       |        |
| Fade Effect   |    1    |   1   |    1   |
| Animated Gif  |    0    |   1   |    1   |
| Video frame   |    0    |   1   |        |
| Progress Bars |    1    |   1   |    1   |

### Caching
* Picasso supports 2 types of cache by default: LRU cache size of 15% of RAM and disk cache from 5 to 
50MB. The functions `memoryPolicy()` and `networkPolicy()` control these values. The first allows you 
to refuse to access the online cache during **image loading** `MemoryPolicy.NO_CACHE` or not to 
**save** the downloaded image to the cache `MemoryPolicy.NO_STORE` and with the second function you 
can regulate the disk cache. Using `NetworkPolicy.NO_CACHE` or `NetworkPolicy.NO_STORE`. Additionally 
`NetworkPolicy.OFFLINE`.
```kotlin
Picasso.get()
  .load(imageUrl)
  .memoryPolicy(MemoryPolicy.NO_CACHE, MemoryPolicy.NO_STORE)
  .networkPolicy(NetworkPolicy.NO_CACHE, NetworkPolicy.NO_STORE)
  .into(test_imageview)
```
* Glide has 4 levels of caching: `active resources` displayed in view, `memory cache` data in RAM, 
  `resource` converted and decoded image, `data` raw image saved to disk. By default before accessing 
  an external resource Glide searches for a suitable image through each level.
```kotlin
Glide.with(this)
  .load(imageUrl)
  .diskCacheStrategy(DiskCacheStrategy.NONE)
  .skipMemoryCache(true)
  .into(test_imageview)
```
* Fresco is also hierarchical and has 3 levels: ***Bitmap*** decoded images ready for display 
post-processing, ***Encoded*** compressed images in the original state in memory, ***Disk*** 
compressed images in the original state on disk

### Transformation
One of the main advantages of image loaders is the ability to process images before display.
* Picasso can `resize`, `centerCrop`, `centerInside`, and `rotate`.
* Glide same as Picasso plus `circlCrop`, `roundedCorners`, `grandularRoundedCorners`
* Fresco has the most sophisticated tools, but has some limitations. It can only resize JPEGs.

### Progress Bars
When downloading large images or in cases of an unstable internet connection, a progress bar may come 
in handy, either infinite or displaying the download status.

* Picasso allows for an infinite progress bar using the `placeHolder()` function. 
`Picasso.get().load(imagUrl).placeHolder(R.drawable.infinitive_progressbar).into(imageView)`
* Glide offers a more comprehensive approach with place holder images but supports gifs
* Fresco is the simplest with a simple xml addition and a dynamic call. It also supports a download 
indicator feature

### Fade Effect
When loading images from disk and not cache images will appear to fade into view.

* Picasso doesn't allow for any tuning other than on or off `Picasso.get().load(imageUrl).noFade().into(imageView)`
* Glide 4 allows for defining the transition time `Glide.with(this).load(imageUrl).transition(DrawableTransitionOptions.withCrossFade(5)).into(imageView)`
* Fresco allows for defining fade duration in the layout xml `fresco:fadeDuration="3000"` or 
dynamically 
`frescoImageView.setHierarchy(GenericDraweeHierarchyBuilder(getResources()).setFadeDuration(3000).build())`

### Glide
[Glide](https://github.com/bumptech/glide) by Bump Tech, allows for complex image transformations. It 
is a fast and efficient open source media management and image loading framework that wraps media 
decoding, memory, disk caching and resource pooling into a simple and easy to use interface.

### Picasso
Created by Square, is know for its minimalistic approach adding only `121 Kb` to your 
apk. `NetworkPolicy.OFFLINE` instructs Picasso to download data only from the cache, without 
accessing the network.

```
Picasso.get()
  .load(imageUrl)
  .memoryPolicy(MemoryPolicy.NO_CACHE, MemoryPolicy.NO_STORE)
  .networkPolicy(NetworkPolicy.NO_CACHE, NetworkPolicy.NO_STORE)
  .into(test_imageview)
```

### Fresco
Created by Facebook, focuses on working efficiently with memory and productivity. Uses its own 
`SimpleDrawView` instead of the traditional `ImageView`.

## Glide
Glide supports the `gif` format out of the box

Resources:
* [Wendergram Example](https://www.raywenderlich.com/2945946-glide-tutorial-for-android-getting-started)
* [Glide Tutorial](https://www.thecrazyprogrammer.com/2017/05/android-glide-tutorial.html)

### Add Glide library
1. Open the gradle script `build.gradle` for the Module app
2. Add the following
```
dependencies {
  implementation 'com.github.bumptech.glide:glide:4.11.0'
  annotationProcessor 'com.github.bumptech.glide:compiler:4.11.0'
}
```

### Load Glide

## Picasso
Picasso is seems like the simplest and most widely used option; but doesn't support `gif`

Resources:
* [Kotlin Picasso example](https://camposha.info/android-examples/android-picasso/#gsc.tab=0)

## Add Picasso
Adding a dependency to your project can be done by
1. Navigate to `File >Project Structure...`
2. Select `Dependencies` on the left hand side
3. Select the `app` module in the middle top
4. Click the little `+` icon in the `Declared dependencies` section
5. Select `Library` and search for `Picasso`
6. Select the `com.squareup.picasso 2.71828` and click `OK`
7. Modify `gradle.properties` to add compatibility for Picasso with
   ```
   android.enableJetifier=true
   ```

# Kotlin
* `val test: Any` type `Any` allows for expressions to return different types
* `kotlin.Unit` is what is returned when there is no value
* Kotlin doesn't allow for declaring multiple variables in a single statement
* Kotlin prefers expressions over statements. This is a modern technique shared by other languages 
like Rust and allows for returning values out of a conditional block.
```kotlin
val bigger = 1000
val smaller = 1
val max = if (bigger > smaller) bigger else smaller
```

## Primitive Types

### Boolean
```kotlin
var exists = false
```

Boolean logic:
```kotlin
if (first || second) {
  println("first or second is true")
}

if (first && !second) {
  println("first is true and second is false")
}
```

### Character
* Char e.g. `var letter = 'A'`
* ASCII `var tab = '\t'`
* Unicode `var infinity = '\u221E'`

### Literals
* Hex e.g. `0xFEDC`
* Binary e.g. `0b101010`

### Numbers
Types can be inferred based on the size and type of number used e.g. `val foo = 1_230.1F`

Oddities:
* Long's require the `L` after the number
* Float requires the `F` after the number else Double is used
* Underscores in numbers are ignored and work well for place separators

* `Byte` 8 bits
* `Short` 16 bits
* `Int` 32 bits
* `Long` 64 bits e.g. 1234L

* `Double` 64 bits
* `Float` 32 bits e.g. 1234.5F

### Nullable types
Kotlin made nullable types special, use `?` suffix to turn any type into nullable. You can also use 
the `?` to make the operation safe.

```kotlin
val person: Person? = null
val greeting: String? = "hello world"
prinln("message: ${greeting?.length}")

// Default to 0 if greeting is null
val len = greeting?.length ?: 0
```

### Strings
```kotlin
val owe = 50
val janet = "I owe Janet \$$owe dollars"
println(janet)
```

## Immutable Variables
Using the `val` declaration makes the value immutable
```kotlin
val foo: Int = 0
```

## Control Flow

### loop
Doesn't have the standard 3 part for loop. Supports modern iteration and standard `break` and 
`contiue`.

```kotlin
for (i in 1..10) {
  println("i = $i")
}

val students = listOf("foo1", "foo2")
for (student in students) {
  println("Current student: $student")
}

for ((i, student) in students.withIndex()) {
  println("Current student: $student, index: $student")
}
```

### when expression
Seems to be basically the same thing as a Rust `match`

* Must be exhaustive
* `else` catch all can be used for non exhaustive lists

## Collections
Fundamental collection types are Lists, Sets and Maps. All come in `Read-only` or `Mutable`. Neither 
has anything to do with `val` or `var`.

### Arrays
Fixed length on declaration

```kotlin
// can be of multiple types
val things = arrayOf(1, 2, 3, "one", "two")

// Must be the same type
val things = arrayOf<Int>(1, 2, 3)
```

### Lists
Mutable length

```kotlin
val names = listOf("foo1", "foo2")
```

<!-- 
vim: ts=2:sw=2:sts=2
-->
