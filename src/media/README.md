# Media <img style="margin: 6px 13px 0px 0px" align="left" src="../data/images/logo_36x36.png" />

Documenting various multimedia technologies

### Quick links
* [.. up dir](../README.md)
* [3D Computer Graphics](#3d-computer-graphics)
  * [FreeCAD](FREECAD.md)
  * [Sweet Home 3D](#sweet-home-3d)
* [Audio](#audio)
  * [Backup an audio CD](#backup-an-audio-cd)
  * [Change speed of audio](audacity/README.md#change-speed-of-audio)
  * [Convert WAV to MP3](#convert-wav-to-mp3)
* [Image](#image)
  * [Printing](#printing)
    * [Print Size](#print-size)
  * [Metadata](#metadata)
  * [Formats](#formats)
    * [JPEG](#jpeg)
  * [Libs](#libs)
  * [Tools](#tools)
    * [Exiftool](#exiftool)
  * [Convert HEIC to JPEG](#convert-heic-to-jpeg)
* [Screen Recorder](#screen-recorder)

### Linked pages
- [Editing](editing/README.md)
- [Self-hosted](self-hosted/README.md)
 
# 3D Computer Graphics
* Sweet Home 3d
* FreeCAD
* LibreCAD
* OpenSCAD
* Remo 3d
* SAS offerings
  * [Sketchup Free](https://app.sketchup.com)
  * [OnShape Free](https://www.onshape.com/en/products/free)

## Sweet Home 3D

# Audio

## Backup an audio CD
1. Install `Asunder`
   ```bash
   $ sudo pacman -S asunder
   ```
2. Configure WAV uncompressed
   1. Click `Preferences`
   2. Select the `Encode` tab
   3. Check the `WAV (uncompressed)` option
3. Configure output directory
   1. Select the `General` tab
   2. Select the `Destination folder`
   3. Unchck `Create M3U playlist`
3. Turn on error correction (will slow it way down)
   1. Navigate to the `Advanced` tab
   2. Uncheck at the bottom the `Faster ripping (no error correction)` option

## Convert WAV to MP3
1. Install `Sound Konverter`
   ```bash
   $ sudo pacman -S soundkonverter
   ```
2. Launch `soundkonverter`
3. Navigate to `File >Add folder...` and select your target audio file directory
4. Click `Proceed` to have it automatically select all audio file types
5. Set `Qualtity: High`, `Destination` directory and hit `OK`
6. Click `Start`

# Image

## Printing

### Print Size
Print Size = Resolution / DPI, thus 1200x1800 image printed at 300 DPI will be `4"x6"`
* 1200/300 = 4"
* 1800/300 = 6"

## Metadata
Images usually store a compressed encoded image. However they can also contain metadata about the 
image as well. This is information about the image such as the date it was taken, the orientation it 
was taken in, the geographical coordinates where the photo was taken etc...

### EXIF

**Resources**
* [MIT Exif description](https://www.media.mit.edu/pia/Research/deepview/exif.html)
* [JPEG Orientation](https://magnushoff.com/articles/jpeg-orientation/)

## Formats

### JPEG
Exif, IPTC and XMP metadata specifications can all be embedded in the JPEG format.

### JFIF
JPEG File Interchange Format (JFIF) is an 
The vanilla JPEG specification doesn't include any method of coding the resolution or aspect ratio 
into the image.


## Libs

### libexif

### Exiv2
[Exiv2](https://exiv2.org/) is a C++ metadata library and toolset for Exif, IPTC, XMP and ICC

### TurboJPEG
[rust-turbojpeg](https://crates.io/crates/turbojpeg) is Rust bindings for 
[TurboJPEG](https://github.com/libjpeg-turbo/libjpeg-turbo) which provides simple fast JPEG 
operations.

* Encoding
* Decoding
* Lossless transformations

## Tools

### Exiftool
Metadata tool for Exif, IPTC, XMP, JFIF, GeoTIFF, ICC Profile, Photoshop IRB, FlashPix, AFCP, and ID3 
as well as many manufacturer specific metadata formats.


## Other Libs
* libJPEG
* libJPEG turbo
* libPNG
* Little CMS
* ZLIB
* JXRLIB
* LibWebP
* LibRaw
* libarchive
* z lib

## Convert HEIC to JPEG
Image conversions are easily done with `imagemagick`
```bash
# Install imagemagick
$ sudo pacman -S imagemagick

# Convert all heic pix to jpeg
$ mogrify -format jpg *.heic

# Delete the original heic files
$ rm *.heic
```

# Screen Recorder
The two best are ***SimpleScreenRecorder*** and ***RecordMyDesktop***

1. Install: `sudo pacman -S simplescreenrecorder`
2. Launch: `simplescreenrecorder`
3. Click ***Continue***
4. Click ***Record a fixed rectangle*** then ***Select window...***
5. Click the content area of the window to be recorder to only record the content
6. Uncheck ***Record cursor***
7. Uncheck ***Record audio***
8. Click ***Continue***
9. Click ***Browse*** to choose a file to record as
10. Choose ***Preset*** as ***faster***
11. Click ***Continue***
12. Click ***Start Recording***

## Strip GPS Location
1. Install: `sudo pacman -S perl-image-exiftool`
2. List out the existing exif info: `exiftool <file>`
3. Remove gps exif data: `exiftool -all= <file>`

<!-- 
vim: ts=2:sw=2:sts=2
-->
