# VLANs <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Virtual Local Area Networks (VLANs) are a form of overlay network that can be configured and deployed 
locally to segregate traffic on your LAN into separate and distinct Virual LANs.

### Quick links
* [VLAN Overview](#vlan-overview)

## VLAN Overview
A single LAN works well for beginners. However there can be some essential benefits for using VLANs 
on top of your LAN such as:

* Reduce lan traffic to only the specific hardware needed rather than all devices
* Restrict access to sensitive networking devices
  * e.g. guest network that only has access to the internet and not other devices
  * e.g. IoT network to quarantine publically connected devices from your personal devices

**References**
* [VLANs - NixOS wiki](https://nixos.wiki/wiki/Networking#VLANs)
* [VLANs - NixOS manual](https://nixos.org/manual/nixos/stable/options.html#opt-networking.vlans)
