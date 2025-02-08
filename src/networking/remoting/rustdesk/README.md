# RustDesk <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Open source project written in Rust providing both a client and server. The project is cross platform 
and available in AUR. Its meant to be a TeamViewer alternative and allows for remote service help 
like TeamViewer using an ID and RustDesk servers to connect in to assist your relatives or whatever. 
However you can also host the server and keep everything tightly controlled for a local solution as 
well.

**Features**
* Cross-platform support, MacOS, Windows, Linux and Android
* Linux is X11 support only for now

### Quick links
* [../](../README.md)
* [Overview](#overview)
* [NixOS Configuration](#nixos-configuration)
  * [NixOS Client](#nixos-client)
  * [NixOS Server](#nixos-server)
* [Arch Linux Configuration](#arch-linux-configuration)

## Overview

**References**
* [RustDesk](https://rustdesk.com/)
* [rustdesk-server](https://github.com/rustdesk/rustdesk-server).

## NixOS Configuration
NixOS provides the `rustdesk-flutter`, `rustdesk` and `rustdesk-server` packages.

### NixOS Client
The RustDesk Client is currently based on [Sciter](https://sciter.com/), an Electron clone using 
HTML, CSS and JS. However they are in the process of migrating to Flutter for their front end which 
is much faster and more capable; hence the two different front end packages `rustdesk` Sciter based 
and `rustdesk-flutter` flutter based. The Sciter frontend is recommended at the moment.

**Installation**
```nix
environment.systemPackages = [
  pkgs.rustdesk
];
```

**Configuration**
The client can be configured programmatically using a configuration string during launch e.g. 
`rustdesk --config $CONFIG_STRING` and you can get the config string 

The client supports the following command line flags:
* Set permanent password: `rustdesk --password $rustdesk_pw &> /dev/null`
* `--config`
* `--option`

1. Click the three dots next to your `ID` and choose `ID/Relay Server`
2. Punch in your server's IP or DNS name as `ID Server`
3. Punch in the value from your `id_*.pub` server file as the `Key`
4. Leave the rest of the values blank

### NixOS Server
RustDesk provides the Signaling and Relay Server components as well for self-hosting. The server 
components are really only useful for external traffic negociation. If you want open up our RustDesk 
server components to the internet they will support help negociate a direct connection between peer 
instances or if hole punching is not possible then the relay server component will act as a middle 
may to relay between the two. However if you are just running locally on a LAN then the server 
component is not needed an just adds extra overhead. You can simply `Enable direct IP access` 
configure a permanent password and use it locally.

* `hbbs` the rendezvous/signaling server which listens on `21114` for http in Pro only and `21115`, 
  `21116`, `21118` for web sockets and UDP `21116`
* `hbbr` the relay server which listens on TCP `21117` and `21119`

Ports:
* TCP: 21114-21119
* UDP: 21116


## Arch Linux Configuration

### Arch Linux Local Relay Server
You can host your own Relay Server so that you don't need to have the mothership involved. This 
allows you to use direct connections and keep things local.

**Configuration**
* `-r <IP or DNS>:21117` for `hbbs` indicates the IP or DNS name for the `hbbr` service
* `-k _` for `hbbs` and `hbbr` prohibits users connecting to your relay without your public key

```
$ mkdir ~/.config/rustdesk
$ cd ~/.config/rustdesk
$ docker run --rm --name hbbs --net=host -v "$PWD/data:/root" rustdesk/rustdesk-server:latest hbbs -r 192.168.1.2 -k _
$ docker run --rm --name hbbr --net=host -v "$PWD/data:/root" rustdesk/rustdesk-server:latest hbbr -k _
```

Running either docker command above will generate the public/private key pair in `~/.config/rustdesk` with prefix `id_`
Examples:
```
-rw-r--r--  1 root   root     88 Apr 18 17:44 id_ed56518
-rw-r--r--  1 root   root     44 Apr 18 17:44 id_ed56518.pub
```

Use the value in the `id_*.pub` file as the `Key` value for the client below.

### Arch Linux Build the Client package
1. Build and install the `python-pynput` package
   ```
   $ yay -Ga python-pynput; cd python-pynput; makepkg -s
   $ sudo pacman -U python-pynput-1.7.6-2-any.pkg.tar.zst
   ```
2. Build and install the `rustdesk-bin` package
   ```
   $ yay -Ga rustdesk-bin; cd rustdesk-bin; makepkg -s
   $ sudo pacman -U rustdesk-bin-1.1.9-3-x86_64.pkg.tar.zst
   ```

