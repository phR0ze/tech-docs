# Incus <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

**Synopsis**: although interesting and a potential solution in its own right, I've found that using 
NixOS to manage my containers directly and QEMU for my virtual machines to be a better fit for my 
needs.

Incus is a modern, secure and powerful system container and virtual machine manager. It provides a 
unified experience for running and managing full Linux systems inside containers or virtual machines. 
Incus supports images for a large number of Linux distributions. Incus can scale from a single 
instance to data center scale size.

### Quick links
* [.. up dir](../README.md)
* [Overview](#overview)
  * [Install Incus](#install-incus)
* [Using Incus](#using-incus)
  * [Create a container](#create-a-container)
  * [Create a VM from a custom ISO](#create-a-vm-from-a-custom-iso)
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

1. Install incus
   ```nix
   virtualisation.incus.enable = true;
   networking.firewall.trustedInterfaces = [ "incusbr0" ];
   ```
2. Create Virtual Bridge Device
   * see `nixos-configs/options/network/default.nix`
3. Intialize Incus
   * see `nixos-configs/options/virtualization/incus.nix`

## Using Incus

### Create a container
1. Create your first LXC container
   ```bash
   $ incus launch images:ubuntu/22.04 first
   ```
2. List out the containers
   ```bash
   $ incus list
   ```

### Create a VM from a custom ISO
To launch a VM that boots from an ISO, you must first create a VM.

**References**
* [Incus instance create](https://linuxcontainers.org/incus/docs/main/howto/instances_create/)

1. Create the empty VM
   ```bash
   $ incus init iso-vm --empty --vm -c limits.cpu=4 -c limits.memory=8GiB -d root,size=50GiB 
   ```
2. Add the local ISO as a disk to the VM by absolute path
  ```bash
  $ incus config device add iso-vm iso-volume disk source=/home/<user>/Downloads/cyberlinux-24.05.20240229.1536926-light.iso boot.priority=0
  ```
2. Import the ISO image so it can be later attached to the VM as a storage volume
   ```bash
   $ incus storage volume import <pool> <path-to-image.iso> iso-volume --type=iso
   ```
3. You can see your new ISO storage volume with
   ```bash
   $ incus storage volume list default
   +-----------------+------------+-------------+--------------+---------+
   |      TYPE       |    NAME    | DESCRIPTION | CONTENT-TYPE | USED BY |
   +-----------------+------------+-------------+--------------+---------+
   | custom          | iso-volume |             | iso          | 0       |
   +-----------------+------------+-------------+--------------+---------+
   | virtual-machine | iso-vm     |             | block        | 1       |
   +-----------------+------------+-------------+--------------+---------+
   ```
4. Attach the custom ISO volume to the VM
   ```bash
   $ incus config device add iso-vm iso-volume disk pool=<pool> source=iso-volume boot.priority=10
   ```
5. Start the VM in LXConsole and attach to the VGA console and walk through the wizard
   * Note: takes forever to work through PXE boot first before booting into the ISO
6. Once installed detach the ISO volume with
   ```bash
   $ incus storage volume detach <pool> iso-volume iso-vm
   ```

### Create a bridge network profile
Create a bridge profile for our `br0` bridge so that VMs and container can use it to get an address 
from your local router and be present on the LAN.

1. Create a new profile
   ```
   $ incus profile create bridged
   ```
2. Add bridge configuration, called `bridged`, to expose container IPs on your LAN
   ```bash
   $ incus profile device add bridged eth0 nic nictype=bridged parent=br0
   ```
3. Create a container using this profile
   ```bash
   $ incus launch images:ubuntu/22.04 second --profile default --profile bridged
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
   1. Open a browser to the LXC container IP e.g. `http://192.168.1.40`
   2. Configure a user account for LXConsole through the web UI
   3. Add your Incus server to the console 
      1. Click `+ Server`
      2. Set `Host Addr:` to the server hosting LXC e.g. `192.168.1.40`
         * Run: `incus config trust add-certificate /var/lib/lxconsole/certs/client.crt`
         * Run: `incus config set core.https_address=192.168.1.50:8443`
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
