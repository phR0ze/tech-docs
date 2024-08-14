# Image

### Quick links
* [Metadata](#metadata)
* [Formats](#formats)
  * [JPEG](#jpeg)
* [Libs](#libs)
* [Tools](#tools)
  * [Exiftool](#exiftool)

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


