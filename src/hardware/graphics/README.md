# Graphics

Notes on how I determined the correct configuration for the video card.

1. The output of `inxi -G`
   ```bash
   Graphics:  Device-1: Advanced Micro Devices [AMD/ATI] Turks GL [FirePro V4900] driver: radeon v: kernel 
              Display: x11 server: X.org 1.21.1.11 driver: loaded: radeon note: n/a (using device driver) 
              resolution: <missing: xdpyinfo> 
              OpenGL: renderer: AMD TURKS (DRM 2.50.0 / 6.6.18 LLVM 16.0.6) v: 4.5 Mesa 24.0.1 
   ```

2. The `vainfo` tool provides the following and matches, so it seems VA-API is supported
   ```bash
   $ vainfo
   ...
   vainfo: VA-API version: 1.20 (libva 2.20.1)
   vainfo: Driver version: Mesa Gallium driver 24.0.1 for AMD TURKS (DRM 2.50.0 / 6.6.18, LLVM 16.0.6)
   vainfo: Supported profile and entrypoints
   ...
   ```

3. Test va-api usage with mpv. Since this is an older card HEVC wasn't supported but using an older 
   h.264 encoded video file I was able to see that VA-API is being used.
   ```bash
   $ mpv --hwdec=vaapi VIDEOFILE
    (+) Video --vid=1 (*) (h264 1920x800 23.976fps)
    (+) Audio --aid=1 --alang=eng (*) (aac 6ch 48000Hz)
    (+) Subs  --sid=1 --slang=eng '2012 (2009) [1920x800.h264-1080p.AAC-2ch].eng.srt' (subrip) (external)
   ...
   Using hardware decoding (vaapi).
   AO: [pulse] 48000Hz 5.1 6ch float
   VO: [gpu] 1920x800 vaapi[nv12]
   ```
