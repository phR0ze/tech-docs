# VPNs

### Quick links
* [OpenConnect](#openconnect)
* [OpenVPN](#openvpn)
  * [Connect via Commandline](#connect-via-commandline)
  * [Connect via NetworkManager](#connect-via-command-line)
* [WireGuard](#wireguard)
  * [WireGuard Client](#wireguard-client)
  * [WireGuard Server](#wireguard-server)

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

## OpenVPN
Many VPN services are based on OpenVPN. In this section I'll be working through common configuration
options. OpenVPN client configuration files are stored in `/etc/openvpn/client` usually with the `.ovpn`
extension.

### Connect via Commandline
This is a `Split DNS` solution meaning we will use the VPN's DNS resolver for all things over the VPN 
and your normal DNS resolver for everything else. This is accomplished using the helper 
[update-systemd-resolved](https://github.com/jonathanio/update-systemd-resolved) script
that reads from the `dhcp-option` field in the server or client config then applies them dynamically 
to `systemd` via the `dbus`.

1. Install from the cyberlinux repo:
   ```bash
   $ sudo pacman -S openvpn-update-systemd-resolved
   ```

2. Install OpenVPN client configuration file:
   ```bash
   $ sudo mv <client>.ovpn /etc/openvpn/client
   ```

3. Revoke read permissions on the client config to keep secrets secure:
   ```bash
   $ sudo chmod og-r /etc/openvpn/client/<client>.ovpn
   ```

4. Establish the VPN connection with Split DNS Resolution:
   ```bash
   $ sudo openvpn --config /etc/openvpn/client/<client>.ovpn --setenv PATH \
       '/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin' \
       --script-security 2 --up /etc/openvpn/scripts/update-systemd-resolved --up-restart \
       --down /etc/openvpn/scripts/update-systemd-resolved --down-pre
   ```

5. Check that the Split DNS was configured correctly:
   ```bash
   # Look for new nameserver entries for the vpn
   $ sudo resolvectl status
   Link 3 (enp1s0)
   ...
   Current DNS Server: 1.1.1.1
   ...
   Link 6 (tun0)
   ...
   Current DNS Server: <NAMESERVER 1>
   ```

### Connect via NetworkManager
1. Install NetworkManager openvpn plugin
   ```bash
   $ sudo pacman -S networkmanager-openvpn gnome-keyring
   ```
2. Left click on the NetworkManager applet in the tray and choose `VPN Connections > Add a VPN 
   connection...`
3. Choose the VPN Connection Type as `Import a saved VPN configuration...` and choose `Create...`
4. Navigate to your `ovpn` file e.g. `~/Downloads/TARGET.ovpn`
   * Note that NetworkManager will be copying out of `~/Downloads/TARGET.ovpn` the configuration it 
   needs into `~/.cert` and `/etc/NetworkManager/system-connections`
5. Set your `User name` and `Password`
   * Change the password option to `Store the password for all users` or it doesn't save at all
6. Switch to the `IPv4 Settings tab`
7. Change the `Method` drop down from `Automatic (VPN)` i.e. send all traffic over the VPN to 
   `Automatic (VPN) addresses only` which will only send vpn address ranges over the VPN.
8. Also set the `DNS servers` and `search domains` to use as desired
9. Click `Save`


VPN connections can be configured to only receive traffic for internal company resources. To use this 
mode with NetworkManager check the box `Use this connection only for resources on its network` at the 
bottom of the IPv4 VPN's configuration.

**Example:**  
Your VPN interface `tun0` has a search domain of `private.company.com` and a routing domain of 
`~company.com`. If you ask for `mail.private.company.com` it is matched by both domains and will be 
routed to `tun0`. Additionally requests for `www.company.com` would match the routing domain and be 
routed over `tun0` as well.

Determine current domain routing with:
```bash
$ resolvectl domain
Global:
Link 2 (eno1):
Link 18 (tun0): company.com
```

In the domain routing list above anything ending in `company.com` would resolve over the `tun0` link. 
And you can check the name servers that will be used per link as well.
```bash
$ resolvectl dns
Global: 1.1.1.1 1.0.0.1
Link 2 (eno1): 1.1.1.1 1.0.0.1
Link 18 (tun0): 10.45.248.15 10.38.5.26
```

### systemd-resolved and vpn dns
`systemd-resolved` first checks for system overrides at `/etc/systemd/resolved.conf` then for network
configuration at `/etc/systemd/network/*.network`, then vpn dns configuration and finally falls back
on the fallback configuration in `/etc/systemd/resolved.conf`. I've found the most reliable way to
get vpn dns to work correctly is to not set anything except the fallback configuration so that dns is
configured by the vpn and when not on the vpn dns is configured by the fallback.


# WireGuard
[WireGuard](https://www.wireguard.com/) is a secure networking tunnel. It can be used as a VPN, for 
connecting datacenters together across the internet any place where you need to join two networks 
together. It was initially released for the Linux kernel, it is now cross-platform (Windows, macOS, 
BSD, iOS, Android) and is widely deployable. WireGuard aims to be as easy to configure and deploy 
as SSH. A VPN connection is made simply by exchanging very simple public keys exactly like exchanging 
SSH keys and all the rest is transparently handled by WireGuard.

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

<!-- 
vim: ts=2:sw=2:sts=2
-->
