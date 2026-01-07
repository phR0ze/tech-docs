# Tailscale <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

***Tailscale*** and its competitors e.g. ***Twingate***, ***netbird***, etc... provide an overlay 
network that allow you to connect to your target network resources anywhere in the world as if you 
were on the same network.

### Quick links
* [.. up dir](../README.md)
* [Overview](#overview)

## Overview

* Pros
  * Zero configuration
  * Built on WireGuard
  * Bypasses firewalls
* Cons
  * Not fully opensource


### Tailscale vs netbird
Netbird is another mesh network solution that provides more customization and configuration than 
Tailscale.
* Tailscale is easier to configure
* Both use Wireguard
* netbird scales at enterprise

### Tailscale Serve
Serve gives you the ability to serve out files or service to anyone on your tailnet locally.

### Tailscale Funnel
Funnel gives you the ability to serve out files or service to anyone publicly

### Exit Node
A node in your tailnet can be specified as an exit node. This means that any device in your tailnet 
can then choose to route all there outgoing traffic through that node to behave as if you were 
executing requests from that machine. Potentially useful for obfuscating your location or working 
around geolocation restrictions. Most useful though is probably the ability to then get access to the 
rest of the devices in your home network through an exit node in your home network.

## Tailnet

