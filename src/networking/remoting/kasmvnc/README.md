# KasmVNC <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Kasm Technologies developed Kasm Workspaces, the Containerized Streaming Platform. Kasm has 
open-sourced the Workspace docker images, including containerized full desktops and apps. These 
containers can be used standalone or within the Kasm Workspaces. KasmVNC is used as the streaming 
tech for Kasm Workspace container images, however, KasmVNC can also be used directly on servers as a 
remote desktop solution.

* Modern - KasmVNC is not compatible with other VNC clients and exposes KVM via Browser technology
* Secure - KasmVNC is regularly pen tested to ensure its secure
* Simple - KasmVNC aims at being simple to deploy and configure

### Quick links
* [../](../README.md)
* [Overview](#overview)
 
## Overview

**References**
* [KasmVNC server docs](https://kasmweb.com/kasmvnc/docs/latest/serverside.html)
* [KasmVNC docs](https://kasmweb.com/kasmvnc/docs/latest/index.html)
* [KasmVNC github](https://github.com/kasmtech/KasmVNC)
* [KasmVNC nixos work](https://github.com/moonpiedumplings/moonpiedumplings.github.io/blob/e127127b73f55a9ccdcb17b0844b9cfe10ca7063/projects/compiling-kasmvnc-on-nixos/index.qmd#L4)
Of all the VNC variations KasmVNC is the most modern and advanced. To do so they are no longer 
* [KasmVNC fixed infra](https://kasmweb.com/docs/latest/how_to/fixed_infrastructure.html#kasmvnc)

### Configuration

**References**
* [KasmVNC yaml config](https://kasmweb.com/kasmvnc/docs/latest/configuration.html)

## NxOS package
[kasmproxy](https://kasmweb.com/kasmvnc/docs/latest/man/kasmxproxy.html?utm_source=chatgpt.com) can 
be used to proxy an existing X11 display to a KasmVNC display. This is usually done in the context of 
providing GPU accelerations to a KasmVNC session.

[Xvnc](https://kasmweb.com/kasmvnc/docs/latest/man/Xvnc.html) is the X Virtual Networking Computing 
server. It serves an X server to the applications and a VNC server for remote VNC clients.


