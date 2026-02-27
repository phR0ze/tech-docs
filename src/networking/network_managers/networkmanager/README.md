# NetworkManager
NetworkManager is the most popular and intuitive way to handle networking in the Linux world if you 
want both wired and wireless networking to just work out of the box.

### Quick links
* [Install](#install)
* [Configure](#configure)
  * [Connection property defaults](#connection-property-defaults)
* [Keyfile Configs](#keyfile)
  * [Generate keyfile config](#generate-keyfile-config)
  * [Connection priority](#connection-priority)
  * [Configure DHCP connection](#configure-dhcp-connection)
  * [Configure Static connection](#configure-static-connection)
  * [Configure Bridge connection](#configure-bridge-connection)
  * [Start connection](#start-connection)
* [resolved](#resolved)
* [nmcli](#nmcli)
  * [Show all connections](#show-all-connecctions)

I've switched over to using NetworkManager with cyberlinux due to the user friendlyness of the 
system and its native support for wireguard.

[NetworkManager](https://wiki.archlinux.org/title/NetworkManager) Network Manager is a frontend for 
backend providers. Network Manager provides a nice system tray icon with UI wizards on par with OSX 
that then automate the configuration of backend providers like `systemd-networkd`, `wpa_supplicant`, 
and `openvpn`. NetworkManager has native support for `WireGuard` all it needs is the `wireguard` 
kernel module. The point of NetworkManager is to make networking configuration and setup as painless 
and automatic as possible. It should just work.

## Install
The following installation provides the systemd service `NetworkManager` and 
`NetworkManager-wait-online` the system tray applet `nm-applet` the graphical editor 
`nm-connection-editor` support for WiFi devices which NetworkManager default to use `wpa_supplicant` 
and openvpn integration.

1. Install Network Manager
   ```bash
   $ sudo pacman -S network-manager-applet networkmanager-openvpn wpa_supplicant
   ```

2. Disable `systemd-networkd`
   ```bash
   $ sudo systemctl disable systemd-networkd
   $ sudo systemctl disable systemd-networkd-wait-online
   $ sudo systemctl stop systemd-networkd.socket systemd-networkd
   ```

3. Enable and start Network Manager
   ```bash
   $ sudo systemctl enable NetworkManager
   ```

## Configure
Configuration load order with later files overriding ealier ones:
1. `/usr/lib/NetworkManager/conf.d`
2. `/run/NetworkManager/conf.d` per boot configuration for one time changes
3. `/etc/NetworkManager.conf`
4. `/etc/NetworkManager/conf.d`
5. `/var/lib/NetworkManager/NetworkManager-intern.conf` not user modifiable but can shadow things

By default if no other connection profiles exist in `/etc/NetworkManager/system-connections` then 
NetworkManager will create an in-memory DHCP connection called `Wired connection 1`. If you have a 
pre-configured connection though it won't do this.

References:
* [Network Manager Settings](https://developer-old.gnome.org/NetworkManager/0.9/ref-settings.html)
* [NetworkManager.conf](https://developer-old.gnome.org/NetworkManager/stable/NetworkManager.conf.html)
* [nmcli examples](https://developer-old.gnome.org/NetworkManager/stable/nmcli-examples.html)
* `man nm-settings`
* `man nmcli`

**See NetworkManager's current settings:**
```bash
$ sudo NetworkManager --print-config
```

### Connection Property Defaults
A number of connection properties can have defaults set that will only be used if the connection is 
configured to explicitely use the defaults on a per property basis. This might be a good place to put 
`TCP Slow Start` speedups etc...

## Keyfile Configs
NetworkManager will read `/etc/NetworkManager/system-connections` for any manually configured 
connections via its `keyfile` plugin which is always enabled. Connection files need to be owned by 
`root` and set to `0600` permissions or they won't be displayed in the list.

**References**
* [network mangager docs on keyfile](https://networkmanager.dev/docs/api/latest/nm-settings-keyfile.html)
* [connection properties](https://www.networkmanager.dev/docs/api/latest/settings-connection.html)

keyfile aliases to keep in mind:
* `802-3-ethernet` = `ethernet`
* `802-11-wireless` = `wifi`
* `802-11-wireless-security` = `wifi-security`

### UUID value
For the `UUID` value present in all the keyfile configs. Just leave it out and NetworkManager will 
generate one for your configuration based on the filename if it is missing.

### Generate keyfile config
You can create a new connection profile via the network manager applet then check what the keyfile 
version of it looks like in `/etc/NetworkManager/system-connections`

### Connection priority
We create the `static` connection profile with a higher connection priority than the DHCP connection 
profile such that it will get tried first if it exists, but we can easily fall back on DHCP by simply 
manually setting it to the active connection profile. To test this you can bring down the connections 
with `nmcli con down "Wired dhcp"` and then `sudo systemctl restart NetworkManager` and 
NetworkManager will restart see the priorities and load the correct one.

Example:
```
autoconnect-priority=0
```

For bridge connections use the `autoconnect-slaves` property
* `0` - leave slave untouched
* `1` - activate all slave connections with the bridge
* `-1` - default, use `global connection.autoconnect-slaves` else `0`

### Configure DHCP connection
```bash
$ sudo cat <<EOF > /etc/NetworkManager/system-connections/dhcp
[connection]
id=Wired dhcp
type=ethernet
autoconnect-priority=0

[ipv4]
method=auto

[ipv6]
method=disabled
EOF
$ sudo chmod 0600 /etc/NetworkManager/system-connections/dhcp
```

### Configure Static connection
```bash
$ sudo cat <<EOF > /etc/NetworkManager/system-connections/static
[connection]
id=Wired static
type=ethernet
autoconnect-priority=1

[ipv4]
method=manual
address=192.168.1.15/24
gateway=192.162.1.1
dns=1.1.1.1;1.0.0.1

[ipv6]
method=disabled
EOF
$ sudo chmod 0600 /etc/NetworkManager/system-connections/static
```

### Configure Bridge connection
Note: this connection is meant to take the place of your main static connection

```bash
$ sudo cat <<EOF > /etc/NetworkManager/system-connections/bridge
[connection]
id=Bridge
type=bridge
interface-name=virbr0
autoconnect-priority=2

[bridge]
stp=false

[ipv4]
method=manual
address=192.168.1.15/24
gateway=192.168.1.1
dns=1.1.1.1;1.0.0.1

[ipv6]
method=disabled
EOF
$ sudo chmod 0600 /etc/NetworkManager/system-connections/bridge
```

### Start connections
Note: if your new connecdtions don't show up check their permission bits are correct `0600`

1. Reload from disk
   ```bash
   $ sudo nmcli con reload
   ```

2. Start static connection
   ```bash
   $ sudo nmcli con up "Wire static"
   ```

3. Start DHCP connection
   ```bash
   $ sudo nmcli con up "Wire dhcp"
   ```

## resolved
NetworkManager will use `systemd-resolved` automatically as its DNS resolver and cache. You just need 
to ensure that `/etc/resolv.conf` is a symlink to `/run/systemd/resolve/stub-resolv.conf` or you can 
explicitely enable it via editing `/etc/NetworkManager/conf.d/dns.conf` and adding:
```
[main]
dns=systemd-resolved
```

Then reload the configuration with: `sudo nmcli general reload`

## nmcli
nmcli is the command line tool for controlling NetworkManager and reporting network status. It can be 
utilized as a replacement for nm-applet or other graphical clients. nmcli is used to create, display, 
edit, activate and deactivate network connections, as well as control and display network device 
status. nmcli is the support way to automate NetworkManager operations for scripts and other 
purposes.

**References**
* [Network Manager cli docs](https://networkmanager.dev/docs/api/latest/nmcli.html)

### Show all connections
```bash
$ sudo nmcli con
```
