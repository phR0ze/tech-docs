# File Sharing <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />
https://wiki.archlinux.org/index.php/NFS

### Quick links
* [NFS Client Config](#nfs-server-config)
* [NFS Server Config](#nfs-server-config)
* [systemd-networkd-wait-online timing out](#systemd-networkd-wait-oneline-timing-out)

## NFS Client Config
* ***nfs*** – calls out the type of technology being used
* ***auto*** – maps the share immediately rather than waiting until it is accessed
* ***noacl*** – turns off all ACL processing, if your not woried about security i.e. home network this is find to turn off
* ***noatime*** – disables NFS from updating the inodes access time, it can be safely ignored to speed up performance a bit
* ***nodiratime*** – same as noatime but for directories
* ***rsize and wsize*** - bytes read from server, default: 1024, larger values e.g. 8192 improve throughput
* ***timeo=14*** – time in tenths of a second to wait before resending a transmission after an RPC timeout, default: 600
* ***_netdev*** – tells systemd to wait until the network is up before tyring to mount the share

```bash
# Check exported list on client side
$ showmount -e 192.168.1.3

# Create local mount points
$ sudo mkdir -p /mnt/{Cache,Documents,Educational,Family,Install,Kids,Movies,Pictures,TV}

# Set local mount point ownership to your user
$ sudo chown -R <user-name>: /mnt/{Cache,Documents,Educational,Family,Install,Movies,Pictures,TV}

# Optionally manually mount/umount remote share to test it out
$ sudo mount 192.168.1.2:/srv/nfs/Movies /mnt/Movies
$ sudo umount /mnt/Movies

# Setup automount for shares
sudo tee -a /etc/fstab <<EOL
192.168.1.2:/srv/nfs/Cache /mnt/Cache nfs auto,noacl,noatime,nodiratime,rsize=8192,wsize=8192,timeo=15,_netdev 0 0
192.168.1.2:/srv/nfs/Documents /mnt/Documents nfs auto,noacl,noatime,nodiratime,rsize=8192,wsize=8192,timeo=15,_netdev 0 0
192.168.1.2:/srv/nfs/Educational /mnt/Educational nfs auto,noacl,noatime,nodiratime,rsize=8192,wsize=8192,timeo=15,_netdev 0 0
192.168.1.2:/srv/nfs/Family /mnt/Family nfs auto,noacl,noatime,nodiratime,rsize=8192,wsize=8192,timeo=15,_netdev 0 0
192.168.1.2:/srv/nfs/Install /mnt/Install nfs auto,noacl,noatime,nodiratime,rsize=8192,wsize=8192,timeo=15,_netdev 0 0
192.168.1.2:/srv/nfs/Movies /mnt/Movies nfs auto,noacl,noatime,nodiratime,rsize=8192,wsize=8192,timeo=15,_netdev 0 0
192.168.1.2:/srv/nfs/Pictures /mnt/Pictures nfs auto,noacl,noatime,nodiratime,rsize=8192,wsize=8192,timeo=15,_netdev 0 0
192.168.1.2:/srv/nfs/TV /mnt/TV nfs auto,noacl,noatime,nodiratime,rsize=8192,wsize=8192,timeo=15,_netdev 0 0
EOL
sudo mount -a
```

## NFS Server Config
Kodi recommends the `(rw,all_squash,insecure)` for the export options. In my experience the nfs root
was also required `/srv/nfs             192.168.1.0/24(rw,fsid=0,no_subtree_check)` and although the
linux nfs client worked fine without it, Kodi wouldn't work until it was added.

* ***.0/24*** - suffix allows all devices on the network to be able to see the share
* ***rw*** - allow read write access to the NFs share
* ***all_squash*** - will map all UIDs and GIDs to the anonymous user
* ***no_all_squash*** - (default) will allow users to be detected
* ***insecure*** - allow the use of a client connecting with a port number above `1024`
* ***no_subtree_check*** - (default) but will warn if not called out
* ***root_squash*** - don't allow remote root user to have root priviledges on this share
* ***no_root_squash*** - allows remote root user's to have root priviledges on this share
* ***sync*** - (default) reply to requests only after changes have been written to disk
* ***async*** - reply to requests before changes are written to disk, faster but dangerous

Setup nfs shares:
```bash
# Create target shared directories
$ sudo mkdir -p /srv/nfs/{Cache,Documents,Educational,Family,Install,Movies,Pictures,TV}

# Add shares to exports file
$ sudo tee -a /etc/exports <<EOL
/srv/nfs             192.168.1.0/24(rw,fsid=0,no_subtree_check)
/srv/nfs/Cache       192.168.1.0/24(rw,no_root_squash,insecure,no_subtree_check)
/srv/nfs/Documents   192.168.1.0/24(rw,root_squash,insecure,no_subtree_check)
/srv/nfs/Educational 192.168.1.0/24(ro,all_squash,insecure,no_subtree_check)
/srv/nfs/Family      192.168.1.0/24(ro,all_squash,insecure,no_subtree_check)
/srv/nfs/Install     192.168.1.0/24(ro,all_squash,insecure,no_subtree_check)
/srv/nfs/Movies      192.168.1.0/24(ro,all_squash,insecure,no_subtree_check)
/srv/nfs/Pictures    192.168.1.0/24(ro,all_squash,insecure,no_subtree_check)
/srv/nfs/TV          192.168.1.0/24(ro,all_squash,insecure,no_subtree_check)
EOL

# Optionally manually bind mount directories as a test
$ sudo mount --bind /mnt/storage/Movies /srv/nfs/Movies

# Auto Bind mount directories
$ sudo tee -a /etc/fstab <<EOL
/mnt/storage/Cache /srv/nfs/Cache none bind 0 0
/mnt/storage/Documents /srv/nfs/Documents none bind 0 0
/mnt/storage/Educational /srv/nfs/Educational none bind 0 0
/mnt/storage/Family /srv/nfs/Family none bind 0 0
/mnt/storage/Install /srv/nfs/Install none bind 0 0
/mnt/storage/Movies /srv/nfs/Movies none bind 0 0
/mnt/storage/Pictures /srv/nfs/Pictures none bind 0 0
/mnt/storage/TV /srv/nfs/TV none bind 0 0
EOL
$ sudo mount -a

# Ensure nfs-server is started
$ sudo systemctl start nfs-server

# Re-export your shares to pick up changes
$ sudo exportfs -r

# Check what is currently being served
$ sudo exportfs -v
```

<!-- 
vim: ts=2:sw=2:sts=2
-->
