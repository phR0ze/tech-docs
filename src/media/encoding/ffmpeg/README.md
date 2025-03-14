# ffmpeg <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

### Quick links
* [.. up dir](../README.md)
* [Encode MP4](#encode-mp4)

## Encode MP4

### Quality controls
Quality is controlled by specifying the bitrate through `-b:v` for video and `-b:a` for audio or by 
specifying any other encoding method the codec my support.

For x264 there are various encoding methods, with the `Constant Rate Factor (CRF)` method being the most 
sophisticated. It results in variable bitrate, but overall good quality in one single pass. CRF 
ranges from 0 to 51, but sane values are usually between 19 and 26. `23` is the default while `18` 
whould be high quality and `28` would be low quality.

```bash
$ ffmpeg -i input.mp4 -c:v libx264 -crf 23 output.mp4
```

