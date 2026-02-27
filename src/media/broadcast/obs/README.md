# OBS <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Open Broadcaster Software (OBS) is the defacto standard when it comes to open-source, cross-platform 
broadcast streaming.

### Quick links
* [.. up dir](../README.md)
* [Getting started](#getting-started)
  * [Install OBS Studio](#install-obs-studio)
* [Sources](#sources)
  * [SRT source](#srt-source)
  * [NDI source](#ndi-source)
  * [Webcam source](#webcam-source)
* [Talking head](#talking-head)
  * [Virtual Camera](#virtual-camera)

## Getting started

### Install OBS Studio
You can install OBS with as just the vanilla package `obs-studio` or with plugins below:

1. Install OBS Studio with the NDI plugin
   ```nix
   boot.extraModulePackages = with config.boot.kernelPackages; [
     v412loopback
   ];
   boot.kernelModules = [ "v412loopback" ];
   boot.extraModprobeConfig = ''
     options v4l2loopback devices=1 video_nr=10 card_label="OBS Virtual Camera" exclusive_caps=1
   '';
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

### Popular plugins


## Sources

### SRT source
In the `Sources` section at the botton near the left
1. Click the `+` button and choose `Media Source`
2. Give it a name e.g. `Mevo SRT` then click `OK`
3. Uncheck `Local File`
4. Set `Input` to your camera's srt address e.g. `srt://192.168.1.112:4201`
5. Set `Input Format` to `mpegts`
6. Check `Use hardware decoding when available`
7. Click `OK` and you should see in a couple seconds the video show up

### NDI source
TBD

### Webcam source
In the `Sources` section at the bottom near the left:
1. Click the `+` button and choose `Video Capture Device (V4L2)`
2. Give it a name e.g. `HP Webcam` then click `OK`
3. Under the `Device` drop down select your device e.g. `HP Webcam 3110`
4. Adjust the new source with the red box controls to move and resize
5. Crop by holding `Alt` while dragging an edge

## Talking head
Here's how to set up OBS with your HDMI capture card as the main source and camera as a
picture-in-picture overlay:

### Add the HDMI Capture Card Source
In the `Sources` section at the bottom near the left:
1. Click the `+` button and choose `Video Capture Device (V4L2)`
2. Ensure `Create new` is selected at the top and click `OK`
   * Select your USB device from the `Device` dropdown e.g. `Guermok USB2 Video`
   * Set `Resolution` to match your HDMI output (e.g. 1920x1080)
   * Set `Frame Rate` to `30.00`
   * Click `OK`
3. Right click on the new sources and rename to `HDMI Capture`
4. If it doesn't fill your canvas right-click it in the `Sources` > `Transform` > `Fit to Screen`

### Add the Camera Source and resize
In the `Sources` section at the bottom near the left:
1. Click the `+` button and choose `Video Capture Device (V4L2)`
2. Give it a name e.g. `HP Webcam` then click `OK`
3. Under the `Device` drop down select your device e.g. `HP Webcam 3110`
4. Adjust the new source with the red box controls to move and resize
5. Crop the camera
   1. Right click the camera preview and navigate to `Transform` > `Edit Transform`
   2. Set the `Crop Left` and the `Right` near the bottom then click `Close`

### Check Layer Order
In the Sources panel, sources are layered — items higher in the list appear on top. Make sure:
- Camera is above HDMI Capture in the list
- If not, drag it up or use the up/down arrows

### Virtual Camera
OBS has a built-in Virtual Camera feature that creates a fake device Zoom can use.

1. Navigate to `Tools >Start Virtual Camera`
