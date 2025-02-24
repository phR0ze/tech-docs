# Editing <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

### Quick links
* [.. up dir](../README.md)
* [Add audio](#add-audio)
* [Backup a Blu-ray](#backup-a-bluray)
* [Backup a DVD](#backup-a-dvd)
* [Burning a DVD](#burning-a-dvd)
* [Cut Video w/out Re-encoding](#cut-video-without-re-encoding)
* [Extracting specific chapters](#extracting-specific-chapters)
* [Burning an DVD](#burning-an-dvd)
* [Encode Blu-ray to x265](#encode-blu-ray-to-x265)
* [Encode DVD to x265](#encode-dvd-to-x265)
* [Lossless Rotate](#lossless-rotate)
* [Strip GPS Location](#strip-gps-location)

### Linked pages
- [Kdenlive](kdenlive/README.md)

## Add audio
Adding and removing audio from a MOV file can be done with OpenShot

1. Click the Add button to add your video
2. Drag your video down into the editing section below
3. Right click on your video track and choose `Add Track Below`
4. Right click on your video and select `Separate Audio >Single Clip (all channels)`
5. Now right click the audio track and select `Remove Clip`
6. Click the Add button to add your new audio
7. Drag your new audio down into the editing section below
8. Offset the new audio as desired
9. 

## Backup a Blu-ray
Instructions for backing up your legally purchased personal collection.

1. Build and install the latest bits
   ```bash
   $ yay -Ga makemkv
   $ cd makemkv
   $ makepkg -s
   $ sudo pacman -U makemkv-1.16.5-1-x86_64.pkg.tar.zst
   $ sudo pacman -S ccextractor
   ```
2. Register makemkv
   a. Launch `makemkv`  
   b. Navigate to the [MakeMKV Forum](https://forum.makemkv.com/forum/viewtopic.php?t=1053)  
   c. Use the key in the app at `Help >Register`  
3. Backup your Blu-ray
   a. Hit thie big button to open up the Blu-ray  
   b. Setup the output directory on the right  
   c. Uncheck everything but the largest title on the left  
   d. Drill in and uncheck any audio and sub-title languages you don't want  
   e. Hit the `Save selected titles` button on the right  

## Backup a DVD
Instructions for backing up your legally purchased personal collection.

1. First install the tooling:
   ```bash
   $ sudo pacman -S dvdbackup libdvdcss
   ```
2. Determine the name of your optical drive:
   ```bash
   $ lsblk
   NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
   sr0     11:0    1   7.2G  0 rom  
   ```
3. Simply clone the entire dvd to make a backup of your personal collection:
   ```bash
   $ dvdbackup -i /dev/sr0 -o <movie-name> -M
   ```
4. Create an ISO from the backup:
   ```bash
   # Create the AUDIO_TS if it doesn't exist, helps with convetional dvd player compatibility
   $ mkisofs -dvd-video -o image.iso ~/dir_containing_video_ts
   ```
5. Verify that your iso is playable:
   ```bash
   $ mplayer dvd::// -dvd-device image.iso
   ```

## Burning a DVD
1. Use step 2 from [Backup a DVD](#backup-a-dvd) to determine your device name

2. Use `growisofs` to burn the image to a disc:
   ```bash
   $ growisofs -dvd-compat -Z /dev/sr0=image.iso
   ```

## Cut Video w/out Re-encoding
1. Install: `sudo pacman -S losslesscut-bin`
2. Launch: `losslesscut`
3. Drag and drop your `mkv` file from [Encode DVD to x265](#encode-dvd-to-x265) into the main window
4. Accept the prompt to `Import chapters`
5. Right click on the first chapter and select `Jump to cut start`
6. Now remove chapters to include in the segment you want up until the last one you want
7. Now use the `Set cut start to current position` button to expand the last piece up to the start
8. Right click on this new expanded chapter and choose `Include ONLY this segment in export`
9. Click `Export` button validate settins and click `Export` again


## Extracting specific chapters
Note often when doing a full dvd backup the first chapter will be the menu

1. Follow the instructions from [Backup a DVD](#backup-a-dvd) to get a local copy to work with

2. Determine which titles you want to extract from:
   ```bash
   # -i needs to refer to the location where the VIDEO_TS is stored
   $ dvdbackup -i backup -I
   ...
   Title set 1
     The aspect ratio of title set 1 is 4:3
   	 Title set 1 has 1 angle
   	 Title set 1 has 1 audio track
   	 Title set 1 has 0 subpicture channels
   
   	 Title included in title set 1 is
   	   Title 1:
   			 Title 1 has 22 chapters
   			 Title 1 has 2 audio channels
   ```

2. Now extract the chapters you'd like e.g. chapter 1:
   ```bash
   # -s start chapter 
   # -e start chapter 
   # -n output directory name to use
   $ dvdbackup -i backup -t 1 -s 1 -e 1 -n Chapter-01
  
   # Ignore the libdvdread
   # libdvdread: Couldn't find device name
   ```

3. Use bash to extract them all skipping menu and offsetting numbers:
   ```bash
   $ for i in {1..21}; do dvdbackup -i backup -t 1 -s $((i+1)) -e $((i+1)) -n "Chapter-0${i}"
   ```

## Encode Blu-ray to x265
1. Install: `sudo pacman -S handbrake`
2. Launch: `ghb`
3. Configure main page settings  
   a. Click `Open Source`  
   b. Navigate to the target `mkv` file and double click it  
   c. Set `Preset` to `Official > General >Super HQ 1080p30 Surround`  
   d. Change `Format` to `Matroska (avformat)`  
   e. Change the `Save As` location to `~/Downloads`  
4. Select the `Dimensions` tab  
   a. Ensure that `Automatic` is selected  
5. Select the `Video` tab  
   a. Ensure that `Video Encoder` is set to `H.265 10-bit (x265)`  
   b. Set `Framerate` to `Same as source` and enable `Variable Framerate`  
   c. Set `Constant Quality` value to `RF: 20` and `Preset` to `slower`  
   d. Set `Tune` to `animation` if applicable  
6. Select the `Audio` tab  
   a. Ensure the `Track List` includes your desired language  
7. Select the `Subtitles` tab  
   a. Ensure the `Track List` says `Foreign Audio Scan -> Burned into Video (Forced Subtitles Only)`  
8. Name the output file  
   a. e.g. `Title (Year) [1080p.x265.AC3.rf20].mkv`  

## Encode DVD to x265
1. Install: `sudo pacman -S handbrake`
2. Launch: `ghb`
3. Configure main page settings  
   a. Click `Open Source`  
   b. Navigate to the `VIDEO_TS` directory and click `Open`  
   c. Set `Preset` to `Official >Matroska >H.265 MKV 480p30`  
4. Select the `Dimensions` tab  
   a. Ensure that `Auto Crop` is selected  
5. Select the `Video` tab  
   a. Ensure that `Video Encoder` is set to `H.265(x265)`  
   b. Set `Framerate` to `Same as source` and enable `Variable Framerate`  
   c. Set `Constant Quality` value to `RF: 18` and `Preset` to `slower`  
   d. Set `Tune` to `animation` if applicable  
6. Select the `Audio` tab  
   a. Ensure the `Track List` includes your desired language  
7. Select the `Subtitles` tab  
   a. Ensure the `Track List` says `Foreign Audio Scan -> Burned into Video (Forced Subtitles Only)`  
8. Name the output file  
   a. e.g. `Title (Year) [480p.x265.AC3.rf19].mkv`  

## Lossless Rotate
This just sets the metadata for the video. Its up to your player to play it with the correct rotation

Rotate clockwise by 90 degrees
```bash
$ ffmpeg -i $INPUTVIDEO -metadata:s:v rotate="90" -codec copy $OUTPUTVIDEO
```

Rotate clockwise by 270 degrees
```bash
$ ffmpeg -i $INPUTVIDEO -metadata:s:v rotate="270" -codec copy $OUTPUTVIDEO
```


