# Overlay Networks <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />

Overlay networks are a super set of VPNs, Mesh technologies like Istio and other types of virtualized 
networks that overlay lower level networks. There is potentially some distinction between VPNs and 
Overlays in that VPNs typically have a client connect to a remote server which then assigns your 
machine an IP from the remote network and you access those services remotely while more modern 
overlay networks often connect peer-to-peer using an encrypted mesh, but there are many variations as 
well.

From a practical perspective it ends up being a similar outcome. Your able to securely connect to a 
remote network and use the resources there regardless of how that network is composed.

### Quick links
* [.. up dir](../README.md)
* [Overview](#overview)
  * [Overlay networks](#overlay-networks)

### Linked pages
* [OpenConnect](#openconnect)
* [OpenVPN](openvpn/README.md)
* [WireGuard](wireguard/README.md)

## Overview

### Overlay networks
Overlay networks require a coordination server to allow peers to find each other. The coordination 
servers will broker connections and hands off the connection to the peers.

Key: Open Source (OS)

| Name          | Wireguard | OS Client | OS server      | Client Support
| ------------- | --------- | --------- | -------------- | --------------------
| Tailscale     | yes       | yes       | no, headscale  | Win, Mac, Linux, Android, pfSense, ISO, BSD, Synology
| Netbird       | yes       | yes       | yes            | Win, Mac, Linux, Android, IOS
| Netmaker      | yes       | yes       | yes            | Win, Mac, Linux, BSD
| Zerotier      | no        | yes       | yes, no web ui | Win, Mac, Linux, BSD, Android, IOS, Synology
| Twingate      | no        | no        | no             | Win, Mac, Linux, Android, IOS, ChromeOS

## OpenConnect
https://wiki.archlinux.org/index.php/OpenConnect  
OpenConnect is a client for Cisco's AnyConnect SSL VPN and Pulse Secure's Pulse Connect Secure.

**Note:** I kept wanting to pass in all the fields I could. It works better to use the minimal args
and OpenConnect will then prompt for information it needs.

```bash
# Install OpenConnect
$ sudo pacman -S openconnect vpnc

# Connect to AnyConnect VPN
$ sudo openconnect --user=<USER-NAME> --authgroup=<AUTH-GROUP> <VPN GATEWAY NAME/IP>
...
Please enter your username and password.
Password:
# When prompted for `Password:` paste in your ldap password
Password:
# When prompted again for `Password:` u may or may not have a second
Verification code:
Response:
# When prompted for `Response:` paste in authy code
Connect Banner:
Welcome to Example VPN!

# Route shows correct routing?
$ route

# Check correct dns
$ systemd-resolved --status
```

<!-- 
vim: ts=2:sw=2:sts=2
-->
