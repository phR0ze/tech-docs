# Vopono <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Vopono provides the ability to run specific applications through a VPN tunnel using network 
namespaces. This allows for specific applications to run through a VPN tunnel while the rest of your 
machine continues to run over the main LAN network.

### Quick links
* [.. up dir](../README.md)
* [Overview](#overview)
  * [Configuration location](#configuration-location)
  * [Sharing network namespaces](#sharing-network-namespace)
* [Private Internet Access](#private-internet-access)
  * [PIA with Wireguard](#pia-with-wireguard)

## Overview
Network namespaces can be used to run specific applications over a specific network or tunnel while 
not affecting the rest of your system. I built an application https://github.com/phR0ze/openvpn-pia 
based on this concept using . However `Vopono` has better VPN support.

* You should run vopono as your own user.
* Post up/down scripts are run outside the network namespace before and after

**References**
* [Vopono User Guide](https://github.com/jamesmcm/vopono/blob/master/USERGUIDE.md)
* [schnouki's write up](https://schnouki.net/posts/2014/12/12/openvpn-for-a-single-application-on-linux/)

### Configuration location
Vopono stores its configuration in `~/.config/vopono/config.toml`

**Example**
```
firewall = "NfTables"
provider = "Mullvad"
protocol = "Wireguard"
server = "usa-us22"
postup = "/home/archie/postup.sh"
predown = "/home/archie/predown.sh"
user = "archie"
dns = "8.8.8.8"
# custom_config = "/home/user/vpn/mycustomconfig.ovpn"
```

### Sharing network namespace
Vopono exec invocations with the same server and provider will share the same network namespace.

**Example**
```bash
$ voponon exec --provider PrivateInternetAccess --server us_west --protocol wireguard firefox
$ voponon exec --provider PrivateInternetAccess --server us_west --protocol wireguard chromium
```

## Private Internet Access
Private Internet Access is the only commercial VPN that accepts Walmart gift cards as payment. They 
also allow for unlimited access with up to 5 connected devices and have the lowest rates out there, 
last time I checked.

**References**
* [PIA foss repo](https://github.com/pia-foss/manual-connections)
* [PIA docs](https://helpdesk.privateinternetaccess.com/guides/linux/linux-manual-connection-scripts)
* [PIA dns docs](https://helpdesk.privateinternetaccess.com/kb/articles/using-pia-dns-in-custom-configurations)

### PIA with Wireguard
[WireGuard](https://www.wireguard.com/) is a modern VPN technology that provides some benefits over 
the original OpenVPN method, namely less resources needed and more security.

1. Install vopono
   ```nix
   environment.systemPackages = [
     pkgs.wireguard-tools
     pkgs.vopono
   ];
   ```
2. Configure PrivateInternetAcces for use with wireguard
   1. Run: `vopono sync --protocol wireguard PrivateInternetAccess`
   2. Enter your PIA credentials
   3. Answer no to the port forwarding only filter
   4. Note the confguration that will be created
      * `~/.config/vopono/pia/wireguard` - all the server configuration files
      * `~/.config/vopono/pia/wireguard/config.txt` - json configuration file
3. Choose a server to use
   1. List them out `vopono servers --protocol wireguard PrivateInternetAccess`
   2. Choose one e.g. `us-saltlakecity`
6. Execution would look like
   ```bash
   $ voponon exec --provider PrivateInternetAccess --server us-saltlakecity --protocol wireguard firefox
   ```
7. Validate that your connection is over the VPN
   1. Browse to [PIA's what's my IP](https://www.privateinternetaccess.com/pages/whats-my-ip)

### DNS Leaks
Browse to [DNS Leak check](http://dnsleak.com)

TBD?
