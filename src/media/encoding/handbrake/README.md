# handbrake <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />

### Quick links
* [.. up dir](../README.md)
* [Encode to AV1](#encode-to-av1)
  * [Animated shows](#animated-shows)
* [Encode Blu-ray to x265](#encode-blu-ray-to-x265)
* [Encode DVD to x265](#encode-dvd-to-x265)
* [Codecs](#codecs)
  * [AV1](#av1)

## Encode to AV1

**References**
* [Handbrake presets](https://handbrake.fr/docs/en/latest/technical/official-presets.html)

Handbrake's `Hardware >AV1 QSV 2160p60 4K` preset seems to be a great option:
* Hardware accelerated by my video card
* Dimensions match the source material
* Video Encoder is `AV1 (SVT)` which is the 8-bit version
* Framerate is `Same as source`
* My only changes were to bump:
  * Constant Quality RF to `30`
  * the AV1 Preset to `8`

### Re-encode Animated show
I tried a few different presets and video tweaks. The simplest that was still fast with decent 
compression was just the out of the box hardware accelerated AV1 with a couple changes.

1. Launch: `ghb`
2. Configure main page settings
   1. Click `Open Source`
   2. Navigate to the target video file and double click it
   3. Set `Preset` to `Official > Hardware >AV1 QSV 2160p60 4K`
   4. Constant Quality RF to `30`
   5. AV1 Preset to `8`
   6. Change the `Save As` location to `~/Videos`
3. Everything else seems fine as is

## Encode Blu-ray to x265
1. Launch: `ghb`
2. Configure main page settings
   1. Click `Open Source`
   2. Navigate to the target `mkv` file and double click it
   3. Set `Preset` to `Official > General >Super HQ 1080p30 Surround`
   4. Change `Format` to `Matroska (avformat)`
   5. Change the `Save As` location to `~/Downloads`
3. Select the `Dimensions` tab
   1. Ensure that `Automatic` is selected
4. Select the `Video` tab
   1. Ensure that `Video Encoder` is set to `H.265 10-bit (x265)`
   2. Set `Framerate` to `Same as source` and enable `Variable Framerate`
   3. Set `Constant Quality` value to `RF: 20` and `Preset` to `slower`
   4. Set `Tune` to `animation` if applicable
5. Select the `Audio` tab
   1. Ensure the `Track List` includes your desired language
6. Select the `Subtitles` tab
   1. Ensure the `Track List` says `Foreign Audio Scan -> Burned into Video (Forced Subtitles Only)`
7. Name the output file
   1. e.g. `Title (Year) [1080p.x265.AC3.rf20].mkv`

## Encode DVD to x265
1. Launch: `ghb`
2. Configure main page settings
   1. Click `Open Source`
   2. Navigate to the `VIDEO_TS` directory and click `Open`
   3. Set `Preset` to `Official >Matroska >H.265 MKV 480p30`
3. Select the `Dimensions` tab
   1. Ensure that `Auto Crop` is selected
4. Select the `Video` tab
   1. Ensure that `Video Encoder` is set to `H.265(x265)`
   2. Set `Framerate` to `Same as source` and enable `Variable Framerate`
   3. Set `Constant Quality` value to `RF: 18` and `Preset` to `slower`
   4. Set `Tune` to `animation` if applicable
5. Select the `Audio` tab
   1. Ensure the `Track List` includes your desired language
6. Select the `Subtitles` tab
   1. Ensure the `Track List` says `Foreign Audio Scan -> Burned into Video (Forced Subtitles Only)`
7. Name the output file
   1. e.g. `Title (Year) [480p.x265.AC3.rf19].mkv`

## Codecs

* 4K HDR10 is around 70-75Mbps with HEVC
* 1080p SDR can be compressed to near original quality with `10-11 Mbps HEVC` which is 60-70% smaller
* SDR
  * Come in 8bit color, SVT-AV1 by default

Gamble: AV1 (SVT) 8-bit RF 30, preset 8 and Opus audio

**References**
* [Handbrake presets](https://johnson.downclimb.com/2023/03/effects-of-handbrake-presets-and-rf.html)
* [Handbrake AV1 presets](https://gitlab.com/AOMediaCodec/SVT-AV1/-/blob/master/Docs/CommonQuestions.md)

### AV1
***AV1 (AOMedia Video 1)***  is a relatively new video codec developed by the Alliance for Open 
Media, a consortium that includes heavyweights like Google, Apple, Netflix, Amazon and Microsoft. AV1 
is designed to be open-source and royalty-free, with a focus on high compression efficiency. AV1 is 
essentially the next gen VP9 with wider support from other large tech companies.

* Pros:
  * License-free and open-source
  * Same quality but 25% less data than H.265
  * Same quality but 50% less data than H.264
* Cons:
  * Limited hardware support
* Summary:
  * AV1 will be the only option that makes sense in a few years

### VP9
***VP9*** is both open-source and royalty-free. It was developed by Google and designed to compete 
directly with HEVC.

* Pros: license-free
* Cons:
  * no hardware support and not widely used
  * only works with webm and mkv container formats

### H.265 or HEVC
***H.265*** or `(HEVC) High Efficiency Video Coding` is a newer, faster version of H.264. It has 
better video quality at the same bitrate or the same quality at a lower bitrte; meaning more space 
savings without quality loss. It can deliver essentiall the same quality for half the space.

* Pros: smaller file sizes compared to H.264
* Cons: no hardware decoding support for older machines
* Summary: not worth it if you have older systems

### H.264 or AVC
***H.264*** also known as `AVC (Advanced Video Coding)` or `MPEG-4 Part 10`, is the most widely used 
video codec in today's digital world. Introduced in 2003, it is remarkably compression efficient. For 
example, compared to the older H.263 and MPEG-2, H.264 can deliver similar quality at half the 
bitrate or less.

* Pros: hardware decoding support in almost all systems
* Cons: larger file sizes compared to more modern codecs
* Summary: best choice for older systems


