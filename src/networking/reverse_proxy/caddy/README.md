# Caddy <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Caddy is an extensible server platform that can be used as a web server or a reverse proxy server.

### Quick links
* [.. up dir](..)
* [Overview](#overview)
* [Configuration](#configuration)
  * [Local TLS Without a Public Domain](#local-tls-without-a-public-domain)
  * [Trusting the Local CA](#trusting-the-local-ca)

## Overview

### Caddy vs Nginx Proxy Manager
* Caddy auto-provisions with Let's Encrypt for free

## Configuration

### Local TLS Without a Public Domain
Browsers refuse to run Web Crypto (and other secure-context-gated APIs) over plain
`http://<lan-ip>:<port>` — only `https://` or `http://localhost` count as a secure context. Caddy's
`tls internal` directive solves this for LAN-only services with no public domain or ACME/port-80
exposure needed: Caddy stands up its own local CA on first run and self-signs certs from it for
whatever site block asks for `tls internal`.

`nixos-config`'s `services.raw.caddy` option wraps this as a small reverse-proxy layer: each entry in
`proxies` gets its own HTTPS port (`port + 1000` by default, e.g. a backend on `8222` becomes
reachable at `https://<host>:9222`) that terminates TLS and forwards to the backend's existing plain
HTTP port over loopback. The backend's original HTTP port is untouched — this is purely additive.

```nix
services.raw.caddy = {
  enable = true;
  proxies = [
    { name = "vaultwarden"; port = 8222; }        # -> https://<host>:9222
  ];
};
```

This is a stopgap for LAN-local HTTPS, not a substitute for a real public-facing reverse proxy (e.g.
[Pangolin](../../reverse_tunnel/pangolin/README.md)) — once something like that fronts a service from
the outside, point it at the plain HTTP port directly and this proxy entry can be dropped.

### Trusting the Local CA
The first connection from any client shows an untrusted-certificate warning, since the CA is unique
to that host and self-signed. To stop seeing it, pull the root cert off the box once and trust it on
each device:
```bash
sudo find /var/lib/caddy -name root.crt
```

