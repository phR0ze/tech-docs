# RustDesk <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Open source project written in Rust providing both a client and server. The project is cross platform 
and available in the AUR. Its meant to be a TeamViewer alternative and allows for remote service help 
like TeamViewer using an ID and RustDesk servers to connect in to assist your relatives or whatever. 
However you can also host the server and keep everything tightly controlled for a local solution as 
well.

### Quick links
* [../](../README.md)
* [Overview](#overview)
* [NixOS Configuration](#nixos-configuration)
  * [NixOS Client](#nixos-client)
  * [NixOS Server](#nixos-server)
* [TCP Tunnel](#tcp-tunnel)

## Overview
**Features**
* Cross-platform support, MacOS, Windows, Linux and Android
* Linux is X11 support only for now

**Review**
* Media playback
  * Video was delayed and a bit lower quality but definitely usable
  * Audio was automatically passed back to the controller took a sec to buffer
* Mobile
  * App only available from github release
  * Surprisingly usable with decent video and audio passthrough 
* Connectivity
  * Connectivity can be done via direct IP and it can be locked down to not allow relay access
  * Installing as a service is apparently possible but I was unable to get it to work
    * Without this you have to have your system automatically login then lock for it to work
  * Server component is not needed and useless for local LAN connectivity

**References**
* [RustDesk](https://rustdesk.com/)
* [rustdesk-server](https://github.com/rustdesk/rustdesk-server).

## NixOS Configuration
NixOS provides the following packages:
* `rustdesk-flutter` is the new Flutter based client which provides a `rustdesk` binary
* `rustdesk` is the deprecated [Sciter](https://sciter.com/) based client
* `rustdesk-server` is the server components

### NixOS Client
We'll be using the `rustdesk-flutter` option since the older Sciter based solution is deprecated.

**Installation**
```nix
environment.systemPackages = [
  pkgs.rustdesk
];
```

**Configuration**
The client can be configured programmatically by manipulating `~/.config/rustdeskt/RustDesk.toml` and 
`~/.config/rustdesk/RustDesk2.toml`

1. Click the three dots next to your `ID` and choose `ID/Relay Server`
2. Punch in your server's IP or DNS name as `ID Server`
3. Punch in the value from your `id_*.pub` server file as the `Key`
4. Leave the rest of the values blank

### NixOS Server
RustDesk provides the Signaling and Relay Server components as well for self-hosting. The server 
components are really only useful for external traffic negociation. If you want open up our RustDesk 
server components to the internet they will help negotiate a direct connection between peer instances 
or if hole punching is not possible then the relay server component will act as a middle may to relay 
between the two. However if you are just running locally on a LAN then the server component is not 
needed an just adds extra overhead. You can simply `Enable direct IP access` configure a permanent 
password and use it locally.

* `hbbs` the rendezvous/signaling server which listens on `21114` for http in Pro only and `21115`, 
  `21116`, `21118` for web sockets and UDP `21116`
* `hbbr` the relay server which listens on TCP `21117` and `21119`

Ports:
* TCP: 21114-21119
* UDP: 21116


## TCP Tunnel
RustDesk has the ability to link a local TCP port to a remote port acting like a proxy for NAT'd 
devices. This unlocks the use of other applications over the connection.

**References**
* [How to setup rustdesk tcp tunneling](https://www.youtube.com/watch?v=VgotqBMLd74&t=6s)
