# Meeting
Linux remote meeting and screen sharing has been historically pretty pathetic; however with remote 
work becoming mainstream as a result of the Covid pandemic a number of Linux friendly solutions have 
popped up that work really well.

### Quick links
* [Zoom](#zoom)
* [Google Meet](#google-meet)

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
