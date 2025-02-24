# Jellyfin <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Jellyfin is an open source, self-hosted media management and playback system for movies, shows, 
music, books and photos. It is a server client architecture with support for live transcoding
to ensure your media always looks good on the target device. Jellyfin is competing with Plex
in this arena, which is more established and has a paid option. That said Jellyfin is close
to feature parity.

**Review**
* Crossplatform client support 
* Web: login, 

### Quick links
* [.. up dir](../README.md)
* [Overview](#overview)
  * [Server Install](#server-install)
  * [Init Configs](#init-configs)
  * [Jellyfin Media Player](#jellyfin-media-player)
* [Media Management](#media-management)
* [User Management](#user-management)
* [Plugins](#plugins)
* [General Configuration](#general-configuration)

## Overview
Jellyfin is a Free Software Media System that puts you in control of managing and streaming your 
media. It is an alternative to the proprietary Emby and Plex.

**References**
* [Jellyfin docs](https://jellyfin.org/docs/)

### Server Install
A standard Jellyfin server install might looks something like this on NixOS

```nix
# Enable Jellyfin media server
services.jellyfin = {
  enable = true;
  openFirewall = true;        # TCP: 8096,8920; UDP: 1900,7359
};

environment.systemPackages = [
  pkgs.jellyfin               # Jellyfin core
  pkgs.jellyfin-web           # Jellyfin web client
  pkgs.jellyfin-ffmpeg        # Jellyfin codecs bundle
  pkgs.jellyfin-media-player  # Crossplatform desktop client similar to Plex Media Player
  pkgs.kodiPackages.jellyfin  # Kodi add-on that combines the best of Kodi w/Jellyfin backend
];    
```

### Init Configs
It works best approach configuration as follows to minimize effort:

1. Open a browser to the sever e.g. `192.168.1.61:8096`
2. Work through the wizard but skip the library setup section
3. Install plugins
3. Configure users

### Jellyfin Media Player
The Jellyfin media player is a standalone application wrapping a browser to provide the same web 
interfaces as if you were to navigate to the server. The player on first boot asks where your 
Jellyfin server is. Once given the address e.g. `192.168.1.100:8096` it proceeds with a wizard to 
guide you through setup. 

* Added benefit of the player is full surround sound

### Jellyfin MPV Shim
[Jellyfin MPV Shim](https://github.com/jellyfin/jellyfin-mpv-shim) is a cross-platform cast client 
for Jellyfin. It has support for all advanced media files without transcoding, as well as tons of 
features.
* Direct play most media using MPV
* Watch videos with friends using SyncPlay
* Jellyfin mobile apps can fully control the client

1. Install with
   ```nix
   environment.systemPackages = [
     pkgs.jellyfin-mpv-shim
   ];    
   ```
2. Configure by launching the `Multimedia >Jellyfin MPV Shim`
   1. Enter your server e.g. `localhost`
   2. Enter an account
   3. Click `Add Server` then `Close`
      * Note: it will stay running in the systray
3. Add `Jellyfin MPV Shim` to your automatically started apps
   ```nix
    environment.etc."xdg/autostart/jellyfin-mpv-shim.desktop".text = ''
      [Desktop Entry]
      Type=Application
      Terminal=false
      Exec=bash -c "sleep 5 && ${pkgs.jellyfin-mpv-shim}/bin/jellyfin-mpv-shim"
    '';
   ```

## Kodi and Jellyfin are better together
Many tech enthusiasts will have installed Kodi and setup a SMB or NFS share to view their media on a 
PC or two in their home. This is great for simple playback; however for the initiated you'll soon 
find that the meta data around the media such as favorites or tracking watched status will be out of 
sync with other Kodi clients because Kodi does everything locally. This is where Jellyfin shines. It 
will give you a central media library with file access similar to NFS or SMB but also with 
centralized meta data tracking to keep all your Kodi instances in sync.

**References**
* [Kodi Jellyfin repo](https://jellyfin.org/docs/general/clients/kodi/#install-add-on-repository)

The `Jellyfin for Kodi Add-on` syncs metadata from selected Jellyfin libraries into the local Kodi 
database. This has the effect of making interacting with it feel very much like a vanilla Kodi with 
local media. Syncing may take a bit of extra time on first boot but is mostly in teh background while 
Kodi is running.

The `JellyCon` add-on behaves more like a standard Kodi streaming add-on. Media is accessed primarily 
by going through the Add-ons > JellyCon menu and isn't as integrated and doesn't sync all the meta 
data locally like the full official add-on.

### Install Kodi Sync Queue
Jellyfin recommends installing the `Kodi Sync Queue` on the Jellyfin server to keep your media 
libraries up to date with Kodi plays.

### Install Jellyfin for Kodi
On the Kodi side we can follow the [Jellyfin instructions](https://jellyfin.org/docs/general/clients/kodi#install-add-on-repository)
to add a new Jellyfin repo and the Jellyfin add-on. Once Jellyfin is configured as the backend for 
Kodi, Kodi will also become a supported Google Cast option from your mobile app or web app to be 
controlled.

1. Download the Kodi repo to your Kodi system
   ```bash
   $ wget https://kodi.jellyfin.org/repository.jellyfin.kodi.zip
   ```
2. Launch Kodi and navigate to `Add-ons >Enter add-on browser`
3. Choose `Install from zip file` and choose the downloaded file
   1. First enable `Unknown Sources`
4. Navigate to `Install from repository >Kodi Jellyfin Addons >Video add-ons`
5. Choose `Jellyfin` and click `Install`
6. Accept the automatically discovered server and sign-in
7. Choose `Add-on(default)` for `Playback mode`
   * **WARNING** Jellfin documentation says `Native (direct)` is no longer functioning
8. Select all the libraries to add

Note: to get access to different Jellyfin accounts you need to create different Kodi profiles per 
account and install the plugin and so forth for each account. This will then show the media as 
desired such as parental controls.

### Kodi create profiles
It is useful for a family to have a few different profiles to filter content to appropriate age 
levels e.g. `Adult`, `Teen`, `Child`.

1. Create `child` profile
2. For a child its best to change `Lock preferences`
   1. **WARNING** come back and do this after you create the profile and enable Jellyfin
   2. Leave `Profile lock` as `Disabled` so that no login is needed
   3. Uncheck all features but `videos`, `music`, `pictures`
2. Ensure you set the `Media info` and `Media sources` at the bottom to `Separate` this will give the 
   new provile a separate isolated space from the other profiles to show different content
3. Choose both `Start fresh` options
4. Login with new user
5. Configure Jellyfin backend
   1. Navigate to `Settings >Add-ons >Add-on browser >My add-ons >Video add-ons >Jellyfin`
   2. Click `Enable` then `Open`
   3. Jellyfin will then automatically popup an server connection modal
   4. Select or enter your server and credentials for the account
   5. Choose `Add-on (default)` and select the appropriate libraries e.g. `All`
      * **WARNING** you have to actually click `All` even though its highlighted to select it

## Media Management
Generally it is best practice to separate our your media from the metadata. There are a couple of 
reasons for this.

* Allows you to keep your media read-only to protect it from accidental modifications
* Allows for keeping media on slower disks with metadata on a faster disk
* Allows for easily switching media management applications
* No need to worry about metadata permissions for the application

### Trailers channel
Apparently a plugin?

### Organization
Having a single top level media share from you NAS to your server is a simple way to get access and 
since the server would be the only direct access machine setting up more granular shares isn't 
necessary. However inside the share granularity is needed to allow for separation of libraries at the 
Jellyfin level. Then using Jellyfin accounts we can provide the granularity over the libraries.

That said its simpler just to create a couple top level folders and use metadata to filter down.

**Folders**
* `Movies >Kids` - separate section for animated content
* `Movies >Adult` - anything not in the kids folder
* `Shows >Kids` - separate section for kids shows
* `Shows >Adult` - anything not in the kids folder

**Libraries**
* `Movies`
* `Kids Movies`
* `Shows`
* `Kids Shows`

**Accounts**
* `family` libraries would be filtered by rating
* `adult` no filters

**Tagging** can be a great way to provide additional organization to your media without requiring 
separate folders.

* Holiday tags: `Christmas`, `Easter`, `Halloween`
* Perhaps ratings: `adult-tag`

### Add Media Library
Note: authors of Jellyfin recommend not using the `Mixed Movies and Shows`

Navigate to `Dashboard >Libraries >Libraries >Add Media Library`

1. Choose content type `Movies`, `Music`, `Shows`, `Books`, `Home Videos and Photos`, `Music Videos`
2. Set the `Folders` where the media resides `/mnt/Media/Movies` can be more than one
3. Set `Preferred download language` and `Country/Region`
4. Keep `Allow all` for `Disable different types of embedded subtitles`
5. Keep `Enable real time monitoring`
6. Leave `Metadata downloaders (Movies)` as is
7. Leave `Automatically refresh metadata from the internet` at `Never`
8. Check `Metadata savers` as `nfo`
9. Don't check `Save artwork into media folders`
10. Check `Enable trickplay image extraction`
11. Check `Extract trickplay images during the library scan`
12. Check `Save trickplay images next to media`
13. Check `Enable chapter image extraction`

### Add Shows Library
Note: authors of Jellyfin recommend not using the `Mixed Movies and Shows`

Navigate to `Dashboard >Libraries >Libraries >Add Media Library`

1. Choose content type `Movies`, `Music`, `Shows`, `Books`, `Home Videos and Photos`, `Music Videos`
2. Set the `Folders` where the media resides `/mnt/Media/Movies` can be more than one
3. Set `Preferred download language` and `Country/Region`
4. Keep `Allow all` for `Disable different types of embedded subtitles`
5. Keep `Enable real time monitoring`
6. Leave `Metadata downloaders (Movies)` as is
7. Leave `Automatically refresh metadata from the internet` at `Never`
8. Check `Metadata savers` as `nfo`
9. Don't check `Save artwork into media folders`
10. Check `Enable trickplay image extraction`
11. Check `Extract trickplay images during the library scan`
12. Check `Save trickplay images next to media`
13. Check `Enable chapter image extraction`

## User Management
Each user has a user profile which can be configured by the administrator to control:
* Access to libraries
* Access to media by ratings

### child account
By creating a separate child account we can:

* limit to PG or better content
* limit access to only approved devices
* remove server modification options
* block content with the `adult` tag

## Hardware acceleration
Really this would only be useful for stream to phones or TVs as all PCs can play most any formats.

At a minimum this would be giving the `jellyfin` server user access to the `video` group.

## Plugins
Jellyfin provides a plugin system has a decent list of plugins available. They also have support for 
additional plugin repositories similar to Kodi's architecture. This allows for almost infinite 
support for different features.

Note: best to install plugins first as they affect how metdata is pulled with the libraries.

There are two main sections:
* `My Plugins` which installed plugins
* `Catalog` which shows available plugins

### Add 3rd-party Repos
* `jellyscrub` - scrub the timeline and see thumbnails
* daniel adov
* `intro skipper` - skip intro button
* tvdb trailers

### Plugins to install
* `Open Subtitles` - for subtitles
* `aniDB` - for Anime metadata
* `Fanart` - additional art
  * Once added update your library to then prioritize `Fanart`
  * Rescan library and replace all metadata
* `TheTVDB` - better at pulling the proper Show metadat than TheMovieDb
  * Once added update your library to then prioritize `TheTVDB`
* `Playback reporting` - shows playback stats
* `VGMdb` - video game music metadata
* `Skin Manager`
* `Simkl`


## General configuration

