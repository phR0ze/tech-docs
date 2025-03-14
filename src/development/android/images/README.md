# Images <img align="left" width="48" height="48" src="../../data/images/logo_256x256.png">

* ***Drawable*** is a general abstraction for something that can be drawn. Subclasses help with specific scenarios
  * Inflate an image resource (a bitmap file) saved in your project
  * Inflate an XML resource that defines the drawable properties
* ***ImageView*** displays image resources like `Bitmap` or `Drawable` and to apply tints and scale images

### Quick links
- [.. up dir](../README.md)
* [Supported media formats](#supported-media-formats)
* [Image loaders](#image-loaders)
* [Glide](#glide)
  * [Add Glide library](#add-glide-library)
  * [Load Glide](#load-glide)
* [Picasso](#picasso)
  * [Add Picasso](#add-picasso)

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
## AGDK
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


