# Media <img style="margin: 6px 13px 0px 0px" align="left" src="../data/images/logo_36x36.png" />

Documenting various multimedia technologies

### Quick links
* [.. up dir](../README.md)
* [Screen Recorder](#screen-recorder)

### Linked pages
- [Audio](audio/README.md)
- [CAD](cad/README.md)
- [Editing](editing/README.md)
- [Encoding](encoding/README.md)
- [Image](image/README.md)
 
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
