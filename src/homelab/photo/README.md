# Photo <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />

The self-hosted space has some awesome photo management options. I'll be documenting my research and 
choices here.

### Quick links
* [.. up dir](../README.md)
* [Comparison](#comparison)

## Comparison
Using my own requirements as the lense through which I view potential solutions I'll be reviewing the 
foss options listed in the [Foss photo library apps list](https://meichthys.github.io/foss_photo_libraries/).

**Contenders**
* [Ente](ente/README.md)
* [Immich](immich/README.md)
* [Librephotos](librephotos/README.md)
* [NextCloud](nextcloud/README.md)
* [Photoprism](photoprism/README.md)
* [Photonix](photonix/README.md)
* [Piwigo](piwigo/README.md)
* [Snapcrescent](snapcrescent/README.md)

**Factors**
* ***Longivity (LO)*** - open-source, not-interfeering paid model, modern language e.g. Rust, Go, Flutter
* ***Community (C)*** - thriving community for additional support
* ***Deployability (D)*** - easily get up and running
* ***File Access (F)*** - easy file recovery for preservation and avoid lock in
* ***Shared Space (S)*** - share all features with multiple accounts in a family use case
* ***Facial and Object Recognition (AI)*** - facial and object recognition
* ***Web Interface (UI)*** - web interface general feel
* ***Mobile Backup (MB)*** - android client that automatically save to the server
* ***Mobile Access (MA)*** - android client that automatically save to the server
* ***Play Animated (PA)*** - play animated images on hover

**Rating**
* ***0*** - no support
* ***1*** - bad
* ***2*** - maybe
* ***3*** - acceptable
* ***4*** - good
* ***5*** - fantastic

| Project       | LO | C | D | F | S | AI | UI | MB | MA | PA |
| ------------- | -- | - | - | - | - | -- | -- | -- | -- | -- |
| `Ente`        | 5  | 2 | 1 | 0 | 1 | ?  | 3  | ?  | ?  | ?  | 
| `Immich`      | 5  | 5 | 5 | 3 | 2 | 5  | 5  | 3  | 4  | ?  |
| `?`           | ?  | ? | ? | ? | ? | ?  | ?  | ?  | ?  | ?  |
| `Photonix`    | 1  | 1 | ? | ? | ? | ?  | 1  | ?  | ?  | ?  |
| `PhotoPrism`  | 3  | 4 | 5 | 3 | 0 | 3  | 2  | ?  | ?  | 5  |
| `Piwigo`      | 1  | 2 | 2 | ? | ? | ?  | 1  | ?  | ?  | ?  |
| `Photoview`   | 3  | 2 | ? | ? | ? | ?  | 2  | ?  | ?  | ?  |
| `Snapcrescent`| 1  | 1 | 1 | ? | ? | ?  | ?  | ?  | ?  | ?  |
| `Synology`    | 2  | 0 | 5 | 5 | 4 | 3  | 5  | 5  | 4  | ?  |

### Photonix
* Longevity: Python and Javascript
* Community: tiny following
* Deployability: Demo works
* UI: slow and clunky

### Photoprism
Portainer test instance can be spun up and available at `0.0.0.0:2342` with `admin/insecure`

* Longevity
  - Using GoLang on the backend
  - Slow dated feeling UI based on Vue progressive Web app
* Community
  - Large open-source community
* Deployability
  - Portainer integration out of box
  - Seems to have full NixOS integration
* Web UI
  - Interface seems clunky and awkward
  - Provides a `Folders` view which is nice
  - Default detailed view of photos is ugly and not useful
  - Animated images will play when hovering with mouse
  - Videos will auto play when selected
  - Modern escape to close view
  - Places map looks nice
* Mobile
  - TBD

### Piwigo
Portainer test instance can be spun up and available at `0.0.0.0:2343`, but some incantation is 
needed to run the php installer. It wasn't intuitive for me.

* Longevity
  - Built on PHP
* Community
  - Smallish open-source community for 22 years of presence
* Deployability
  - Portainer integration out of box
  - Seems to have full NixOS integration
* Web UI
  - Slow, dated, and clunky
* Mobile
  - TBD

### Photoview
* Longevity: Go and Typescript
* Community: tiny following
* Deployability: Demo works
* UI: slow and clunky

### Snapcrescent
* Longevity: Built on JAVA
* Community: Non-existant
* Deployability: Demo won't even work
