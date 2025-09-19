# Selkies <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Selkies is a high-performance replacement for older Linux remote desktop solutions, delivering 60 fps 
at 1080p with comparable performance to proprietary remote desktop solutions like `Google Stadia`, 
`GeForce NOW` or `Parsec`. Selkies is open-source, low-latency, high-performance, Linux-native and 
GPU/CPU-accelerated. It can stream Linux X11 desktops or Docker/Kubernetes containers to any recent 
web browser utilizing WebRTC HTML5. 

### Quick links
* [../](../README.md)
* [Overview](#overview)
  * [Benefits of Selkies](#benefits-of-selkies)
  * [Getting Started](#getting-started)

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

