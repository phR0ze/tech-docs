# Cloudflare DNS <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Cloudflare is not a replacement for web hosting. Cloudflare is used as your name server and provides 
DNS lookups for your domain i.e. `vanity name` => `Cloudflare DNS` => `homelab web server`. Because 
of Cloudflare's giant network they will likely have a server close to your potential client and will 
speed things up. Cloudflare is free to use for personal or hobby projects for basic DNS, Global CDN, 
Unmetered DDoS Protection, Universal SSL Certificate, Free Managed Ruleset and Simple bot 
mitigation.

### Quick links
* [.. up dir](../README.md)
* [Overview](#overview)
  * [Domain Name Registrar](#domain-name-registrar)

## Overview

**References**
* [Craylor overview](https://www.youtube.com/watch?v=IXpvUD5SDzA)
* [Craylor setup guide](https://www.youtube.com/watch?v=f1EoQygkJ0E)

### Domain Name Registrar
The first thing you need is to purchase a domain name to then use with Cloudflare DNS. There are 
numerous registrars but Cloudflare is reliable, keeps prices steady and integrates well with the 
rest of their system.

### DDNS with Cloudflare
Since most home networks are fronted by an ISP that is reusing dynamic IPs your IP address will 
likely change over time. To avoid getting locked out from remote access you'll need to have a client 
running on your home network that connects out to Cloudflare to update Cloudflare with the latest IP 
address.

### Cloudflare vs Github free pages
Both are essentially only for static web pages which leaves you with many limitations.

## How to setup DNS
1. Navigate to the `DNS` entry on the left


## Cloudflare and Tailscale Funnel
Tailscale Funnel publishes a specific service to the public internet that is still running over the 
Tailscale infrastructure and thus removes the need for a firewall hole to be made for your web 
server. Thus you can have both:

1. Private home network access
  * Access your home services via foobar.example.com
  * No open ports on your router
  * End-to-end encryption
  * Access control (login required)
  * Works from anywhere
2. Public web server access
  * Tailscale provides
    * No open ports on your router
    * Works from anywhere
  * Cloudflare provides
    * Domain name
    * DDoS protection
    * WAF
    * Rate limiting
    * Access rules


Public users
   ↓
Cloudflare (DNS / WAF)
  foobar.example.com → <machine>.<tailnet>.ts.net
   ↓
Tailscale Funnel
   ↓
Your home service



### Web service
* Keep the service isolated in a container or VM
* Only expose HTTPS
