# Pangolin <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Pangolin is the closest approximation to Cloudflare Tunnels without the limitations. It has a
polished dashboard, auto SSL, and access control built in. Being able to hand out a custom domain
name instead of random IPs makes it easier for less technical family members - they just go to the
URL, log in with Google SSO, and use the service. This is the strongest self-hosted option.

Cloudflare Tunnel isn't an option for me because of the `100mb` upload limit and the ToS that blocks
the use of Jellyfin.

### Quick links
- [.. up dir](../README.md)
- [Overview](#overview)
  - [Security Advantage](#security-advantage)
  - [Concepts](#concepts)
- [Configure VPS](#configure-vps)
  - [Recommended Resources](#recommended-resources)
  - [Configure VPS](#configure-vps)
- [Configure Domain Name](#configure-domain-name)
  - [Purchase Domain Name](#purchase-domain-name)
  - [Configure DNS](#configure-dns)
- [Deploy Pangolin](#deploy-pangolin)
- [Configure Pangolin](#configure-pangolin)
  - [Add Site](#add-site)
  - [Install Newt](#install-newt)
  - [Create Resource](#create-resource)

## Overview
Pangolin uses `Traefik` as its reverse proxy and `Gerbil` for `WireGuard tunnel management`. It is
setup on a VPS that has a publically reachable IP that accepts traffic on ports 80/443 and securely
tunnels it to your server. Your homelab just runs a lightweight `newt` client that makes an outbound
connection to the VPS. From that point on the VPS bridges internet users to your homelab without your
home IP ever being exposed. In this way the VPS is essentially playing the same role as Cloudflare's
edge network tunnel, but you own and control it. Most homelab's use a cheap $4-6/month instance from
Hetzner or another Cloud service.

* [Pangolin - DB Tech](https://www.youtube.com/watch?v=a-a-Xk1hXBQ)

### Security Advantage
Pangolin has a security advantage over reverse proxies. 

**Pros**
* The VPS is the only public-facing endpoint
* No inbound ports are open on your home router
* Access control lives at the edge
* You own the relay, not TLS cracking
* WireGuard between VPS and homelab
* Blast radious containment

**Cons**
* The VPS is now a target you must maintain
* You are the security team
* VPS provider is a trust boundary
* Single point of failure for auth
* Misconfiguration risk
* No built-in WAF or bot protection
  - fail2ban, crowdsec, etc...

### Concepts

#### Sites
Sites in Pangolin are tunnels. You'll need a site per subnet you want to expose. The Wireguard tunnel
is established using `newt` running on your homelab host. 

#### Resources
Resources in Pangolin are the services you'd like expose over the tunnel (a.k.a. site).

## Configure VPS
The first thing we need is a Cloud VPS to host Pangolin on. After some cursory research I landed on
RackNerd as a budget option.

### Recommended Resources
Pangolin itself is lightweight — the VPS does no transcoding or storage. The binding constraint for
media-heavy workloads is monthly transfer, not compute.

**Minimum VPS specs (per official docs)**
- vCPU: 1
- RAM: 1.5 GB
- Storage: 8 GB SSD

**Recommended VPS specs**
- vCPU: 2
- RAM: 2 GB
- Storage: 20 GB SSD
- Transfer: sized to your workload (see [Cloud Budget Comparison](../../../../cloud/README.md#budget-comparison))

### Configure VPS
For a homelab serving Jellyfin (3× 1080p movies/day) and Immich photo/video browsing, estimated
transfer is ~0.85 TB/month (~1.7 TB with 2× headroom). The
[RackNerd 2 GB KVM](../../../../cloud/racknerd/README.md#pangolin-vps) at $35.99/yr is the
cheapest option that meets all requirements.

## Configure Domain Name

### Purchase Domain Name
A domain name is required to route public traffic to the VPS. [Cloudflare](../../../dns/cloudflare_dns/README.md)
is the recommended registrar — it keeps prices steady, has no surprise renewal markups, and the
domain integrates directly with Cloudflare DNS which handles the rest of the setup.

### Configure DNS
Pangolin needs a domain name pointing at the VPS's static public IPv4 address. See
[Configure DNS for your Domain Name](../../../dns/cloudflare_dns/README.md#configure-dns-for-your-domain-name)
for the full setup — Cloudflare is the recommended registrar and DNS provider as it integrates
cleanly with Pangolin's auto-SSL and is free for personal use.

## Deploy Pangolin

* [Thomas Wilde - Pangolin guide](https://www.youtube.com/watch?v=ISEP6SIrEVE)


## Google as OAuth2 provider
* [Setup GCP OAuth2](https://youtu.be/Bu8WFh1ns4c?t=655)


## Configure Pangolin

### Add Site
Create a new site for your homelab subnet

1. Set the `Name` e.g. `redfish`
2. Set the `Method` to `Newt`
3. Copy the newt configuration line 
4. Check `I have copied the config`
5. Click `Create Site`

Note: the new site will be offline until you install the `Newt` agent on your homelab host using the
supplied configuration.

### Install Newt
In order for your site to be active you need to complete the tunnel configuration with `Newt`

1. Browse to the [Pangoline newt instructions](https://docs.pangolin.net/manage/sites/install-site#docker-compose)
2. Grab the docker compose configuration
```yaml
services:
  newt:
    image: fosrl/newt
    container_name: newt
    restart: unless-stopped
    environment:
      - PANGOLIN_ENDPOINT=https://app.pangolin.net
      - NEWT_ID=2ix2t8xk22ubpfy
      - NEWT_SECRET=nnisrfsdfc7prqsp9ewo1dvtvci50j5uiqotez00dgap0ii2
```
3. Update the environment variables with the configuration copied in the [Add Site](#add-site) section
   1. Set `PANGOLIN_ENDPOINT`
   2. Set `NEWT_ID`
   3. Set `NEWT_SECRET`

4. Once newt is up and running you should see your site go `active`

### Create Resource
Creating a resource is essentially configuring your homelab application to be accessible over the
tunnel that the site configuration and newt created.

1. Set the `Name` e.g. `Jellyfin`
2. Set the `Subdomain` e.g. `jellyfin.example.com`
3. Select the correct site e.g. `redfish`
4. Click `Create Resource`
5. Flip the toggle to `Enable SSL (https)`
6. Enter the IP address e.g. `192.168.0.3` of Jellyfin
7. Enter the Port e.g. `8096` of Jellyfin
8. Click `Add Target` then `Save Target`

