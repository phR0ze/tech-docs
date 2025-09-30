# Selkies <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Selkies is a high-performance replacement for older Linux remote desktop solutions, delivering 60 fps 
at 1080p with comparable performance to proprietary remote desktop solutions like `Google Stadia`, 
`GeForce NOW` or `Parsec`. Selkies is open-source, low-latency, high-performance, Linux-native and 
GPU/CPU-accelerated. It can stream Linux X11 desktops or Docker/Kubernetes containers to any recent 
web browser utilizing WebRTC HTML5. 

**Summary**
* Pros
  * Lauded as the next best solution
* Cons
  * All the documentation and examples are focused on container usage
  * Spent several days building NixOS package and getting it to run
  * When it finally did run it didn't seem to run the server component
* Result
  * I may try this again later

### Quick links
* [../](../README.md)
* [Overview](#overview)
  * [Benefits of Selkies](#benefits-of-selkies)
  * [Getting Started](#getting-started)
* [Packaging for NixOS](#packaging-for-nixos)

## Overview
VNC's underlying protocol RFB is 30 years old and shows it. There are better ways to do things now. 
`Selkies-GStreamer` is that better way. It streams a Linux X11 desktop or Docker®/Kubernetes 
container to a recent web browser using WebRTC with GPU hardware or CPU software acceleration from 
the server and the client. Selkies is a hybrid VNC using modern technologies.

`RustDesk` is really the only modern open source competition, but really competes more accurately with 
TeamViewer for remote support rather than large-scale client-server deployments such as cloud or 
clustered deployments of many end systems and applications.

The `linuxserver.io` community is using this technology to build docker based apps that expose the 
app UI over a port to be opened in a browser. Its a very cool idea, but my brief experimentation with 
their [Selkies based images](https://www.linuxserver.io/blog/webtop-3-0-part-2-the-eagle-has-landed#selkies-based-images)
seemed to be pretty sluggish, but that could have been the docker memory constraints or something. 
Regardless it could be a neat way to spin up ephemeral type environments. Selkies is actually a 
multi-generational choice by the linuxserver.io group. They have evolved from a Guacuamole based 
experiment to KasmVNC based to finally arrive at Selkies as their choice to power their desktop 
application containers a.k.a. Web-Based Linux Desktops

I only mention this as an aside as the 
real as my intent is to use Selkies for a remote desktop solution instead.
**References**
* [Selkies project](https://github.com/selkies-project/)
* [Selkies github](https://github.com/selkies-project/selkies)
* [Selkies motivation](https://selkies-project.github.io/selkies/design/#motivation)
* [Selkies getting started](https://selkies-project.github.io/selkies/start/)
* [Webtop with Selkies](https://www.linuxserver.io/blog/webtop-3-0-part-1-a-call-to-revolutionize-web-based-linux-desktops#the-foundation-the-selkies-project)
* [lsio based images](https://hub.docker.com/r/lsiobase/selkies)

### Benefits of Selkies
`Selkies-GStreamer` has several strenghts over other game streaming or remote desktop services:
1. Selkies is much more flexible
   * HTTP based so it can leverage the existing ecosystem
   * Single port needed
2. Modern video compression
   * Audio with `Opus`
   * Leverages `H.264 encoding with hardware acceleration`
   * Fallsback on software acceleration with H.264, H.265, VP8, VP9, AV1
3. Supports Docker and Kubernetes containers
   * No dependencies on special devices or systemd
   * Now you can remote desktop into a container instead of only a VM

### Getting Started

## Packaging for NixOS
When you run `selkies-gstreamer` the default being run are
```selkies-gstreamer --addr=0.0.0.0 --port=8080 --enable_https=false --https_cert=/etc/ssl/certs/ssl-cert-snakeoil.pem --https_key=/etc/ssl/private/ssl-cert-snakeoil.key --basic_auth_user=user --basic_auth_password=mypasswd --encoder=x264enc --enable_resize=false
```
to change the encoder use `--encoder=` and specify `nvh264enc`, `vah264enc`, `vp9enc`, or `vp8en`
vaapi `DRI_NODE=/dev/dri/renderD128`






Selkies is composed of multiple “mandatory components”:
1. A Python backend / signaling / host-side part (distributed as a wheel) selkies-project.github.io
2. A GStreamer build / media pipeline / native dependencies component (which deals with encoding, GPU interfaces, etc.) 
3. A web (HTML5 / frontend) component (JavaScript / web assets) 
The upstream project publishes release artifacts (wheel for the Python parts, tarballs for web 
frontend, prebuilt GStreamer binaries) via GitHub Actions. 

The “Getting Started” docs show how to install dependencies, unpack the web assets, install the 
Python wheel, set up GStreamer, etc. 

So, a Nix package (or set of packages) would need to:
* Pull in or build the GStreamer / native dependencies + correct plugins
* Install / wrap the Python side
* Install the web assets (into a “static web root” that the packaged server can serve)
* Provide a service or “runScript” so that selkies-gstreamer (or whatever the entrypoint is) can find the necessary native dependencies (GStreamer path, libraries, etc.) at runtime

Steps to build a NixOS package
1. Fetch source artifacts either fetchFromGitHub or builtins.fetchTarball
   * The Python wheel (or source) — you might even build from source if upstream provides it
   * The web tarball (frontend assets)
   * The GStreamer binaries or build GStreamer (or the required plugins)

If the upstream artifacts are sufficient (wheel + web + precompiled native libs), your Nix expression 
can simply download and “install” them (i.e. unpack into the right place, wrap paths). If not, you 
may have to build GStreamer or parts of Selkies from source, which is more work.

3. Define nativeBuildInputs / buildInputs
   * GStreamer and the required gstreamer plugins (e.g. x264, vaapi, etc.)
   * Python & setuptools, maybe gst-python bindings
   * Node / npm / yarn (if you need to build the web assets from source)
   * Other system libraries referenced in the docs (e.g. libx11, libxcb, libpulse, etc.) 
   * In Nix you’d import the relevant gstreamer packages (e.g. gstreamer, gst-plugins-good, etc.) from Nixpkgs.

4. Staging / install phases
   * Put the Python wheel into the output’s Python site-packages or create a wrapper
   * Copy / unpack the web assets into e.g. $out/share/selkies/web
   * Install the native libs / binaries (if you compiled them) into $out/lib, $out/bin
   * Write a wrapper script for the selkies-gstreamer command so that it sets environment variables 
   such as `GSTREAMER_PATH`, `LD_LIBRARY_PATH`, `SELKIES_WEB_ROOT`, etc., so that the binary and Python 
   side picks up the correct artifacts

You may also need to patch paths in configuration files, JSON manifests in the web assets (e.g. 
manifest.json), etc., to point to correct web-root. The upstream docs mention customizing 
manifest.json and sw.js for rebranding the web interface. 

5. Wrap the executable / runtime environment
   * `LD_LIBRARY_PATH` includes your package’s lib and gstreamer plugin directories
   * `GSTREAMER_PATH` is set so that the GStreamer plugins in your package are found
   * `SELKIES_WEB_ROOT` or equivalent environment variables point to the installed web assets

6. Optionally provide a NixOS module / systemd service
