# Virtualization <img style="margin: 6px 13px 0px 0px" align="left" src="../data/images/logo_36x36.png" />

Documenting various virtualization technologies

### Quick links
- [.. up dir](..)
- [Overview](#overview)

### Linked pages
- [Containers](containers/README.md)
  - [Docker](containers/docker/README.md)
    - [dind](containers/docker/dind/README.md)
  - [Flatpak](containers/flatpak/README.md)
  - [Podman](containers/podman/README.md)
  - [systemd-nspawn](containers/systemd-nspawn/README.md)
- [Networking](networking/README.md)
  - [Bridge](networking/bridge/README.md)
  - [MacVLAN](networking/macvlan/README.md)
- [Orchestration](orchestration/README.md)
  - [Incus](orchestration/incus/README.md)
  - [Kasm](orchestration/kasm/README.md)
  - [Kubernetes](orchestration/kubernetes/README.md)
  - [Orbstack](orchestration/orbstack/README.md)
  - [Portainer](orchestration/portainer/README.md)
  - [ProxMox](orchestration/proxmox/README.md)
- [Virtual Machines](virtual_machines/README.md)
  - [libvirt](virtual_machines/libvirt/README.md)
  - [QEMU](virtual_machines/qemu/README.md)
    - [QEMU Networking](virtual_machines/qemu/qemu_networking/README.md)
    - [NIX QEMU](virtual_machines/qemu/nix_qemu/README.md)
  - [VirtManager](virtual_machines/virt_manager/README.md)

## Overview

### Apps to evaluate
[Proxmox related Helper Scripts repo](https://community-scripts.github.io/ProxmoxVE/scripts) has some 
great applications packaged up for easy deployment and gives some exposure to the current open source 
landscape.

| App           | Deployment methods        | Description
| ------------- | ------------------------- | ----------------------------------------------------
| AdGuard Home  | Docker, tar, deb          | DNS blocker with more features than Pi-Hole
| Dashy         | Docker, Nixpkgs, deb, LXC | DNS blocker with more features than Pi-Hole
| Nextcloud     | Docker, LXC               | Private Google Suite alternative
| Jellyfin      | LXC               | Plex like media server

* Servarr Suite a.k.a ARR stack - media automation
  * Lidarr
  * Prowlarr - 
  * Radarr - 
  * Readarr
  * Sonarr - 
  * Whisparr
  * Bazarr
  * Notifiarr
* Dashboards
  * Dashy
  * Homarr
    * said to be better than Heimdall
    * Integration with `Sevarr` suite of apps
* emby
* plex
* home assistant os
* Nextcloud
* Cockpit
  * Easy to use Web GUI for linux servers
  * Provides support for creating network shares
  * `apt install --no-install-recommends cockpit -y`
  * Add Cockpit plugins: `File Sharing`, `Identities`, `Navigator`
  * wget them then `apt install ./*deb`
* Services (UNRAID)
  * Plex - exellent for streaming externally
  * Lidarr - 
  * Sonarr - Shows, same as Radarr
  * Radarr - Movies, same as Sonarr
  * Prowlarr - track your indexers for Sonarr and Radarr
  * Overseerr - use for discovering new media and tracks requests for aquiring them
  * Kavita - ebook management and reader
  * Tautulli - plex server metrics on play counts, duration, last 30 days.
  * FreshRSS - news collection and management
  * Mealie - Recipies collection management
  * Audiobookshelf - audio streaming, own version of audible
  * NZBGet - News server download client using delugevpn network
    * Uses index to download all components of upload then unzip together
  * DelugeVPN - Deluge with built in VPN (OpenVPN with PIA)
  * Jellyfin
  * Kasm
  * Portainer
  * Watchtower
* Hardware
  * Intel Nuc with Ubuntu and Portainer
  * UNRAID
  * Synology NAS

## Kasm
[Kasm](https://www.kasmweb.com/) is a virtualization system for running containers. You can even 
self-host it in a Proxmox VM. It allows for launching single applications to full linux distributions 
as Containers. I see Kasm as a Portainer like solution that does a better job than Portainer. 
Although, depending on your needs, it might work as a Proxmox replacement; Proxmox provides better 
Hypervisor VM support for general purpose needs.

Docker container streaming to your Browser

**Features**
* Self-hosting free Community option is limited to 5 concurrent sessions
* Registry with pre-packaged containers
* Custom containers

### 

<!-- 
vim: ts=2:sw=2:sts=2
-->
