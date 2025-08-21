# Editing <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

### Quick links
* [.. up dir](../README.md)
* [Add audio](#add-audio)
* [Backup a Blu-ray](#backup-a-bluray)
* [Backup a DVD](#backup-a-dvd)
* [Burning a DVD](#burning-a-dvd)
* [Video operations without re-encoding](#video-operations-without-re-encoding)
  * [Drop tracks](#drop-tracks)
  * [Cut video](#cut-video)
  * [losslesscut](#losslesscut)
* [Extracting specific chapters](#extracting-specific-chapters)
* [Burning an DVD](#burning-an-dvd)
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

## Video operations without re-encoding

### Drop tracks
Using `mkvtoolnix-gui` we can drop un-needed tracks without re-encoding the original video. This 
allows for a storage savings and is lightning fast.

1. Launch `mkvtoolnix-gui`
2. Add your file
   1. Right click in the `Source files` pane and select `Add source files`
   2. Select your target MKV file
3. Remove tracks as desired
   1. Remove foreign audio
   2. Remove foreign subs
   3. Mark English tracks as defaults as desired
5. Click `Start multiplexing`

### Cut video

***First find out the exact time where you want to cut the video***
1. This can be done with `smplayer` by enabling OSD settings
   1. Navigate to `View >OSD`
   2. Choose the `Volume + Seek + Timer + Total time`
   3. Repeat and choose at the bottom `Show times with miliseconds`
2. Alternatively you can load it into `losslesscut`

***Cut the video with mkvmerge***
I noticed some glitches when I used losslesscut the last time so switched to mkvmerge which processed 
it way faster and was perfect quality.

```bash
$ mkvmerge -o output.mkv --split parts:00:00:27.945-01:43:21.696 input.mkv
```

***Cut the video with losslesscut***
**WARNING:** I had some weird file size bloat i.e. doubled the size with lossless

1. Install: `sudo pacman -S losslesscut-bin`
2. Launch: `losslesscut`
3. Drag and drop your `mkv` file from [Encode DVD to x265](#encode-dvd-to-x265) into the main window
4. Accept the prompt to `Import chapters`
5. Enable the advanced options of the app in the bottom left
6. Select the location in the video where you would like to start
   1. Mark the start by using the `Start current segement at current time` button
7. Select the location in the video where you would like to stop
   1. Mark the end by using the `End current segement at current time` button
8. Click `Export+merge`
   1. Choose `Merge cuts`
   2. Choose the Output container format e.g. `mp4`
   3. Select the appropriate audio and subs tracks to keep
   4. Finally click the `Export+merge` button again

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


