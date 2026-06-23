# Pangolin <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Pangolin is the closest approximation to Cloudflare Tunnels without the limitations. It has a
polishsed dashboard, auto SSL, and access control built in. Being able to hand out a custom domain
name instead of random IPs makes it easier for less technical family members - they just go to the
URL, log in with Google SSO, and use the service. This is the strongest self-hosted option.

### Quick links
- [.. up dir](../README.md)
- [Overview](#overview)

## Overview
Pangolin uses Traefik as its reverse proxy and Gerbil for WireGuard tunnel management. It is setup on
a VPS that has a publically reachable IP that accepts traffic on ports 80/443 and securely tunnels it
to your server. Your homelab just runs a lightweight `newt` client that makes an outbound connection
to the VPS. From that point on the VPS bridges internet users to your homelab without your home IP
ever being exposed. In this way the VPS is essentially playing the same role as Cloudflare's edge
network tunnel, but you own and control it. Most homelab's use a cheap $4-6/month instance from
Hetzner or another Cloud service.
