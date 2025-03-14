# WireGuard <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

[WireGuard](https://www.wireguard.com/) is a secure networking tunnel. It can be used as a VPN, for 
connecting datacenters together across the internet any place where you need to join two networks 
together. It was initially released for the Linux kernel, it is now cross-platform (Windows, macOS, 
BSD, iOS, Android) and is widely deployable. WireGuard aims to be as easy to configure and deploy 
as SSH. A VPN connection is made simply by exchanging very simple public keys exactly like exchanging 
SSH keys and all the rest is transparently handled by WireGuard.

### Quick links
* [.. up dir](../README.md)
* [Overview](#overview)
* [WireGuard Client](#wireguard-client)
  * [Install WireGuard](#install-wireguard)
  * [Starting WireGuard](#starting-wireguard)
* [WireGuard Server](#wireguard-server)

## Overview
WireGuard uses modern ciphers like the `Noise protocol framework`, `Curve25519`, `ChaCha20`, 
`Poly1305`, `BLAKE2`, `SipHash24`, `HKDF`. It works by adding a network interface like `eth0` or 
`wlan0` called `wg0`. This network interface can then be configured normally using `ip` with routes 
for it added and removed via `route` or `ip-route` and so on using all the normal networking 
utilities.

References:
* [Arch Linux WireGuard](https://wiki.archlinux.org/title/WireGuard)

Features:
* Faster, lighter and better than OpenVPN and IPsec and other VPN tech made in the 90s.
* Can't scan for it on the internet, its undetectable unless you know where it is
* Has a very small code base that can fit into the kernel

## WireGuard Client

### Install WireGuard
```bash
$ sudo pacman -S wireguard-tools
```

### Generate private/public key pair

### Configure all traffic tunnel
```
cat /etc/wireguard/wg0.conf
[Interface]
PrivateKey = `PRIVATEKEY`
Address = `IPV4FROMVPNPROVIDER`,`IPV6FROMVPNPROVIDER`
DNS = `VPNDNS4`,`VPNDNS6`
PostUp = ip route add `192.168.1.0/24 via 192.168.1.1`;
PreDown = ip route delete `192.168.1.0/24`;

[Peer]
PublicKey = `PUBLICKEY`
AllowedIPs = `0.0.0.0/0`,`::0/0`
Endpoint = `PUBLICVPNSERVERIP`:`PORT`
PersistentKeepalive = 25
```

### Starting WireGuard
```bash
$ sudo systemctl enable wg-quick@wg0
$ sudo systemctl start wg-quick@wg0
```

## WireGuard Server
1. Enable kernel ipv4 forwarding
   ```bash
   $ echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.d/10-ipv4-forwarding.conf
   ```


