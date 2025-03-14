# qBittorrent <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

### Quick links
* [.. up dir](../README.md)
* [Backup and Restore](#backup-and-restore)
  * [Backup](#backup)
  * [Restore](#restore)

## Configure

1. Lauch `qbittorrent` and open `Preferences`
2. Configure `Behavior`
   1. Check `Minimize qBittorrent to notification area`
3. Configure `Downloads`
   1. Check `Do not start the download automatically`
   2. Set `Default Save Path` to `/mnt/storage1/torrents`
4. Configure `Connection`
5. Configure `Speed`
6. Configure `BitTorrent`
   1. Check `Enable DHT`
   2. Check `Enable Peer Exchange`
   3. Check `Enable Local Peer Discovery`
   4. Change `Maximum active uploads` to `50`
   5. Change `Maximum active torrents` to `60`
7. Configure `Web UI`
8. Configure `Advanced`

## Backup and Restore

### Backup
Backup the following paths
* `~/.config/qBittorrent` - qBittorrent's configuration
* `~/.local/share/data/qBittorrent` - qBittorrent's fast resume files
* `/mnt/storage1/torrents` - actual shares

### Restore
[NixOS config reference](https://github.com/phR0ze/nixos-config/blob/main/options/services/raw/private-internet-access.nix)

1. Restore the following paths
   1. `~/.config/qBittorrent` - qBittorrent's configuration
   2. `~/.local/share/data/qBittorrent` - qBittorrent's fast resume files
   3. `/mnt/storage1/torrents` - actual shares
2. Configure and launch via VPN
   1. Manually launch `xdg-open /etc/xdg/autostart/qbittorrent-over-vpn.desktop`
3. Relink sources if you moved the torrent location
   1. Right click on the torrent and choose `Torrent options...`
   2. Ensure the `Save at` and `Use another path for incomplete torrent` are both set to `/mnt/storage1/torrents`
