# Kdenlive <img style="margin: 6px 13px 0px 0px" align="left" src="../../../../data/images/logo_36x36.png" />

Just worked out of the box. Clean, intuitive and didn't seem bloated at all. I'm quite happy with the 
result of the simple cut and join exercise I did. I think I'll switch from OpenShot which is always 
broken.

### Quick links
* [.. up dir](../README.md)
* [Overview](#overview)
  * [New project](#new-project)
* [Cut and Join](#cut-and-join)

### Linked pages

## Overview
I've been trying to use Openshot as my primary video editor, however every time I go to use it I find 
that it is broken in some way e.g. won't build, exports video without audio, crashes during exports. 
So I'm trying Kdenlive.

### New project
Drag and drop your videos in Kdenlive and they will be transcoded to an edit friendly format. MP4s by 
default are a compressed group-of-pictures i.e. the whole key frame thing. By converting it to a edit 
friendly format you are making readable frame-by-frame.

## Cut and Join
The process was very intuitive using the razor tool and simply selecting and deleting the unwanted 
footage then dragging in the want to join with.

1. Trigger export by running `Project >Render`
2. Choose `Generic >MP4-H264/AAC` as the output type
3. Click `Render to File`
4. It exported to `~/Videos/<project>.mp4` by default
