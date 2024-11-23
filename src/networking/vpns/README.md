# VPNs <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />

Virtual Private Networks provide an encrypted connection to a remote network. Your machine gets 
assigned an IP address on that network and you can access services on that network.

Overlay networks take a different approach. Rather than tunneling all traffic through a centralized 
VPN server, they create a secure peer-to-peer encrypted mesh network.

From a practical perspective it ends up being a similar outcome. Your able to securely connect to a 
remote network and use the resources there.

### Quick links
* [Overview](#overview)
  * [Overlay networks](#overlay-networks)
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
