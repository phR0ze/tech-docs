# Ad blocking <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

### Quick links
* [Overview](#overview)

## Overview

### Typical Goals
* Reduce Ads
  * Blocks objectionable content
  * Increases privacy by reducing tracking
  * Increases network performance by reducing ad overhead resource consumption

### Types of Ad Blockers
There are at least four different types of ad blockers.

1. ***Browser extensions*** are add-ons to your browser that analyze and filter out content as it 
   loads in the browser, preventing ads from displaying.

2. ***Standalone applications*** can block ads across various other apps and browsers

3. ***DNS-based blockers*** alter the DNS resolution to reroute requests for known ad servers to 
   prevent ads from being downloaded and displayed.

4. ***Network-level blockers*** are the most robust and are installed on the router protecting all 
   devices connected to the network.

### Self-Hosted Reverse Proxy Servers
Self-hosted reverse proxy servers sit between your local network and the internet acting as the 
perfect layer to handle filter, blocking, tracking, caching and more.

***Privoxy proxy*** is a non-caching web proxy built to enhance privacy. It boasts enhanced filtering 
capabilities for modifying HTTP headers and web page data, controlling access, and removing ads. You 
can then setup Firefox and other browsers to use the proxy. It is available for `Llnux and DD-WRT`.

***Squid Proxy*** supports HTTP, HTTPS, FTP, and more. It features a reverse proxy which serves as a 
web cache daemon that caches incoming requests for outgoing data. It features several traffic 
optimization options, access control, authorization and logging facilities.

***Traefik Proxy*** is a modern, fast HTTP reverse proxy and load balancer written in Golang. It can 
be automatically and dynamically configured by any user and does not require extensive knowledge of 
networking of proxy servers. Supports access logs and metrics.

***Tinyproxy*** is a lightweight open-source HTTP/HTTPS proxy daemon designed to be fast and small. 
Supports URL-based filtering, access control using subnets and IP addresses, transparent proxying and 
an extensive privacy feature. Its privacy features allow you to restrict the data from an HTTP server 
to your web browser.

***HAProxy*** is primarily a load balancer but functions as a reverse proxy for TCP and HTTP. 
Supports caching and protects agaisnt DDoS attacks.

***Pi-hole*** is a DNS sickhole that protects your devices from unwanted content, without installing 
any client side software.
* blocks content network wide
* speeds up daily browsing with DNS caching
* runs smoothly on minimal hardware
* command line interface is simple and robust
* responsive web interface dashboard
* optionally functions as a DHCP server ensuring all devices are protected automatically
* blocks ads over both IPv4 and IPv6
* open source

***Polipo*** was a HTTP caching proxy, but is no longer maintained
