# AdGuard Home <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

AdGuard started in Moscow in 2009 with a free web analytics service. They have grown to offer a 
catalog of paid services utilizing their ad-blocking software. However they offer a free open source 
solution called ***AdGuard Home*** that competes with other open source options like ***Pi-Hole***.

### Quick links
* [Overview](#overview)
* [Install on Proxmox](#install-on-proxmox)
* [Configure Adguard Home](#configure-adguard-home)
* [Configure Clients](#configure-clients)
  * [NixOS Client](#nixos-client)

## Overview
[AdGuard Home](https://github.com/AdguardTeam/AdGuardHome) is written in Golang. It is a network wide 
ad and tracker blocking DNS server. It is on par with the with Pi-Hole. Like Pi-hole AdGuard Home 
isn't a true proxy/filter and only operates at the DNS level which mean it can block requests to ads, 
block sites and content but can't cosmetically clean the rendered web pages final display thus 
blocked components may leave noticible gaps or layout issues.

**References**
* [PI-Hole vs Adguard home](https://www.smarthomebeginner.com/pi-hole-vs-adguard-home/)

### Features
* Block ads, phishing and malware domains - configurable lists
* HTTPS admin interface - provides an easy-to-use GUI for settings and statistics
  * Shows DNS queries as well as which devices are making the queries
  * Provides controls to block for a specific device
* Built in DHCP/DNS servers - provide network wide support
* Encrypted DNS upstream servers
* Running DNS-over-HTTPS
* Parental control (blocking adult domains)
* Force Safe search on search engines
* Per-client (device) configuration
* Access settings
* Running without root

## Install on Proxmox
Using the proxmox helper scripts we can easily install an AdGuard Home linux container

1. Copy the Proxmox helper script
   1. Navigate to [Proxmox VE Helper-Scripts](https://community-scripts.github.io/ProxmoxVE/scripts)
   2. Select the `AdBlocker - DNS >AdGuard Home LXC`
      * Note: you can view what the script is doing via the `Source Code` button
   3. Copy the bash script
      ```bash
      $ bash -c "$(wget -qLO - https://github.com/community-scripts/ProxmoxVE/raw/main/ct/adguard.sh)"
      ```
2. Now we'll create the container
   1. In Proxmox select the server e.g. `server3` then select `>_ Shell` to get a shell
   2. Paste in the copied shell script to launch the installer
   3. Select `Yes` to proceed then `Advanced` for advanced settings
   4. Navigate through selecting `Debian`, `12 Bookworm`, `Unprivileged`
   5. Set Hostname `adguard`, `2 GB` Disk size, `1` cpu core, `1024` MB Ram, `vmbr0` bridge
   6. Set `dhcp` for IPv4 cider, `Disable IPv6`, leave DNS settings as Host
   7. `No` to root ssh access we can access via Proxmox directly
   8. Note the IP address it ended up with and make it static on your router

3. Configure the container
   1. Navigate to `Options`
   2. Set the Start/Shutdown order to `1`
   3. Switch to the `Network` tab
   4. Set a static ip for your instance e.g. `192.168.1.53/24` and gateway
   5. Reboot the container

## Configure Adguard Home

### Basic configuration
1. Configure first run wizard
   1. Open a browser up to e.g. `http://192.168.1.53:3000` this will change to just port `80` after 
      the first run configuration is done
   2. Switch to dark mode but clicking the button at the bottom
   3. Advance through the setup leaving the defaults
   4. Create an administrator account
2. Configure upstream DNS server
   1. Navigate to `Settings >DNS settings`
   2. Replace the default `https://dns10.quad9.net/dns-query` with cloudflare's 
      `https://dns.cloudflare.com/dns-query`
   3. Scroll down and set the fallback to google's `https://dns.google/dns-query`
   4. Click the `Test upstreams`
3. Configure General settings
   1. Navigate to `Settings >General settings`
   2. Enable `Block domains using filters and hosts files`
   3. Enable `Use AdGuard browsing security web service`
   4. Enable `Use AdGuard parental control web service`
   5. Enable `Use Safe Search`
   6. Set `Logs configuration >Query logs rotation` to `90 days`
   6. Set `Statistics configuration >Statistics retension` to `90 days`

### Setup device profiles
AdGuard Home has the capability to setup profiles for particular devices. You can identify devices by 
their static ip addresses or MAC addresses.

* Scheduled blocking of services

### Configure DNS blocklists
There are many blocklists that are compatible with AdGuard Home

**References**
* [DNS Blocklists aggregate](https://github.com/hagezi/dns-blocklists)
* [The Big Blocklist Collection](https://firebog.net/)
* [Repo for Firebog lists](https://github.com/WaLLy3K/wally3k.github.io)
* [Adguard Home settings by Celenityy](https://github.com/celenityy/adguard-home-settings)
* [Block adult content](https://github.com/Bon-Appetit/porn-domains)
* [Blocklist project](https://github.com/blocklistproject/Lists)

1. Login to AdGuard Home
2. Navigate to `Filters >DNS blocklists`
3. Click `Add blocklist` then `Choose from the list`
  * General
    * `AdAway Default Blocklist`
    * `AdGuard DNS Popup Hosts filter`
    * `AWAvenue Ads Rule`
    * `Dan Pollock's List`
    * `HaGeZi's Pro++ Blocklist`
    * `OISD Blocklist Big`
    * `Peter Lowe's Blocklist`
    * `Steven Black's List`
  * Other
    * `Dandelion Sprout's Anti Push Notifications`
    * `Dandelion Sprout's Game Console Adblock List`
    * `HaGeZi's Allowlist Referral`
    * `Perflyst and Dandelion Sprout's Smart-TV Blocklist`
    * `WindowsSpyBlocker - Hosts spy rules`
  * Security
    * `Phishing URL Blocklist (PhishTank and OpenPhish)`
    * `Dandelion Sprout's Anti-Malware List`
    * `HaGeZi's Badware Hoster Blocklist`
    * `HaGeZi's DynDNS Blocklist`
    * `HaGeZi's Threat Intelligence Feeds`
    * `NoCoin Filter List`
    * `Phishing Army`
    * `Scam Blocklist by DurableNapkin`
    * `ShadowWhisperer's Malware List`
    * `Stalkerware Indicators List`
    * `The Big List of Hacked Malware Web Sites`
    * `uBlock filters - Badware risks`
    * `Malicious URL Blocklist (URLHaus)`
  * Custom lists outside AdGuard
    * [EasyList](https://v.firebog.net/hosts/Easylist.txt)
    * [EasyPrivacy](https://v.firebog.net/hosts/Easyprivacy.txt)
    * [Blocklist adult content](https://blocklistproject.github.io/Lists/adguard/porn-ags.txt)

### Configure Local DNS entries
You can use AdGuard Home to setup DNS names for your local systems so you can stop using their IP 
addresses on your home lan without having to maintain a hosts list on every system in your home 
network.

1. Login to AdGuard Home
2. Navigate to `Filters >DNS rewrites`
3. Click `Add DNS rewrite`
4. Entry the DNS name you'd like e.g. `adguard.local`
5. Set the IP address to the system e.g. `192.168.1.53`

### Configure Blocked services
AdGuard supports a large list of well known sites that can be disabled with the flip of a switch. 
This is just an easy way to add specific sites to an internal block list. 

Personally it seems like simply adding them directly to the `Custom filtering rules` would make more 
sense as then you'd have a complete list that could be saved for the future.

### Configure Custom filtering rules
AdGuard Home supports custom one off rules to block or allow sites

1. Login to AdGuard Home
2. Navigate to `Filters >Custom filtering rules`
3. Add items
```
# Ads/Tracking allowed by AdGuard
||adservice.google.*^$important
||adsterra.com^$important
||amplitude.com^$important
||analytics.edgekey.net^$important
||analytics.twitter.com^$important
||app.adjust.*^$important
||app.*.adjust.com^$important
||app.appsflyer.com^$important
||doubleclick.net^$important
||googleadservices.com^$important
||guce.advertising.com^$important
||metric.gstatic.com^$important
||mmstat.com^$important
||statcounter.com^$important
```

### Manually block from query log
You can browse the Query log in AdGuard Home and select and block entries directly from there

1. Login to AdGuard Home
2. Navigate to `Query Log`
3. Search or find your log entry
4. Click the option menu on the right then `Block`
   * This will which will add a rule to the `Filters >Custom filtering rules` section.

## Configure Clients

### NixOS Client
Set the nix configuration for your system using 

Ensure you don't have either of these settings set as we want to use your home router as the default 
DNS provider which will in turn be using your adbuard home instance's as the DNS provider.

Remove settings for both the nameservers and their fallbacks
```nix
networking.nameservers = [ "1.1.1.1" "1.0.0.1"];
services.resolved.fallbackDns = [ "8.8.8.8" "8.8.4.4" ];
```
