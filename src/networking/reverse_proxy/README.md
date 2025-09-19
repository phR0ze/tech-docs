# Reverse Proxy <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />

Proxies are systems that forward taffic for a service. Whether the proxy is considered a ***Forward 
proxy*** a.k.a just a ***Proxy*** or a ***Reverse proxy*** depends on which way the requests are 
being made.

* ***Forward proxy*** accepts requests from systems going out to the internet.
  * protects the client's oneline identity
  * potentially bypass browsing restrictions
  * blocks access to certain content

* ***Reverse proxy*** accepts requests from the internet going into your internal systems
  * protects a web site, hiding the web site's IP address
  * load balancing across a pool of servers 
  * caching for static content
  * handles SSL encryption for the server

### Quick links
* [.. up dir](../README.md)
* [Overview](#overview)
  * [Traefik vs Nginx Proxy Manager vs Caddy](#traefik-vs-nginx-proxy-manager-vs-caddy)

### Linked pages
* [Caddy](caddy/README.md)
* [Traefik](traefik/README.md)
* [Proxy Manager](proxy_manager/README.md)

## Overview

### Traefik vs Nginx Proxy Manager vs Caddy
Proxy Manager is a simple reverse proxy manager using Nginx with a simple and intuitive web 
interface. However it has some issues with automatic certificate renewal. With less features and some 
known issues in Proxy Manager the community is switching to Traefik. Traefik is more capable and 
featureful and being used in enterprises as well. It is super flexible.

* Caddy
  * pros
    * Written in Golang
    * Simpler to configure than others
    * Let's Encrypt automation for automatic cert renewal
  * cons
    * No visualization UI
* Traefik
  * pros
    * Has a web UI for showing how routes are handled
    * More modern than others
    * Let's Encrypt automation for automatic cert renewal
    * Cloud native enterprise grade solution
      * Intgrates with Prometheus, Grafana, and logs, etc...
    * Dyamic service discovery
    * Better support for protocols beyond just HTTP and HTTPS
    * Can centrally manage multiple environments
  * cons
    * Overly complicated to configure
    * Based on docker labels which is fragile
    * Community support is poor
    * Documentation is fragmented
* Nginx Proxy Manager
  * pros
    * Extremly powerful
    * Let's Encrypt automation
  * cons
    * Difficult to setup
    * Can't handle more complex enviroments
    * Not able to handle dynamic container env well
