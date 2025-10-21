# Download <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

### Quick links
* [.. up dir](../README.md)

## yt-dlp
yt-dlp can be an excellent way to archive your uploaded youtube videos.

### Latest on Nix
Frequently you'll want the latest version of `yt-dlp` on Nix. This is super easy using `nix-shell` 
just point to the latest unstable and your good.

```bash
$ nix-shell -p '(import (fetchTarball "https://github.com/NixOS/nixpkgs/archive/nixpkgs-unstable.tar.gz") {}).yt-dlp'
```

### Choose the avilable formats
You can list out all the available formats for a given youtube link using the `-F` option.

```bash
$ yt-dlp -F https://SHORT-SHARABLE-LINK

ID  EXT   RESOLUTION FPS CH │  FILESIZE   TBR PROTO │ VCODEC        VBR ACODEC      ABR ASR MORE INFO
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────
sb2 mhtml 48x27        0    │                 mhtml │ images                                storyboard
sb1 mhtml 80x45        0    │                 mhtml │ images                                storyboard
sb0 mhtml 160x90       0    │                 mhtml │ images                                storyboard
140 m4a   audio only      2 │   5.25MiB  129k https │ audio only        mp4a.40.2  129k 44k [en] medium, m4a_dash
251 webm  audio only      2 │   4.66MiB  115k https │ audio only        opus       115k 48k [en] medium, webm_dash
91  mp4   256x144     30    │ ~ 6.33MiB  156k m3u8  │ avc1.4D400C       mp4a.40.5           [en]
160 mp4   256x144     30    │   2.78MiB   69k https │ avc1.4d400c   69k video only          144p, mp4_dash
93  mp4   640x360     30    │ ~22.58MiB  557k m3u8  │ avc1.4D401E       mp4a.40.2           [en]
134 mp4   640x360     30    │  11.24MiB  277k https │ avc1.4d401e  277k video only          360p, mp4_dash
18  mp4   640x360     30  2 │ ≈16.44MiB  405k https │ avc1.42001E       mp4a.40.2       44k [en] 360p
243 webm  640x360     30    │   8.57MiB  211k https │ vp9          211k video only          360p, webm_dash
95  mp4   1280x720    30    │ ~63.24MiB 1560k m3u8  │ avc1.64001F       mp4a.40.2           [en]
136 mp4   1280x720    30    │  42.38MiB 1045k https │ avc1.64001f 1045k video only          720p, mp4_dash
```

* Download specific option: `yt-dlp -f 95 $LINK`
* Download specific video and audio: `yt-dlp -f 136+251 $LINK`
