# pfsense <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

pfSense is a firewall/router computer software distribution based on FreeBSD. The open source 
Community Edition (CE) can be installed on a virtual machine or physical computer dedicated as a 
firewall/router. It can be configured and upgraded through a web based interface.

OPNsense was forked from pfSense. It is an open source FreeBSD based firewall and router developed by 
Deciso, a company in the Netherlands that makes hardware and sells support packages for OPNsense.

### Quick links
* [pfSense Overview](#pfSense overview)

## pfSense Overview

### Bare metal or Virtualized
Virtualization can be done but poses some issues:
* Passing NICs down can be complicated
* Ensuring VLANs are correct can be complicated
* Ensuring proper performance can be complicated
* Always up and available might be a problem

**Hardware** requirements are not complicated, just x86 low power.

**Zima Board**
* Compatible with Linux, OpenWrt, pfSense
* Intel Celeron N3450 Quad Core 1.1-2.2GHz
* 8GB RAM, 32 GB eMMC, (2) lan ports, (2) sata connections
* Intel VT-d, VT-x virtualization
* H.264, H.265, MPEG-2, VC-1
* Pre-installed CasaOS

### UniFi vs pfSense
**UniFi** is a complete centralized management platform for all their firewall, router switch 
devices. It is prosumer hardware

| Features                      | pfSense | Unifi Dream | Notes
| ----------------------------- | ------- | ----------- | -------------------------
| Can Run on your own hardware  | yes     | no          | 
| Can be virtualized            | yes     | no          | 
| Supports routing              | yes     | yes         | 
| Centralized management        | no      | yes         | 
| Web interface                 | yes     | yes         | 
| License fees                  | no      | no          | 
| Operating system              | FreeBSD | Linux       |
| Automated updates             | no      | yes         | 
| Granular update rollback      | yes     | no          | 
| VLAN support                  | yes     | yes         | 
| Inteface groupings            | yes     | no          | pfSense: WireGuard or Tailscale groups  
| BGP / OSPF                    | yes     | yes         |
| Captive portal                | yes     | yes         |
| OpenVPN                       | yes     | yes (basic) | pfSense: a lot more options
| IPSec                         | yes     | yes         | pfSense: a lot more options
| WireGuard                     | yes     | yes         | 
| L2TP VPN                      | yes     | yes         |
| Automatic Site to Site        | no      | yes         | only useful for multiple locations
| Tailscale                     | yes     | no          | 
| IDS/IPS                       | yes     | yes         | pfSense: a lot more options
| Content filtering & Controls  | no      | yes         | adguard is way better
| DNS filtering                 | yes     | yes         | adguard is way better
| Advanced DNS (host overrides) | yes     | no          |
| GeoIP Filtering               | yes     | yes         |
| Traffice shaping              | yes     | yes         |
| Multi LAN failover            | yes     | yes (basic) | pfSense: a lot more options
| SNMP monitoring               | yes     | yes         |
| Active Directory integration  | yes     | yes         | 
| Policy routing                | yes     | yes (basic) |
| Packet Capture & Diag Tools   | yes     | no          |
| Reversee Proxy or WAF         | yes     | no          | homelab use
| Let's encrypt certificate     | yes     | no          | homelab use
| Firewall rules                | yes     | yes         | pfSense: way more options and better layout

### UniFi Devices compared
UniFi devices come in to categories: they have a built in controller or can be managed by an external 
controller like UXG Lite, UXG Pro which are for multiple sites. Dreamwall all in one solution is also 
not a great fit.

**Routers**
UniFi Dream Router (UDR), UniFi Express (UE), UniFi Cloud Gateway Ultra (UCG), UDM Pro/UDM SE (UDM)
* Built in controller
* VLAN and firewall capable

| Feature                       | UE  | UDR | UCG | UDM | Notes
| ----------------------------- | --- | --- | --- | --- | -------------------------------------
| Price                         | 149 | 199 | 129 | 499 | 
| Device support                | 4   | 4   |     |     | UE: AP and switch combinations up to 4
| Number of UniFi apps          | 1   | 2   | 1   | all | UDM: runs Network, Protect, Access, Talk, Connect, InnerSpace
| Built in WiFi 6               | yes | yes | no  | no  | 
| Wireless mesh support         | yes | yes | no  | no  | 
| VPN support                   | yes | yes | yes | yes | 
| Port speeds                   | 1G  | 1G  | 2.5G| 2.5 | UDM: up to 10G lan
| Ports                         | 2   | 5   |  5  | 8   | UDR: PoE power on 2 ports, UDM SE provides PoE
| IDS/IPS                       | no  | yes | yes |     | UCG: all security featurs on full speed
| Camera number support         | 0   | 4   |  0  | all | 
| Storage                       | no  | SD  | no  | HDD | UDR: all security features slows it down
| |     | |     |     | 
| |     | |     |     | 

**Access Points**
* Wireless mesh support

