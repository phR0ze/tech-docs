# Images <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

### Quick links
* [.. up dir](../README.md)
* [Animated Images](#animated-images)
* [Disk Cache](#disk-cache)

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


