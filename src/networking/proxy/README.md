# Proxy <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />

### Quick links
* [Proxy Requirements](#proxy-requirements)
  * [Ad blocking](#ad-blocking)
  * [Content blocking](#content-blocking)
* [App research](#app-research)
  * [AdGuard Home](adguard_home/README.md)
  * [Privoxy](privoxy/README.md)
  * [Squid](squid/README.md)
  * [Deprecated or Eliminated](#deprecated-or-eliminated)

## Proxy Requirements
Comparing various proxy and ad-blocking application features for use in my home network. I'm 
excluding any solution that doesn't address the full network transparently i.e. browser based 
solutions, socks or DNS based solutions that require individual device configuration. Additionally 
I'm excluding paid services and external services that are not self-hosted in my network. Mostly that 
leaves network level proxies like Privoxy, Squid, Privaxy or others in a similar category.

**References**
* [Enhancing privacy with Squid and Privoxy](https://www.christianschenk.org/blog/enhancing-your-privacy-using-squid-and-privoxy/)
* [DansGuardian and TinyProxy](https://www.instructables.com/Set-up-web-content-filtering-in-4-steps-with-Ubunt/)
* [AdGuard on Android Review](https://adguard.com/en/blog/adguard-vs-adaway-dns66.html)

### Ad blocking
**Ad blocking requirements**
* Block ad network requests to save bandwidth
* Intelligently cleanup rendered page such that the absence of the ad is not noticed

**DNS and/or Hosts based approaches** such as ***Pi-Hole*** or ***AdGuard Home*** and the like are unable 
to provide a clean solution on their own as they leave cosmetic holes in your rendered web page where 
the ad should have been thus making it difficult to consume.

**Possible** 
* AdGuard Home might be a possibility but does leave cosmetic holes in rendered pages

**Eliminated** 
* DNS or Hosts based apps
  * ***Pi-Hole*** as AdGuard Home claims to do it better

### Content blocking
**Content blocking reqquirements**
* Block sites by category
* Block sites manually

**Possible** 
* AdGuard Home claims to have parental controls and per site settings


## App research

* [Technitium DNS Server](https://technitium.com/dns/)
  * Block ads and malware at the DNS level
* [Blocky](https://0xerr0r.github.io/blocky/latest/)
  * Golang ad blocker
  * Client group settings
  * External allow denylist reloads

### Reverse Proxy Servers
Self-hosted reverse proxy servers sit between your local network and the internet acting as the 
perfect layer to handle filter, blocking, tracking, caching and more.

* ***Privoxy proxy*** is a non-caching web proxy built to enhance privacy. It boasts enhanced filtering 
  capabilities for modifying HTTP headers and web page data, controlling access, and removing ads. You 
  can then setup Firefox and other browsers to use the proxy. It is available for `Llnux and DD-WRT`.

* ***Squid Proxy*** supports HTTP, HTTPS, FTP, and more. It features a reverse proxy which serves as a 
  web cache daemon that caches incoming requests for outgoing data. It features several traffic 
  optimization options, access control, authorization and logging facilities.

* ***Traefik Proxy*** is a modern, fast HTTP reverse proxy and load balancer written in Golang. It can 
  be automatically and dynamically configured by any user and does not require extensive knowledge of 
  networking of proxy servers. Supports access logs and metrics.

* ***Tinyproxy*** is a lightweight open-source HTTP/HTTPS proxy daemon designed to be fast and small. 
  Supports URL-based filtering, access control using subnets and IP addresses, transparent proxying and 
  an extensive privacy feature. Its privacy features allow you to restrict the data from an HTTP server 
  to your web browser.

* [Privaxy](https://github.com/Barre/privaxy)
  * Rust based
  * Support for adblock plus filters like easylist
  * Web GUI for statistics
  * Support for uBlock origin's `js`, `redirect`, `scriptlets`
  * Support for custom filters
  * Support for excluding sites from MITM
  * Automatic filter lists updates
* [Brave adblock](https://github.com/brave/adblock-rust/)
  * Rust based
  * Network blocking
  * Cosmetic filtering
  * uBlock Origin syntax extensions
* [Cloudflare's Oxy](https://blog.cloudflare.com/introducing-oxy/)
  * ***Proprietary***
  * Rust based
  * Protocol decapsulation
  * Traffic analysis
  * Routing
  * Tunneling logic
  * DNS resolution
* [Cloudflare's Pingora](https://github.com/cloudflare/pingora)
  * Rust based
  * HTTP 1/2 end to end proxy
  * TLS over OpenSSL, BordingSSL or rustls(experimental)
  * gRPC and websocket proxying
  * Support for a variety of observability tools
* [rathole](https://github.com/rapiz1/rathole)
  * Rust based
  * A secure, stable and high performance reverse proxy for NAT traversal

### Web Filtering
* ***e2guardian*** is an open source web content filtring solution featuring
  * Features
    * supports multiple filter profiles
    * whitelist and black list urls
    * deny regular expressions on urls, body content, and headers
    * deep url scanning to spot URLS and block images in Google images
    * advanced advert blocking
    * time based blocking as well
    * https://github.com/e2guardian/e2guardian
    * Fork of DansGuardian

***NxFilter*** is web filtering software for controlling user activity on the Internet. You can 
monitor Internet usage in your network and block user request for websites.

* Adguard Home
* WebCleaner

* Squidguard

### Deprecated or Eliminated
* ***DansGuardian*** is an award winning, open source web content filter which runs on Linx and 
  others. It filters the actual content of pages based on many methods including phrase matching, PICS 
  filter and URL filtering.
  * Reasoning
    * no longer maintained

* ***HAProxy*** is primarily a load balancer but functions as a reverse proxy for TCP and HTTP. 
  Supports caching and protects agaisnt DDoS attacks.
  * Reasoning
    * load balancer with little other functionality

* ***Pi-hole*** is a DNS sickhole that protects your devices from unwanted content, without installing 
  any client side software.
  * Reasoning
    * Seems like a decent option but AdGuard Home covers the same space and more
  * Features
    * blocks content network wide
    * speeds up daily browsing with DNS caching
    * runs smoothly on minimal hardware
    * command line interface is simple and robust
    * responsive web interface dashboard
    * optionally functions as a DHCP server ensuring all devices are protected automatically
    * blocks ads over both IPv4 and IPv6
    * open source

* ***Polipo*** is a HTTP caching proxy
  * Reasoning
    * no longer maintained

* [Shadowsocks Rust](https://github.com/shadowsocks/shadowsocks-rust)
  * Reasoning
    * focused on tunneling
