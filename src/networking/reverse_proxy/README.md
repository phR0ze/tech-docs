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
* [Traefik](traefik/README.md)
* [Proxy Manager](proxy_manager/README.md)

### Traefik vs Proxy Manager
Proxy Manager is a simple reverse proxy manaber using Nginx with a simple and intuitive web 
interface. However it has some issues with automatic certificate renewal. With less features and some 
known issues in Proxy Manager the community is switching to Traefik. Traefik is more capable and 
featureful and being used in enterprises as well. It is super flexible.

