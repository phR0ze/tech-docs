# Audio <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />

### Quick links
* [.. up dir](../README.md)
* [Backup an audio CD](#backup-an-audio-cd)
* [Change speed of audio](audacity/README.md#change-speed-of-audio)
* [Convert WAV to MP3](#convert-wav-to-mp3)

## Backup an audio CD
1. Install `Asunder`
   ```bash
   $ sudo pacman -S asunder
   ```
2. Configure WAV uncompressed
   1. Click `Preferences`
   2. Select the `Encode` tab
   3. Check the `WAV (uncompressed)` option
3. Configure output directory
   1. Select the `General` tab
   2. Select the `Destination folder`
   3. Unchck `Create M3U playlist`
3. Turn on error correction (will slow it way down)
   1. Navigate to the `Advanced` tab
   2. Uncheck at the bottom the `Faster ripping (no error correction)` option

## Convert WAV to MP3
1. Install `Sound Konverter`
   ```bash
   $ sudo pacman -S soundkonverter
   ```
2. Launch `soundkonverter`
3. Navigate to `File >Add folder...` and select your target audio file directory
4. Click `Proceed` to have it automatically select all audio file types
5. Set `Qualtity: High`, `Destination` directory and hit `OK`
6. Click `Start`


