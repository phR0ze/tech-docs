# OBS <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Open Broadcaster Software (OBS) is the defacto standard when it comes to open-source, cross-platform 
streaming.

### Quick links
* [.. up dir](../README.md)
* [Getting started](#getting-started)
  * [Install OBS Studio](#install-obs-studio)
* [Sources](#sources)
  * [SRT source](#srt-source)
  * [NDI source](#ndi-source)
* [Virtual Camera](#virtual-camera)

## Getting started

### Install OBS Studio

1. Install OBS Studio with the NDI plugin
   ```nix
   environment.systemPackages = [
     (pkgs.wrapOBS {
       plugins = with pkgs.obs-studio-plugins; [
         obs-backgroundremoval                 # Remove or blur the video background
         obs-pipewire-audio-capture            # Capture appliation audio with PipeWire
         obs-ndi                               # NDI support
       ];
     })
   ];
   
   # polkit needs to be enabled so that OBS can access the virtual camera device
   security.polkit.enable = true;
   
   # Provide support for OBS virtual display
   boot.extraModprobeConfig = ''options v4l2loopback devices=1 video_nr=1 card_label="OBS Cam" exclusive_caps=1'';
   ```
2. Configure machine as an NDI source i.e. will send this machine's screen as output to a client
   1. Launch OBS Studio
   2. Navigate to `Tools >NDI Output settings`
   3. Check the `Main Output` box and click `OK`

## Sources


### SRT source
1. In the `Sources` section at the botton near the left
2. Click the `+` button and choose `Media Source`
3. Give it a name e.g. `Mevo SRT` then click `OK`
4. Uncheck `Local File`
5. Set `Input` to your camera's srt address e.g. `srt://192.168.1.112:4201`
6. Set `Input Format` to `mpegts`
7. Check `Use hardware decoding when available`
8. Click `OK` and you should see in a couple seconds the video show up

### NDI source
TBD
