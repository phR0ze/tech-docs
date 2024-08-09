# Imaging

### Quick links
* [Metadata](#metadata)
  * [EXIF](#exif)
* [Image libs](#image-libs)
  * [zune-image](#zune-image)
* [Image Viewers](#image-viewers)
  * [avis-imgv](#avis-imgv)
* [Formats](#formats)
  * [JPEG](#jpeg)
    * [turbojpeg](#turbojpeg)
  * [TIFF](#tiff)
* [XDG thumbnail database](#xdg-thumbnail-database)

## Metadata
Image metadata is additional contextual data stored in the image itself. Many image formats have 
their own way of doing this but some specifications have been designed that are used across formats 
such as `EXIF`, `IPTC` and `XMP`

### EXIF
Existing crate review

**References**
* [EXIF 3.0 spec](https://archive.org/details/exif-specs-3.0-dc-008-translation-2023-e/)
* [XMP meta data spec](https://www.cipa.jp/std/documents/download_e.html?CIPA_DC-010-2024_E)

| Crate                                             | Desc                              | Status
| ------------------------------------------------- | --------------------------------- | -----------
| [exif](https://crates.io/crates/exif)             | FFI bindings to `libexif`         | Abandoned
| [exif-rs](https://crates.io/crates/exif-rs)       | Native, partial implementation    | Abandoned
| [exif-sys](https://crates.io/crates/exif-sys)     | Clone of `exif` with new name     | Abandoned
| [exifsd](https://crates.io/crates/exifsd)         | Native, JPEG exif read            | Abandoned
| [gexiv2](https://crates.io/crates/gexiv2-sys) 	  | FFI bindings for gexiv2           | `Active`
| [gufo-exif](https://crates.io/crates/gufo-exif)   | Native, focused on editing exif   | `Active`
| [imagemeta](https://crates.io/crates/imagemeta)   | Native, read/write partial        | Abandoned 
| [img-parts](https://crates.io/crates/img-parts)   | Native, read/write parts          | Abandoned
| [kamadak-exif](https://crates.io/crates/kamadak-exif) | Native, some write support    | Quasi standard
| [little\_exif](https://crates.io/crates/little_exif) | Native, partial implementation | `Active`
| [nom-exif](https://crates.io/crates/nom-exif)     | Native, read JPEG,HEIF,MOV,MP4    | `Active`
| [peck-exif](https://crates.io/crates/peck-exif) 	| Wrapper around `exiftool`         | Abandoned
| [quickexif](https://crates.io/crates/img-parts)   | Native, parse only needed tags    | Abandoned
| [rexif](https://crates.io/crates/rexif) 	        | Native, JPEG, TIFF readonly       | `Active`
| [rexiv2](https://crates.io/crates/rexiv2) 	      | FFI bindings for gexiv2 	        | `Active`

**Date**
* `DateTimeOriginal`
* `CreationDate`
* `DateTimeCreated`
* `CreateDate`
* `DateCreated`
* `Datecreate`
* `TrackCreateDate`

## Image libs

### zune-image
An opinionated image library supporting: BMP, Farbfeld, HDR, JPEG, JPEG-XL, PNG, PPM, QOI

## Image Viewers
### avis-imgv
[avis-imgv](https://github.com/hats-np/avis-imgv) is a rust image viewer using egui

## Formats

### JPEG
* Start signature `0xFF, 0xD8, 0xFF, 0xE0`
* End signature `0xFF, 0xD9`

#### jpeg extractor
[jpeg extractor](https://crates.io/crates/jpeg_extractor) will extract jpegs from any binary file

#### turbojpeg
[rust-turbojpeg](https://crates.io/crates/turbojpeg) is Rust bindings for 
[TurboJPEG](https://github.com/libjpeg-turbo/libjpeg-turbo) which provides simple fast JPEG 
operations.

**Features**
* Lossless JPEG transformations

### TIFF
* [TIFF digital format](https://web.archive.org/web/20220412210014/https://www.loc.gov/preservation/digital/formats/content/tiff_tags.shtml)
* [tiff\_tags rust crate](https://crates.io/crates/tiff_tags)

## XDG thumbnail database
[AllMyToes](https://crates.io/crates/allmytoes) follows the XDG thumbnail data-base to create cached 
thumbnails for many file types.


