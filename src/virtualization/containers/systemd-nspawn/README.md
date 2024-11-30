# systemd-nspawn <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Systemd has built in LXC like containerization support called `systemd-nspawn` that allows for 
creating and managing containers. Their implementation is separate from both LXC and libvirt-lxc but 
aims to play well with them via its generic management tool `machinectl`. The intent is to be able to 
manage containers and integrate smoothly with `sytemctl` such that machines could be spun up and used 
in a similar fashion as other services.

### Quick links
* [.. up dir](../README.md)
* [Overview](#overview)

## Overview
`systemd-nspawn` may be used to run a command or operating system in a light-weight namespace 
container. It is more powerful than chroot since it fully virtualizes the file system hierarchy, as 
well as the process tree and limits access to various kernel interfaces including network interfaces.

`machinectl` is provided as a way to interact with systemd-nspawn containers. `tar`, `raw`, `qcow2`, 
and `dkr` i.e. (Docker) image formats are supported. There is no centralized repo for images however. 
Images are managed based on the btrfs file system.

**References**
* [systemd-nspawn](https://wiki.archlinux.org/title/Systemd-nspawn)
* [NixOS Containers - wiki](https://nixos.wiki/wiki/NixOS_Containers)
* [NixOS Containers - manual](https://nixos.org/manual/nixos/stable/#ch-containers)
* [Docker explained](https://github.com/paulboot/docker-explained/blob/master/Systemd%20Containers%20and%20systemd-nspawn%20Explained.md)
