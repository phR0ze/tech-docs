# systemd-networkd

### Quick links
* [DHCP Networking](#dhcp-networking-systemd-networkd)
* [Static Networking](#static-networking-systemd-networkd)
* [Wifi Networking](#wifi-networking-systemd-networkd)
* [systemd-networkd-wait-online](#systemd-networkd-wait-online)

# systemd-networkd
`systemd-networkd` is a bare bones, light and simple networking configuration. In conjunction with 
`wpa_supplicant` and `WPA_UI` I've got by just fine. However it does lack some of the elegance 
heavier weight solutions like Network Manager provide.

## Install systemd-networkd
1. Disable `NetworkManager`
   ```bash
   $ sudo systemctl disable NetworkManager
   $ sudo systemctl disable NetworkManager-wait-online
   ```
2. Disable `nm-applet`
   ```bash
   $ sudo rm /etc/xdg/autostart/nm-applet.desktop
   ```
3. Enable `systemd-networkd`
   ```bash
   $ sudo systemctl enable systemd-networkd
   $ sudo systemctl enable systemd-networkd-wait-online
   ```

## DHCP Networking
```bash
# Create config file
sudo tee -a /etc/systemd/network/10-static.network <<EOL
[Match]
Name=en* eth*

[Network]
DHCP=ipv4
IPForward=kernel

[DHCP]
UseDomains=true
EOL

# Configure DNS
sudo ln -sf /run/systemd/resolve/resolv.conf /etc/resolv.conf

# Enable/start resolved
sudo systemctl enable systemd-networkd systemd-resolved
sudo systemctl start systemd-networkd systemd-resolved

# Restart networking
sudo systemctl restart systemd-networkd
```

## Static Networking
```bash
# Create config file
sudo tee -a /etc/systemd/network/10-static.network <<EOL
[Match]
Name=en* eth*

[Network]
Address=192.168.1.6/24
Gateway=192.168.1.1
DNS=1.1.1.1
DNS=1.0.0.1
IPForward=kernel
EOL

# Restart networking
sudo systemctl restart systemd-networkd
```

## Wifi Networking
1. Ensure kernel driver is accurate:
  ```bash
  inxi -N
  # Network:   Card-1: Intel Ethernet Connection I217-LM driver: e1000e 
  #            Card-2: Intel Centrino Advanced-N 6235 driver: iwlwifi 
  ```
2. Rename wifi id to something predictable:
  ```bash
  sudo tee /etc/systemd/network/10-wlo1.link <<EOL
  [Match]
  OriginalName=wl*

  [Link]
  Name=wlo1
  EOL
  sudo reboot
  ```

3. Ensure ***systemd-networkd*** has been configured:
  ```bash
  sudo tee /etc/systemd/network/30-wireless.network <<EOL
  [Match]
  Name=wl*

  [Network]
  DHCP=ipv4
  IPForward=kernel

  [DHCP]
  RouteMetric=20
  UseDomains=true
  EOL
  sudo systemctl daemon-reload
  sudo systemctl restart systemd-networkd
  ```
4. Create minimal ***wpa_supplicant*** config:
  ```bash
  sudo pacman -S wpa_supplicant wpa_gui
  sudo tee /etc/wpa_supplicant/wpa_supplicant-wlo1.conf <<EOL
  ctrl_interface=/run/wpa_supplicant
  ctrl_interface_group=wheel
  update_config=1
  p2p_disabled=1
  EOL
  sudo systemctl daemon-reload
  sudo systemctl enable wpa_supplicant@wlo1
  sudo systemctl start wpa_supplicant@wlo1
  sudo systemctl status wpa_supplicant@wlo1
  ```
4. Configure Wifi Connection:
   a. Launch the gui: `sudo wpa_gui`  
   b. Click `Scan >Scan`  
   c. Double click the target network   
   d. Choose `CCMP` as the Encryption method for `AES` endpoints  
   e. Enter the `PSK` and click `Add`  
   f. Click `Close` and you should see it is already `Completed` i.e. connected  

## systemd-networkd-wait-online timing out
While trouble shooting a NFS share failing to mount I ran into this interesting tid bit of
information. Turns out that `systemd-networkd-wait-online` by default will wait for all network
interfaces to be ready. This means if you have a system with multiple nics and only use one the
others will be in a perpetual `configuring` state which cause `systemd-networkd-wait-online` to
always time out which is super annoying. A better default would be to have it to use the `--any` flag
which will cause it to succeed if any nics are online.

```bash
# Find name of mount unit
$ sudo systemctl | grep Cache
mnt-Cache.mount

# List out the dependencies and found `systemd-networkd-wait-online.service` was in red
$ sudo systemctl list-dependencies mnt-Cache.mount
mnt-Cache.mount
● ├─system.slice
● └─network-online.target
●   └─systemd-networkd-wait-online.service

# Checking the status of `system-networkd-wait-online`
$ sudo systemctl status systemd-networkd-wait-online.service
...
     Active: failed (Result: exit-code) since Sun 2020-12-27 16:36:39 MST; 9min ago
       Docs: man:systemd-networkd-wait-online.service(8)
   Main PID: 468 (code=exited, status=1/FAILURE)
...

# Network status indicated it timed out waiting for a network interface that was not yet configured
# https://github.com/systemd/systemd/issues/2713
$ sudo networkctl status
...
Dec 27 16:36:39 main4 systemd[1]: Failed to start Wait for Network to be Configured.

# Checkint my interfaces I have one in a `configuring` state i.e. not ready
$ sudo networkctl -a
IDX LINK    TYPE     OPERATIONAL SETUP      
  1 lo      loopback carrier     unmanaged  
  2 eno1    ether    no-carrier  configuring
  3 enp1s0  ether    routable    configured 

# Turns out that by default `systemd-networkd-wait-online` will wait for `all` network interfaces
# to be line even if they are not currently being used. So we need to modify its configuration
# so that it only waits for at least 1 to be ready by default.
# https://askubuntu.com/questions/1217252/boot-process-hangs-at-systemd-networkd-wait-online

# Create this override file to tell it to only wait for 1 interface
$ sudo mkdir -p /etc/systemd/system/systemd-networkd-wait-online.service.d
$ sudo tee -a /etc/systemd/system/systemd-networkd-wait-online.service.d/override.conf <<EOL
[Service]
# To replace values here we need to first clear out the value
ExecStart=
# Then set it to what we want, else it will be addative
ExecStart=/usr/lib/systemd/systemd-networkd-wait-online --any
EOL

# Restarting the service completes immediately meaning we fixed it
$ sudo systemctl restart systemd-networkd-wait-online
```
