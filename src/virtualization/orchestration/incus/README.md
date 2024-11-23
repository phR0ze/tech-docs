# Incus <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Incus is a moder, secure and powerful system container and virtual machine manager.

It provides a unified experience for running and managing full Linux systems inside containers or 
virtual machines. Incus supports images for a large number of Linux distributions. Incus can scale 
from a single instance to data center scale size.

### Quick links
* [../](../README.md)
* [Overview](#overview)
  * [Install Incus](#install-incus)
  * [Incus Storage Pools](#incus-storage-pools)
  * [Create Virtual Bridge Device](#create-virtual-bridge-device)
* [Templates](#templates)
* [Using Incus](#using-incus)
  * [Create a container](#create-a-container)
  * [Create a bridge network profile](#create-a-bridge-network-profile)
* [LXConsole](#lxconsole)
  * [Install LXConsole](#install-lxconsole)
  * [Create VM from Image](#create-vm-from-image)
  * [Create VM from ISO](#create-vm-from-iso)

## Overview
Canonical is [now managing the LXD project](https://linuxiac.com/lxd-containers-project-goes-under-canonical-wing/).
This action generated mixed reactions within the open-source community [triggering a fork of LXD 5.16](https://linuxiac.com/incus-project-lxd-fork/) named `Incus` that is invisioned as being driven by the community rather than a commercial entity. 
Incus as of Aug. 7th, 2023 is an official member of the Linux Containers project.

[Incus has continued to mature](https://tadeubento.com/2024/replace-proxmox-with-incus-lxd/) providing a
unified experience for dealing with both LXC containers and VMs. Incus has put a lot of work into 
managing both types of virtualization in a similar fashion. It now comes with a web interface as 
well. It is purely open source and can be installed on any Debian system with little to no overhead 
and as of Debian 13 will be included in the repositories.

### Install Incus
Hardware requirements: 4 core cpu and 16 GB memory minimum

**References**
* [Incus step by step by Scotti-BYTE](https://www.youtube.com/watch?v=ULPuU9aKyoU)

1. Create an Ubuntu Server 22.04 LTS 
   1. Download the ISO from Ubuntu
   2. Create a new VM with 4 cores, 8 GB ram
2. Login and upgrade machine
   ```
   $ sudo su
   $ apt update && apt upgrade -y
   ```
3. [Create Virtual Bridge Device](#create-virtual-bridge-device)
4. Import the Zabbly Incus GPG keys
   1. List them out
      ```bash
      $ curl -fsSL https://pkgs.zabbly.com/key.asc | gpg --show-keys --fingerprint
      ```
   2. Create a folder to store them 
      ```bash
      $ mkdir -p /etc/apt/keyrings
      ```
   3. Download the keys
      ```bash
      $ curl -fsSL https://pkgs.zabbly.com/key.asc -o /etc/apt/keyrings/zabbly.asc
      ```
5. Add zabbly respository
   ```
   sh -c 'cat <<EOF > /etc/apt/sources.list.d/zabbly-incus-stable.sources
   Enabled: yes
   Types: deb
   URIs: https://pkgs.zabbly.com/incus/stable
   Suites: $(. /etc/os-release && echo ${VERSION_CODENAME})
   Components: main
   Architectures: $(dpkg --print-architecture)
   Signed-By: /etc/apt/keyrings/zabbly.asc
   EOF'
   ```
6. Install Incus and add your user to the `incus-admin` group
   ```bash
   $ apt update
   $ apt install incus
   $ sudo usermod -aG incus-admin $USER
   $ newgrp incus-admin
   ```
7. Initialize Incus
   1. Run: `incus admin init`
   2. `Would you like to use clustering`: choose `no`
   3. `Do you want to configure a new storage pool` choose `yes`
   4. `Name of the new storage pool` choose `default`
   5. `Name of the storage backend to use` choose `dir`
   6. `Where should this storage pool store its data` choose `/data/incus`
   7. `Would you like to create a new local network bridge` choose `yes`
   8. `What should the new bridge be called` choose `incusbr0`
   9. `What IPv4 address should be used` choose `auto`
   10. `What IPv6 address should be used` choose `auto`
   11. `Would you like the server to be available over the network` choose `yes`
   12. `Address to bind to` choose `all`
   13. `Port to bind to` choose `8443`
   14. `Would you like stale cached image to be updated automatically` choose `yes`
   15. `Would you like your YAML "init" preseed to be printed` choose `no`

### Incus Storage Pools
Sharing the file system with the host is usually the most space-efficient way to run Incus. This 
option is supported for the `dir` driver and the `btrfs` driver (if the host is using Btrfs and you 
point Incus to a dedicated sub-volumn) and the `zfs` driver (if the host is ZFS and you point Incus 
to a dedicated zpool).

You must use `dir`, which is slower, if you want to share the host drive and the host drive is not 
setup with Btrfs or ZFS.

**dir driver**
```bash
$ incus storage create default dir source=/data/incus
```


### Create Virtual Bridge Device
Physical NICs can't be shared amongst multiple systems without first virtualizing it.

Note: netplan is Canonical's abstraction over `networkd` and `NetworkManager` its two supported 
backends that allows configuration via a yaml file.

1. Install `openvswitch-switch`
   ```bash
   $ sudo apt install openvswitch-switch
   ```
2. Edit `sudo vim /etc/netplan/50-cloud-init.yaml` and change it to
   ```yaml
   network:
     version: 2
     ethernets:
       ens18:
         dhcp4: false
     bridges:
       bridge0:
         interfaces: [ens18]
         addresses: [192.168.1.208/24]
         routes:
           - to: default
             via: 192.168.1.1
         nameservers:
           addresses:
             - 1.1.1.1
             - 8.8.8.8
         parameters:
           stp: true
           forward-delay: 4
         dhcp4: no
   ```
   Note:
   * Use the same IP address for your bridge as your NIC already is using
3. Apply the new netplan configuration
   1. Run: `sudo netplan apply`
   2. Note that `ip a` shows the NIC as not having an address and a new `bridge0` as having the 
      address instead.

## Templates
LXC templates from the likes of TurnKey

## Using Incus

### Create a container
```bash
$ incus launch images:ubuntu/22.04 first
```

### Create a bridge network profile
1. Create a new profile
   ```
   $ incus profile create bridgeprofile
   ```
2. Add bridge configuration to expose container IPs on your LAN
   ```bash
   $ incus profile device add bridgeprofile eth0 nic nictype=bridged parent=bridge0
   ```
3. Create a container using this profile
   ```bash
   $ incus launch images:ubuntu/22.04 second --profile default --profile bridge
   ```
4. Takes a min but will get a LAN IP
   ```bash
   $ incus list
   ```
5. Install openssh
   ```bash
   $ apt install openssh-server
   $ adduser NEW_USER
   ```

## LXConsole
Stephane Graber, the lead on Incus, has no plans for an official web interface for Incus which has 
spawned the creation of a few different web interfaces, one of which is LXConsole.

LXConsole is a python Web Interface for Incus
* Provides a full desktop session GUI similar to Proxmox
* Provides shell login session
* Provides LXC exec directly into a shell

### Install LXConsole
Note: we're creating an LXC container into which we'll install Docker which will be where LXConsole 
is actually deployed with Docker compose. This could also just be done directly on a server that has 
Docker installed on it.

1. Create an Ubuntu container
   ```bash
   $ incus launch images:ubuntu/22.04 lxconsole --profile default --profile bridge -c \
     security.nesting=true -c boot.autostart=true
   ```
2. Shell in and install Docker and Compose
   1. Shell in
      ```bash
      $ incus shell lxconsole
      ```
   2. Upgrade the system and install tools
      ```bash
      $ apt update && apt upgrade -y
      $ apt install curl vim openssh-server
      ```
   3. Add a new user account
      ```bash
      $ adduser $USER
      ```
   4. Install docker and compose
      ```bash
      $ curl -sSL https://get.docker.com | sh
      $ apt install docker-compose
      $ usermod -aG docker $USER
      ```
3. Finally setup a docker compose setup for lxconsole
   1. Create directories for lxconsole
      ```bash
      $ su - $USER
      $ mkdir lxconsole lxcbackups
      $ cd lxconsole
      ```
   2. Create the compose file
      ```yaml
      services:
        lxconsole:
          image: 'penninglabs/lxconsole:latest'
          restart: unless-stopped
          ports:
            - '80:5000'
          volumes:
            - /home/USER/lxcbackups:/opt/lxconsole/backups
            - ./certs:/opt/lxconsole/certs
            - ./server:/opt/lxconsole/instance
      ```
   3. Launch which will create the two folders `~/.certs` and `~/.server`
      ```bash
      $ docker compose up -d
      ```
4. Login to the UI
   1. Open a browser to the LXC container IP e.g. `http://192.168.1.199`
   2. Configure a user account for LXConsole through the web UI
   3. Add your Incus server to the console 
      1. Click `+ Server`
      2. Set `Host Addr:` to the server hosting LXC e.g. `192.168.1.208`
         * Now click the `here` link at the top to get the lxconsole cert
         * Save the cert as `lxconsole.crt` on the incuse server
         * Run: `incus config trust add-certificate lxconsole.crt`
         * Run: `incus config set core.https_address=[::]:8443`
      3. Leave `8443` for the port
      4. Leave `Proxy` empty
      5. Leave `SSL Verify` set to `False`

### Create VM from Image
Incus VMs can be created from pre-built images from https://linuxcontainers.org which saves a lot of 
time creating them.

1. Download your target image
   1. Navigate to `Images` on the left
   2. Click th `+Image` button in the top right
   3. Select the target image and version
   4. Click `Ok` to download
2. Create VM
   1. Navigate to `Instances >Virtual-Machines` then click `+Instance`
   2. Fill it out selecting the image you downloaded
   3. Add your `bridge` profile to get an IP on the LAN
   4. Set `Autostart:` to `true`

### Create VM from ISO
You can create a VM from a bootable ISO in the traditional way as well.

1. Create a VM stub
   ```bash
   $ incus init Test-VM --empty --vm
   ```
