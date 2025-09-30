# x2go <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

X2Go is a Modified NX3 protocol that provides all the classic remoting desktop features. It is lauded 
as being secure, efficient and faster than VNC to access Linux desktop environments over the network. 
It compresses and tunnels sessions through SSH by default, ensuring encryption and low-latency 
performance, even on slow connections.

**Summary**
* Pros
  * Readily available in nixpkgs
* Cons
  * Seems to required a separate plugin and configuration for desktop sharing
* Result
  * The separte plugin is throwing my off
  * The whole paradigm seems to be from the original X days and dated

### Quick links
* [../](../README.md)
* [Overview](#overview)

## Overview
The benefits over VNC are:
* low bandwidth support
* audio pass through with Puleaudio support
* traffic is tunneled over SSH
* file sharing from client to server
* desktop sharing for remote support
* separate sessions for remote access
* native headless Linux server support

**References**
* [X2go vs VNC](https://theserverhost.com/blog/post/x2go-vs-vnc)

### NixOS
NixOS has both the `x2goserver` and the `x2goclient` readily available.

```nix
{
  # Enable X2Go server
  services.x2go.server.enable = true;

  # Install a lightweight desktop environment (e.g., XFCE)
  services.xserver.enable = true;
  services.xserver.desktopManager.xfce.enable = true;

  # Optional: Enable a display manager (for full desktop login)
  services.xserver.displayManager.lightdm.enable = true;

  # Allow SSH (X2Go uses SSH)
  services.openssh.enable = true;

  # (Optional) Set firewall rules to allow X2Go (via SSH, port 22)
  networking.firewall.allowedTCPPorts = [ 22 ];
}
```
