# Firewall <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />

The default firewall in NixOS is `iptables` and enabled by default. It makes all local ports and 
services unreachable from external connections.

### Quick links
* [Overview](#overview)

## Overview

## Allow ports

### On all interfaces
```nix
networking.firewall = {
  enable = true;
  allowedTCPPorts = [ 80 443 ];
  allowedUDPPortRanges = [
    { from = 4000; to = 4007; }
    { from = 8000; to = 8010; }
  ];
};
```
