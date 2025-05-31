# PDFs <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

### Quick links
- [.. up dir](../README.md)
- [PDF Operations](#pdf-operations)
  - [PDF Manipulation](#pdf-manipulation)
  - [Convert Images to PDF](#convert-images-to-pdf)

## PDF Operations

### PDF Manipulation
For general PDF manipulation `pdfslicer` works well. It is a simple UI tool to `extract`, `merge`, 
`rotate` and `reorder` pages of PDF documents.

```bash
$ nix-shell -p pdfslicer
$ pdfslicer
```

### Convert Images to PDF
```bash
# Install imagemagick
$ sudo pacman -S imagemagick

# Comment out the stupid security policy default in Arch Linux
$ sudo vim /etc/ImageMagick-7/policy.xml
# <policy domain="coder" rights="none" pattern="{PS,PS2,PS3,EPS,PDF,XPS}" />

# Convert images to pdf
# -quality is the jpeg compression to use for images
$ convert -resize 50% -quality 98 -units pixelsperinch -density 150 image1.jpg image2.jpg output.pdf
```

