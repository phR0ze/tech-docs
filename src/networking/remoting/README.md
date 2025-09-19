# Remoting <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />

Full remote desktop solutions in the Linux world have been rather poor in my opinion. Most early 
solutions used `VNC`, `XRDP` or remote `X11`. None of these technologies though were very good. 
Yes it is possible to carefully tune them to provide an acceptable solution; but out of the box usage 
for non-experts was painful at best resulting in tearing, choppy, kludgy solutions that we're 
difficult to setup and maintain. There were a few paid options (`Zoho Assist`, `Remote Access Plus`, 
and others) that had better results. TeamViewer, despite its reputation, isn't that bad and I've used 
it for years in personal solutions. That said it would frequently break depending on the kernel 
version your on.

**Requirements**
- Free for personal use, preferably FOSS
- Low latency andio and gpu accelerated video
- Linux server metal and virtual machine support

### Quick links
* [../](../README.md)
* [Overview](#overview)
  * [Browser based](#browser-based)
  * [Server Comparison](#server-comparison)
  * [Client Comparison](#client-comparison)
* [Lightdm remote login](#lightdm-remote-login)
* [Guacamole](#guacamole)
* [Teamviewer](#teamviewer)
* [x11spice](#x11spice)
* [Clients](#clients)
  * [Remmina](#remmina)
* [Local Device Sharing](#local-device-sharing)
  * [Barrier](#barrier)

### Linked pages
- [Guacamole](guacamole/README.md)
- [Kasm](kasm/README.md)
* [RustDesk](rustdesk/README.md)
* [Selkies](selkies/README.md)
 
## Overview
**References**
* [NixOS Remoting](https://nixos.wiki/wiki/Remote_Desktop)
* [Comparison - Wikipedia](https://en.wikipedia.org/wiki/Comparison_of_remote_desktop_software)

### Browser based
There is a subset of the remoting software space that deals with setting up a server that will serve 
up an HTML5 experience of the desktop into a browser.

* Selkies
  * Modern, 60fps at 1080p hardware accelerated
* Guacamole
  * Older Apache solution Works for every OS
* Kasm VNC
  * Has their own flavor of VNC that works really well
  * Works for every OS

### Server Comparison

| Name          |
| ------------- | ----------------------------
| NX/NoMachine  |  

### Client Comparison

| Name              | FOSS | UI      | Platform | Other 
| ----------------- | ---- | ------- | -------- | --------------------
| GNOME Connctions  | yes  | GTK     | Linux    | RDP, VNC
| Remmina           | yes  | GTK     | Linux    | RDP, VNC, SPICE, X2Go, SSH, HTTP(S)
| RustDesk          | yes  | Flutter | Cross    | P2P, end-to-end encryption
| X2Go              | yes  | ?       | ?        | X2Go(NX3)
| Cendio ThinLinc   |
AnyDesk
Ericom Connect
NetSupport Manager
NX
RealVNC
ConnectWise Control
Teamviewer
xpra
x11vnc
Citrix XenApp
QVD
Veyon
XRDP
TightVNC
TigerVNC
TurboVNC
Sunshine
Remotely

Zoho Assist

## Lightdm remote login
Some VNC servers like `x11vnc` and TigerVNC's `x0vncserver` make the primary X session available for 
remote connections. Lightdm provides the ability to using XDMCP, VNC for incoming remote logins.

**References**
* [Arch linux docs - lightdm](https://wiki.archlinux.org/title/LightDM)
* [Arch linux docs - x11vnc](https://wiki.archlinux.org/title/X11vnc)
* [Super user post](https://superuser.com/questions/1718646/xvnc-and-lightdm-connect-to-primary-display-0-and-login-remotely)
* [Super user x11vnc](https://superuser.com/questions/1375625/vncviewer-connect-to-primary-display-of-lightdm-on-debian-jessie-share-with-phy)

x11vnc doesn't require any special configuration from lightdm. however you can have lightdm start the 
vnc server using its configuration for Tiger VNC

It depends on `tigervnc` which needs to be installed on the server side.

1. Setup an authentication password
   ```
   $ vncpasswd /etc/vncpasswd
   ```

2. Edit `/etc/lightdm/lightdm.conf` configs to below.
   ```
   [VNCServer]
   enabled=true
   command=Xvnc -rfbauth /etc/vncpasswd
   port=5900
   listen-address=localhost
   width=1024
   height=768
   depth=24
   ```

### x11vnc
x11vnc doesn't require any special configuration from lightdm

### TigerVNC x0vncserver
* [TigerVNC x0vncserver docs](https://tigervnc.org/doc/x0vncserver.html)

## NX
NX or NoMachine is proprietary but free-of-charge for non-commercial use. Created in 2003, NX 
compresses the X display protocol to improve the preformance to be used over slow connections. It 
also reduces round trips to improve the speed.

**Spinoffs**
* Freenx
* Google's Neatx

## X2Go
X2Go is a Modified NX 3 protocol that provides all the classic remote features.

### X2Go features
* low bandwidth support
* audio pass through with Puleaudio support
* traffic is tunneled over SSH
* file sharing from client to server
* desktop sharing for remote support
* separate sessions for remote access

### Install X2Go on NixOS
1. Install the server by simply including
   ```nix
   services.x2goserver.enable = true;
   ```

## xdmcp
Is similar to telnet, using unencrypted authentication.

## Teamviewer
Teamviewer is a free for personal use remote desktop solution that in my opinion provided one of the 
first decent out of box experiences out there.

Typically I configure TV to only be accessible from my LAN and just use it locally or tunnel in over 
SSH if I'm remote.

1. Install Teamviewer
  ```bash
  sudo pacman -S teamviewer
  sudo systemctl enable teamviewerd
  sudo systemctl start teamviewerd
  ```
2. Autostart Teamviewer
  ```bash
  cp /usr/share/applications/teamviewer.desktop ~/.config/autostart
  ```
3. Configure Teamviewer  
  a. Start teamviewer: `teamviewer`  
  b. Click ***Accept License Agreement***  
  c. Navigate to ***Extras >Options***  
  d. Click ***General*** tab on left  
  e. Set ***Your Display Name***  
  f. Check the box ***Start Teamviwer with system***  
  g. Set drop down ***Incoming LAN connections*** to ***accept exclusively***  
  h. Click ***Security*** tab   
  i. Set ***Password*** and ***Confirm Password***  
  j. Leave ***Start Teamviewer with Windows*** checked and click ***OK*** then ***OK***  
  k. Click ***Advanced*** tab on left  
  l. Disable log files  
  m. Check ***Disable TeamViewer shutdown***  
  n. Click ***OK***  

## x11spice
`x11spice` is patterned after `x11vnc` and allows for connecting to a running X server via a Spice 
server. Configured via `~/.config/x11spice/x11spice.conf` or `/etc/xdg/x11spice/x11spice.conf` with 
`xdg/x11spice//x11spice.conf` being an example of all options.

## Clients

### Remmina
* simple GTK app
* easy to use
* supports RDP, VNC, SPICE, X2GO, SSH and HTTP(S)

## Local Device Sharing

### Barrier
Barrier allows you to share a keyboard and mouse between local machines (e.g. desktop and laptop). It 
is only useful if you have all machines your sharing at the same local workstation. It is a fork of 
the `Synergy 1.9` codebase and the go forward open source solution.

#### Install Barrier
```bash
$ sudo pacman -S barrier
```

#### Barrier Server
Note: you can watch the logs with `journalctl --user -u barriers -f` or use
`barriers -f -c /etc/barrier.conf --disable-crypto` for foreground debugging

1. From your workstation launch ***Barrier*** from the ***Network*** menu
2. Work through the wizard
3. Select ***Server (share this computer's mouse and keyboard)*** and click ***Finish***
4. Select ***Configure interactively*** and then click ***Configure Server...***
5. Drag a new monitor from top right down to be to the right of ***desktop***
6. Double click the new monitor and name it ***laptop*** and click ***OK***
   Note: the name used here must match the 'Client name' used in the Client section  
7. Navigate to ***File >Save configuration as...*** and save ***barrier.conf*** in your home dir  
8. Now move it to etc: `sudo mv ~/barrier.conf /etc`
9. Enable barriers: `systemctl --user enable barriers`  
10. Start barriers: `systemctl --user start barriers`  

#### Barrier Client
1. Launch: `barrierc --disble-crypto IP-ADDRESS-SERVER`
2. Click ***Next*** to accept ***English*** as the default language
3. Select ***Client (use another computer's mouse and keyboard)*** then ***Finish***
4. Uncheck ***Auto config***
5. Enter server hostname e.g. ***192.168.1.4***
6. Click ***Start***
7. Navigate to ***Edit >Settings*** and check ***Hide on startup*** then ***OK***
8. Click ***File >Save configuration as...*** and save as ***~/.config/barrier.conf***
9. Create autostart for client: `cp /usr/share/applications/barrier.desktop ~/.config/autostart`

## Meeting Screen Sharing
Linux remote meeting and screen sharing has been historically pretty pathetic; however with remote 
work becoming mainstream as a result of the Covid pandemic a number of Linux friendly solutions have 
popped up that work really well.

### Zoom
Zoom is one of the most popular Linux friendly remote meeting apps out there. Its pretty good 
quality and all I did was simply install it and selected my plantronics headset and audio worked 
great.  My laptop webcam also worked without doing anything.

**Install Manually**
```bash
$ yaourt -G zoom; cd zoom
$ makepkg -s
$ sudo pacman -U zoom-2.4.121350.0816-1-x86_64.pkg.tar.xz
```

**Install from cyberlinux-repo**
```bash
$ sudo tee -a /etc/pacman.conf <<EOL
[cyberlinux]
SigLevel = Optional TrustAll
Server = https://phR0ze.github.io/cyberlinux-repo/$repo/$arch
EOL
$ sudo pacman -Sy zoom
```

### Google Meet
Google meet, part of the Google Suite, and available as a web app is another great option. It worked 
out of the box with Chromium and my plantronics headset and webcam without any issues.

<!-- 
vim: ts=2:sw=2:sts=2
-->
