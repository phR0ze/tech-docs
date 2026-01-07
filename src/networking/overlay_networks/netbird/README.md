# Netbird <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Open source Wireguard based overlay VPN that can be self-hosted. Essentially it allows you to bypass 
firewalls and connect to a remote peer over an enrypted tunnel and optionally to use the remote peer 
as an exit node to make networking requests on its behalf. Netbird is based in Germany which has 
strong privacy laws and with the self-hosting option this is quite an attractive option.

### Quick links
* [.. up dir](../README.md)
* [Overview](#overview)
  * [Features](#features)
  * [Mesh network](#mesh-network)

## Overview
Open source zero trust networking platform that allows you to create secure private networks that can 
be accessible from anywhere in the world without opening any ports or complex configuration.

**References**
* [netbird overview](https://www.youtube.com/watch?v=CFa7SY4Up9k)
* [Christian Lempa](https://www.youtube.com/watch?v=_Fgwap-sl3A)

### Fetures
* Self-hosting Peer-to-peer connections & full path encryption with WireGuard
  * uses netbird coordination server to establish point-to-point connections
* Supports relaying traffic to a different computer with exit nodes
  * thus one computer can act as an entry into your network
* Create private networks
* Simple and fast remoting from anywhere in the world
* SSO with Google Workspace, Azure, Okta
  * Paid??
* Access controls
* Private DNS
* Mesh network routing
* management activity logging

### Mesh network
Netbird is installed on every device in the network allowing for direct secure connections and full 
control between any devices in the mesh.

### Routing peers
Only install agents on particular devies (i.e. peers) to include them in the mesh and allow those 
devices to act as proxies to other devices in the networks they are attached to. Thus you don't have 
direct access to every device but indirect access via the proxy. 

Multiple routing peers can be used for failover if one goes down or you need to reboot one for 
updates or something you still have another link to keep things live until the first comes back 
online.

### Management Service
Acts as the central control plane for your netbird networks orchestrating all the peers and their 
connections. The management service can be self-hosted. It maintaines the overall state of the 
privatge networks including information about active peers, ip addresses and their assigned roles. 
Policy controls allow you to provide which devices have access to which other devices.

### Signal service
Primary purpose is to ochrestrate peer-to-peer connections by allowing each device to connect to the 
signal service as a rendezvous point where each device connects in to discover other devices on the 
private network and then get connection information to then make direct connections. It doesn't proxy 
any data just facilitates the direct wireguard connections.

### Relay service
This service is a fallback which allows for relaying traffic between two peers that can't connect 
directly.

### Cloud service
Free to use for up to 5 users and 100 machines. Just install netbird on all your machines then login 
to their web portal to manage them.

### Self-hosted
