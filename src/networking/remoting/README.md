# Remoting
Full remote desktop solutions in the Linux world have been rather poor in my opinion. Most early 
solutions used `VNC`, `XRDP` or remote `X11`. None of these technologies though were very good. 
Yes it was possible to carefully tune them to provide a decent solution; but out of the box usage for 
non-experts was painful at best resulting in tearing, choppy, kludgy solutions that we're difficult 
to setup, maintain and still had poor results. There were a few paid options (`Zoho Assist`,
`Remote Access Plus`, and others) that had better results.

### Quick links
* [RustDesk](#rustdesk)
* [Teamviewer](#teamviewer)
* [Local Device Sharing](#local-device-sharing)
  * [Barrier](#barrier)

## x11vnc

## x2go

## xdmcp
Is similar to telnet, using unencrypted authentication.

## xpra

## xrdp

## TigerVNC

## RustDesk
Open source project written in Rust providing both a client and server. The project is cross platform 
and available in AUR. Its meant to be a TeamViewer alternative and allows for remote service help 
like TeamViewer using an ID and RustDesk servers to connect in to assist your relatives or whatever. 
However you can also host the server and keep everything tightly controlled for a local solution as 
well.

**Resources**
* [rustdesk-server](https://github.com/rustdesk/rustdesk-server).

**Features**
* Cross-platform support, MacOS, Windows, Linux and Android
* Linux is X11 support only

#### Configure Local Relay Server
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

#### Build the Client package
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

#### Configure Client ot use local Relay Server
Client options allow for Relay Server configuration
1. Click the three dots next to your `ID` and choose `ID/Relay Server`
2. Punch in your server's IP or DNS name as `ID Server`
3. Punch in the value from your `id_*.pub` server file as the `Key`
4. Leave the rest of the values blank

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
