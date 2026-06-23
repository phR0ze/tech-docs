# Reverse Tunnel <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />


### Quick links
- [.. up dir](../README.md)
- [Overview](#overview)
  - [Pangolin vs Cloudflare Tunnel](#pangoline-vs-cloudflare-tunnel)

### Linked pages
* [Cloudflare Tunnel](cloudflare_tunnel/README.md)
* [Pangolin](pangolin/README.md)

## Overview

### Why Reverse Tunnel
There are a number of ways to expose services to the public internet; three of which I've dug into as
potential solutions to the way I'd like to expose my services.

***Reverse Proxy*** such as Caddy, Traefik, or Nginx:
* require your server to have a publically reachable IP address via DDNS-managed public IP
* require 80/443 open on router exposing your home network to attacks

***Overlay Mesh*** such as Tailscale, Twingate or WireGuard
* require a special app to run in the background on all devices
* mobile clients often get killed by battery management
* free tier is limited to 6 at most users

***Reverse Tunnel*** such as Pangolin, Cloudflare Tunnel, Rathole
* no public IP needed at home
* no open inbound ports on home router
* users need nothing installed
* requires a VPS through which all traffic flows
* slightly higher latency

Because I require the end user to not need to install anything that eliminates the Overlay Mesh
option. Ideally for network security the Reverse Proxy option is out. This leaves self-hosted
versions of Reverse Tunnel as my target.


**Honerable mentions**
* frp (Fast Reverse Proxy) - one of the most feature rich self-hosted options
* Rathole - high performance, written in Rust, low resouce consumption, high security via Noise
Protocol, and no third-party bandwidth caps. More technical to setup than Pangolin

### Pangolin vs Cloudflare Tunnel
Although Cloudflare Tunnel is turn key for the most part and a great solution, it limits file uploads
to `100mb` and its ToS blocks streaming services like Jellyfin which is a non-starter for me.
