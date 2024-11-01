# OpenVPN

Many VPN services are based on OpenVPN. In this section I'll be working through common configuration
options. OpenVPN client configuration files are stored in `/etc/openvpn/client` usually with the `.ovpn`
extension.

### Quick links
* [Connect via Commandline](#connect-via-commandline)
* [Connect via NetworkManager](#connect-via-command-line)
* [Troubleshooting DNS](#troubleshooting-dns)
  * [Check VPN dns routing](#check-vpn-dns-routing)

## Connect via Commandline
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

## Connect via NetworkManager
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

9. Click `Routes` and check `Use this connection only for resources on its network`

10. Click `Save`

## Troubleshooting DNS

**DNS Example routing search domain:**  
Your VPN interface `tun0` has a search domain of `private.company.com` and a routing domain of 
`~company.com`. If you ask for `mail.private.company.com` it is matched by both domains and will be 
routed to `tun0`. Additionally requests for `www.company.com` would match the routing domain and be 
routed over `tun0` as well.

### Check VPN dns routing
see [../../dns/README.md#split-dns](../../dns/README.md#split-dns)

